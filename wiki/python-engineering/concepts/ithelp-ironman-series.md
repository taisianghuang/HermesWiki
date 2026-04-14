---
title: "30 天 Python 專案工坊 — ithelp 鐵人賽系列"
type: concept
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 工程化, 鐵人賽, Hatch, FastAPI, Docker, CI/CD]
sources: []
related: [python-engineering]
summary: ithelp 2025 鐵人賽系列，30天不談語法、專講工程化：從環境管理到 Docker + CI/CD + 監控追蹤。
---

# 30 天 Python 專案工坊 — ithelp 鐵人賽系列

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **標題** | 30 天 Python 專案工坊：環境、結構、測試到部署全打通 |
| **作者** | shothead6062 |
| **團隊** | Cyber Edge Runners |
| **平台** | ithelp 鐵人賽（2025） |
| **總篇數** | 30 篇 |

## 系列目標

> 30 天不談語法、專講工程化：Hatch 環境、pyproject 依賴、測試與型別、FastAPI 六邊形設計、Docker＋CI/CD、監控追蹤。結尾交付可 fork 的模板與 Checklist。

## 核心主題

### 環境與工具鏈（Day 1-9）
- Hatch 作為統一的環境管理器
- pyproject.toml 作為專案設定中心
- 依賴管理與鎖定版本策略
- 自動化工作流（pre-commit、lint、format）

### 程式品質（Day 10-18）
- 型別檢查（mypy / pyright）
- Pydantic v2 資料驗證
- pytest 測試策略與 fixtures
- structured logging（structlog）
- 例外處理與重試機制（tenacity）
- 快取策略

### 架構設計（Day 19-21）
- SQLAlchemy 2.x + Repository Pattern
- FastAPI + Ports/Adapters（六邊形架構）
- 背景作業（Celery / Dramatiq / asyncio）

### 部署與維運（Day 22-30）
- Hatch build 發布流程
- SemVer 版本策略
- Docker 多階段建置 + 非 root
- Dev Container 開發體驗
- GitHub Actions CI/CD
- pip-audit 安全掃描
- Gunicorn/Uvicorn、Cloud Run、K8s
- OpenTelemetry 監控追蹤
- AI 協作（Vibe Coding）

## 教學方法

- **每天一主題**：一天一個具體課題
- **實作導向**：每篇都有可運行的程式碼
- **最終交付**：可 fork 的完整專案模板 + Checklist

## See also

- [Python 工程化](../concepts/python-engineering.md)
- [Day 1 — 為什麼要工程化](../sources/ithelp-ironman-day01.md)
