---
title: Hermes Agent Repo - NousResearch
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, nous-research, ai-agent, open-source]
sources: []
related: [hermes-agent-core-loop, hermes-tool-registry, hermes-gateway, hermes-cli-architecture]
summary: Hermes Agent 是 NousResearch 開發的自我改進 AI Agent，具備內建學習循環、跨平台Gateway、多工具系統，是一個可運行於 VPS 到 GPU 集群的通用 Agent 框架。
---

# Hermes Agent Repo - NousResearch

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **Repo** | https://github.com/NousResearch/hermes-agent |
| **官方文件** | https://hermes-agent.nousresearch.com/docs/ |
| **Discord** | https://discord.gg/NousResearch |
| **License** | MIT |
| **主要語言** | Python 3.11+ |

## 核心定位

> **"The self-improving AI agent built by Nous Research."**

Hermes Agent 是一個具備**自我改進能力**的通用 AI Agent 框架，特色包括：
- 內建學習循環（從經驗中創建技能、改善技能）
- 跨平台訊息 Gateway（Telegram、Discord、Slack、WhatsApp、Signal）
- 40+ 內建工具，可擴展的 Tool Registry 系統
- 支援多種 LLM Provider（OpenRouter、MiniMax、Kimi、OpenAI、Anthropic 等）
- 可運行於本地、Docker、SSH、Daytona、Modal、Singularity 等多種後端

## 專案結構總覽

```
hermes-agent/
├── run_agent.py           # AIAgent 類 — 核心對話循環（~10800行）
├── model_tools.py         # 工具調度、handle_function_call()
├── toolsets.py            # 工具集定義
├── cli.py                 # HermesCLI 類 — 互動式 CLI 協調器（~447KB）
├── hermes_state.py        # SessionDB — SQLite 對話存儲（FTS5搜尋）
├── agent/                 # Agent 內部模組
│   ├── prompt_builder.py      # 系統提示組裝
│   ├── context_compressor.py  # 自動上下文壓縮
│   ├── prompt_caching.py      # Anthropic Prompt Caching
│   ├── auxiliary_client.py     # 輔助 LLM 客戶端
│   ├── display.py             # KawaiiSpinner、工具預覽格式化
│   └── skill_commands.py      # Skill 斜線指令
├── hermes_cli/            # CLI 子命令
│   ├── main.py           # 入口點
│   ├── commands.py       # 斜線指令定義（CommandDef  registry）
│   ├── skin_engine.py    # 皮膚/主題引擎
│   └── ...
├── tools/                # 工具實現（59個檔案）
│   ├── registry.py       # 中央工具註冊表
│   ├── terminal_tool.py  # 終端協調
│   ├── mcp_tool.py       # MCP 客戶端（~1050行）
│   └── environments/     # 終端後端
├── gateway/              # 訊息平台閘道
│   ├── run.py            # 主循環、斜線指令
│   ├── session.py        # SessionStore
│   └── platforms/        # 適配器（Telegram、Discord、Slack...）
├── acp_adapter/          # ACP 適配器（VS Code / Zed / JetBrains）
├── cron/                 # 排程器
└── tests/                # ~3000 測試
```

## 核心設計原則

### 1. 檔案依賴鏈（單一來源）

```
tools/registry.py  (無依賴 — 所有工具檔案都 import 它)
       ↑
tools/*.py  (每個工具檔案在 import 時呼叫 registry.register())
       ↑
model_tools.py  (導入 registry + 觸發工具發現)
       ↑
run_agent.py, cli.py, batch_runner.py
```

### 2. 斜線指令集中定義

所有 `/command` 在 `hermes_cli/commands.py` 的 `COMMAND_REGISTRY` 集中定義，自動影響：
- CLI help
- Gateway help
- Telegram Bot menu
- Slack subcommand routing
- Autocomplete

### 3. Tool Registry 模式

每個工具檔案在 module-level 呼叫 `registry.register()`，而非在中央設定檔定義。model_tools.py 透過 `_discover_tools()` 自動發現所有工具。

## 支援的 LLM Provider

| Provider | 特色 |
|---------|------|
| OpenRouter | 200+ 模型 |
| MiniMax | MiniMax API |
| Kimi/Moonshot | Moonshot API |
| OpenAI | GPT 系列 |
| Anthropic | Claude 系列 |
| Nous Portal | Nous 自有服務 |
| 自定義 endpoint | OpenAI-compatible API |

## 支援的訊息平台

- Telegram
- Discord
- Slack
- WhatsApp
- Signal
- Home Assistant
- Email

## 與其他框架的比較

| 特性 | Hermes | LangChain | AutoGPT |
|------|--------|-----------|---------|
| 工具註冊 | Registry 模式 | Tool 類 | Plugin |
| 學習循環 | 內建 | 無 | 無 |
| 多平台 Gateway | 原生支援 | 無 | 無 |
| 部署方式 | 輕量（$5 VPS 到 GPU） | 較重 | 較重 |
| 框架自建程度 | 幾乎全部自建 | 大量依賴第三方 | 大量依賴第三方 |

## 關鍵檔案

- `run_agent.py` (~10800 lines) — 核心 Agent 迴圈
- `cli.py` (~447KB) — CLI 實現
- `tools/mcp_tool.py` (~1050 lines) — MCP 客戶端
- `AGENTS.md` (~20851 chars) — 開發者指南

## See also

- [Hermes Agent Core Loop](wiki/concepts/hermes-agent-core-loop.md)
- [Hermes Tool Registry](wiki/concepts/hermes-tool-registry.md)
- [Hermes Gateway](wiki/concepts/hermes-gateway.md)
- [Hermes CLI Architecture](wiki/concepts/hermes-cli-architecture.md)
