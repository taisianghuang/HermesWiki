---
title: Hermes Agent Core Loop
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, agent-loop, core, architecture, provider, api-modes]
sources: [../../raw/hermes-agent/docs/architecture.md, ../../raw/hermes-agent/docs/agent-loop.md]
related: [hermes-prompt-assembly, hermes-context-compression, hermes-provider-runtime, hermes-tool-registry]
summary: Hermes 核心同步 tool-calling 迴圈，運行在 run_agent.py 的 AIAgent.run_conversation() 中。三種 API modes（chat_completions/codex_responses/anthropic_messages），支援中斷、ThreadPool 並行工具執行、Iteration Budget 和 Fallback Model。
---

# Hermes Agent Core Loop

## 概述

核心orchestration engine是 `run_agent.py` 的 `AIAgent` 類，約 9,200 行，處理從 prompt 組裝到 tool dispatch 到 provider failover 的所有事情。

## 兩個進入點

```python
# 簡單介面 — 返回最終回覆字串
response = agent.chat("Fix the bug in main.py")

# 完整介面 — 返回 dict，包含 messages, metadata, usage stats
result = agent.run_conversation(
    user_message="Fix the bug in main.py",
    system_message=None,           # 自動構建
    conversation_history=None,     # 自動從 session 載入
    task_id="task_abc123"
)
```

`chat()` 是 `run_conversation()` 的 thin wrapper，提取 `final_response` 欄位。

## API Modes

Hermes 支援三種 API execution modes：

| API mode | 用於 | Client |
|----------|------|--------|
| `chat_completions` | OpenAI-compatible endpoints（OpenRouter、custom、大多數 providers）| `openai.OpenAI` |
| `codex_responses` | OpenAI Codex / Responses API | `openai.OpenAI`（Responses format）|
| `anthropic_messages` | Native Anthropic Messages API | `anthropic.Anthropic`（via adapter）|

**Mode resolution order**:
1. Explicit `api_mode` constructor arg（最高優先）
2. Provider-specific detection（例如 `anthropic` provider → `anthropic_messages`）
3. Base URL heuristics（例如 `api.anthropic.com` → `anthropic_messages`）
4. Default: `chat_completions`

## Turn Lifecycle（每輪迭代順序）

```
run_conversation()
  │
  ├── 1. Generate task_id if not provided
  │
  ├── 2. Append user message to conversation history
  │
  ├── 3. Build or reuse cached system prompt (prompt_builder.py)
  │
  ├── 4. Check if preflight compression needed (>50% context)
  │
  ├── 5. Build API messages from conversation history
  │     ├── chat_completions: OpenAI format as-is
  │     ├── codex_responses: convert to Responses API input items
  │     └── anthropic_messages: convert via anthropic_adapter.py
  │
  ├── 6. Inject ephemeral prompt layers (budget warnings, context pressure)
  │
  ├── 7. Apply prompt caching markers if on Anthropic
  │
  ├── 8. Make interruptible API call (_api_call_with_interrupt)
  │
  └── 9. Parse response:
          ├── If tool_calls: execute them, append results, loop back to step 5
          └── If text response: persist session, flush memory, return
```

## Message Format

內部統一使用 OpenAI-compatible 格式：

```python
{"role": "system", "content": "..."}
{"role": "user", "content": "..."}
{"role": "assistant", "content": "...", "tool_calls": [...]}
{"role": "tool", "tool_call_id": "...", "content": "..."}
```

Reasoning content（支援 extended thinking 的模型）存在 `assistant_msg["reasoning"]`。

## Message Alternation Rules

嚴格的 role alternation：
- System 後：User → Assistant → User → Assistant → ...
- Tool calling 時：Assistant (tool_calls) → Tool → Tool → ... → Assistant
- **禁止**：連續兩個 assistant、連續兩個 user

## Interruptible API Calls

```
┌──────────────────────┐     ┌──────────────┐
│  Main thread         │     │  API thread   │
│  wait on:            │────▶│  HTTP POST    │
│  - response ready    │     │  to provider  │
│  - interrupt event   │     └──────────────┘
│  - timeout           │
└──────────────────────┘
```

