---
title: "Day 14 - 失敗即常態：例外分層、重試與降級（tenacity）"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 例外處理, 重試, 降級, tenacity, 容錯, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-14.md]
related: [ithelp-ironman-series]
summary: 工程化 Python 專案必須把「失敗」當成一等公民。本篇介紹例外分層設計（可恢復 vs 不可恢復 vs 業務錯誤）、tenacity 優雅重試框架（指數回退、jitter）、以及降級策略（快取、預設值、局部停用）。
---

# Day 14 - 失敗即常態：例外分層、重試與降級（tenacity）

## 摘要

Day 14 建立錯誤處理的完整策略：例外分層 → 重試 → 降級。解決「一刀切錯誤處理、不可觀測、系統直接崩潰」的問題。介紹 tenacity 框架的優雅重試機制，以及與 structlog 日誌、測試的整合實踐。

## 關鍵論點

### 為什麼要設計「失敗處理」？

沒設計的錯誤處理：
```python
try:
    call_api()
except Exception:
    print("Something went wrong")
```

問題：
- 一刀切：所有錯誤被同等對待
- 不可觀測：只會 print()，無法追蹤
- 不可恢復：沒有 retry 或替代方案

### 例外分層：區分錯誤種類

| 類型 | 說明 | 範例 |
|------|------|------|
| **可恢復錯誤（Retryable）** | 暫時性失敗，可重試 | HTTP 503、DB 連線中斷、網路抖動 |
| **不可恢復錯誤（Fatal）** | 程式碼 bug 或無法解析配置 | KeyError、TypeError、資料遷移失敗 |
| **業務錯誤（Business Logic）** | 業務規則判斷 | 餘額不足、權限驗證失敗 |

自訂例外階層：
```python
class ExternalServiceError(Exception): ...
class TemporaryNetworkError(ExternalServiceError): ...
class FatalSystemError(Exception): ...
```

### tenacity 優雅重試

基本用法：
```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
def fetch_data():
    print("calling API ...")
    raise TimeoutError("service not responding")
```

- 最多重試 3 次
- 指數回退：2 秒、4 秒、8 秒
- 不用手寫 try/except + sleep
- 輕鬆整合 logging

與 structlog 整合：
```python
from tenacity import retry, stop_after_attempt, before_sleep_log

@retry(stop=stop_after_attempt(3), before_sleep=before_sleep_log(logger, "warning"))
def fetch_user():
    logger.info("fetching user")
    raise TimeoutError("service timeout")
```

### 降級策略：系統不能完全掛掉

常見降級做法：
- **回傳快取結果**：天氣 API 掛掉，回傳最後一次快取值
- **回傳預設值**：比崩潰更好，回傳「系統忙碌中」
- **局部停用功能**：非關鍵模組暫停，核心功能繼續

```python
from tenacity import retry, stop_after_attempt, RetryError

@retry(stop=stop_after_attempt(3))
def call_payment_api():
    raise TimeoutError("payment service down")

def safe_call_payment():
    try:
        return call_payment_api()
    except RetryError:
        return {"status": "fallback", "message": "Payment service unavailable"}
```

### 降級測試

```python
def test_payment_fallback():
    result = safe_call_payment()
    assert result["status"] == "fallback"
```

確保 fallback 策略不被遺漏。

### 最佳實務

- 錯誤不是例外，而是常態
- 例外分層：區分「需要重試」還是「立即告警」
- tenacity：複雜 retry/backoff/jitter 策略封裝
- 降級：非核心失敗不拖垮整個系統
- 整合日誌與測試，形成完整閉環

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列 Day 14，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Day 13 - 結構化日誌](../sources/ithelp-ironman-day13.md) — structlog 與日誌整合
- [Day 11 - 測試策略藍圖](../sources/ithelp-ironman-day11.md) — 測試基礎與覆蓋率
- [Day 15 - 序列化與設定格式](../sources/ithelp-ironman-day15.md) — orjson、YAML、TOML
