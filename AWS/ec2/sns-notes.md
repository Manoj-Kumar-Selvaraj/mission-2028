## SNS — Simple but Complete

### 1. What is SNS?

**Amazon SNS = Simple Notification Service.**

It is a **publish/subscribe messaging service**.

One producer publishes a message to an SNS **topic**, and SNS can deliver that message to **multiple subscribers**.

```text
Producer
   ↓
SNS Topic
   ├── Email
   ├── Lambda
   ├── SQS
   └── HTTP endpoint
```

Easy memory:

> **SNS = publish once, deliver to many.**

---

## 2. Topic

A **topic** is the logical communication channel.

Example:

```text
Topic:
prod-alerts
```

Applications or AWS services publish messages to that topic.

Then subscribers receive them.

Think:

> **Topic = central broadcast channel.**

---

## 3. Publisher

A **publisher** sends a message to the SNS topic.

Examples:

```text
CloudWatch Alarm
Application
Lambda
EventBridge
CLI / SDK
```

Example:

```text
CloudWatch Alarm
     ↓
SNS Topic: prod-alerts
```

---

## 4. Subscriber

A **subscriber** receives messages from the topic.

Common subscribers include:

```text
Email
SMS
SQS
Lambda
HTTP/HTTPS endpoint
```

Example:

```text
SNS Topic
   ├── Email → Ops team
   ├── Lambda → Automation
   └── SQS → Processing application
```

---

## 5. Fanout

This is one of the most important SNS concepts.

A single published message can be distributed to many subscribers.

```text
Message
   ↓
SNS
   ├── Consumer A
   ├── Consumer B
   └── Consumer C
```

This is called:

> **Fanout**

Example:

An order is created.

```text
Order Service
     ↓
SNS Topic
     ├── Billing Queue
     ├── Inventory Queue
     └── Notification Queue
```

All three systems can process the same event independently.

---

## 6. Push Model

SNS normally **pushes** messages to subscribers.

The subscriber does not continuously ask SNS:

> "Do you have a message?"

Instead:

```text
SNS receives message
      ↓
SNS delivers it
      ↓
Subscriber receives message
```

This is a key difference from SQS, which we'll cover next.

---

## 7. Standard vs FIFO SNS

### Standard Topic

Most common.

Good when you need:

- Very high throughput
- Fanout
- At-least-once delivery
- Strict ordering is not required

---

### FIFO Topic

Use when:

- Message order matters
- Duplicate processing must be minimized
- Subscribers support FIFO behavior

Think:

```text
Standard → scale + flexibility

FIFO → ordering + deduplication
```

---

## 8. Delivery Retry

If SNS cannot deliver to some subscriber types, it can retry according to its delivery policy.

Example:

```text
SNS
 ↓
HTTP endpoint unavailable
 ↓
Retry
 ↓
Retry
```

For workloads requiring durable buffering, a common design is:

```text
SNS
 ↓
SQS
 ↓
Consumer
```

because SQS holds the message until the consumer processes it.

---

## 9. SNS + SQS

This is a very common production pattern.

```text
Publisher
    ↓
SNS Topic
 ├──────────┬───────────┐
 ↓          ↓           ↓
SQS-A      SQS-B       SQS-C
 ↓          ↓           ↓
App-A      App-B       App-C
```

Why?

SNS gives:

> **fanout**

SQS gives:

> **buffering + durable asynchronous processing**

---

## 10. Example — CloudWatch Alert

```text
EC2 CPU
   ↓
CloudWatch Metric
   ↓
CloudWatch Alarm
   ↓
SNS Topic
   ├── Email → DevOps
   └── Lambda → Automation
```

If CPU becomes too high:

```text
CPU > 80%
   ↓
Alarm → ALARM state
   ↓
SNS notification
   ↓
Engineer receives email
```

We'll configure this practically after SQS.

---

## Quick Memory

```text
Publisher
   ↓
Topic
   ↓
Subscribers
```

And:

> **SNS = push + pub/sub + fanout**

### Interview one-liner

> **Amazon SNS is a publish/subscribe messaging service where publishers send messages to a topic and SNS pushes those messages to one or more subscribers such as email, Lambda, HTTP endpoints, or SQS queues.**

