---
title: "Day 3 - pyproject.toml：現代 Python 專案的核心設定檔"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, pyproject.toml, PEP 518, 設定檔, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-03.md]
related: [ithelp-ironman-day02, ithelp-ironman-day04, python-engineering]
summary: 介紹 pyproject.toml 如何統一 Python 專案的建置系統與工具設定，取代散落的 setup.py、requirements.txt、setup.cfg 等檔案，並說明其與 requirements.txt 的共存情境。
---

# Day 3 - pyproject.toml：現代 Python 專案的核心設定檔

## 摘要

pyproject.toml（PEP 518）是現代 Python 專案的核心設定檔，統一了建置系統與工具設定。本篇說明如何用它取代散落的 setup.py、requirements.txt、setup.cfg，並介紹與 requirements.txt 的共存情境。

## 關鍵論點

### 歷史演進：從混亂到整合

- **舊時代**：setup.py + requirements.txt + setup.cfg + tox.ini + .flake8 散落各處
- **PEP 518 (2016)**：提出 pyproject.toml，統一建置方式與工具設定
- **現在**：pyproject.toml 成為現代 Python 專案的「心臟」

### pyproject.toml 最小範例

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"
```

### 工具設定整合

```toml
[tool.black]
line-length = 88
target-version = ["py310"]

[tool.isort]
profile = "black"

[tool.mypy]
ignore_missing_imports = true
strict = true
```

### pyproject.toml vs requirements.txt

| 情境 | 需要的檔案 |
|------|------------|
| 發佈套件 | pyproject.toml（必須） |
| 部署服務 / CI/CD | requirements.txt（常見） |

兩者可共存：pyproject.toml 負責建置與發佈，requirements.txt 負責鎖定版本供部署。

### Hatch 的 pyproject.toml

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-awesome-project"
version = "0.1.0"
dependencies = ["requests>=2.28", "pydantic>=2.0"]

[tool.hatch.envs.default]
dependencies = ["pytest", "black"]

[tool.hatch.envs.dev]
dependencies = ["pytest", "black", "mypy"]
```

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 3，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-day02](../sources/ithelp-ironman-day02.md) — Day 2：環境管理工具比較
- [ithelp-ironman-day04](../sources/ithelp-ironman-day04.md) — Day 4：專案目錄結構
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
