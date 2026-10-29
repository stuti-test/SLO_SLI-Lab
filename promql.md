# PromQL

Assumed names (verify in Grafana Explore for your demo version):
- Metric: `traces_span_metrics_calls_total` (older versions: `calls_total`)
- Span: `oteldemo.CheckoutService/PlaceOrder`
- Latency bucket `le="1000"` must exist

## SLI 1: Availability (5m)
```promql
1 - (
  (sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder",status_code="STATUS_CODE_ERROR"}[5m])) or vector(0))
  /
  sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder"}[5m]))
)
```

## SLI 2: Latency under 1s (5m)
```promql
sum(rate(traces_span_metrics_duration_milliseconds_bucket{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder",le="1000"}[5m]))
/
sum(rate(traces_span_metrics_duration_milliseconds_count{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder"}[5m]))
```

## Error budget remaining (30d)
Only meaningful with long metric retention.
```promql
1 - (
  (sum(increase(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder",status_code="STATUS_CODE_ERROR"}[30d])) or vector(0))
  / sum(increase(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder"}[30d]))
) / 0.005
```

## Fast burn (14.4x, threshold 7.2%) - lab windows 15m / 5m
Production windows: 1h / 5m.
```promql
(
  (sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder",status_code="STATUS_CODE_ERROR"}[15m])) or vector(0))
  / sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder"}[15m]))
) > (14.4 * 0.005)
and
(
  (sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder",status_code="STATUS_CODE_ERROR"}[5m])) or vector(0))
  / sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder"}[5m]))
) > (14.4 * 0.005)
```

## Slow burn (6x, threshold 3%) - lab windows 1h / 10m
Production windows: 6h / 30m.
```promql
(
  (sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder",status_code="STATUS_CODE_ERROR"}[1h])) or vector(0))
  / sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder"}[1h]))
) > (6 * 0.005)
and
(
  (sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder",status_code="STATUS_CODE_ERROR"}[10m])) or vector(0))
  / sum(rate(traces_span_metrics_calls_total{service_name="checkout",span_name="oteldemo.CheckoutService/PlaceOrder"}[10m]))
) > (6 * 0.005)
```

The queries use `and`, so they return a value only when both windows exceed the
threshold. When healthy they return nothing, so alerts use "no data = Normal".
