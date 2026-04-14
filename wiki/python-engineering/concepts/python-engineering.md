---
title: "Python 工程化"
type: concept
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 工程化, 軟體工程, 最佳實踐]
sources: [../../raw/ithelp-ironman-30day/day-01.md]
related: [ithelp-ironman-series]
summary: Python 工程化是指將 Python 專案從「能跑」提升到「可維護、可擴展、可協作」的軟體工程標準。
---

# Python 工程化

## 定義

Python 工程化是將 Python 專案從「能跑」提升到「可維護、可擴展、可協作」的軟體工程標準。核心問題：Python 長期被視為「腳本語言」，缺乏明確的專案結構與工程化工具。

## 四大核心價值

| 價值 | 說明 |
|------|------|
| **可維護性** | 程式碼結構清楚、檔案命名合理，半年後回來也能理解 |
| **一致性** | 本地、伺服器、CI/CD pipeline 執行結果一致 |
| **可測試性** | 自動化測試降低人為錯誤風險 |
| **可擴展性** | 專案從小腳本能自然演進，不需要推倒重來 |

## 傳統 Python 的問題

- 沒有標準專案結構（vs Java 的 Maven/Gradle、Node 的 package.json）
- 環境管理混亂（venv、conda、pipenv、poetry 各有優缺點）
- 依賴版本不確定，部署環境差異導致「在我機器上能跑」
- 缺乏統一的測試/發布流程

## 工程化工具全景圖

| 層面 | 工具 |
|------|------|
| **環境管理** | Hatch、venv、conda、poetry |
| **專案設定** | pyproject.toml（PEP 621 標準） |
| **依賴鎖定** | constraints file、lock file |
| **自動化工作流** | Hatch scripts、Nox、pre-commit |
| **程式品質** | Black、Ruff、isort、mypy、pyright |
| **測試** | pytest + fixtures + coverage |
| **API 框架** | FastAPI（Port/Adapter 架構） |
| **容器化** | Docker（多階段建置、非 root） |
| **CI/CD** | GitHub Actions |
| **監控** | OpenTelemetry |

## 與傳統 RAG 的對比（呼應 LLM Wiki 概念）

工程化 Python 就像給 wiki 建立 CLAUDE.md schema——不是每次都從零開始，而是建立結構讓知識/程式持續複利累積。

## See also

- [ithelp 鐵人賽系列](../concepts/ithelp-ironman-series.md)
- [Day 1 source](../sources/ithelp-ironman-day01.md)
