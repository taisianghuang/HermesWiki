---
title: Hermes Plugin System — Build a Plugin Guide
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, plugin-system, tools, hooks, extensibility]
sources: [../../raw/hermes-agent/website/docs/guides/build-a-hermes-plugin.md]
related: [hermes-tool-registry, hermes-toolsets, hermes-skills-system]
summary: Hermes Plugin 開發完整指南，含 Plugin 架構（manifest + schemas + handlers + registration）、工具/生命週期 hooks 註冊、data files、bundled skills、env gates。
---

# Hermes Plugin System — Build a Plugin Guide

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | `raw/hermes-agent/website/docs/guides/build-a-hermes-plugin.md` |
| **涵蓋內容** | Plugin 開發完整流程（manifest、schemas、handlers、registration）、8種 hooks、skill bundling、env gates |

## Plugin 架構（四件套）

```
~/.hermes/plugins/<name>/
├── plugin.yaml      # Manifest：宣告 Plugin 身份
├── __init__.py     # Registration：連接 schemas ↔ handlers
├── schemas.py       # Tool schemas：LLM 看到的描述
└── tools.py        # Tool handlers：實際執行的程式碼
```

### 1. Manifest（plugin.yaml）

```yaml
name: calculator
version: 1.0.0
description: Math calculator
provides_tools:
  - calculate
  - unit_convert
provides_hooks:
  - post_tool_call
```

### 2. Schemas（schemas.py）

定義 LLM 看到的工具描述，必須包含 `name`、`description`、`parameters`。

### 3. Handlers（tools.py）

實際執行邏輯，**規則**：
- 函式簽名：`def my_handler(args: dict, **kwargs) -> str`
- **永遠回傳 JSON string**（成功或錯誤都是）
- 抓所有 exception，回傳 error JSON
- 接受 `**kwargs` 保留未來擴展性

### 4. Registration（__init__.py）

```python
def register(ctx):
    ctx.register_tool(name="calculate", toolset="calculator",
                      schema=schemas.CALCULATE, handler=tools.calculate)
    ctx.register_hook("post_tool_call", _on_post_tool_call)
```

## 8 種 Hooks

| Hook | 時機 | 可 injection？ |
|------|------|---------------|
| `pre_tool_call` | 任何 tool 執行前 | 否 |
| `post_tool_call` | 任何 tool 返回後 | 否 |
| `pre_llm_call` | 每 turn 的 LLM 呼叫前 | **是（context injection）** |
| `post_llm_call` | 每 turn 的 LLM 呼叫後 | 否 |
| `on_session_start` | 新 session 建立時 | 否 |
| `on_session_end` | `run_conversation` 結束或 CLI 退出 | 否 |
| `pre_api_request` | 發送 HTTP 請求到 LLM provider 前 | 否 |
| `post_api_request` | 收到 LLM provider HTTP 回應後 | 否 |

### pre_llm_call 的 context injection

這是唯一有回傳價值的 hook。可回傳：
```python
return {"context": "要注入的文字"}  # dict with context key
return "要注入的文字"               # plain string（等同上者）
return None                         # 不注入
```

注入到 **user message**（不是 system prompt），保持 system prompt 緩存命中。

## Plugin 擴展功能

### Ship data files

在 plugin 目錄放任何檔案，執行時讀取：
```python
_PLUGIN_DIR = Path(__file__).parent
_DATA_FILE = _PLUGIN_DIR / "data" / "languages.yaml"
```

### Bundle a skill

可在 `register()` 時將 `skill.md` 複製到 `~/.hermes/skills/`：
```python
def _install_skill():
    dest = get_hermes_home() / "skills" / "my-plugin" / "SKILL.md"
    if not dest.exists():
        shutil.copy2(Path(__file__).parent / "skill.md", dest)
```

### Environment variable gates

```yaml
# plugin.yaml
requires_env:
  - WEATHER_API_KEY                    # 簡單格式
  - name: WEATHER_API_KEY              # 豐富格式
    description: "API key for OpenWeather"
    url: "https://openweathermap.org/api"
    secret: true
```

### Conditional tool availability

```python
ctx.register_tool(
    name="my_tool", schema={...}, handler=my_handler,
    check_fn=lambda: _has_optional_lib(),  # False = tool 對 LLM 隱藏
)
```

## 與 Tool Registry 的關係

- **Tool Registry**（`tools/registry.py`）：內建工具的中央註冊表，module-level `registry.register()` 自動註冊
- **Plugin System**：使用者安裝的 Plugin，透過 `ctx.register_tool()` 動態加入同一個 Registry

Plugin 和內建工具最終都在同一個 Registry 中，LLM 看到的工具清單是兩者的聯集。

## See also

- [Hermes Tool Registry](hermes-tool-registry.md)
- [Hermes Toolsets](hermes-toolsets.md)
- [Hermes Skills System](hermes-skills-system.md)
