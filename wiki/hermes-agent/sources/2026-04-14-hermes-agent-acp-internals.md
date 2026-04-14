---
title: Hermes Agent ACP Internals — Editor Integration
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, acp, editor-integration, vscode, zed, jetbrains]
sources: [../../raw/hermes-agent/docs/acp-setup.md]
related: [hermes-agent-core-loop, hermes-gateway]
summary: Hermes Agent 的 ACP（Agent Client Protocol）設定指南，涵蓋 VS Code、Zed、JetBrains 三大編輯器的整合設定，以及終端命令審批流程。
---

# Hermes Agent ACP Internals — Editor Integration

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | `raw/hermes-agent/docs/acp-setup.md` |
| **涵蓋內容** | VS Code / Zed / JetBrains 設定、審批流程、Toolsets 差異、Troubleshooting |

## 什麼是 ACP？

ACP（Agent Client Protocol）讓 Hermes Agent 作為編碼代理運行在你的編輯器內。編輯器發送任務給 Hermes，Hermes 回傳檔案編輯、終端命令和說明，全部顯示在編輯器原生介面。

## 三大編輯器設定

### VS Code

1. 安裝 **ACP Client** 擴充：`code --install-extension anysphere.acp-client`
2. 在 `settings.json` 設定：
```json
{
  "acpClient.agents": [{
    "name": "hermes-agent",
    "registryDir": "/path/to/hermes-agent/acp_registry"
  }]
}
```
3. 重啟 VS Code，在 ACP agent picker 中選擇 Hermes Agent

### Zed

在 `settings.json` 設定：
```json
{
  "agent_servers": {
    "hermes-agent": {
      "type": "custom",
      "command": "hermes",
      "args": ["acp"]
    }
  }
}
```

### JetBrains

1. 安裝 **ACP** plugin（Marketplace 搜尋「ACP」）
2. 設定 registry directory 為 `acp_registry/` 資料夾路徑

## ACP 提供的介面

| 功能 | 說明 |
|------|------|
| **Chat Panel** | 對話介面，描述任務、提問、下指令 |
| **File Diffs** | Hermes 編輯檔案時顯示標準 diff，可逐筆 accept/reject |
| **Terminal Commands** | 在整合終端中顯示命令，可設自動審批 |
| **Approval Flow** | 破壞性操作（刪除、shell、Git）會先請求審批 |

## Toolsets 差異

ACP sessions 使用精選的 `hermes-acp` toolset，專為編輯器工作流程設計，**刻意排除**訊息投遞、cronjob 管理、音訊優先 UX 等功能。

## 與 Gateway 的關係

- Gateway 是**異步訊息平台**介面（LINE、Discord、Slack...）
- ACP 是**同步編輯器整合**介面（VS Code、Zed、JetBrains...）
- 兩者共享同一個 `AIAgent` 核心

## See also

- [Hermes Gateway](hermes-gateway.md)
- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
