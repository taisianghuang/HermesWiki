---
title: "Day 6 - 依賴管理策略：直接安裝 vs. 鎖定版本"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 依賴管理, Hatch, PEP 440, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-06.md]
related: [ithelp-ironman-day05, ithelp-ironman-day07, ithelp-ironman-series]
summary: 探討 Python 專案的兩種依賴管理策略——直接安裝（浮動版本）與鎖定版本，以及 Hatch 的 extra-dependencies 與 optional-dependencies + features 的使用時機。
---

# Day 6 - 依賴管理策略：直接安裝 vs. 鎖定版本

## 摘要

本篇探討 Python 專案中的依賴管理策略對立：直接安裝（Floating Dependency）追求便利性，鎖定版本（Pinned Dependency）追求穩定性。並介紹如何在 pyproject.toml 中使用 PEP 440 語法精確表達依賴範圍，以及 Hatch 的 extra-dependencies 與 optional-dependencies + features 的差異與適用場景。

## 關鍵論點

### 兩種依賴策略

| 策略 | 優點 | 缺點 | 適用情境 |
|------|------|------|----------|
| **直接安裝** | 快速、方便 | 結果不一致，可能「在我電腦能跑，在你電腦卻壞掉」 | 個人實驗、一次性腳本 |
| **鎖定版本** | 團隊一致性高、可重現性好 | 需定期手動更新 | 團隊合作、長期維護專案 |

### PEP 440 版本約束語法

- `>=2.32,<3` — 區間鎖定（常見做法），允許範圍內升級
- `==2.8.*` — 完全指定，仅允許 patch 變動
- `~=1.26` — 相容版號，允許小版本更新但不跳大版本

### 專案依賴 vs. 開發依賴

- **專案依賴**：放在 `[project].dependencies`，專案運行時必須存在
- **開發/測試依賴**：定義在 Hatch 環境設定或 extras

### Hatch 依賴管理方式對比

| 寫法 | 範例 | 適合情境 |
|------|------|----------|
| `extra-dependencies` | `[tool.hatch.envs.dev].extra-dependencies = ["pytest"]` | 只想在 Hatch 中加特定套件，不需要跨工具支援 |
| `optional-dependencies + features` | `[project.optional-dependencies].dev = [...]` + `[tool.hatch.envs.dev].features = ["dev"]` | 希望依賴群組可被 pip/Poetry/Hatch 等工具重用 |

## 重要參考

- [PEP 440 — Version Identification and Dependency Specification](https://peps.python.org/pep-0440/)
- [Hatch — Modern Python project management](https://github.com/pypa/hatch)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 6，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [Day 5 - 結構開箱：Hatch 預設的專案骨架](../sources/ithelp-ironman-day05.md)
- [Day 7 - 可重現環境策略：constraints / lock 與快取](../sources/ithelp-ironman-day07.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
