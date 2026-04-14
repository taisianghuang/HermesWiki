---
title: "Day 11 - 測試策略藍圖：pytest 目錄結構、fixtures 與 coverage"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 測試, pytest, fixtures, coverage, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-11.md]
related: [ithelp-ironman-series]
summary: 建立完整的測試藍圖，涵蓋 pytest 目錄結構設計、conftest.py 與共用 fixtures 策略、單元/整合/端到端測試分层标记，以及 coverage 覆蓋率與 Hatch/Nox 串接的一鍵化工作流。
---

# Day 11 - 測試策略藍圖：pytest 目錄結構、fixtures 與 coverage

## 摘要

Day 11 建立完整的測試策略藍圖，解決「測試檔案四散無章、重複邏輯、缺乏覆蓋率檢查」三大痛點。涵蓋 pytest 目錄結構設計、fixtures 共用策略、三層測試切分（unit/integration/e2e）、coverage 門檻設定，以及 Hatch × Nox 一鍵化工作流。

## 關鍵論點

### 測試目錄結構

```
my_project/
├─ src/my_project/
│ ├─ __init__.py
│ ├─ core.py
│ └─ adapters/
│   └─ web.py
└─ tests/
  ├─ conftest.py          # 共用 fixtures
  ├─ unit/
  │  └─ test_core.py     # 單元測試
  ├─ integration/
  │  └─ test_api.py      # 整合測試（API/DB）
  └─ e2e/
     └─ test_cli.py      # 端到端測試
```

- **tests/unit/**：快速驗證函式與類別邏輯
- **tests/integration/**：測試 API、DB、檔案等組件交互
- **tests/e2e/**：模擬真實使用場景，確保主要路徑走通

### conftest.py 與共用 Fixtures

pytest 的 fixtures 能統一產生測試資源，減少重複：

```python
class CliResult(NamedTuple):
    code: int
    out: str
    err: str

@pytest.fixture
def run_cli(tmp_path: Path) -> callable:
    def _run(args: list[str] | None = None) -> CliResult:
        cmd = [sys.executable, "-m", "demo_app.main"]
        if args:
            cmd.extend(args)
        proc = subprocess.run(cmd, capture_output=True, text=True, cwd=str(tmp_path))
        return CliResult(proc.returncode, proc.stdout, proc.stderr)
    return _run
```

### 實戰測試範例

| 類型 | 範例 | 工具 |
|------|------|------|
| 單元測試 | `test_add_basic()` | pytest |
| API 整合測試 | FastAPI `client.post("/avg", ...)` | `pytest-httpx` |
| SQLite 整合測試 | `temp_db.cursor()` 建立資料表 | `tmp_path` fixture |
| 外部服務 mock | `mocker.patch.object(httpx, "get")` | `pytest-mock` |
| 日誌驗證 | `caplog.set_level(logging.INFO)` | `caplog` fixture |

### Coverage 設定

在 `pyproject.toml` 加入：

```toml
[tool.pytest.ini_options]
addopts = """
  -ra
  --strict-markers
  --disable-warnings
  --cov=your_package
  --cov-report=term-missing
  --cov-report=xml:coverage.xml
  --cov-fail-under=80
"""
```

- 顯示哪些行沒測到
- CI 強制覆蓋率低於 80% 時失敗

### Hatch × Nox 串接

```toml
[tool.hatch.envs.test]
dependencies = ["pytest", "pytest-cov", "pytest-mock", "httpx", "fastapi"]
[tool.hatch.envs.test.scripts]
unit = "pytest -m 'not slow and not e2e'"
all = "pytest"
ci = "pytest --junitxml=report.xml"
```

```python
@nox.session(python=["3.10", "3.11", "3.12"])
def tests(session):
    session.install(".[dev]")
    session.run("pytest", "--cov=your_package", "--cov-fail-under=80")
```

結合 pre-commit hook 可在 push 前檢查測試覆蓋率。

### 最佳實務

- 單元測試多、整合測試少但真實、端到端測試更少但關鍵
- fixtures 提供共用資源，避免複製貼上
- 覆蓋率報告納入 CI，品質有數字依據
- Hatch/Nox 一鍵化測試，避免「本地 OK、CI 爆炸」

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列 Day 11，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
- [Day 09 - 程式風格與規範](../sources/ithelp-ironman-day09.md) — Ruff 程式風格一致性
- [Day 10 - 型別與資料契約](../sources/ithelp-ironman-day10.md) — Pydantic 型別安全