中斷時：API thread 被放棄（response 丟棄），agent 可乾淨處理新輸入或關閉。

## Tool Execution

### Sequential vs Concurrent

- **單一** tool call → 直接在主執行緒執行
- **多個** tool calls → 透過 ThreadPoolExecutor 並行執行
- 例外：標記為 `interactive` 的工具（例如 `clarify`）強制順序執行

### Execution Flow

```python
for each tool_call in response.tool_calls:
    1. Resolve handler from tools/registry.py
    2. Fire pre_tool_call plugin hook
    3. Check if dangerous command (tools/approval.py)
       - If dangerous → invoke approval_callback, wait for user
    4. Execute handler with args + task_id
    5. Fire post_tool_call plugin hook
    6. Append {"role": "tool", "content": result} to history
```

## Agent-Level Tools（被 run_agent.py 攔截）

| Tool | 為什麼攔截 |
|------|-----------|
| `todo` | 讀寫 agent-local task state |
| `memory` | 寫入 persistent memory files（有字元限制）|
| `session_search` | 透過 agent 的 session DB 查詢 |
| `delegate_task` | Spawn subagent，隔離 context |

## Callback Surfaces

| Callback | 時機 | 使用者 |
|---------|------|--------|
| `tool_progress_callback` | 每個 tool 執行前後 | CLI spinner, gateway progress |
| `thinking_callback` | Model 開始/停止思考 | CLI "thinking..." 指示器 |
| `reasoning_callback` | Model 返回 reasoning content | CLI reasoning 顯示 |
| `clarify_callback` | clarify tool 被呼叫 | CLI input prompt |
| `step_callback` | 每個完整 agent turn | Gateway step tracking |
| `stream_delta_callback` | 每個 streaming token | CLI streaming 顯示 |
| `tool_gen_callback` | 從 stream 解析 tool call | CLI spinner 中的 tool preview |
| `status_callback` | State changes | ACP status updates |

## Iteration Budget

- 預設：90 iterations（可透過 `agent.max_turns` 配置）
- **共享**：parent 和 child agents 共享同一個 budget
- 兩層 budget pressure：
  - 70%+ usage：append `[BUDGET: Iteration X/Y...]` caution
  - 90%+ usage：append `[BUDGET WARNING: ...]` warning
  - 100%：停止並返回 work summary

## Fallback Model

當 primary model 失敗時（429 rate limit、5xx server error、401/403 auth error）：

```
Check fallback_providers list in config
  │
  ├── Try each fallback in order
  │     └── On success → continue with new provider
  │
  └── On 401/403 → attempt credential refresh before failing
```

## Compression and Persistence

### When Compression Triggers
- **Preflight**（API call 前）：對話超過 50% context window
- **Gateway auto-compression**（turns 中間）：超過 85%（更積極）

### What Happens During Compression
1. Memory 先 flush 到磁碟
2. Middle turns 濃縮成 summary
3. 最後 N 條訊息完整保留（`protect_last_n`，預設 20）
4. Tool call/result pairs 保持在一起
5. 生成新的 session lineage ID

## Key Source Files

| 檔案 | 用途 |
|------|------|
| `run_agent.py` | AIAgent 類 — 完整 agent loop（~9,200 行）|
| `agent/prompt_builder.py` | System prompt 組裝 |
| `agent/context_engine.py` | ContextEngine ABC — 可插拔 context 管理 |
| `agent/context_compressor.py` | Default engine — lossy summarization algorithm |
| `agent/prompt_caching.py` | Anthropic prompt caching markers |
| `agent/auxiliary_client.py` | Auxiliary LLM client（vision、summarization）|
| `model_tools.py` | Tool schema collection、handle_function_call() dispatch |

## See also

- [Hermes Tool Registry](hermes-tool-registry.md)
- [Hermes Prompt Assembly](hermes-prompt-assembly.md)
- [Hermes Context Compression](hermes-context-compression.md)
- [Hermes Provider Runtime](hermes-provider-runtime.md)
- [Hermes Session Storage](hermes-session-storage.md)
