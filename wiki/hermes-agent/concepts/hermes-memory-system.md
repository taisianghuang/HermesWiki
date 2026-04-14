---
title: Hermes Memory System
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, memory, session, fts5]
sources: [../sources/2026-04-14-hermes-agent-repo.md]
related: [hermes-agent-core-loop, hermes-skills-system]
summary: Hermes 的記憶系統分為短期（Session History）和長期（Persistent Memory），長期記憶又分為 user profile（用戶偏好）和 agent memory（自我認知），並支援 FTS5 全文搜尋。
---

# Hermes Memory System

## 概述

Hermes 的記憶系統是**多層次的**，分為：
- **短期記憶**：對話歷史（Session）
- **長期記憶**：持久化記憶（user profile + agent memory）

## 記憶層次架構

```
┌─────────────────────────────────────────────┐
│         Hermes Memory System                 │
├─────────────────────────────────────────────┤
│                                             │
│  短期記憶 (Short-term)                      │
│  └── Session History                        │
│      └── 當前對話的所有 messages           │
│                                             │
│  長期記憶 (Long-term)                       │
│  ├── user_profile (USER.md)                 │
│  │   └── 用戶偏好、習慣、背景              │
│  └── agent_memory (MEMORY.md)               │
│      └── Agent 對世界的認知、學習           │
│                                             │
│  跨 Session 搜尋                             │
│  └── FTS5 全文搜尋 Session History          │
│      └── 透過 LLM summarization 召回        │
└─────────────────────────────────────────────┘
```

## 儲存位置

```
~/.hermes/
├── memory/              # 長期記憶目錄
│   ├── USER.md         # 用戶 profile
│   └── MEMORY.md       # Agent 自我認知
├── sessions.db         # SQLite (FTS5) — Session 歷史
└── skills/             # Skills (程序記憶)
```

## Session History (短期記憶)

透過 `hermes_state.py` 的 `SessionDB` 管理：

```python
class SessionDB:
    """SQLite + FTS5 session store"""
    
    def search_sessions(self, query: str) -> list:
        """FTS5 全文搜尋歷史對話"""
        
    def save_message(self, session_id: str, role: str, content: str):
        """保存單條訊息"""
        
    def get_history(self, session_id: str) -> list:
        """獲取對話歷史"""
```

## FTS5 全文搜尋

Hermes 使用 SQLite 的 FTS5 模組實現高效的全對話歷史搜尋：

```python
# 搜尋過去所有對話中提到「營造業」的内容
results = session_db.search_sessions("營造業")

# 返回格式：
# [
#   {"session_id": "xxx", "snippet": "...營造業相關段落...", "date": "2026-01-15"},
#   ...
# ]
```

搜尋結果會透過 **LLM summarization** 濃縮，產生可讀的摘要。

## Persistent Memory (長期記憶)

### USER.md（用戶 Profile）

儲存用戶的：
- 姓名、背景
- 偏好設置
- 通訊方式
- 常用語言
- 專案上下文

由 `agent/memory_manager.py` 的 `build_memory_context_block()` 處理。

### MEMORY.md（Agent 自我認知）

儲存 Agent 的：
- 對用戶的理解
- 持續學習的知識
- 跨 session 的狀態

## Memory Manager

```python
# agent/memory_manager.py
def build_memory_context_block():
    """構建要注入到 system prompt 的記憶區塊"""
    
    # 讀取 USER.md 和 MEMORY.md
    # 格式化為字串
    # 返回 {"role": "system", "content": "..."}
```

## 與 Skills System 的差異

| 特性 | Memory | Skills |
|------|--------|--------|
| 內容 | 事實、偏好 | 操作程序 |
| 創建者 | Agent 或用戶 | Agent 自動生成 |
| 顆粒度 | 片段/文件 | 完整技能文件 |
| 使用方式 | 注入到 system prompt | 自動觸發 |

## Honcho 相容性

Hermes 的用戶模型與 [Honcho](https://github.com/plastic-labs/honcho) dialectic user modeling 系統相容，支援：
- 對話式用戶建模
- 跨 session 的用戶理解演化

## 記憶的召回觸發

記憶會在以下時機被召回：

1. **新對話開始時**：`build_memory_context_block()` 讀取 USER.md
2. **用戶提到相關主題**：透過 `/remember` 或自動關鍵詞
3. **跨 session 搜尋**：透過 FTS5 + LLM summarization

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Skills System](hermes-skills-system.md)
