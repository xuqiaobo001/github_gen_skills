# Skill 8: `otel-observability` — OpenTelemetry 全链路可观测性

**适用场景**: 需要分布式追踪、结构化日志、性能指标的后端系统。

**Dify 源码引用**:

- `api/extensions/ext_otel.py` — 统一初始化 OTEL Instrumentor
- `api/core/logging/context.py` — 请求级别上下文注入 Trace/Span ID
- `api/extensions/ext_sentry.py` — Sentry 错误追踪集成

**Instrumentor 初始化骨架**:

```python
# ext_otel.py
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor
from opentelemetry.instrumentation.celery import CeleryInstrumentor
from opentelemetry.instrumentation.redis import RedisInstrumentor

def init_app(app: Flask):
    provider = TracerProvider(resource=Resource.create({
        "service.name": "dify-api",
        "deployment.environment": app.config["DEPLOY_ENV"],
    }))
    # 选择 Exporter：OTLP / Jaeger / Zipkin
    exporter = OTLPSpanExporter(endpoint=app.config["OTEL_EXPORTER_ENDPOINT"])
    provider.add_span_processor(BatchSpanProcessor(exporter))
    trace.set_tracer_provider(provider)

    # 自动埋点
    FlaskInstrumentor().instrument_app(app)
    SQLAlchemyInstrumentor().instrument(engine=db.engine)
    CeleryInstrumentor().instrument()
    RedisInstrumentor().instrument()
```

**日志上下文注入**:

```python
# core/logging/context.py — 每条日志自动携带 Trace ID
@app.after_request
def inject_trace_headers(response):
    span = trace.get_current_span()
    ctx = span.get_span_context()
    response.headers["X-Trace-Id"] = format(ctx.trace_id, "032x")
    response.headers["X-Span-Id"] = format(ctx.span_id, "016x")
    return response
```

**关键规则**:

- 所有 Instrumentor 在 `init_app` 中统一初始化，不分散到各模块
- 响应头暴露 `X-Trace-Id`，便于前端排查问题时关联后端日志
- Sentry 作为错误追踪补充，与 OTEL 并行使用
- 生产环境使用 `BatchSpanProcessor`，开发环境可用 `SimpleSpanProcessor`

**反模式**:

- ❌ 手动在每个函数中创建 Span（应依赖自动埋点）
- ❌ 使用 `SimpleSpanProcessor` 上生产（同步导出阻塞请求）
- ❌ 日志中不包含 Trace ID，无法关联分布式调用链
