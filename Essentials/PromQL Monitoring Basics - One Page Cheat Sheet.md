# PromQL Monitoring Basics - One Page Cheat Sheet

## 1) Monitoring in one sentence

Monitoring = collecting numeric measurements over time so you can detect issues, understand trends, and answer operational questions.

---

## 2) Core concepts

### Metric
A numerical measurement about a system.

Examples:

- CPU usage
- memory used
- requests/sec
- latency
- error count

### Time series
A metric sampled over time.

Example idea:

- metric name: `http_requests_total`
- labels: `method="GET"`, `status="200"`
- values: 100, 110, 125 at different timestamps

### Labels
Metadata attached to a metric.

Examples:

- `job="api"`
- `instance="node-1"`
- `status="500"`
- `endpoint="/login"`

---

## 3) Common metric types

### Counter
Only increases.

Use for:

- total requests
- total errors
- total bytes sent

Example:

- `http_requests_total`

### Gauge
Can go up or down.

Use for:

- memory usage
- active connections
- CPU usage

Example:

- `node_memory_used_bytes`

### Histogram
Tracks distribution by buckets.

Use for:

- request durations
- latency buckets

Example:

- `http_request_duration_seconds_bucket`

### Summary
Also captures distribution summaries, often for quantiles.

---

## 4) PromQL basics

PromQL lets you query and transform time series.

### Select series by label

```promql
http_requests_total{job="api"}
```

### Match multiple labels

```promql
http_requests_total{job="api",status="500"}
```

### Regex match

```promql
http_requests_total{job=~"api.*"}
```

---

## 5) Rate / change over time

This is one of the most important PromQL ideas.

### Request rate over last 5 minutes

```promql
rate(http_requests_total[5m])
```

### Error rate (5xx only)

```promql
rate(http_requests_total{status=~"5.."}[5m])
```

### Increase over a time window

```promql
increase(http_requests_total[1h])
```

Use `rate()` to answer “how fast?” and `increase()` to answer “how much total growth?”

---

## 6) Aggregation

### Sum by label

```promql
sum by (job) (rate(http_requests_total[5m]))
```

### Average by instance

```promql
avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100
```

### Max latency

```promql
max by (job) (rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m]))
```

Common functions:

- `sum()`
- `avg()`
- `max()`
- `min()`
- `count()`

---

## 7) Simple dashboard questions

These are the kinds of questions monitoring answers:

- What is the current request rate?
- Which service has the most errors?
- Which endpoint is slowest?
- Did CPU spike after deployment?
- Is latency trending upward?

---

## 8) Alerting idea

Alerts are usually expressions like:

```promql
rate(http_requests_total{status=~"5.."}[5m]) > 0.05
```

Meaning:

- there are too many 5xx responses
- over a recent time window
- beyond an acceptable threshold

---

## 9) Visual mental model

Monitoring stack:

- Metrics → numbers
- Time series → values over time
- Labels → dimensions
- PromQL → query language
- Dashboards → visual answers
- Alerts → notifications

---

## 10) Quick cheat sheet

```promql
# request rate by service
sum by (job) (rate(http_requests_total[5m]))

# errors by status
rate(http_requests_total{status=~"5.."}[5m])

# CPU percent by instance
avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100

# memory usage
node_memory_used_bytes

# requests per second total
rate(http_requests_total[5m])
```

---

## 11) Beginner rule of thumb

If you see a metric in Prometheus, ask:

- What is being measured?
- Is it a counter, gauge, histogram, or summary?
- Which labels matter?
- Should I look at a rate, increase, or average?
- Am I looking at a point in time or over a time window?

---

## 12) Final takeaway

PromQL is easiest to understand when you remember:

- metrics are numbers
- time series are numbers over time
- labels are dimensions
- `rate()` tells you how fast something changes
- `sum by (...)` groups results by labels
- dashboards and alerts are just practical uses of these queries

---

## 13) One-line summary

PromQL is the language for asking “how is this system behaving over time?” using metrics, labels, and time windows.
