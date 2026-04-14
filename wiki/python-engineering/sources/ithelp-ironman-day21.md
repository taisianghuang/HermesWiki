---
title: Day 21 - 背景作業選型：Celery / Dramatiq / asyncio 任務
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, engineering, background-tasks, celery, dramatiq, asyncio]
sources: [../../raw/ithelp-ironman-30day/day-21.md]
related: [python-engineering, ithelp-ironman-series]
summary: 背景作業三選一：Celery（企業級工作流）、Dramatiq（輕量日常）、asyncio（零外部依賴）
---

# Day 21 - 背景作業選型：Celery / Dramatiq / asyncio 任務

## Summary

本文聚焦 Python 背景任務處理的三條路線：Celery（成熟企業級）、Dramatiq（輕量直覺）、asyncio（零 broker）。涵蓋何時該把工作丟到背景、任務框架選型、Celery/Dramatiq 實作範例、重試策略、排程、與 API 的整合方式，以及可觀測性建設。

## Key Findings

### 什麼工作該丟背景
- 慢 I/O 或高不確定性：第三方 API、郵件、檔案轉碼、Webhook
- 可重試且可延後：失敗不影響當下使用者體驗
- CPU 吃重且離線可接受：報表產出、ML 特徵抽取
- 需要排程/節流/聚合：批次同步、Rate limit 保護

### 三框架比較

| 面向 | Celery | Dramatiq | asyncio 任務 |
|------|--------|----------|--------------|
| Broker | Redis、RabbitMQ | Redis、RabbitMQ | 無需外部 broker |
| 工作流 | group/chord/map 豐富 | 輕量，middleware 組裝 | TaskGroup/自寫 DAG |
| 重試/延遲 | 內建策略完整 | 直覺裝飾器 | 自行用 tenacity |
| 排程 | celery beat | dramatiq-cron | APScheduler/系統 cron |
| 典型場景 | 複雜工作流、大團隊 | 中小團隊、80% 需求 | 邊車任務、單機、PoC |

### Celery 實作重點
- 使用 `pyproject.toml` 設定 optional-dependencies
- `task_serializer="json"`、worker_prefetch_multiplier=1
- 搭配 celery beat 做排程任務

### Dramatiq 實作重點
- `@dramatiq.actor(max_retries=5, retry_when=...)` 裝飾器
- RedisBroker 簡潔設定
- 適合不想在設定上花太多力氣的團隊

### asyncio 任務適用場景
- 單節點服務、內部工具
- 已廣泛使用 asyncio/anyio 的專案
- 任務失敗可容忍、量級中小

### 可觀測性建設
- JSON 結構化日誌：event/task/task_id/retry_count/latency_ms
- 指數回退 + 抖動重試策略
- 版本化快取 key 避免 stampede
- 設定全部走設定層，broker URL、Queue 名稱等

### 與 API 整合
- API 端點立刻回 202 Accepted
- 搭配 request_id 與 task_id 形成端到端追蹤鏈

### 一鍵化
- `hatch run worker:worker` 啟 worker
- `hatch run dq:worker` 啟動 Dramatiq worker
- `hatch run worker:beat` 啟排程

## See also

- [python-engineering](../concepts/python-engineering.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
