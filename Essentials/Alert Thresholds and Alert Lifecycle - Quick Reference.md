# Alert Thresholds and Alert Lifecycle - Quick Reference

## 1) What is a threshold?

A threshold is the value at which a metric is considered abnormal enough to trigger an alert.

Examples:

- CPU usage > 85%
- Error rate > 5%
- p95 latency > 500ms
- Queue depth > 1000

The alert is usually written as a PromQL expression with a comparison operator:

```promql
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
```

This means:

- compute the error rate
- if it is greater than 5%
- then the condition is true

---

## 2) Thresholds are not random

Good thresholds come from a combination of:

- business impact
- service SLO / target
- observed normal behavior
- historical baselines
- acceptable noise level

A threshold should be:

- high enough to avoid noisy alerts
- low enough to catch real problems early

---

## 3) Typical alert rule structure

```yaml
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
  for: 10m
  labels:
    severity: warning
```

Important parts:

- `expr`: the PromQL condition
- `>`: threshold comparison
- `for`: how long the condition must remain true before alerting
- `labels`: metadata such as severity, team, service

---

## 4) Why `for:` matters

`for:` prevents short spikes from triggering noisy alerts.

Example:

- `expr` says error rate > 5%
- `for: 10m` means it must remain above 5% for 10 minutes before the alert is fired

This reduces false positives.

---

## 5) How to choose a threshold

### Start with service intent

Ask:

- What is the acceptable error rate?
- What latency is still acceptable?
- What CPU level is considered abnormal?
- At what point does the user feel the problem?

### Look at historical data

Check normal ranges over days or weeks:

- CPU typical range
- latency pattern
- error rate in steady state

### Add headroom

Do not set the threshold at the absolute maximum.

Example:

- CPU usually around 50–70%
- alert at 85%, not 99%
- latency target is 200ms, alert at 300ms rather than 1s

---

## 6) Example thresholds by type

### CPU

```yaml
expr: avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100 > 85
for: 10m
```

### Error rate

```yaml
expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
for: 10m
```

### Latency

```yaml
expr: histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket[5m]))) > 0.5
for: 10m
```

---

## 7) What happens when an alert fires?

When the alert expression remains true long enough, the alert becomes active.

Then the system typically does one or more of the following:

- sends a notification
- pages on-call engineers
- creates an incident ticket
- triggers automation or runbooks
- shows alert in monitoring dashboards

---

## 8) Alert routing and severity

Alert severity often controls who gets notified and how.

Examples:

- `critical` → page SRE / on-call immediately
- `warning` → Slack or email
- `info` → ticket or internal dashboard

This is usually configured in Alertmanager or an incident platform.

---

## 9) Alert lifecycle

The usual lifecycle is:

1. Metric is collected
2. PromQL expression is evaluated
3. Condition is true
4. Alert is active
5. `for:` duration is satisfied
6. Alert is sent to Alertmanager
7. Routing decides notification/integration
8. Incident may be created if configured
9. Engineers investigate and resolve
10. Alert is cleared when conditions recover

---

## 10) Alert vs incident

Not every alert becomes an incident.

An alert may simply be:

- a warning
- a service health issue
- a notification for a team

An incident is typically when the problem has:

- user impact
- operational coordination needs
- broader service degradation
- escalation or formal response

---

## 11) Practical rule of thumb

Set thresholds based on:

- what is acceptable
- what is normal
- what is user-impacting
- how noisy the alert can be
- how quickly the team needs to act

Then route by severity so the right people get notified at the right time.

---

## 12) One-line summary

Alert thresholds are the decision boundaries that tell the monitoring system “this metric is abnormal enough to deserve attention,” and when the condition fires, it may notify people, escalate, or trigger an incident workflow.
