---
title: Hermes Agent Loop Internals — Official Doc
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, agent-loop, official-docs]
sources: [../../raw/hermes-agent/docs/agent-loop.md]
related: [hermes-agent-core-loop, hermes-prompt-assembly, hermes-context-compression, hermes-provider-runtime]
summary: 官方 Agent Loop Internals 文檔，~9,200 行核心 orchestration engine。涵蓋 API Modes、Turn Lifecycle、Message Alternation、Interruptible API Calls、Tool Execution、Callbacks、Iteration Budget、Fallback Model、Compression Trigger 時機。
---

# Hermes Agent Loop Internals — Official Doc

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | https://hermes-agent.nousresearch.com/docs/developer-guide/agent-loop |
| **Raw 位置** | `raw/hermes-agent/docs/agent-loop.md` |
| **涵蓋內容** | API Modes、Turn Lifecycle、Message Format、Interruptible API、Tool Execution、Callbacks、Budget、Fallback |

## 與 AGENTS.md 的差異

| 項目 | AGENTS.md（既有） | Agent Loop Internals（官方） |
|------|------------------|------------------------------|
| **API Modes** | 提及 3 種 mode | 詳細表格 + resolution order |
| **Turn Lifecycle** | 簡化流程圖 | 9 步詳細 sequence |
| **Message Alternation** | 無 | 完整 alternation rules |
| **Interruptible API** | 無 | 有詳細時序圖 |
| **Tool Execution** | 簡單描述 | Sequential vs Concurrent + execution flow |
| **Agent-Level Tools** | 無 | 具體列出被攔截的 4 個 tools |
| **Callbacks** | 提及 callback surfaces | 完整表格（8 種 callbacks）|
| **Iteration Budget** | 簡單提及 | 兩層 pressure + fallback |
| **Compression Timing** | 無 | 區分 preflight 50% vs gateway 85% |

## 官方文檔原始 content

（完整內容收錄於 `raw/hermes-agent/docs/agent-loop.md`）
