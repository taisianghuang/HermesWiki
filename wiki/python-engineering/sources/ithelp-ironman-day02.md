---
title: "Day 2 - Python 環境管理：venv、conda 與 hatch 的選擇"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 環境管理, venv, conda, Hatch, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-02.md]
related: [ithelp-ironman-day01, ithelp-ironman-day03, python-engineering]
summary: 比較 Python 環境管理工具（venv、conda、poetry、hatch），並建議依場景選擇：小專案用 venv、資料科學用 conda、中小專案用 poetry、長期規模化用 hatch。
---

# Day 2 - Python 環境管理：venv、conda 與 hatch 的選擇

## 摘要

Python 環境管理工具百家爭鳴，本篇比較了 virtualenv、venv、conda、pipenv、poetry 與 hatch 六種工具的適用場景，並建議：學習/小專案用 venv、資料科學/AI 用 conda、中小專案用 poetry、長期規模化/團隊用 hatch。

## 關鍵論點

### 環境管理工具比較

| 工具 | 推出時間 | 功能範疇 | 適合場景 |
|------|----------|----------|----------|
| venv | 2012 (Python 3.3 內建) | 虛擬環境 | 學習、個人小專案 |
| conda | Anaconda 發行 | 環境 + 套件 | 資料科學、AI/ML |
| poetry | 2018 | 環境 + 套件 + 發佈 | 中小型專案、套件開發 |
| hatch | 2022 (PyPA) | 環境 + 套件 + 專案全流程 | 長期團隊專案、工程化管理 |

### 建議

- **小專案 / 學習用途**：用內建的 venv 就足夠
- **資料科學 / AI 領域**：建議用 conda（解決 Python + C/CUDA 相依問題）
- **中小型專案**：建議用 poetry，社群生態成熟
- **長期規模化 / 團隊維運 / 發佈套件**：建議用 hatch

### Hatch 快速上手

```bash
# 安裝 hatch（建議用 pipx）
pipx install hatch

# 建立新專案
hatch new demo-app

# 啟動虛擬環境
cd demo-app
hatch shell

# 執行測試
hatch run test
```

## 重要參考

- [Hatch — Modern Python project management](https://github.com/pypa/hatch)
- [PyPA - Python Packaging Authority](https://packaging.python.org/)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 2，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-day01](../sources/ithelp-ironman-day01.md) — Day 1：工程化的必要性与价值
- [ithelp-ironman-day03](../sources/ithelp-ironman-day03.md) — Day 3：pyproject.toml 详解
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
