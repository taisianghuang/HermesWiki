---
title: Hermes ACP Integration
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, acp, editor-integration, vscode, zed, jetbrains]
sources: [../../raw/hermes-agent/docs/acp-setup.md, ../sources/2026-04-14-hermes-agent-acp-internals.md]
related: [hermes-gateway, hermes-agent-core-loop]
summary: Hermes Agent 的 ACP（Agent Client Protocol）讓 Agent 运行在 VS Code、Zed、JetBrains 編輯器中，透過檔案 diffs、終端整合、審批流程提供原生的編碼輔助體驗。
---

# Hermes ACP Integration

## 概述

ACP（Agent Client Protocol）是 Hermes Agent 對**編碼助理場景**的解答。不同於 Gateway 的異步訊息平台模型，ACP 是**同步編輯器整合**模型——讓 Hermes 直接嵌入你的開發環境。

## 支援的編輯器

| 編輯器 | 設定方式 |
|--------|----------|
| **VS Code** | 安裝 `anysphere.acp-client` 擴充，設定 `registryDir` |
| **Zed** | 內建 ACP 支援，設定 `agent_servers` JSON |
| **JetBrains** | 安裝 ACP plugin，設定 registry directory |

## 核心能力

- **Chat Panel**：編輯器內對話，直接描述任務
- **File Diffs**：標準 diff 視圖，可逐筆 accept/reject
- **Terminal Commands**：整合終端顯示，支援自動或手動審批
- **Approval Flow**：刪除、shell、Git 等危險操作需確認

## 與 Gateway 的架構差異

| 維度 | Gateway | ACP |
|------|---------|-----|
| 訊息模型 | 異步（訊息佇列） | 同步（請求-回應） |
| 典型用途 | LINE/Discord/Slack Bot | VS Code/Zed 編碼助理 |
| 平台數 | 14+ | 3（主要） |
| Session 管理 | 長期對話歷史 | 編輯器工作區綁定 |

兩者共享同一個 `AIAgent` 核心，只是入口不同。

## Toolsets 差異

ACP sessions 使用精選的 `hermes-acp` toolset，刻意排除：
- 訊息投遞功能（LINE/Discord delivery）
- Cronjob 管理
- 音訊優先 UX 功能

這是因為編輯器場景不需要這些功能，且可避免不必要的 tool call 困惑。

## See also

- [Hermes Gateway](hermes-gateway.md)
- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
