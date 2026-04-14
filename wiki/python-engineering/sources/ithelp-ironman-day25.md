---
title: Day 25 - CI/CD 範本：GitHub Actions（lint → test → build → publish）
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, engineering, cicd, github-actions, hatch, testing, publishing]
sources: [../../raw/ithelp-ironman-30day/day-25.md]
related: [python-engineering, ithelp-ironman-series]
summary: GitHub Actions CI/CD 範本：uv 鎖檔還原依賴、多版本矩陣測試、覆蓋率門檻、TestPyPI/ PyPI 雙軌發佈
---

# Day 25 - CI/CD 範本：GitHub Actions（lint → test → build → publish）

## Summary

本文提供完整的 GitHub Actions CI/CD 範本，涵蓋 ci.yml（lint → typecheck → test → build）和 release.yml（build → publish）。使用 uv 鎖檔確保重現性、多版本矩陣測試、覆蓋率門檻、TestPyPI/ PyPI 雙軌發佈策略。

## Key Findings

### 為何用 uv 而不是 pip
- `uv.lock` 鎖定完整依賴樹
- `uv sync --locked` 確保版本 100% 一致
- 快取加速 CI 安裝

### CI Workflow（ci.yml）

觸發條件：push 到 main / pull_request

流程：
1. `actions/checkout@v4` 取出程式碼
2. `astral-sh/setup-uv@v3` 安裝 uv，啟用 cache
3. `uv sync --locked` 依鎖檔還原依賴
4. Lint：`ruff check .`、`black --check .`
5. Type check：`pyright` / `mypy src/`
6. Test：`pytest -q --cov=your_package --cov-report=xml:coverage.xml --cov-fail-under=80`
7. Build：`uv run hatch build`

矩陣策略：Python 3.10、3.11、3.12 三版本並行

### Release Workflow（release.yml）

觸發條件：push tags (`v*.*.*`)

根據 tag 判斷預發佈或正式版：
- 含 `a`、`b`、`rc` → TestPyPI
- 穩定版 → PyPI

```yaml
- name: Publish to TestPyPI (pre-release)
  if: contains(github.ref_name, 'a') || contains(github.ref_name, 'b') || contains(github.ref_name, 'rc')
  env: TEST_PYPI_TOKEN: ${{ secrets.TEST_PYPI_TOKEN }}
  run: uv run hatch publish --repo testpypi --user __token__ --auth "$TEST_PYPI_TOKEN"

- name: Publish to PyPI (stable)
  if: !(contains(github.ref_name, 'a') || contains(github.ref_name, 'b') || contains(github.ref_name, 'rc'))
  env: PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}
  run: uv run hatch publish --user __token__ --auth "$PYPI_TOKEN"
```

### 與專案既有工法對齊
- 一鍵化腳本：Lint/Typecheck/Test 收斂到 Hatch scripts，CI 直接呼叫同一入口
- 型別與資料契約：Pydantic v2 契約測試區
- 覆蓋率門檻：用 `-cov-fail-under` 鎖住最低標
- Secrets 與設定：CI 注入環境變數，不把 .env committed

### 常見踩雷速查

| 症狀 | 解法 |
|------|------|
| CI 安裝依賴忽快忽慢 | setup-uv@v3 打開 enable-cache: true，使用 uv sync --locked |
| 本地過、CI 不過 | 把 Lint/Typecheck/Test 收斂到 Hatch scripts |
| 釋出上 PyPI 失敗 | 檢查 tag 規則與條件區塊，token 放 Secrets |
| 覆蓋率門檻不一致 | 固定在 pyproject.toml 的 pytest options |

### 延伸：容器化與 Dev Container
- 多階段 Dockerfile + uv.lock → `uv sync --frozen` 可重現
- Dev Container 配合 `postCreateCommand` 跑 Hatch 一鍵化腳本

## See also

- [python-engineering](../concepts/python-engineering.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
