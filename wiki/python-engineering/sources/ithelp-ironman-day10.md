---
title: "Day 10 - 型別與資料契約：mypy / pyright 與 Pydantic v2"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 型別檢查, mypy, pyright, Pydantic, 資料驗證, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-10.md]
related: [ithelp-ironman-day09, ithelp-ironman-day11, ithelp-ironman-series]
summary: 介紹如何使用 mypy/pyright 在靜態分析階段攔截型別錯誤，以及如何使用 Pydantic v2 實作資料邊界驗證（CLI、環境變數、HTTP），並與 Hatch scripts、Nox、pre-commit 整合。
---

# Day 10 - 型別與資料契約：mypy / pyright 與 Pydantic v2

## 摘要

本篇介紹如何透過 mypy / pyright 在靜態分析階段攔截型別錯誤，以及如何使用 Pydantic v2 把輸入/輸出資料做成可驗證、可序列化、有結構化錯誤回報的契約。文章並展示如何與 Hatch scripts、Nox、pre-commit 整合，確保型別檢查與資料契約在本機、提交前、CI 都一致運作。

## 關鍵論點

### 工具選擇建議

| 工具 | 特性 | 適用情境 |
|------|------|----------|
| **pyright / basedpyright** | 極速、IDE 體驗佳 | 日常開發的「即時回饋」 |
| **mypy** | 規則久經社群驗證 | CI 做第二層更嚴格的把關 |

可二擇一；或本機走 pyright、CI 再跑 mypy。

### pyproject.toml 型別檢查設定

```toml
[tool.pyright]
pythonVersion = "3.11"
typeCheckingMode = "strict"
reportMissingTypeStubs = "warning"

[tool.mypy]
python_version = "3.11"
strict = true
warn_unused_ignores = true
warn_redundant_casts = true
no_implicit_optional = true
disallow_any_generics = true
plugins = ["pydantic.mypy"]
```

口訣：**先嚴格再標註**。先把規則拉高，用型別註記與局部忽略收斂歷史區塊。

### Pydantic v2：把「邊界資料」變成契約

#### 欄位驗證 + 結構化錯誤

```python
from typing import Annotated, Literal
from pydantic import BaseModel, Field, field_validator, ValidationError

Username = Annotated[str, Field(min_length=3, max_length=20)]
Role = Literal["admin", "staff", "guest"]

class User(BaseModel):
    id: int
    username: Username
    email: Annotated[str, Field(pattern=r"^[^@]+@[^@]+\.[^@]+$")]
    role: Role = "guest"

    @field_validator("username")
    @classmethod
    def no_spaces(cls, v: str) -> str:
        if " " in v:
            raise ValueError("username must not contain spaces")
        return v
```

#### 跨欄位規則（模型級）

```python
from pydantic import BaseModel, model_validator

class PasswordPair(BaseModel):
    password1: str
    password2: str

    @model_validator(mode="after")
    def passwords_match(self):
        if self.password1 != self.password2:
            raise ValueError("passwords do not match")
        return self
```

#### 任意型別一次性驗證：TypeAdapter

```python
from typing import List
from pydantic import TypeAdapter

ta = TypeAdapter(List[int])
assert ta.validate_python(["1","2","3"]) == [1,2,3]
```

### 邊界三例

| 邊界 | 工具 | 範例 |
|------|------|------|
| **CLI** | argparse + TypeAdapter | `TypeAdapter(Args).validate_python(vars(ns))` |
| **環境變數** | pydantic-settings | `class AppSettings(BaseSettings): model_config = SettingsConfigDict(env_prefix="APP_")` |
| **HTTP** | FastAPI + Pydantic | `def create_user(payload: CreateUser):` |

### Hatch + Nox + pre-commit 整合

```toml
[tool.hatch.envs.default.scripts]
typecheck = ["pyright .", "mypy src/"]
contracts = "pytest -q tests/contracts"
```

```python
# noxfile.py
@nox.session(name="typecheck", venv_backend="none")
def typecheck(session):
    session.run("pyright", ".")
    session.run("mypy", "src/")

@nox.session(name="contracts", venv_backend="none")
def contracts(session):
    session.run("pytest", "-q", "tests/contracts")
```

### 常見錯誤與收斂策略

- **嚴格模式一開錯誤爆量** → 列出「技術債清單」，先針對活躍目錄補註記，其餘以 `# type: ignore[code]` 擋住，逐步還債
- **Any 擴散** → 啟用 `disallow_any_generics`、觀察 `reportAny*` 系列診斷
- **所有內部資料都做成 Pydantic 模型** → 避免。Pydantic 專注在邊界（I/O、HTTP、Config），內部資料結構用標準 typing 即可
- **測試缺席** → 為關鍵 Model 準備至少一個「有效/無效樣本」的測試檔（例如 `tests/contracts/`）

## 重要參考

- [mypy — Static type checking for Python](https://mypy-lang.org/)
- [Pyright — Static type checker for Python](https://github.com/microsoft/pyright)
- [Pydantic v2](https://docs.pydantic.dev/)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 10，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [Day 9 - 風格統一：Black + Ruff + isort + pre-commit](../sources/ithelp-ironman-day09.md)
- [Day 11 - 測試驅動開發（TDD）](../sources/ithelp-ironman-day11.md)（待創建）
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
