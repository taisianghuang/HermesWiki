---
title: Hermes Agent Architecture — Official Docs
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, architecture, official-docs]
sources: [../../raw/hermes-agent/docs/architecture.md]
related: [hermes-agent-core-loop, hermes-tool-registry, hermes-gateway, hermes-cli-architecture, hermes-toolsets, hermes-skills-system, hermes-memory-system]
summary: Hermes Agent 官方架構文檔，包含 System Overview、Directory Structure（詳細行數）、Data Flow（CLI/Gateway/Cron 三種流程）、Major Subsystems 說明、以及 Design Principles。
---

# Hermes Agent Architecture — Official Docs

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | https://hermes-agent.nousresearch.com/docs/developer-guide/architecture |
| ** Raw 位置** | `raw/hermes-agent/docs/architecture.md` |
| **涵蓋內容** | System Overview、Directory Structure、Data Flow、Major Subsystems、Design Principles |

## 與 AGENTS.md 的差異摘要

官方 Architecture 文件相比 AGENTS.md 有以下補充：

| 項目 | AGENTS.md（既有） | 官方 Architecture（新版） |
|------|-------------------|--------------------------|
| **工具數量** | 40+ tools | **47 tools**, 20 toolsets |
| **平台適配器** | ~6 個（Telegram, Discord, Slack, WhatsApp, Signal, HomeAssistant） | **15 個**，包含 matrix, mattermost, email, sms, dingtalk, feishu, wecom, weixin, bluebubbles, webhook |
| **行數資訊** | 無精確行數 | `run_agent.py` ~9,200 行、`cli.py` ~8,500 行、`gateway/run.py` ~7,500 行、`mcp_tool.py` ~2,200 行 |
| **agent/ 目錄** | 10 個模組 | **14 個模組**，新增 `context_engine.py`、`memory_manager.py`、`memory_provider.py`、`anthropic_adapter.py` |
| **hermes_cli/ 目錄** | ~10 個模組 | **15 個模組**，新增 `runtime_provider.py`、`plugins.py`、`delivery.py`、`pairing.py`、`hooks.py`、`mirror.py`、`status.py`、`builtin_hooks/` |
| **Plugin System** | 未提及 | 有完整章節，支援 3 種發現源 |
| **ACP Integration** | 僅提到 | 有獨立章節 |
| **Data Flow** | 僅有 Agent Loop | **三種完整流程**：CLI Session、Gateway Message、Cron Job |
| **Design Principles** | 無 | 有六項設計原則 |
| **Provider Resolution** | 僅提到多種 provider | 有獨立 subsystem 說明（runtime_provider.py，18+ providers） |
| **行數估計** | AGENTS.md ~20KB | Architecture doc ~14KB |

## System Overview（系統概觀）

```
Entry Points
  CLI (cli.py) | Gateway (gateway/run.py) | ACP (acp_adapter/)
  Batch Runner | API Server | Python Library
         │
         ▼
┌──────────────────────────────────────┐
│     AIAgent (run_agent.py)           │
│                                      │
│  Prompt Builder  │ Provider Resolution │ Tool Dispatch │
│  Compression     │ 3 API Modes        │ Tool Registry  │
│  & Caching      │ (chat/completions, │ 47 tools      │
│                  │  codex, anthropic) │ 40 toolsets    │
└──────────────────────────────────────┘
         │                      │
         ▼                      ▼
  Session Storage           Tool Backends
  (SQLite + FTS5)           Terminal (6)
  hermes_state.py           Browser (5)
  gateway/session.py         Web (4)
                            MCP (dynamic)
                            File, Vision, etc.
```

## 三種 Data Flow

### 1. CLI Session
```
User input → HermesCLI.process_input()
  → AIAgent.run_conversation()
    → prompt_builder.build_system_prompt()
    → runtime_provider.resolve_runtime_provider()
    → API call
    → tool_calls? → model_tools.handle_function_call() → loop
    → final response → display → save to SessionDB
```

### 2. Gateway Message
```
Platform event → Adapter.on_message() → MessageEvent
  → GatewayRunner._handle_message()
    → authorize user → resolve session key
    → create AIAgent with session history
    → AIAgent.run_conversation()
    → deliver response back through adapter
```

### 3. Cron Job
```
Scheduler tick → load due jobs from jobs.json
  → create fresh AIAgent (no history)
  → inject attached skills as context
  → run job prompt
  → deliver response to target platform
  → update job state and next_run
```

## Design Principles

| 原則 | 實踐 |
|------|------|
| **Prompt stability** | System prompt 不會在對話中途改變，只有明確的 `/model` 才會造成 cache break |
| **Observable execution** | 每個 tool call 都透過 callbacks 對用戶可見 |
| **Interruptible** | API 呼叫和 tool 執行都可以被使用者輸入或訊號取消 |
| **Platform-agnostic core** | 同一個 AIAgent class 同時服務 CLI、Gateway、ACP、batch、API server |
| **Loose coupling** | 選用性子系統（MCP、plugins、memory providers、RL environments）使用 registry pattern，而非 hard dependencies |
| **Profile isolation** | 每個 profile (`hermes -p <name>`) 擁有獨立的 HERMES_HOME、config、memory、sessions、gateway PID |

## Major Subsystems

1. **Agent Loop** — 同步協調引擎，支援 3 種 API modes
2. **Prompt System** — 包含 prompt_builder、prompt_caching、context_compressor
3. **Provider Resolution** — 18+ providers，OAuth flows，credential pools
4. **Tool System** — 47 tools，20 toolsets，6 種 terminal backends
5. **Session Persistence** — SQLite + FTS5，lineage tracking
6. **Messaging Gateway** — 14 platform adapters
7. **Plugin System** — 3 discovery sources，memory providers + context engines
8. **ACP Integration** — VS Code / Zed / JetBrains
9. **RL / Environments / Trajectories** — Atropos 整合

## 精確行數對照

| 檔案 | 行數 |
|------|------|
| `run_agent.py` | ~9,200 |
| `cli.py` | ~8,500 |
| `gateway/run.py` | ~7,500 |
| `hermes_cli/main.py` | ~5,500 |
| `hermes_cli/setup.py` | ~3,100 |
| `mcp_tool.py` | ~2,200 |

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Tool Registry](hermes-tool-registry.md)
- [Hermes Gateway](hermes-gateway.md)
- [Hermes CLI Architecture](hermes-cli-architecture.md)
- [Hermes Toolsets](hermes-toolsets.md)
- [Hermes Skills System](hermes-skills-system.md)
- [Hermes Memory System](hermes-memory-system.md)
