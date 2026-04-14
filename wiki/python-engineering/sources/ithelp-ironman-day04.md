---
title: "Day 4 - 專案目錄結構：從腳本到可維護的專案設計"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 目錄結構, 專案架構, 工程化, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-04.md]
related: [ithelp-ironman-day03, ithelp-ironman-day05, python-engineering]
summary: 介紹 Python 專案目錄結構的演進：從單一腳本到模組化結構，探討 src/ 佈局的原因，以及針對函式庫、CLI 工具、Web API、資料科學等不同場景的專案結構建議。
---

# Day 4 - 專案目錄結構：從腳本到可維護的專案設計

## 摘要

本篇介紹 Python 專案目錄結構的演進：從最簡單的單一腳本，到可擴充的資料夾結構，再到模組化的 src/ 架構，並針對不同應用場景（函式庫、CLI、Web API、資料科學）提供具體的目錄建議。

## 關鍵論點

### 專案結構演進

1. **最簡單的腳本式專案**：只有 `main.py`，適合一次性任務
2. **可擴充的進階結構**：加入 docs/、scripts/、notebooks/、ci/ 等資料夾
3. **模組化的專案結構**：使用 src/ + tests/ 骨架，符合社群慣例

### 推薦的模組化結構

```
my_project/
├── pyproject.toml
├── README.md
├── src/my_project/
│   ├── __init__.py
│   ├── core.py
│   ├── data.py
│   └── model.py
└── tests/
    └── test_core.py
```

### 為什麼要用 src/ 目錄？

- **防呆設計**：強迫開發者必須透過正確的安裝或設定路徑才能引入程式碼
- **避免測試與正式安裝環境不一致**的問題
- 確保即使沒有安裝套件也能 import 的錯誤不會發生

### 不同場景的專案結構

| 場景 | 特色結構 |
|------|----------|
| **函式庫 / SDK** | src/my_project/core/ + public_api.py + version.py |
| **CLI 工具** | src/my_project/adapters/cli/main.py + commands/ |
| **Web API** | src/my_project/adapters/web/app.py + routers/ + deps.py |
| **資料科學 / ML** | data/(raw/interim/processed) + notebooks/ + src/my_project/(features/training/pipelines) |

### 核心設計原則

- **降低耦合**：不同模組之間依賴關係要清晰
- **提高內聚**：每個模組的職責要單一

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 4，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-day03](../sources/ithelp-ironman-day03.md) — Day 3：pyproject.toml 详解
- [ithelp-ironman-day05](../sources/ithelp-ironman-day05.md) — Day 5：Hatch 基本操作
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
