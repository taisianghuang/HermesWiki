---
title: "Day 9 - 風格統一：Black + Ruff + isort + pre-commit"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 程式碼風格, Black, Ruff, pre-commit, CI/CD, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-09.md]
related: [ithelp-ironman-day08, ithelp-ironman-day10, ithelp-ironman-series]
summary: 介紹如何透過 Black（formatter）、Ruff（linter）、pre-commit（git hook）實現程式碼風格自動化，並與 Hatch scripts、Nox 整合，減少團隊無意義的風格爭論。
---

# Day 9 - 風格統一：Black + Ruff + isort + pre-commit

## 摘要

本篇介紹如何透過工具組合實現程式碼風格自動化：Black 作為唯一 formatter、Ruff 整合 lint 與 import 排序、pre-commit 在 git commit 前自動把關入口。並展示如何與 Hatch scripts、Nox 整合，確保所有工具使用同一份設定，減少團隊無意義的風格爭論。

## 關鍵論點

### 為什麼要統一風格？

- **減少無意義差異**：PR diff 乾淨，reviewer 專注在邏輯
- **降低心智負擔**：不用記住規範，交給工具自動處理
- **新手友善**：pre-commit 幫忙修正，不怕踩雷

👉 **結論：交給工具，不要交給人。**

### 工具組合

| 工具 | 職責 |
|------|------|
| **Black** | 唯一 formatter，專注排版 |
| **Ruff** | 超快 linter，整合 flake8/pyflakes/pycodestyle/isort，可自動修正 |
| **isort**（可選） | 傳統 import 排序器；若團隊已習慣可保留，否則讓 Ruff 的 I 規則取代 |
| **pre-commit** | 在 git commit 前自動執行，把關入口 |

### pyproject.toml 設定

```toml
[tool.black]
line-length = 100
target-version = ["py312"]

[tool.ruff]
line-length = 100
target-version = "py312"
select = ["E", "F", "I", "B", "UP", "SIM"]
ignore = ["E203"]  # 與 Black 衝突的 slicing 規則
exclude = ["build", "dist", ".venv"]

[tool.ruff.lint.isort]
known-first-party = ["your_project"]
combine-as-imports = true
force-sort-within-sections = true
```

### pre-commit 設定（呼叫 Hatch scripts）

```yaml
repos:
 - repo: local
   hooks:
   - id: hatch-fmt
     name: hatch fmt
     entry: hatch
     language: system
     args: ["run", "fmt"]
     types: [python]
     pass_filenames: false

   - id: hatch-check
     name: hatch check
     entry: hatch
     language: system
     args: ["run", "check"]
     types: [python]
     pass_filenames: false
```

### Hatch scripts 整合

```toml
[tool.hatch.envs.default.scripts]
fmt = ["black .", "ruff --fix ."]
lint = ["ruff ."]
check = ["black --check .", "ruff ."]
```

使用：`hatch run fmt`（自動修正）、`hatch run check`（只檢查）

### 常見問題與解法

- **Black 與 Ruff 衝突** → 忽略 Ruff 的 E203
- **導入初期改動巨大** → 建議分兩步：先全專案格式化，再啟用 pre-commit
- **既有 isort 配置** → 先跑一次 `isort .`，再逐步切換到 Ruff 的 I 規則

## 重要參考

- [Black — The uncompromising Python code formatter](https://github.com/psf/black)
- [Ruff — An extremely fast Python linter, written in Rust](https://github.com/astral-sh/ruff)
- [pre-commit — A framework for managing and maintaining multi-language pre-commit hooks](https://pre-commit.com/)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 9，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [Day 8 - 一鍵化開發工作流：Hatch scripts × Nox](../sources/ithelp-ironman-day08.md)
- [Day 10 - 型別與資料契約：mypy / pyright 與 Pydantic v2](../sources/ithelp-ironman-day10.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
