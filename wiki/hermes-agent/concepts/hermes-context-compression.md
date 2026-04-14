---
title: Hermes Context Compression and Caching
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, context-compression, prompt-caching, anthropic, context-engine]
sources: [../../raw/hermes-agent/docs/architecture.md, ../../raw/hermes-agent/docs/context-compression-and-caching.md]
related: [hermes-agent-core-loop, hermes-prompt-assembly, hermes-session-storage]
summary: Hermes 採用雙層壓縮系統（Gateway Session Hygiene 85% + Agent ContextCompressor 50%），以及 Anthropic Prompt Caching（system_and_3 策略，節省 ~75% input tokens）。ContextCompressor 是一個可插拔的 engine，預設使用 lossy summarization，也可替換成 lossless 引擎。
---

# Hermes Context Compression and Caching

## 概述

Hermes 使用**雙層壓縮系統**和 **Anthropic prompt caching** 來有效管理長對話中的 context window 使用。

## Pluggable Context Engine

Context 管理基於 `ContextEngine` ABC（`agent/context_engine.py`）。內建的 `ContextCompressor` 是預設實現，但可以被 plugins 替換。

```yaml
context:
  engine: "compressor"    # default — lossy summarization
  engine: "lcm"          # plugin — lossless context management
```

Engine 職責：
- 決定何時觸發压缩（`should_compress()`）
- 執行压缩（`compress()`）
- 可選暴露 tools（例如 `lcm_grep`）
- 追蹤 API 回報的 token 使用量

## 雙層壓縮系統

```
                     ┌──────────────────────────┐
  Incoming message   │   Gateway Session Hygiene │  Fires at 85% of context
  ─────────────────► │   (pre-agent, rough est.) │  Safety net
                     └─────────────┬────────────┘
                                   │
                                   ▼
                     ┌──────────────────────────┐
                     │   Agent ContextCompressor │  Fires at 50% of context (default)
                     │   (in-loop, real tokens)  │  Normal management
                     └──────────────────────────┘
```

### Layer 1: Gateway Session Hygiene（85% threshold）

位於 `gateway/run.py`（`_maybe_compress_session`）。這是**安全網**，在 agent 處理訊息前執行。防止 API failure，例如 Telegram/Discord 過夜累積的大型 sessions。

- **Threshold**：Fixed at 85% of model context length
- **Token source**：Prefers actual API-reported tokens；falls back to rough character-based estimate
- **Fires**：Only when `len(history) >= 4` and compression is enabled
- **Purpose**：Catch sessions that escaped the agent's own compressor

### Layer 2: Agent ContextCompressor（50% threshold）

位於 `agent/context_compressor.py`。這是**主要壓縮系統**，在 agent tool loop 內執行，擁有準確的 API-reported token counts。

## Configuration

```yaml
compression:
  enabled: true              # 啟用/停用（預設: true）
  threshold: 0.50           # Context window 的 fraction（預設: 50%）
  target_ratio: 0.20         # Tail protection token budget（預設: 20%）
  protect_last_n: 20         # 最少保護的 tail messages（預設: 20）
  summary_model: null        # Summary model 覆寫（預設: 使用 auxiliary）
```

## 參數細節

| 參數 | 預設 | 範圍 | 說明 |
|------|------|------|------|
| `threshold` | 0.50 | 0.0-1.0 | Compression 觸發時的 prompt tokens fraction |
| `target_ratio` | 0.20 | 0.10-0.80 | 控制 tail protection token budget |
| `protect_last_n` | 20 | ≥1 | 永遠保留的最近訊息數 |
| `protect_first_n` | 3 | (hardcoded) | System prompt + 第一次 exchange 永遠保留 |

### 計算範例（200K context model, defaults）

```
context_length       = 200,000
threshold_tokens     = 200,000 × 0.50 = 100,000
tail_token_budget    = 100,000 × 0.20 = 20,000
max_summary_tokens   = min(200,000 × 0.05, 12,000) = 10,000
```

## 壓縮演算法（4 階段）

### Phase 1: Prune Old Tool Results（無 LLM 呼叫，便宜）

保護的 tail 外的舊 tool results（>200 chars）被替換為：
```
[Old tool output cleared to save context space]
```

### Phase 2: Determine Boundaries

```
[0..2]  ← protect_first_n（system + first exchange）
[3..N]  ← middle turns → SUMMARIZED
[N..end] ← tail（by token budget OR protect_last_n）
```

Tail protection 基於 token budget：从結尾往回走，累積 tokens 直到預算用完。如果 budget 會保護更少的訊息，fallback 到固定的 `protect_last_n`。

Boundaries 對齊避免 split tool_call/tool_result groups。`_align_boundary_backward()` 方法會走過連續的 tool results 找到 parent assistant message，保持 groups 完整。

### Phase 3: Generate Structured Summary

使用 auxiliary LLM 產生結構化 summary：

```markdown
## Goal
[What the user is trying to accomplish]

## Constraints & Preferences
[User preferences, coding style, constraints, important decisions]

## Progress
### Done
[Completed work — specific file paths, commands run, results]
### In Progress
[Work currently underway]
### Blocked
[Any blockers or issues encountered]

## Key Decisions
[Important technical decisions and why]

## Relevant Files
[Files read, modified, or created — with brief note on each]

## Next Steps
[What needs to happen next]

## Critical Context
[Specific values, error messages, configuration details]
```

Summary budget formula：`content_tokens × 0.20`（`_SUMMARY_RATIO` constant）

### Phase 4: Assemble Compressed Messages

壓縮後的訊息列表：
1. Head messages（第一次 compression 會 append note 到 system prompt）
2. Summary message
3. Tail messages（未修改）

## Iterative Re-compression

後續 compressions，previous summary 會傳給 LLM，指示它「更新」而非「從頭總結」。這保留跨多次壓縮的資訊——項目從 "In Progress" 移到 "Done"，新增進度，移除過時資訊。

## Prompt Caching（Anthropic）

Source: `agent/prompt_caching.py`

透過快取對話前綴減少 ~75% input token costs。

### Strategy: system_and_3

```
Breakpoint 1: System prompt              (所有 turns 穩定)
Breakpoint 2: 3rd-to-last non-system    ─┐
Breakpoint 3: 2nd-to-last non-system     ├─ Rolling window
Breakpoint 4: Last non-system            ─┘
```

### Cache Marker Format

```python
marker = {"type": "ephemeral"}
# 或 1-hour TTL:
marker = {"type": "ephemeral", "ttl": "1h"}
```

### Cache-Aware Design Patterns

| Pattern | 說明 |
|---------|------|
| Stable system prompt | 作為 breakpoint 1，快取跨所有 turns。避免在中途 mutate。 |
| Message ordering matters | 快取命中需要 prefix matching。 |
| Compression cache interaction | Compression 後，compressed 區域的 cache 被 invalidate，但 system prompt cache 存活。滾動 3-message window 在 1-2 turns 內重建快取。 |
| TTL selection | Default: 5m。Long-running sessions 使用 1h。 |

### 啟用條件

自動啟用當：
- Model 是 Anthropic Claude model
- Provider 支援 cache_control（native Anthropic API 或 OpenRouter）

## Context Pressure Warnings

在 compression threshold 的 85% 發出警告：

```
⚠️  Context is 85% to compaction threshold (42,500/50,000 tokens)
```

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Prompt Assembly](hermes-prompt-assembly.md)
- [Hermes Session Storage](hermes-session-storage.md)
