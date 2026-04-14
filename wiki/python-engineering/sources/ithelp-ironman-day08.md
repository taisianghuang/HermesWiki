---
title: "Day 8 - 一鍵化開發工作流：Hatch scripts × Nox"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 工作流自動化, Hatch, Nox, CI/CD, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-08.md]
related: [ithelp-ironman-day07, ithelp-ironman-day09, ithelp-ironman-series]
summary: 介紹如何使用 Hatch scripts 集中定義開發指令（test/lint/format/typecheck），以及如何透過 Nox 實現跨 Python 版本測試，並將兩者整合為 `hatch run ci` 單一入口。
---

# Day 8 - 一鍵化開發工作流：Hatch scripts × Nox

## 摘要

本篇介紹如何透過 Hatch scripts 與 Nox 實現開發工作流一鍵化。Hatch scripts 類似 Node.js 的 npm run，可集中定義測試、lint、格式化等指令；Nox 則支援跨 Python 版本測試與任務自動化。兩者整合後只需 `hatch run ci` 即可完成所有檢查，確保本地與 CI 環境完全一致。

## 關鍵論點

### 為什麼需要一鍵化工作流？

| 傳統問題 | 一鍵化好處 |
|----------|------------|
| 每人執行方式不同，有人漏跑 lint | **一致性**：所有人用同樣的入口 |
| 格式化忘記加 --check | **可維護性**：修改工作流只需改一個地方 |
| CI 跑的流程跟本地端不一致 | **易用性**：新人只要會跑一個指令，就能正確參與開發 |

### Hatch scripts：集中入口

```toml
[tool.hatch.envs.default.scripts]
test = "pytest -q"
lint = "ruff check ."
format = "black ."
typecheck = "mypy src/"
check-all = ["ruff check .", "black --check .", "mypy src/", "pytest -q"]
```

使用方式：`hatch run test`、`hatch run check-all`

### Nox：跨版本與自動化測試

```python
import nox

@nox.session(python=["3.10", "3.11", "3.12"])
def tests(session):
    session.install(".[dev]")
    session.run("pytest", "-q")

@nox.session
def lint(session):
    session.install("ruff", "black")
    session.run("ruff", "check", ".")
    session.run("black", "--check", ".")
```

執行：`nox`（跑所有 session）、`nox -s lint`（只跑 lint）、`nox -s tests-3.11`（只跑 3.11 測試）

### Hatch × Nox 整合

```toml
[tool.hatch.envs.default.scripts]
ci = "nox"
```

使用：`hatch run ci` → 等同於本地跑 lint、格式化檢查、型別檢查、測試

### 示範專案結構

```
./
├── pyproject.toml
├── noxfile.py
├── src/demo_app/
│   ├── __init__.py
│   ├── __about__.py
│   └── main.py
└── tests/
    ├── __init__.py
    └── test_main.py
```

## 重要參考

- [Hatch scripts](https://hatch.pypa.io/latest/scripts/)
- [Nox — Command-line tool that runs tests in multiple Python environments](https://nox.thea.codes/)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 8，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [Day 7 - 可重現環境策略：constraints / lock 與快取](../sources/ithelp-ironman-day07.md)
- [Day 9 - 風格統一：Black + Ruff + isort + pre-commit](../sources/ithelp-ironman-day09.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
