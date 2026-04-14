---
title: "Day 7 - 可重現環境策略：constraints / lock 與快取"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 可重現環境, UV, Hatch, CI/CD, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-07.md]
related: [ithelp-ironman-day06, ithelp-ironman-day08, ithelp-ironman-series]
summary: 介紹 Python 可重現環境的三個進化階段：Constraints（傳統補救）、Hatch dependencies（工程化表達）、UV lock（現代最佳實務），以及 GitHub Actions 中的 UV 快取策略。
---

# Day 7 - 可重現環境策略：constraints / lock 與快取

## 摘要

本篇探討 Python 可重現環境的三個進化階段：Constraints（傳統補救措施）、Hatch dependencies（PEP 440 標準化表達）、UV lock（現代最佳實務）。文章說明如何透過 `uv.lock` 完整鎖定依賴樹，並展示 GitHub Actions 中的 UV 快取配置，確保 CI 環境與本地完全一致。

## 關鍵論點

### 可重現環境的三個進化階段

| 階段 | 控制範圍 | 易用性 | 重現性 | 適用情境 |
|------|----------|--------|--------|----------|
| **Constraints** | 部分套件（補充規則） | 需額外檔案，人工維護 | 部分一致 | 臨時修補 / 白名單 |
| **Hatch dependencies** | 主依賴 + 範圍 | pyproject.toml 直接宣告 | 視間接依賴而定 | 專案需求表達 |
| **UV lock** | 全依賴樹 | 自動生成 uv.lock | 完全一致 | 團隊工程化 / CI/CD |

### Constraints：傳統補救措施

- `requirements.txt` → 主依賴清單（要安裝哪些套件）
- `constraints.txt` → 補充規則（套件若被安裝，必須符合指定版本）
- 安裝時帶入：`pip install -c constraints.txt requests`

### UV lock：現代可重現解法

```bash
uv lock # 產生 / 更新 uv.lock
uv sync --locked # 嚴格依照 uv.lock 安裝
uv lock --upgrade # 升級所有套件
uv lock --upgrade-package requests # 升級特定套件
```

### GitHub Actions + UV 快取

```yaml
- uses: astral-sh/setup-uv@v3
  with:
    enable-cache: true
```

## 重要參考

- [UV — Fast Python package installer](https://github.com/astral-sh/uv)
- [Hatch — Modern Python project management](https://github.com/pypa/hatch)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 7，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [Day 6 - 依賴管理策略：直接安裝 vs. 鎖定版本](../sources/ithelp-ironman-day06.md)
- [Day 8 - 一鍵化開發工作流：Hatch scripts × Nox](../sources/ithelp-ironman-day08.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
