## Load Balancer + EC2

A **Load Balancer distributes incoming traffic across multiple backend targets**, such as EC2 instances.

Typical architecture:

```text
Users
  |
  v
Load Balancer
  |
  v
Target Group
  |
  +---------+---------+
  |         |         |
 EC2-1     EC2-2     EC2-3
```

The Load Balancer sends traffic only to targets considered healthy by its target group's health checks. 

---

# 1. ALB — Application Load Balancer

ALB works mainly at **Layer 7 — Application Layer**.

It understands:

```text
HTTP
HTTPS
```

So ALB can make routing decisions based on HTTP information. 

### Example

```text
Users
  |
  v
 ALB
  |
  +--- /api/* ------> API Target Group
  |
  +--- /images/* ---> Image Target Group
```

This is called **content-based routing**.

Common ALB use cases:

- Web applications
- REST APIs
- Microservices
- Host/path-based routing
- HTTP/HTTPS workloads

### Easy Memory

> **ALB = Application-aware HTTP/HTTPS load balancing.**

---

# 2. NLB — Network Load Balancer

NLB operates at the **network/transport level** and is used when you need high-performance connection-based load balancing.

It supports protocols including TCP, UDP, TLS, TCP_UDP, QUIC and TCP_QUIC. 

Example:

```text
Client
  |
 TCP :443
  |
  v
 NLB
  |
  +---------+
  |         |
 EC2       EC2
```

Common NLB use cases:

- TCP applications
- Very high connection volumes
- Low-latency network workloads
- Applications that need static load-balancer IP characteristics

### Easy Memory

> **NLB = Network/connection-level load balancing.**

---

# ALB vs NLB

| ALB | NLB |
|---|---|
| Layer 7 | Layer 4/network-oriented |
| HTTP/HTTPS aware | TCP/UDP/TLS etc. |
| Path/host routing | Connection-based routing |
| Web apps/APIs | Network/high-performance workloads |

---

# 3. Listener

A **Listener receives incoming connections on the Load Balancer**.

Example:

```text
ALB

Listener:
HTTP  :80
HTTPS :443
```

A listener has rules that determine where traffic should go.

Example:

```text
HTTPS :443
    |
    v
/api/*
    |
    v
API Target Group
```

---

# 4. Target Group

A **Target Group is a logical collection of backend targets**.

For example:

```text
Target Group: app-prod

EC2-1 :8080
EC2-2 :8080
EC2-3 :8080
```

The Load Balancer forwards requests to these registered targets using the configured protocol and port. 

Conceptually:

```text
ALB
 |
Listener :443
 |
Target Group
 |
 +------ EC2-1 :8080
 +------ EC2-2 :8080
 +------ EC2-3 :8080
```

---

## Target Types

Depending on the load balancer/target group configuration, targets can include resources such as:

```text
EC2 Instances
IP addresses
```

ALB also supports additional target types such as Lambda in supported configurations. 

For EC2 interviews, focus mainly on:

> **Load Balancer → Listener → Target Group → EC2**

---

# 5. Health Checks

A Load Balancer periodically checks whether targets are healthy.

Example:

```text
Target Group Health Check

Protocol = HTTP
Port     = 8080
Path     = /health
```

ALB sends:

```text
GET /health
```

to each registered target.

Example:

```text
EC2-1 /health → 200 OK ✅
EC2-2 /health → 200 OK ✅
EC2-3 /health → timeout ❌
```

Traffic is routed to healthy targets rather than the unhealthy target. 

---

# Health Check Settings

Typical settings include:

```text
Protocol
Port
Path
Interval
Timeout
Healthy Threshold
Unhealthy Threshold
Expected success code
```

These settings are configured **per Target Group**. 

---

# Example

Suppose:

```text
ALB
 |
Target Group
 |
 +-- EC2-A ✅
 +-- EC2-B ✅
 +-- EC2-C ❌
```

The application on EC2-C crashes.

```text
EC2-C
   |
/health fails
   |
   v
Target marked unhealthy
```

The Load Balancer stops routing normal application traffic to that unhealthy target. 

If this Target Group is also connected to an **ASG with ELB health checks enabled**, ASG can use that health information to replace the unhealthy instance.

