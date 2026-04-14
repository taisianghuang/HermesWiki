---
title: "Day 5 - Hatch 基本操作：建立與管理虛擬環境"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, Hatch, 虛擬環境, 環境管理, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-05.md]
related: [ithelp-ironman-day02, ithelp-ironman-day04, ithelp-ironman-day06, python-engineering]
summary: 介紹 Hatch 的環境管理功能，包括建立環境、啟動 shell、多環境管理、移除環境等操作，以及如何透過 pyproject.toml 統一管理環境設定。
---

# Day 5 - Hatch 基本操作：建立與管理虛擬環境

## 摘要

Hatch 是現代 Python 專案管理工具，本篇介紹其核心功能：建立環境、啟動 shell、多環境管理、編譯後執行，並說明如何透過 pyproject.toml 統一管理環境設定，解決傳統 venv 的繁瑣流程。

## 關鍵論點

### 為什麼選擇 Hatch 而非傳統 venv？

傳統 venv 流程：
1. 手動建立 `venv/`
2. 執行 `source venv/bin/activate`
3. 手動安裝套件
4. 自己維護 requirements.txt

Hatch 將這些步驟整合進 pyproject.toml，統一管理專案結構與環境設定。

### Hatch 核心指令

```bash
# 建立環境
hatch env create

# 進入環境 shell
hatch shell

# 編譯後執行
hatch build
hatch run start

# 多環境管理
hatch env create dev
hatch env create test
hatch env show

# 進入指定環境
hatch shell dev
hatch shell test

# 移除環境
hatch env remove dev
```

### pyproject.toml 中的多環境設定

```toml
[tool.hatch.envs.dev]
dependencies = ["black", "ruff"]

[tool.hatch.envs.test]
dependencies = ["pytest"]
```

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 5，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-day04](../sources/ithelp-ironman-day04.md) — Day 4：專案目錄結構
- [ithelp-ironman-day06](../sources/ithelp-ironman-day06.md) — Day 6：Hatch 管理套件依賴
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
