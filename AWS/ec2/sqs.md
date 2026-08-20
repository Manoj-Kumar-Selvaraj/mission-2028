## SQS — Simple but Complete

### 1. What is SQS?

**Amazon SQS = Simple Queue Service.**

It is a **message queue** used to decouple producers and consumers.

```text
Producer
   ↓
SQS Queue
   ↓
Consumer
```

The producer does not need the consumer to be available at the same time. SQS stores the message until it is processed or expires. 

> **SQS = buffer messages between systems.**

---

# 2. Producer and Consumer

### Producer

Sends messages to the queue.

Examples:

```text
Application
Lambda
EC2
ECS/EKS
SNS
```

### Consumer

Reads and processes messages.

Example:

```text
Order Service
    ↓
SQS
    ↓
Payment Worker
```

---

# 3. Message Lifecycle

This is the most important SQS flow:

```text
Producer
   ↓
Send message
   ↓
SQS Queue
   ↓
Consumer receives message
   ↓
Message becomes invisible temporarily
   ↓
Consumer processes it
   ↓
Consumer deletes message
```

If processing succeeds:

```text
Receive
  ↓
Process
  ↓
Delete
```

If it is **not deleted**, it becomes visible again after the visibility timeout and can be processed again. 

---

# 4. Visibility Timeout

Suppose:

```text
Queue:
Order-123
```

Consumer A receives it.

SQS temporarily hides that message from other consumers:

```text
Consumer A receives Order-123
        ↓
Invisible to others
        ↓
Consumer A processing
```

Default visibility timeout:

> **30 seconds**

It can be configured up to **12 hours**. 

### Example

```text
Visibility Timeout = 60 sec
```

Consumer gets message:

```text
12:00:00 → message received
```

Until:

```text
12:01:00
```

other consumers normally cannot receive that same message.

If Consumer A successfully processes it:

```text
DeleteMessage
```

and it disappears permanently.

If Consumer A crashes:

```text
No delete
   ↓
60 seconds expires
   ↓
Message visible again
   ↓
Consumer B can process it
```

### Very important

> **Receiving a message does NOT delete it.**

The consumer must explicitly delete it after successful processing. 

---

# 5. Standard Queue

This is the normal/default queue type.

Use when:

- Very high throughput is needed
- Strict ordering isn't required
- Applications can tolerate occasional duplicate delivery

Conceptually:

```text
Messages sent:

A
B
C

Could be processed:

A
C
B
```

Standard queues provide **at-least-once delivery**, so applications should generally be designed to tolerate duplicate processing. 

### Memory

> **Standard = scale first; strict ordering not guaranteed.**

---

# 6. FIFO Queue

**FIFO = First-In, First-Out.**

Use when:

- Ordering matters
- Duplicate processing needs stronger control

Example:

```text
A → B → C
```

must be processed:

```text
A → B → C
```

FIFO queues use concepts such as:

```text
MessageGroupId
MessageDeduplicationId
```

Messages within the same message group maintain their FIFO ordering. 

### Example

Banking events:

```text
Deposit $100
      ↓
Withdraw $50
      ↓
Check balance
```

Order matters.

### Memory

```text
Standard
→ high scale

FIFO
→ order + deduplication
```

---

# 7. Short Polling

Consumers get messages by calling:

```text
ReceiveMessage
```

With **short polling**, SQS responds immediately.

If nothing is found:

```text
Consumer → SQS
           ↓
        No message
           ↓
Immediate empty response
```

The application may then call again.

This can generate many API calls. 

---

# 8. Long Polling

With long polling:

```text
Consumer
   ↓
ReceiveMessage
   ↓
SQS waits for a message
```

Example:

```text
WaitTimeSeconds = 20
```

SQS can wait for up to **20 seconds** before returning an empty response. 

If a message arrives after 5 seconds:

```text
Message arrives
      ↓
SQS returns immediately
```

Advantages:

- Fewer empty responses
- Fewer unnecessary API calls
- Lower cost
- More efficient message consumption 

### Memory

```text
Short Polling
→ Ask and return immediately

Long Polling
→ Ask and wait for message
```

Long polling is generally preferred.

---

# 9. Message Retention

What if nobody consumes the message?

SQS stores it for the configured **message retention period**.

Default:

```text
4 days
```

Configurable:

```text
1 minute → 14 days
```

After retention expires:

```text
Message deleted automatically
``` 


Do not confuse:

```text
Visibility Timeout
```

with:

```text
Message Retention
```

### Difference

**Visibility timeout**

> Consumer received the message; how long should it remain hidden while processing?

**Retention period**

> How long can the message remain in SQS before SQS deletes it?

---

# 10. Dead-Letter Queue — DLQ

Suppose one bad message keeps failing.

```text
Message
  ↓
Consumer
  ↓
FAIL
  ↓
Visible again
  ↓
Consumer
  ↓
FAIL
```

You don't want this forever.

Configure:

```text
maxReceiveCount = 5
```

After repeated failures:

```text
Main Queue
    ↓
Failed 5 times
    ↓
Dead-Letter Queue
```

Example:

```text
orders-queue
      ↓
orders-dlq
```

The DLQ lets you inspect failed messages separately without blocking normal processing. AWS recommends configuring DLQ retention longer than the source queue retention. 

### Memory

> **DLQ = quarantine for repeatedly failing messages.**

---

# 11. Redrive

After fixing the application/problem, messages in a DLQ may need to be moved back for processing.

Conceptually:

```text
DLQ
 ↓
Fix issue
 ↓
Redrive
 ↓
Source Queue
 ↓
Consumer retries
```

This is called **redrive**.

---

# 12. SNS vs SQS

This distinction is extremely important.

### SNS

```text
Publisher
    ↓
SNS
 ├── Consumer A
 ├── Consumer B
 └── Consumer C
```

SNS:

> **Push + fanout**

One message can go to many subscribers.

---

### SQS

```text
Producer
   ↓
Queue
   ↓
Consumer
```

SQS:

> **Store + buffer + consumer polls**

---

## Example

Suppose an order is created.

We want:

- Billing to process it
- Inventory to process it
- Analytics to process it

If all consumers use one SQS queue:

```text
        SQS
       / | \
Billing Inventory Analytics
```

they normally compete for messages.

A message consumed by one worker isn't intended to be independently delivered to every other application.

Instead:

```text
Order Service
     ↓
SNS
 ┌────┼────┐
 ↓    ↓    ↓
SQS  SQS  SQS
 ↓    ↓    ↓
Billing Inventory Analytics
```

Now each application gets its **own copy** through its own queue.

This is:

> **SNS + SQS fanout architecture.**

---

# Quick Memory

```text
SQS
 ↓
QUEUE / BUFFER

Producer
 ↓
Queue
 ↓
Consumer

Visibility Timeout
→ hide during processing

DeleteMessage
→ processing complete

Retention
→ how long SQS stores it

Long Polling
→ wait efficiently for messages

DLQ
→ repeatedly failed messages

Standard
→ scale

FIFO
→ ordered processing
```

### Interview one-liner

> **Amazon SQS is a managed message-queue service used to decouple producers and consumers. Producers place messages into a queue, consumers poll and process them, visibility timeout prevents concurrent processing during that window, successful messages are deleted, and repeatedly failing messages can be moved to a dead-letter queue.** 