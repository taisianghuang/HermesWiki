---
title: "Day 26 - 安全與授權：pip-audit、Safety、授權掃描"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 安全, 授權, pip-audit, Safety, SBOM, 容器, CI/CD]
sources: [../../raw/ithelp-ironman-30day/day-26.md]
related: [ithelp-ironman-series, python-engineering]
summary: 介紹 Python 安全掃描工具 pip-audit 與 Safety 的比較、SBOM 生成（CycloneDX）、授權白名單治理，以及如何在 CI/容器建置中整合安全檢查。
---

# Day 26 - 安全與授權：pip-audit、Safety、授權掃描

## 摘要

本篇介紹 Python 專案的安全與授權治理流程，涵蓋弱點掃描（pip-audit、Safety）、SBOM 生成（CycloneDX）、第三方授權清單與白名單守門。這些工具透過 Hatch scripts 與 Nox 一鍵化，並整合進 pre-commit 與 CI workflow。

## 關鍵論點

### 工具比較

| 面向 | 工具 | 資料庫來源 | 特色 | 報告格式 | 是否可修復 |
|------|------|-----------|------|----------|------------|
| 弱點掃描 | pip-audit | PyPI/OSV | 官方系，預設準確、更新快 | text/json/cyclonedx | --fix 可嘗試升級 |
| 弱點掃描 | Safety | 專有與社群 DB | 覆蓋廣、規模化用得多 | text/json/sarif | 需付費帳號才吃全庫 |
| 授權清單 | pip-licenses | wheel metadata | 產 Markdown/CSV 表格 | md/csv/json | 不適用 |
| SBOM | cyclonedx-bom / cyclonedx-py | 解析環境/lock | 產 CycloneDX（業界通吃） | json/xml | 不適用 |

建議：兩個弱點掃描都跑，資料庫不同、誤報與覆蓋互補。

### Hatch scripts 整合

```toml
[project.optional-dependencies]
sec = [
 "pip-audit>=2.7",
 "safety>=3.2",
 "pip-licenses>=4.5",
 "cyclonedx-bom>=4.0",
]

[tool.hatch.envs.sec.scripts]
audit = "pip-audit -l --desc --progress-spinner off"
safety = "safety check --full-report --continue-on-error 0"
licenses = "pip-licenses --format markdown --with-authors --with-urls --output-file LICENSES-3RD-PARTY.md"
sbom = "cyclonedx-bom -o sbom.cdx.json -F json"
check-all = [
 "pip-audit -l --desc --progress-spinner off",
 "safety check --full-report --continue-on-error 0",
 "pip-licenses --format markdown --with-authors --with-urls --output-file LICENSES-3RD-PARTY.md",
 "cyclonedx-bom -o sbom.cdx.json -F json"
]
```

### 授權白名單與守門

```python
import json, sys, subprocess

ALLOWED = {"MIT","BSD","Apache-2.0","MPL-2.0"}
raw = subprocess.check_output(["pip-licenses", "--format", "json"]).decode()
pkgs = json.loads(raw)
bad = [(p["Name"], p["License"]) for p in pkgs if p["License"] not in ALLOWED]
if bad:
 print("Forbidden licenses detected:")
 for name, lic in bad:
 print(f"- {name}: {lic}")
 sys.exit(1)
print("Licenses OK")
```

### CI workflow 整合

```yaml
security:
 runs-on: ubuntu-latest
 steps:
 - uses: actions/checkout@v4
 - uses: astral-sh/setup-uv@v3
 - name: Sync deps from lock (no dev)
   run: uv sync --locked --no-dev
 - name: pip-audit
   run: uv run pip-audit -l --desc --progress-spinner off
 - name: Safety
   env:
     SAFETY_API_KEY: ${{ secrets.SAFETY_API_KEY }}
   run: uv run safety check --full-report --continue-on-error 0
 - name: Generate licenses & SBOM
   run: |
     uv run pip-licenses --format markdown --with-authors --with-urls --output-file LICENSES-3RD-PARTY.md
     uv run cyclonedx-bom -o sbom.cdx.json -F json
 - name: Upload artifacts
   uses: actions/upload-artifact@v4
   with:
     name: security-reports
     path: LICENSES-3RD-PARTY.md sbom.cdx.json
```

## 重要參考

- [pip-audit](https://github.com/trailofbits/pip-audit)
- [Safety](https://safety.pyup.io/)
- [CycloneDX](https://cyclonedx.org/)
- [pip-licenses](https://github.com/raimon49/pip-licenses)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 26，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
