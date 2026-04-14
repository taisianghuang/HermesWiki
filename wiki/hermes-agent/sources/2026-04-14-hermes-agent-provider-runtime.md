---
title: Hermes Provider Runtime Resolution — Official Doc
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, provider, official-docs]
sources: [../../raw/hermes-agent/docs/provider-runtime.md]
related: [hermes-provider-runtime, hermes-agent-core-loop]
summary: 官方 Provider Runtime Resolution 文檔，涵蓋 Resolution Precedence、18+ Provider Families、API Mode Resolution、Credential Resolution、Auxiliary Model Routing、Fallback Model 機制。
---

# Hermes Provider Runtime Resolution — Official Doc

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | https://hermes-agent.nousresearch.com/docs/developer-guide/provider-runtime |
| **Raw 位置** | `raw/hermes-agent/docs/provider-runtime.md` |
| **涵蓋內容** | Resolution Precedence、18+ Providers、Credential Resolution、Auxiliary Routing、Fallback |

## 與既有概念頁的差異

| 項目 | 既有概念頁 | 官方文檔 |
|------|----------|---------|
| **Provider Families** | 列舉 10 個 | 完整清單（18+）|
| **Resolution Order** | 有 | 更詳細說明 + 為何這樣設計 |
| **Credential Resolution** | 提及 | 詳細的 per-provider 流程 |
| **Auxiliary Routing** | 有 | 完整 YAML config 範例 |
| **Fallback** | 有 | 補充 401/403 credential refresh 機制 |

## 官方文檔原始 content

（完整內容收錄於 `raw/hermes-agent/docs/provider-runtime.md`）
