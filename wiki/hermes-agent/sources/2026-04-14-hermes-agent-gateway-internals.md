---
title: Hermes Gateway Internals — Official Doc
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, gateway, official-docs]
sources: [../../raw/hermes-agent/docs/gateway-internals.md]
related: [hermes-gateway, hermes-session-storage]
summary: 官方 Gateway Internals 文檔，涵蓋 Key Files 表格、Architecture Overview、Two-Level Message Guard、Authorization layers、DM Pairing Flow、14+ Platform Adapters、Hook Events、Memory Provider Integration。
---

# Hermes Gateway Internals — Official Doc

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | https://hermes-agent.nousresearch.com/docs/developer-guide/gateway-internals |
| **Raw 位置** | `raw/hermes-agent/docs/gateway-internals.md` |
| **涵蓋內容** | Key Files、Architecture、Message Flow、Two-Level Guard、Authorization、Platform Adapters、Hooks |

## 與既有概念頁的差異

| 項目 | 既有概念頁（hermes-gateway.md）| 官方文檔 |
|------|---------------------------|---------|
| **Key Files** | 列出 ~10 個 | 完整表格 + 行數（gateway/run.py ~7,500 行）|
| **Architecture** | ASCII flow | 更完整的 Overview diagram |
| **Two-Level Guard** | 提及 | 詳細說明 Level 1（Base adapter）vs Level 2（Gateway runner）|
| **Authorization** | 多層檢查 | 完整 flow + priority |
| **DM Pairing** | 提及 | 有完整 pairing flow 對話範例 |
| **Config Sources** | 無 | 差異說明（gateway 用 YAML loader，CLI 用 load_cli_config）|
| **Hooks** | 列舉 8 events | 完整表格 + lifecycle 說明 |
| **Memory Flush** | 無 | 有 Lifecycle 詳細說明 |
| **Process Management** | 提及 | 有 profile-scoped vs global 差異 |

## 14+ Platform Adapters

| Adapter | 說明 |
|---------|------|
| telegram.py | Telegram Bot API |
| discord.py | Discord bot |
| slack.py | Slack Socket Mode |
| whatsapp.py | WhatsApp Business Cloud |
| signal.py | Signal via signal-cli |
| matrix.py | Matrix via mautrix |
| mattermost.py | Mattermost WebSocket |
| email.py | Email via IMAP/SMTP |
| sms.py | SMS via Twilio |
| dingtalk.py | DingTalk WebSocket |
| feishu.py | Feishu/Lark |
| wecom.py | WeCom |
| weixin.py | Weixin via iLink |
| bluebubbles.py | Apple iMessage |
| webhook.py | Inbound/outbound webhook |
| api_server.py | REST API server |
| homeassistant.py | Home Assistant |

## 官方文檔原始 content

（完整內容收錄於 `raw/hermes-agent/docs/gateway-internals.md`）
