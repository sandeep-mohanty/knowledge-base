# Alert Thresholds and Alert Lifecycle - Cheat Sheet

## Alert threshold

A threshold is the value that triggers an alert.

Examples:

- `cpu > 85%`
- `error_rate > 5%`
- `latency > 500ms`

PromQL example:

```promql
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
```

This means:

- compute 5xx request rate
- divide by total request rate
- trigger when it is greater than 5%

---

## Alert rule structure

```yaml
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
  for: 10m
  labels:
    severity: warning
```

Key parts:

- `expr`: the condition to evaluate
- `>`: threshold comparison
- `for`: how long condition must remain true
- `labels`: metadata like severity/team/service

---

## Choosing thresholds

A good threshold is:

- low enough to catch problems early
- high enough to avoid noisy alerts

Use:

- service target / SLO
- historical normal ranges
- headroom above expected values

Examples:

- CPU: alert at 85%, not 99%
- error rate: alert at 5% if that is meaningful for the service
- latency: alert at p95 above design target

---

## Why `for:` matters

```yaml
for: 10m
```

This means the condition must stay true for 10 minutes before alerting.

Purpose:

- avoid alert spam from short spikes
- reduce false positives

---

## What happens when an alert fires?

Usually:

- notification is sent
- PagerDuty / Slack / email may be triggered
- incident may be created if configured
- automation may run
- system shows alert in dashboards

---

## Common outcome paths

### Notification
- Slack
- email
- Teams
- PagerDuty
- Opsgenie

### Incident creation
- Jira ticket
- ServiceNow
- PagerDuty incident
- incident timeline / runbook

### Automated action
- scale up service
- restart pod
- rollback deployment
- trigger webhook

---

## Severity example

```yaml
labels:
  severity: critical
```

This typically means:

- page on-call immediately
- notify SREs
- escalate faster

Warning severity may mean:

- Slack only
- less urgent routing

---

## Alert vs incident

Alert = a metric condition has crossed a threshold.

Incident = the problem has broader operational impact and requires coordinated response.

Not all alerts become incidents.

---

## Simple lifecycle

1. metric collected
2. expression evaluated
3. threshold crossed
4. condition is true long enough
5. alert is fired
6. notification / routing occurs
7. incident may be opened
8. team responds and resolves
9. alert clears when service recovers

---

## 1-line summary

Thresholds define when a metric is “bad enough to alert,” and when an alert fires, it usually triggers notifications, escalations, and sometimes an incident workflow.
