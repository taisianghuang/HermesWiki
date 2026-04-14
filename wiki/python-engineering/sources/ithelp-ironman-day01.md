---
title: "Day 1 - 為什麼要「工程化的 Python」：系列導讀與範例專案藍圖"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 工程化, Hatch, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-01.md]
related: [ithelp-ironman-series]
summary: 探討 Python 為何需要工程化——從腳本語言到 ML/企業級應用的轉變，以及工程化的四大核心價值。
---

# Day 1 - 為什麼要「工程化的 Python」：系列導讀與範例專案藍圖

## 摘要

Python 從 1980 年代誕生，2014 年後因 ML/深度學習熱潮躍升主流。隨著應用場景從小腳本擴展到大型專案，工程化成為必要。本篇介紹工程化的四大價值，以及 30 天系列的學習路線圖。

## 關鍵論點

### Python 的演進
- 1980 年代末誕生，長期被視為腳本語言
- 2014 年前後因 ML/深度學習生態（NumPy、TensorFlow、PyTorch）躍升主流
- 底層高效能套件以 C/C++ 撰寫，Python 提供簡單直覺介面
- 應用場景：從「小腳本」擴展到「大型專案」
- **核心問題**：Python 缺乏像 Java/.NET Core 那樣明確的標準專案結構與工程化工具

### 工程化的四大價值

| 價值 | 說明 |
|------|------|
| **可維護性** | 程式碼結構清楚、檔案命名合理，半年後回來也能理解 |
| **一致性** | 本地、伺服器、CI/CD pipeline 執行結果一致 |
| **可測試性** | 自動化測試降低人為錯誤風險 |
| **可擴展性** | 專案從小腳本能自然演進，不需要推倒重來 |

### 30 天學習路線圖

| 階段 | 天數 | 內容 |
|------|------|------|
| 環境與結構 | Day 1-9 | Hatch、pyproject.toml、目錄結構、依賴管理、工作流 |
| 品質與穩定性 | Day 10-18 | 型別檢查、測試、日誌、例外、快取、效能觀測 |
| 資料與 API | Day 19-21 | SQLAlchemy、FastAPI、背景作業 |
| 發布與部署 | Day 22-28 | 發布版本、Docker、Dev Container、CI/CD、監控 |
| 總結 | Day 29-30 | AI 協作、模板交付 |

## 重要參考

- [Hatch — Modern Python project management](https://github.com/pypa/hatch)
- [PEP 621 — Storing project metadata in pyproject.toml](https://peps.python.org/pep-0621/)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的第一篇，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
