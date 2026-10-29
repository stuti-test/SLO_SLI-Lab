# Creating the alerts in the Grafana UI

Alerting > Alert rules > New alert rule. Repeat for fast and slow burn.

| Setting | Fast burn | Slow burn |
|---|---|---|
| Name | Checkout PlaceOrder - fast burn | Checkout PlaceOrder - slow burn |
| Query A | Code mode, Instant, expression from queries/promql.md | same |
| Expressions | Reduce (A, Last) then Threshold (B, IS ABOVE 0) | same |
| Folder | SLOs | SLOs |
| Evaluation | every 1m | every 1m |
| Pending period | 2m | 5m (15m in production) |
| If no data | Normal | Normal |
| Labels | severity=page, service=checkout | severity=ticket, service=checkout |

## How it fires
Every minute Grafana runs the query. It returns a value only when both windows exceed
the threshold. A value makes the rule Pending; staying true for the whole pending
period makes it Firing. If the condition drops during Pending, it resets to Normal.

## Where to watch
- Alerting > Alert rules (Normal / Pending / Firing)
- Rule > State history tab for timestamps
- Optional: a webhook contact point (webhook.site) routed by severity label
