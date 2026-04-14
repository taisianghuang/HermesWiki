---
title: Day 22 - 發佈與版本：Hatch build wheel、SemVer、Changelog
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, engineering, packaging, hatch, semver, changelog, publishing]
sources: [../../raw/ithelp-ironman-30day/day-22.md]
related: [python-engineering, ithelp-ironman-series]
summary: Python 發佈三要素：Hatch 建置發佈、SemVer 版本管理、Keep a Changelog 維護變更紀錄
---

# Day 22 - 發佈與版本：Hatch build wheel、SemVer、Changelog

## Summary

本文解說如何用 Hatch 建置與發佈 Python 套件（wheel/sdist），以 SemVer 管理版本節奏，以及維護高品質 Changelog。涵蓋 pyproject.toml 最小骨架、版本來源策略（SSOT）、預發佈標記、建置命令、PyPI 發佈流程、Conventional Commits 搭配版本自動化。

## Key Findings

### 最小可發佈專案必備
- `pyproject.toml`（PEP 621，Hatchling 建置後端）
- `README.md`（長描述，PyPI 顯示用）
- `LICENSE`
- `CHANGELOG.md`
- `src/my_project/` + `tests/`

### 單一可信版本來源（SSOT）
兩種好選擇：
1. 直接在 `pyproject.toml` 設定 `version = "0.1.0"`，用 `hatch version` 指令管理
2. 由 `src/my_project/version.py` 提供，`[tool.hatch.version] path = "src/my_project/version.py"`

### SemVer 語義化版本
- **MAJOR**: BREAKING 變更或移除 API
- **MINOR**: 新增相容功能
- **PATCH**: 修正 bug、內部最佳化

預發佈標記：`1.2.0a1`、`1.2.0b1`、`1.2.0rc1`（CI 偵測到送往 TestPyPI）

### 建置產物
- **wheel**：已編譯可安裝檔，安裝最快，首選
- **sdist**：原始碼分佈，某些環境需要從原始碼建置時用到

命令：`hatch build`、`hatch build -t wheel`、`hatch build -t sdist`

### Hatch 發佈到 PyPI
```bash
# TestPyPI
hatch publish --repo testpypi --user __token__ --auth $TEST_PYPI_TOKEN
# PyPI 正式
hatch publish --user __token__ --auth $PYPI_TOKEN
```

### Keep a Changelog 分類
- Added（新功能）
- Changed（變更或行為調整）
- Deprecated（仍可用但不鼓勵）
- Removed（破壞性移除）
- Fixed（修 bug）
- Security（安全性修補）

### Conventional Commits 搭配
- `feat:` → MINOR
- `fix:` → PATCH
- `feat!:` 或 `refactor!:` → MAJOR（BREAKING）

### 一鍵化 Hatch scripts
```toml
[tool.hatch.envs.release]
scripts.check = ["ruff .", "mypy src/", "pytest -q"]
scripts.build = "hatch build"
scripts.publish-test = "hatch publish --repo testpypi ..."
scripts.publish = "hatch publish ..."
scripts.tag = "git tag v{project:version} && git push --tags"
```

### GitHub Actions 觸發策略
- `ci.yml`：每個 push / PR 都跑（lint、test、build）
- `release.yml`：只在打 tag 時觸發（`on: push: tags: - "v*.*.*"`）

## See also

- [python-engineering](../concepts/python-engineering.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
