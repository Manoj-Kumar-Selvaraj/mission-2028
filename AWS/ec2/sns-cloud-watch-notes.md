## CloudWatch Alarm + SNS Integration

This is the normal alerting flow:

```text
EC2 / Application
      ↓
CloudWatch Metric
      ↓
CloudWatch Alarm
      ↓
SNS Topic
      ↓
Email / SMS / Lambda / SQS
```

### 1. CloudWatch Alarm

A **CloudWatch Alarm watches a metric and compares it against a threshold**.

Example:

```text
Metric    = CPUUtilization
Threshold = > 80%
Period    = 5 minutes
```

If the metric breaches the configured condition, the alarm can change state and trigger an action. 

### 2. Alarm states

CloudWatch alarms have three states:

```text
OK
ALARM
INSUFFICIENT_DATA
```

- **OK** → metric is within the expected condition.
- **ALARM** → configured threshold condition is breached.
- **INSUFFICIENT_DATA** → CloudWatch doesn't currently have enough data to determine the state. 

---

## 3. Period

The **period** is the size of each metric time bucket.

Example:

```text
Period = 60 seconds
```

CloudWatch evaluates one-minute datapoints.

Or:

```text
Period = 300 seconds
```

means five-minute periods.

---

## 4. Evaluation Periods

Suppose:

```text
Period             = 1 minute
Evaluation Periods = 3
```

CloudWatch evaluates the last:

```text
3 × 1 minute
= 3 minutes
```

of datapoints.

---

## 5. Datapoints to Alarm

This controls **how many of those evaluation periods must breach**.

Example:

```text
Evaluation Periods = 3
Datapoints to Alarm = 2
```

This means:

> At least 2 of the last 3 datapoints must be breaching.

Example:

```text
CPU threshold = 80%

Minute 1 = 85% ❌
Minute 2 = 70% ✅
Minute 3 = 90% ❌
```

Two out of three breached:

```text
Alarm → ALARM
```

This is often called an **M-out-of-N alarm**. 

---

# 6. Connect Alarm to SNS

Suppose we create:

```text
SNS Topic:
prod-alerts
```

Subscriptions:

```text
prod-alerts
   ├── devops@example.com
   └── Lambda
```

Then configure the CloudWatch Alarm action:

```text
When state becomes ALARM
        ↓
Publish to SNS topic
        ↓
prod-alerts
```

SNS then delivers the notification to its subscribers. CloudWatch can configure SNS actions for transitions to `ALARM`, `OK`, or `INSUFFICIENT_DATA`. 

---

# 7. Practical EC2 CPU Alarm

Example:

```text
EC2
 ↓
CPUUtilization
 ↓
CloudWatch Alarm

Threshold = > 80%
Period = 5 minutes
Evaluation Periods = 2
Datapoints to Alarm = 2
```

Meaning:

```text
CPU > 80%
for 2 consecutive 5-minute periods
```

Then:

```text
Alarm → ALARM
     ↓
SNS → prod-alerts
     ↓
Email → DevOps Team
```

---

# 8. Memory Alarm

For memory, remember:

```text
EC2 default metrics
      ↓
Memory ❌
```

So:

```text
CloudWatch Agent
      ↓
mem_used_percent
      ↓
CloudWatch Alarm
      ↓
SNS
```

Example:

```text
mem_used_percent > 85%
for 3 out of 5 minutes
```

---

# 9. Disk Alarm

Same idea:

```text
CloudWatch Agent
      ↓
disk_used_percent
      ↓
Alarm > 90%
      ↓
SNS
      ↓
Email / automation
```

---

# 10. Setup in AWS Console

Typical flow:

```text
SNS
→ Topics
→ Create topic
→ prod-alerts
→ Create subscription
→ Email
→ Confirm email subscription
```

Then:

```text
CloudWatch
→ Metrics
→ Select metric
→ Create alarm
→ Define threshold
→ Configure notification
→ Select prod-alerts SNS topic
→ Create alarm
```

Email subscriptions remain pending until the recipient confirms them. 

---

## Quick Memory

```text
Metric
 ↓
WHAT are we monitoring?

Threshold
 ↓
WHEN is it bad?

Period
 ↓
How large is each datapoint window?

Evaluation Periods
 ↓
How many periods do we inspect?

Datapoints to Alarm
 ↓
How many must breach?

SNS
 ↓
WHO should be notified?
```

### Interview one-liner

> **I configure a CloudWatch alarm against a metric with an appropriate threshold, period and M-out-of-N evaluation policy, and configure an SNS topic as the alarm action so notifications or automation are triggered when the alarm changes state.**