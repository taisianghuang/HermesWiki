---
title: "Day 28 - 監控與追蹤：OpenTelemetry 指標/追蹤"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 監控, OpenTelemetry, OTel, 追蹤, Metrics, Prometheus, Grafana, 可觀測性]
sources: [../../raw/ithelp-ironman-30day/day-28.md]
related: [ithelp-ironman-series, python-engineering]
summary: 介紹 OpenTelemetry（OTel）在 Python 的應用，包括 Traces、Metrics、Logs 的整合，OTel Collector 部署，以及如何將結構化日誌與追蹤關聯，實現雲原生可觀測性。
---

# Day 28 - 監控與追蹤：OpenTelemetry 指標/追蹤

## 摘要

本篇介紹如何用 OpenTelemetry（OTel）建立雲原生可觀測性架構，涵蓋 Traces（追蹤）、Metrics（指標）、Logs（日誌）三大支柱的 Python SDK 整合、OTel Collector 部署，以及 Log 與 Trace 的關聯。搭配 Prometheus、Tempo/Grafana 實現從「感覺」到「證據」的轉變。

## 關鍵論點

### 為什麼要 OTel

- 雲原生場景同時需要：指標（告警/SLO）、追蹤（端對端延遲分析）、日誌（細節脈絡）
- OTel 提供統一的 API/SDK、OTLP 傳輸協定、Collector 中繼架構
- 減少廠商綁定：一次整合，可切換後端（Prometheus、Tempo/Jaeger、Datadog）

### 總架構

```
App (SDK) --OTLP--> OTel Collector --export--> Tempo/Jaeger + Prometheus/OTLP + Logs Sink
```

### Python OTel SDK 整合

```python
# 安裝
# opentelemetry-sdk>=1.27, opentelemetry-exporter-otlp>=1.27,
# opentelemetry-instrumentation-fastapi>=0.49b0,
# opentelemetry-instrumentation-httpx>=0.49b0,
# opentelemetry-instrumentation-logging>=0.49b0

from opentelemetry import trace, metrics
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter
from opentelemetry.sdk.metrics.export import PeriodicExportingMetricReader

def setup_otel() -> None:
    res = Resource.create({
        "service.name": os.getenv("OTEL_SERVICE_NAME", "awesome-api"),
        "service.version": os.getenv("SERVICE_VERSION", "0.0.0"),
        "service.namespace": os.getenv("OTEL_NAMESPACE", "default"),
        "deployment.environment": os.getenv("ENV", "dev"),
    })
    # Traces
    tp = TracerProvider(resource=res)
    tp.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
    trace.set_tracer_provider(tp)
    # Metrics
    reader = PeriodicExportingMetricReader(OTLPMetricExporter())
    mp = MeterProvider(resource=res, metric_readers=[reader])
    metrics.set_meter_provider(mp)
```

### FastAPI 自動化框架追蹤

```python
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.httpx import HTTPXClientInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor

def create_app() -> FastAPI:
    setup_otel()
    app = FastAPI(title="Awesome API", version="1.2.3")
    FastAPIInstrumentor.instrument_app(app)
    HTTPXClientInstrumentor().instrument()
    RequestsInstrumentor().instrument()
    return app
```

### 自訂 Span

```python
from opentelemetry import trace
tracer = trace.get_tracer(__name__)

def compute_quote(user_id: str, items: list[dict]) -> int:
    with tracer.start_as_current_span("compute_quote") as span:
        span.set_attribute("user.id", user_id)
        span.set_attribute("items.count", len(items))
        # ... heavy logic ...
        price = 123
        span.add_event("quote_computed", {"price": price})
        return price
```

### Metrics：RED/USE 框架

```python
from opentelemetry import metrics
meter = metrics.get_meter(__name__)

orders_success = meter.create_counter("orders_success_total")
orders_failed = meter.create_counter("orders_failed_total")
latency = meter.create_histogram("orders_latency_ms", unit="ms")

def place_order(user_id: str, payload: dict) -> str:
    import time
    t0 = time.perf_counter()
    try:
        # ... business ...
        orders_success.add(1, {"route": "POST /v1/orders"})
        return "ok"
    except Exception:
        orders_failed.add(1, {"route": "POST /v1/orders"})
        raise
    finally:
        latency.record((time.perf_counter() - t0) * 1000, {"route": "POST /v1/orders"})
```

### Structlog 注入 trace_id

```python
import structlog
from opentelemetry.trace import get_current_span

def _otel_ids(_, __, event_dict):
    span = get_current_span()
    ctx = span.get_span_context()
    if ctx and ctx.is_valid:
        event_dict["trace_id"] = f"{ctx.trace_id:032x}"
        event_dict["span_id"] = f"{ctx.span_id:016x}"
    return event_dict

def setup_logging():
    structlog.configure(
        processors=[
            structlog.processors.add_log_level,
            _otel_ids,
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.processors.JSONRenderer(),
        ]
    )
```

### OTel Collector 設定

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

exporters:
  otlp:
    endpoint: tempo:4317
    tls:
      insecure: true
  prometheus:
    endpoint: "0.0.0.0:9464"

processors:
  batch: {}

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp]
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
```

## 重要參考

- [OpenTelemetry Python SDK](https://opentelemetry.io/docs/instrumentation/python/)
- [OTel Collector](https://opentelemetry.io/docs/collector/)
- [Prometheus](https://prometheus.io/)
- [Grafana Tempo](https://grafana.com/docs/tempo/)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 28，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
