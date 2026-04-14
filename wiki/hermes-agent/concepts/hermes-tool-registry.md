---
title: Hermes Tool Registry
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, tool-registry, plugin, architecture]
sources: [../sources/2026-04-14-hermes-agent-repo.md]
related: [hermes-agent-core-loop, hermes-cli-architecture]
summary: Hermes 的工具系統採用 Registry 模式，每個工具檔案在 import 時自動向中央 Registry 註冊，model_tools.py 透過 _discover_tools() 自動發現所有工具。
---

# Hermes Tool Registry

## 概述

Hermes 的工具系統採用 **Registry 模式**（註冊表模式），每個工具檔案在 Python import 時自動向中央 `ToolRegistry` 註冊自己的 schema 和 handler。這使得新增工具只需要新增一個檔案，無需修改任何中央設定。

## 設計哲學

> **"每個工具檔案自己管理自己，Registry 只是鏡子"**

- **非侵入式**：工具檔案在 module-level 呼叫 `registry.register()`，框架無需為每個工具維護單獨的設定
- **自動發現**：`model_tools.py` 在啟動時透過 `_discover_tools()` 自動發現所有已註冊的工具
- **無循環依賴**：`tools/registry.py` 是唯一沒有任何 import 的檔案，所有工具檔案都依賴它

## 檔案依賴鏈

```
tools/registry.py  (無依賴 — 底部)
       ↑
tools/*.py  (每個工具檔案 import registry，module-level 呼叫 register())
       ↑
model_tools.py  (導入 registry + 觸發工具發現)
       ↑
run_agent.py, cli.py, batch_runner.py
```

## ToolRegistry 類

位置：`tools/registry.py`

```python
class ToolEntry:
    """單一工具的元數據"""
    __slots__ = (
        "name", "toolset", "schema", "handler", "check_fn",
        "requires_env", "is_async", "description", "emoji",
        "max_result_size_chars",
    )

class ToolRegistry:
    """收集所有工具 schema + handlers 的單例"""
    def __init__(self):
        self._tools: Dict[str, ToolEntry] = {}
        self._toolset_checks: Dict[str, Callable] = {}
```

## 註冊工具

每個工具檔案（例如 `tools/file_tools.py`）長這樣：

```python
# tools/registry.py 的工廠函式
def register(
    name: str,
    toolset: str,
    schema: dict,
    handler: Callable,
    check_fn: Callable = None,
    requires_env: list = None,
    is_async: bool = False,
    description: str = "",
    emoji: str = "",
    max_result_size_chars: int | float | None = None,
):
    ...

# tools/file_tools.py 中的使用方式
def check_requirements() -> bool:
    return True  # 或檢查環境變數等

def file_read(path: str, ...) -> str:
    return json.dumps({"content": "...", ...})

registry.register(
    name="file_read",
    toolset="file",
    schema={
        "name": "file_read",
        "description": "Read a file with line numbers...",
        "parameters": {"type": "object", "properties": {...}}
    },
    handler=lambda args, **kw: file_read(...),
    check_fn=check_requirements,
    requires_env=[],
    emoji="📄",
)
```

## 新增工具的步驟

1. **建立工具檔案** `tools/your_tool.py`
2. **在 `model_tools.py` 的 `_discover_tools()` 加入 import**
3. **在 `toolsets.py` 加入 toolset 定義**（或加入現有的 `_HERMES_CORE_TOOLS`）

## ToolEntry 欄位說明

| 欄位 | 說明 |
|------|------|
| `name` | 工具名稱，unique identifier |
| `toolset` | 所屬工具集（如 "file"、"web"、"terminal"） |
| `schema` | JSON schema，描述工具的輸入參數 |
| `handler` | 可呼叫的函式，處理實際邏輯 |
| `check_fn` | 環境檢查函式，判斷工具是否可用 |
| `requires_env` | 需要的環境變數列表 |
| `is_async` | 是否為 async 工具 |
| `emoji` | CLI 顯示用的 emoji |
| `description` | 工具描述 |

## Toolset 概念

Toolset 是一組相關工具的集合。`toolsets.py` 定義了：

```python
_HERMES_CORE_TOOLS = [
    "file", "terminal", "web", "code", ...
]
```

可以单独啟用/停用某個 toolset，透過 `hermes tools` 命令配置。

## 內建工具集

| Toolset | 工具數 | 範例工具 |
|---------|--------|---------|
| `file` | 5+ | file_read, file_write, search_files, patch |
| `terminal` | 3+ | terminal, background, process |
| `web` | 4+ | web_search, browser_navigate |
| `code` | 2+ | code_execution, delegate_task |
| `mcp` | 1 | mcp_tool |
| `memory` | 3+ | memory, session_search, todo |

## 工具發現機制

`model_tools.py` 中的 `_discover_tools()`：

```python
def _discover_tools():
    """在啟動時被呼叫，觸發所有工具檔案的 import"""
    # 透過 import tools.registry 間接 import 所有工具檔案
    # 每個工具檔案在 import 時自動註冊
```

## 與其他框架的比較

| 特性 | Hermes | LangChain | CrewAI |
|------|--------|-----------|--------|
| 註冊方式 | Module-level register() | @tool 裝飾器 | @tool 裝飾器 |
| 發現機制 | 自動（import 觸發）| 自動 | 自動 |
| Toolset 支援 | 原生 | 需自建 | Tools 類 |
| Schema 格式 | OpenAI function | LangChain format | Pydantic |

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes CLI Architecture](hermes-cli-architecture.md)
