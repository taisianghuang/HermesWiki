---
title: Hermes Session Storage — Official Doc
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, session-storage, sqlite, official-docs]
sources: [../../raw/hermes-agent/docs/session-storage.md]
related: [hermes-session-storage, hermes-agent-core-loop]
summary: 官方 Session Storage 文檔，涵蓋完整 SQLite Schema（sessions + messages + FTS5 triggers）、Schema Migrations v1→v6、Write Contention Handling、Common Operations API、Session Lineage SQL 範例。
---

# Hermes Session Storage — Official Doc

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | https://hermes-agent.nousresearch.com/docs/developer-guide/session-storage |
| **Raw 位置** | `raw/hermes-agent/docs/session-storage.md` |
| **涵蓋內容** | 完整 SQLite Schema、Schema Migrations、Write Contention、Operations API、Lineage SQL |

## 與既有概念頁的差異

| 項目 | 既有概念頁 | 官方文檔 |
|------|----------|---------|
| **完整 Schema** | 有（truncated）| 完整含所有 columns |
| **Migrations** | 列舉版本 | 詳細說明每個版本的變更 |
| **Write Contention** | 有常數 | 詳細說明 BEGIN IMMEDIATE + WAL checkpoint |
| **Operations** | 主要方法 | 完整含 filtered search、title resolution |
| **Lineage SQL** | 提及 | 完整 WITH RECURSIVE 範例 |
| **Query Examples** | 無 | 有 Token Usage + Recent Sessions + Preview 範例 |

## 官方文檔原始 content

（完整內容收錄於 `raw/hermes-agent/docs/session-storage.md`）
