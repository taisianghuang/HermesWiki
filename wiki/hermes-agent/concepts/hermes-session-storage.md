---
title: Hermes Session Storage
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, session-storage, sqlite, fts5, session-lineage]
sources: [../../raw/hermes-agent/docs/architecture.md, ../../raw/hermes-agent/docs/session-storage.md]
related: [hermes-agent-core-loop, hermes-gateway, hermes-memory-system]
summary: Hermes 使用 SQLite（~/.hermes/state.db，WAL mode）持久化 session 資料。包含 sessions 表（metadata + billing）、messages 表（完整歷史）、messages_fts（FTS5 全文搜尋）。支援 Session Lineage（parent_session_id chains）處理 compression-triggered splits。
---

# Hermes Session Storage

## 概述

Hermes 使用 SQLite 資料庫 (`~/.hermes/state.db`) 持久化：
- Session metadata + billing
- 完整訊息歷史
- Model 配置

Source: `hermes_state.py`

## 架構

```
~/.hermes/state.db (SQLite, WAL mode)
├── sessions          — Session metadata, token counts, billing
├── messages          — Full message history per session
├── messages_fts      — FTS5 virtual table for full-text search
└── schema_version    — Migration state tracking
```

Key design decisions:
- **WAL mode**：Concurrent readers + one writer（gateway multi-platform）
- **FTS5**：Fast 全文搜尋
- **Session lineage**：via `parent_session_id` chains（compression-triggered splits）
- **Source tagging**：平臺過濾（cli, telegram, discord 等）

## Schema

### Sessions Table

```sql
CREATE TABLE sessions (
    id TEXT PRIMARY KEY,
    source TEXT NOT NULL,           -- 'cli', 'telegram', 'discord', etc.
    user_id TEXT,
    model TEXT,
    model_config TEXT,
    system_prompt TEXT,
    parent_session_id TEXT,         -- For lineage chains
    started_at REAL NOT NULL,       -- Unix epoch
    ended_at REAL,
    end_reason TEXT,
    message_count INTEGER DEFAULT 0,
    tool_call_count INTEGER DEFAULT 0,
    input_tokens INTEGER DEFAULT 0,
    output_tokens INTEGER DEFAULT 0,
    cache_read_tokens INTEGER DEFAULT 0,
    cache_write_tokens INTEGER DEFAULT 0,
    reasoning_tokens INTEGER DEFAULT 0,
    billing_provider TEXT,
    billing_base_url TEXT,
    billing_mode TEXT,
    estimated_cost_usd REAL,
    actual_cost_usd REAL,
    cost_status TEXT,
    cost_source TEXT,
    pricing_version TEXT,
    title TEXT,
    FOREIGN KEY (parent_session_id) REFERENCES sessions(id)
);
```

### Messages Table

```sql
CREATE TABLE messages (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    session_id TEXT NOT NULL REFERENCES sessions(id),
    role TEXT NOT NULL,
    content TEXT,
    tool_call_id TEXT,
    tool_calls TEXT,                -- JSON string
    tool_name TEXT,
    timestamp REAL NOT NULL,        -- Unix epoch
    token_count INTEGER,
    finish_reason TEXT,
    reasoning TEXT,                 -- Raw reasoning text
    reasoning_details TEXT,         -- JSON string
    codex_reasoning_items TEXT      -- JSON string
);
```

### FTS5 Virtual Table

```sql
CREATE VIRTUAL TABLE messages_fts USING fts5(
    content,
    content=messages,
    content_rowid=id
);
```

三個 triggers 保持 FTS5 與 messages 表同步（INSERT/UPDATE/DELETE）。

## Schema Migrations（版本 1→6）

| 版本 | 變更 |
|------|------|
| 1 | Initial schema |
| 2 | Add finish_reason to messages |
| 3 | Add title to sessions |
| 4 | Add unique index on title |
| 5 | Add billing columns（cache_read/write_tokens, reasoning_tokens, billing_*, cost_*）|
| 6 | Add reasoning columns（reasoning, reasoning_details, codex_reasoning_items）|

## Write Contention Handling

```python
_WRITE_MAX_RETRIES = 15
_WRITE_RETRY_MIN_S = 0.020   # 20ms
_WRITE_RETRY_MAX_S = 0.150   # 150ms
_CHECKPOINT_EVERY_N_WRITES = 50
```

策略：
- Short SQLite timeout（1s，而非預設 30s）
- Application-level retry with random jitter
- `BEGIN IMMEDIATE` transactions（在 transaction 開始時就暴露 lock contention）
- Periodic WAL checkpoints（PASSIVE mode）

## 常見操作

```python
from hermes_state import SessionDB
db = SessionDB()

# Create session
db.create_session(session_id="sess_abc", source="cli", model="anthropic/claude-sonnet-4.6")

# End session
db.end_session("sess_abc", end_reason="user_exit")

# Append message
msg_id = db.append_message(
    session_id="sess_abc",
    role="assistant",
    content="Here's the answer...",
    tool_calls=[{"id": "call_1", "function": {"name": "terminal", "arguments": "{}"}}],
    token_count=150,
    finish_reason="stop",
    reasoning="Let me think..."
)

# Get messages
messages = db.get_messages("sess_abc")
conversation = db.get_messages_as_conversation("sess_abc")  # OpenAI format

# Session titles
db.set_session_title("sess_abc", "Fix Docker Build")
session_id = db.resolve_session_by_title("Fix Docker Build")

# FTS search
results = db.search_messages("docker deployment")
results = db.search_messages("error", source_filter=["cli"], role_filter=["user"])

# Export/cleanup
data = db.export_session("sess_abc")
db.prune_sessions(older_than_days=90)
db.delete_session("sess_abc")
```

## Session Lineage

Sessions 透過 `parent_session_id` 形成鏈。當 context compression 觸發 session split 時（gateway），會建立 lineage。

```sql
-- Find all ancestors of a session
WITH RECURSIVE lineage AS (
    SELECT * FROM sessions WHERE id = ?
    UNION ALL
    SELECT s.* FROM sessions s
    JOIN lineage l ON s.id = l.parent_session_id
)
SELECT id, title, started_at, parent_session_id FROM lineage;
```

Compression 會：
1. Flush memory to disk first
2. Summarize middle turns
3. Generate new session lineage ID（建立 "child" session）

## FTS5 Query Syntax

```sql
-- Keywords (implicit AND)
docker deployment

-- Quoted phrase
"exact phrase"

-- Boolean OR
docker OR kubernetes

-- Boolean NOT
python NOT java

-- Prefix match
deploy*
```

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Gateway](hermes-gateway.md)
- [Hermes Memory System](hermes-memory-system.md)