---

# ASG + ALB Full Flow

```text
                    Users
                      |
                      v
                     ALB
                      |
                  Listener :443
                      |
                      v
                 Target Group
                  /        \
                 /          \
              EC2-1        EC2-2
               AZ-A         AZ-B
                 \          /
                  \        /
                      ASG
                       |
                Launch Template
```

Flow:

```text
Traffic increases
      ↓
ASG launches EC2
      ↓
EC2 registered with Target Group
      ↓
Health check passes
      ↓
Load Balancer sends traffic
```

---

# Important Interview Scenario

EC2 shows:

```text
Running ✅
```

but ALB shows:

```text
Unhealthy ❌
```

Why?

Because these check different things.

EC2 may be running successfully while:

```text
Application crashed
Wrong application port
Wrong /health path
Security Group blocking ALB
Application not listening
Health check timeout
Wrong expected HTTP code
```

Target-health failure reasons can include response-code mismatch, timeout, failed connection/health checks, or registration/configuration issues. 

---

# Quick Memory

```text
Load Balancer
      ↓
Distributes traffic

Listener
      ↓
Accepts traffic on port/protocol

Target Group
      ↓
Collection of backend servers

Health Check
      ↓
Determines which targets are healthy

ALB
      ↓
HTTP/HTTPS + application routing

NLB
      ↓
TCP/UDP/TLS + network performance
```

### Interview one-liner

> **A Load Balancer receives client traffic through a listener, forwards it to healthy backends in a target group, and uses health checks to avoid routing traffic to unhealthy EC2 instances.**


Sure. The easiest way is to separate **ALB routing into two stages**.

## 1. Listener rules — Which Target Group?

ALB first decides **which target group should receive the request** based on listener rules. 

Example:

```text
User Request
    ↓
   ALB
    ↓
Check Listener Rules
    ↓
Choose Target Group
```

Common rules:

- **Host-based**  
  `api.example.com` → API target group  
  `app.example.com` → App target group

- **Path-based**  
  `/api/*` → API target group  
  `/images/*` → Image target group

- **Header-based**  
  `X-Version: beta` → Beta target group

- **Query-string-based**  
  `/app?version=v2` → V2 target group

- **Source-IP-based**  
  `10.20.0.0/16` → Internal target group

You can also use **weighted forwarding**:

```text
90% → Version 1
10% → Version 2
```

Useful for canary or blue/green deployments.

---

## 2. Routing algorithm — Which EC2 inside that Target Group?

Once ALB chooses the target group, it decides which **healthy EC2 target** gets the request.

AWS currently supports three ALB target-group algorithms: **Round Robin, Least Outstanding Requests, and Weighted Random**. Round Robin is the default. 

### Round Robin

Requests are distributed one after another.

```text
Request 1 → EC2-A
Request 2 → EC2-B
Request 3 → EC2-C
Request 4 → EC2-A
```

Use when servers and requests are roughly similar.

**Memory:**  
> Take turns.

---

### Least Outstanding Requests

Send the new request to the server currently handling the **fewest unfinished requests**. 

Example:

```text
EC2-A → 20 active requests
EC2-B → 4 active requests
EC2-C → 12 active requests

New request → EC2-B
```

Good when some requests take much longer than others.

**Memory:**  
> Send work to the least busy server.

---

### Weighted Random

ALB dynamically distributes requests using weights and can optionally use anomaly mitigation. 

You don't need to go very deep into this for interviews.

**Memory:**  
> ALB adjusts how much traffic targets receive.

---

## 3. NLB routing

NLB is different.

It doesn't inspect `/api`, hostnames, headers, etc. like an ALB. It selects a backend using a **flow-hash algorithm**. A connection is inherently sticky to the selected target for that connection. 

```text
Client
  ↓
NLB
  ↓
Flow Hash
  ↓
EC2-B
```

### Final memory

```text
ALB Listener Rule
      ↓
Which TARGET GROUP?

Host / Path / Header / Query / Source IP
      ↓
Target Group Algorithm
      ↓
Which EC2?

Round Robin
Least Outstanding Requests
Weighted Random
```

The most important two to understand deeply are **Round Robin** and **Least Outstanding Requests**.

