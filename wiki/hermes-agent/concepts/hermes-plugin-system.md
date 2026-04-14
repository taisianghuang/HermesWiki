---
title: Hermes Plugin System
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, plugin-system, tools, hooks, extensibility]
sources: [../../raw/hermes-agent/website/docs/guides/build-a-hermes-plugin.md, ../sources/2026-04-14-hermes-agent-plugin-system.md]
related: [hermes-tool-registry, hermes-toolsets, hermes-skills-system, hermes-acp-integration]
summary: Hermes Plugin System 是 Hermes 的擴充機制，透過 plugin.yaml manifest + schemas + handlers + registration 四件套，讓使用者可新增工具、註冊生命週期 hooks、附帶資料檔案和技能文件。
---

# Hermes Plugin System

## 概述

Hermes Plugin System 是 Hermes 的**使用者擴充機制**。不同於內建工具（透過 `tools/registry.py` 註冊），Plugin 允許使用者在 `~/.hermes/plugins/` 安裝自己的工具和 hooks，且享有與內建工具完全平等的待遇（都在同一個 Registry 中）。

## 四件套結構

```
~/.hermes/plugins/<name>/
├── plugin.yaml      # Manifest：名稱、版本、提供哪些工具/hooks
├── __init__.py     # Registration：連接 schemas ↔ handlers ↔ hooks
├── schemas.py       # Tool schemas：LLM 看到的描述（name、description、parameters）
└── tools.py        # Tool handlers：實際執行的 Python 程式碼
```

## 與 Tool Registry 的關係

| 系統 | 誰使用 | 註冊方式 |
|------|--------|----------|
| **Tool Registry** | 內建工具 | `tools/*.py` 的 `registry.register()`（module-level） |
| **Plugin System** | 使用者插件 | `__init__.py` 的 `ctx.register_tool()`（動態） |

最終 LLM 看到的工具清單是兩者的**聯集**，沒有差別。

## 工具開發要點

### Schema 描述決定 LLM 行為

`description` 欄位是 LLM 判斷「何時該用這個工具」的依據。需具體說明：
- 工具用途
- 支援的參數格式
- 適當使用時機

### Handler 規則

1. 函式簽名：`def handler(args: dict, **kwargs) -> str`
2. **永遠回傳 JSON string**（含錯誤）
3. 不 raise exception，抓取後回傳 error JSON
4. 接受 `**kwargs`（未來可能傳入更多參數）

## 8 種 Hooks

| Hook | 時機 | injection？ |
|------|------|-----------|
| `pre_tool_call` | Tool 執行前 | 否 |
| `post_tool_call` | Tool 返回後 | 否 |
| `pre_llm_call` | LLM 呼叫前 | **是** |
| `post_llm_call` | LLM 呼叫後 | 否 |
| `on_session_start` | Session 建立時 | 否 |
| `on_session_end` | Session 結束時 | 否 |
| `pre_api_request` | HTTP 發出前 | 否 |
| `post_api_request` | HTTP 回應後 | 否 |

### pre_llm_call：唯一有意義的回傳值

可注入 context 到 user message（不是 system prompt）：
```python
def inject_memory(**kwargs):
    return {"context": "用戶上次提到他喜歡 Python"}
```

這是 Plugin 實作 memory、RAG、guardrails 的基礎。

## 實用擴充功能

### Environment Variable Gates

Plugin 可要求環境變數，缺失時 Plugin 被禁用但 Hermes 繼續正常運行：
```yaml
requires_env:
  - WEATHER_API_KEY
  - name: OPENAI_KEY
    description: "OpenAI API key"
    url: "https://platform.openai.com/api-keys"
    secret: true
```

### Conditional Tool Availability

透過 `check_fn` 可根據條件動態隱藏工具：
```python
ctx.register_tool(name="my_tool", schema={...}, handler=my_handler,
                  check_fn=lambda: _has_gpu())
```

### Bundled Skills

Plugin 可附帶 `skill.md`，在 `register()` 時自動安裝到 `~/.hermes/skills/`：
```python
def _install_skill():
    dest = get_hermes_home() / "skills" / plugin_name / "SKILL.md"
    if not dest.exists():
        shutil.copy2(source_skill_md, dest)
```

## Plugin 與 Skills 的比較

| 維度 | Plugin | Skills |
|------|--------|--------|
| 本質 | 工具/hooks 擴充 | 教 Agent 如何做事的文件 |
| 對 LLM 的影響 | 新工具（tool call） | System prompt 知識 |
| 觸發方式 | LLM 主動呼叫 | LLM 自動參考 |
| 用途 | 執行特定任務 | 學習複雜流程/領域知識 |

## See also

- [Hermes Tool Registry](hermes-tool-registry.md)
- [Hermes Toolsets](hermes-toolsets.md)
- [Hermes Skills System](hermes-skills-system.md)
- [Hermes ACP Integration](hermes-acp-integration.md)
