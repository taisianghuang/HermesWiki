---
title: Hermes Gateway
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, gateway, messaging, platform-adapter]
sources: [../sources/2026-04-14-hermes-agent-repo.md]
related: [hermes-agent-core-loop, hermes-cli-architecture]
summary: Hermes Gateway 是訊息平台適配器系統，透過統一的 Message Handler 對接 Telegram、Discord、Slack 等多個平台，支援斜線指令和跨平台對話持久化。
---

# Hermes Gateway

## 概述

Hermes Gateway 是 Hermes 的**多平台訊息適配器系統**。它讓同一個 Agent 可以同時對接多個訊息平台（Telegram、Discord、Slack 等），並支援跨平台對話。

## 架構結構

```
gateway/
├── run.py            # 主循環、斜線指令、訊息分發
├── session.py        # SessionStore — 對話持久化
└── platforms/        # 平台適配器
    ├── telegram.py    ← Telegram
    ├── discord.py     ← Discord
    ├── slack.py       ← Slack
    ├── whatsapp.py    ← WhatsApp
    ├── signal.py      ← Signal
    ├── homeassistant.py
    └── ...
```

## 核心概念

### 1. Adapter Pattern（適配器模式）

每個平台有一個對應的 Adapter class，實作相同的介面：

```python
class PlatformAdapter:
    def connect() -> None: ...
    def disconnect() -> None: ...
    def send_message(chat_id: str, message: str) -> None: ...
    def on_message(handler: Callable) -> None: ...
```

### 2. Session Store（對話持久化）

`session.py` 中的 `SessionStore` 負責：
- 持久化對話歷史到 SQLite
- 管理 session 生命週期
- 支援 FTS5 全文搜尋

### 3. 斜線指令分發

Gateway 和 CLI 共用同一個 `COMMAND_REGISTRY`：

```python
# gateway/run.py
if canonical in GATEWAY_KNOWN_COMMANDS:
    return await self._handle_command(event, canonical)
```

## 訊息流程

```
User 在 Telegram 發送訊息
        │
        ▼
telegram.py adapter 接收
        │
        ▼
gateway/run.py 主循環
        │
        ▼
SessionStore 載入對話歷史
        │
        ▼
AIAgent.run_conversation() 處理
        │
        ▼
回覆訊息
        │
        ▼
telegram.py adapter 發送回 Telegram
```

## 支援的平台

| 平台 | Adapter 檔案 | 狀態 |
|------|-------------|------|
| Telegram | `platforms/telegram.py` | 主力支援 |
| Discord | `platforms/discord.py` | 支援 |
| Slack | `platforms/slack.py` | 支援 |
| WhatsApp | `platforms/whatsapp.py` | 支援 |
| Signal | `platforms/signal.py` | 支援 |
| Home Assistant | `platforms/homeassistant.py` | 支援 |
| Email | 透過 SMTP | 支援 |

## 跨平台對話

Hermes 支援透過 `/sethome` 設定「主通道」（Home Channel）：

```
/hermes config home telegram:8483266010
```

這樣定時任務（cron）的結果會發送到指定的平台。

## 斜線指令（Slash Commands）

Gateway 支援與 CLI 相同的斜線指令集合。關鍵代碼：

```python
# gateway/run.py
async def _handle_command(self, event, canonical):
    handler = self._resolve_handler(canonical)
    if handler:
        return await handler(event)
```

## 與 CLI 的關係

| 特性 | CLI | Gateway |
|------|-----|---------|
| 輸入方式 | prompt_toolkit 終端 | 訊息平台（Telegram 等） |
| 斜線指令 | 完整支援 | 完整支援 |
| 對話歷史 | 存 shell session | 存 SessionStore |
| 部署 | 本地 | 遠端伺服器 |

## 啟動方式

```bash
hermes gateway setup    # 首次設定（配置平台憑證）
hermes gateway start    # 啟動 gateway
```

## 安全性

- **DM Pairing**：只有允許的用戶可以與 Agent 對話
- **Command Approval**：危險指令需要二次確認
- **Token Lock**：每個平台 adapter 使用 scoped lock 防止憑證衝突

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes CLI Architecture](hermes-cli-architecture.md)
