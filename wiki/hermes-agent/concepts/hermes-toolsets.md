---
title: Hermes Toolsets
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, toolsets, tool-management]
sources: [../sources/2026-04-14-hermes-agent-repo.md]
related: [hermes-tool-registry, hermes-agent-core-loop]
summary: Hermes Toolsets 是一個靈活的工具分組系統，可將多個工具組合成工具集（Toolset）供不同場景使用，並支援工具集的繼承和動態解析。
---

# Hermes Toolsets

## 概述

Toolsets 是 Hermes 的**工具分組系統**，讓工具可以被分組、繼承、動態啟用/停用。可以把工具集想成「工具的配置檔」，一個工具集就是一組工具的集合。

## _HERMES_CORE_TOOLS

核心工具列表，CLI 和所有訊息平台工具集共享同一份：

```python
_HERMES_CORE_TOOLS = [
    # Web
    "web_search", "web_extract",
    # Terminal + process management
    "terminal", "process",
    # File manipulation
    "read_file", "write_file", "patch", "search_files",
    # Vision + image generation
    "vision_analyze", "image_generate",
    # Skills
    "skills_list", "skill_view", "skill_manage",
    # Browser automation
    "browser_navigate", "browser_snapshot", "browser_click",
    "browser_type", "browser_scroll", "browser_back",
    "browser_press", "browser_get_images",
    "browser_vision", "browser_console",
    # Text-to-speech
    "text_to_speech",
    # Planning & memory
    "todo", "memory",
    # Session history search
    "session_search",
    # Clarifying questions
    "clarify",
    # Code execution + delegation
    "execute_code", "delegate_task",
    # Cronjob management
    "cronjob",
    # Cross-platform messaging
    "send_message",
    # Home Assistant
    "ha_list_entities", "ha_get_state", "ha_list_services", "ha_call_service",
]
```

## TOOLSETS 定義

```python
TOOLSETS = {
    "web": {
        "description": "Web research and content extraction tools",
        "tools": ["web_search", "web_extract"],
        "includes": []  # 不繼承其他工具集
    },
    
    "search": {
        "description": "Web search only (no content extraction/scraping)",
        "tools": ["web_search"],
        "includes": []
    },
    
    "vision": {
        "description": "Image analysis and vision tools",
        "tools": ["vision_analyze"],
        "includes": []
    },
    
    "terminal": {
        "description": "Terminal/command execution and process management tools",
        "tools": ["terminal", "process"],
        "includes": []
    },
    
    "moa": {
        "description": "Mixture of Agents — research using multiple specialized agents",
        "tools": ["delegate_task"],
        "includes": []
    },
    # ... 更多工具集
}
```

## 工具集的繼承（includes）

工具集可以繼承其他工具集：

```python
"full_stack": {
    "description": "All tools for full-stack development",
    "tools": [],
    "includes": ["web", "terminal", "file"]  # 繼承這些工具集的所有工具
}
```

## 動態解析

```python
from toolsets import get_toolset, resolve_toolset, get_all_toolsets

# 獲取工具集定義
tools = get_toolset("web")  # ["web_search", "web_extract"]

# 解析工具集（包含繼承）
all_tools = resolve_toolset("full_stack")  # ["web_search", "web_extract", "terminal", ...]

# 獲取所有工具集名稱
all = get_all_toolsets()  # ["web", "search", "vision", "terminal", "full_stack", ...]
```

## 使用方式

在 `hermes tools` CLI 中：

```
hermes tools list              # 列出所有工具集
hermes tools enable web       # 啟用 web 工具集
hermes tools disable terminal  # 停用 terminal 工具集
```

## Toolset 與 Registry 的關係

```
ToolRegistry (tools/registry.py)
  │
  ├── 儲存所有工具的 schema + handler
  │
  └── model_tools.py 在此基礎上支援 toolsets
        │
        └── resolve_toolset() 解析工具集
              │
              └── 回傳該工具集包含的所有工具名稱
```

## 預設工具集

| 工具集 | 包含工具 |
|--------|---------|
| `web` | web_search, web_extract |
| `search` | web_search |
| `vision` | vision_analyze |
| `image_gen` | image_generate |
| `terminal` | terminal, process |
| `moa` | delegate_task |
| `full_stack` | 繼承 web + terminal + file + ... |

## See also

- [Hermes Tool Registry](hermes-tool-registry.md)
- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
