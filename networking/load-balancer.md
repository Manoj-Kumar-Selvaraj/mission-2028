# Load Balancer — In-Depth Networking Notes

## 1. What is a Load Balancer?

A **Load Balancer (LB)** distributes incoming traffic across multiple backend servers.

```text
Users
  ↓
Load Balancer
  ├── Server 1
  ├── Server 2
  └── Server 3
```

Main goals:

- Distribute traffic
- Improve availability
- Avoid overloading one server
- Stop sending traffic to unhealthy servers
- Allow backend servers to scale horizontally

---

## 2. Basic Traffic Flow

Suppose a user opens:

```text
https://app.example.com
```

Flow:

```text
User
  ↓
DNS resolves app.example.com
  ↓
Load Balancer
  ↓
Select healthy backend
  ↓
Backend Server
  ↓
Response
  ↓
Load Balancer
  ↓
User
```

The user usually communicates with the **load balancer endpoint**, not directly with individual application servers.

---

# 3. Frontend vs Backend

A load balancer has two sides.

## Frontend

The side facing clients.

Example:

```text
Client → Load Balancer:443
```

Frontend configuration may contain:

```text
Protocol: HTTPS
Port: 443
Certificate
Public/private IP
```

---

## Backend

Servers that actually process requests.

Example:

```text
Load Balancer
   ↓
10.0.1.10:8080
10.0.2.10:8080
10.0.3.10:8080
```

These are commonly called:

```text
Backends
Targets
Pool members
Upstreams
```

---

# 4. Listener

A **listener** defines which traffic the load balancer accepts.

Example:

```text
Protocol: HTTPS
Port: 443
```

Meaning:

> Listen for HTTPS traffic on port 443.

Another example:

```text
HTTP :80
HTTPS:443
TCP  :3306
```

---

# 5. Backend Pool / Target Group

A target group is a collection of backend servers.

Example:

```text
Target Group: web-app

10.0.1.10:8080
10.0.2.10:8080
10.0.3.10:8080
```

The load balancer chooses one healthy target from this group.

In AWS ALB/NLB terminology:

```text
Backend Pool = Target Group
```

---

# 6. Health Checks

A load balancer continuously checks whether backends are healthy.

Example HTTP health check:

```text
GET /health
```

Expected response:

```text
200 OK
```

If Server 2 fails:

```text
Server 1 → Healthy
Server 2 → Unhealthy
Server 3 → Healthy
```

Traffic becomes:

```text
Load Balancer
   ├── Server 1
   ├── Server 2  X
   └── Server 3
```

The LB stops routing new traffic to Server 2.

---

# 7. Health Check Parameters

Common health-check settings:

```text
Protocol
Port
Path
Interval
Timeout
Healthy threshold
Unhealthy threshold
Expected status code
```

Example:

```text
Protocol: HTTP
Port: 8080
Path: /health
Interval: 30 seconds
Timeout: 5 seconds
Healthy threshold: 2
Unhealthy threshold: 3
```

---

# 8. Layer 4 Load Balancer

Layer 4 works mainly with:

```text
IP addresses
TCP
UDP
Ports
```

It does not normally need to understand HTTP URLs or headers.

Example:

```text
Client
  ↓ TCP 443
L4 Load Balancer
  ↓
Server
```

It makes forwarding decisions based mainly on:

```text
Source IP
Source port
Destination IP
Destination port
Protocol
```

---

# 9. Layer 7 Load Balancer

Layer 7 understands application protocols such as:

```text
HTTP
HTTPS
```

It can inspect:

```text
Hostname
URL path
HTTP method
Headers
Cookies
```

Example:

```text
/api/*      → API servers
/images/*   → Image servers
/admin/*    → Admin servers
```

This is called **content-based routing**.

---

# 10. Layer 4 vs Layer 7

| Feature | Layer 4 LB | Layer 7 LB |
|---|---|---|
| OSI layer | Transport | Application |
| Understands | TCP/UDP | HTTP/HTTPS |
| Routes using | IP + Port | Host/path/header/etc. |
| HTTP path routing | No | Yes |
| TLS termination | Limited/pass-through common | Common |
| Performance | Very high | More processing |
| AWS example | NLB | ALB |

Easy memory:

```text
L4 → connection/transport aware
L7 → application aware
```

---

# 11. AWS ALB

**ALB = Application Load Balancer**

Primarily:

```text
Layer 7
```

Supports:

```text
HTTP
HTTPS
HTTP/2
WebSocket
gRPC
```

Useful for:

```text
Web applications
APIs
Microservices
Containers
Path-based routing
Host-based routing
```

---

# 12. ALB Path-Based Routing

Example:

```text
app.example.com/api/*
        ↓
API Target Group
```

```text
app.example.com/images/*
        ↓
Image Target Group
```

Flow:

```text
ALB
├── /api/*    → API servers
├── /images/* → Image servers
└── /*        → Web servers
```

---

# 13. Host-Based Routing

An ALB can route based on hostname.

Example:

```text
api.example.com
        ↓
API servers
```

```text
admin.example.com
        ↓
Admin servers
```

One load balancer can therefore serve multiple applications.

---

# 14. AWS NLB

**NLB = Network Load Balancer**

Primarily:

```text
Layer 4
```

Supports:

```text
TCP
UDP
TLS
TCP_UDP
```

Useful when you need:

```text
Very high throughput
Very low latency
Static IP capability
Preserved source IP in supported scenarios
Non-HTTP protocols
```

---

# 15. ALB vs NLB

| Feature | ALB | NLB |
|---|---|---|
| Layer | L7 | L4 |
| HTTP aware | Yes | No |
| Path routing | Yes | No |
| Host routing | Yes | No |
| TCP support | Through HTTP protocols | Native |
| UDP | No | Yes |
| Static IP | Not directly per node | Supported |
| Typical use | Web/API | TCP/UDP/high-performance |

---

# 16. Load Balancing Algorithms

A load balancer needs to choose a backend.

Common approaches include:

## Round Robin

Requests are distributed sequentially.

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
```

---

## Least Connections

Send traffic to the server with the fewest active connections.

Useful when connections have different durations.

---

## Weighted Routing

Servers can receive different percentages of traffic.

Example:

```text
Server A → weight 70
Server B → weight 30
```

Useful for:

```text
Canary deployment
Gradual migration
Different server capacities
```

---

## Hash-Based Routing

A value such as client IP may be hashed to select a backend.

Useful when consistent backend selection is desirable.

---

# 17. Sticky Sessions

Normally requests from one client can go to different servers.

```text
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
```

With **sticky sessions**:

```text
Client A
  ↓
Server B
  ↓
Future requests
  ↓
Server B
```

This is also called:

```text
Session Affinity
```

---

# 18. Why Sticky Sessions Exist

Suppose an application stores login/session state locally:

```text
Server A:
Session ABC123
```

If the next request goes to Server B:

```text
Server B:
Doesn't know ABC123
```

User may appear logged out.

Sticky sessions solve this temporarily by keeping the client on the same backend.

Better application architecture usually stores session state centrally:

```text
Redis
Database
Shared session store
```

Then sticky sessions become less necessary.

---

# 19. TLS Termination

A load balancer can terminate TLS.

Flow:

```text
Client
   ↓ HTTPS
Load Balancer
   ↓ HTTP or HTTPS
Backend
```

The certificate is installed on the load balancer.

Benefits:

```text
Central certificate management
Reduce TLS workload on applications
Simpler backend configuration
```

---

# 20. TLS Passthrough

Instead of decrypting TLS, the load balancer can forward encrypted traffic directly.

```text
Client
   ↓ TLS
Load Balancer
   ↓ TLS
Backend
```

The backend performs TLS termination.

Common with Layer 4 load balancing.

---

# 21. TLS Re-Encryption

Another design:

```text
Client
   ↓ HTTPS
Load Balancer
   ↓ decrypt
   ↓ encrypt again
Backend
```

So both:

```text
Client ↔ LB
LB ↔ Backend
```

are encrypted.

---

# 22. Source IP

With some load balancers, the backend may see:

```text
Source IP = Load Balancer
```

instead of the actual client.

For HTTP-based load balancers, the original client IP is commonly passed through headers such as:

```text
X-Forwarded-For
```

Example:

```text
X-Forwarded-For: 203.0.113.10
```

---

# 23. X-Forwarded Headers

Common headers:

```text
X-Forwarded-For
X-Forwarded-Proto
X-Forwarded-Port
```

Example:

```text
X-Forwarded-For: 203.0.113.10
X-Forwarded-Proto: https
X-Forwarded-Port: 443
```

These help the backend know details about the original client connection.

---

# 24. Public vs Internal Load Balancer

## Public / Internet-Facing

Accessible from the Internet.

```text
Internet
   ↓
Public Load Balancer
   ↓
Private Backends
```

Typical web application setup.

---

## Internal Load Balancer

Accessible only from private networks.

```text
Frontend service
   ↓
Internal LB
   ↓
Backend service
```

Useful for internal microservices.

---

# 25. Load Balancer and DNS

Users normally access:

```text
app.example.com
```

DNS points toward the load balancer.

```text
app.example.com
      ↓
DNS
      ↓
Load Balancer
      ↓
Backend Servers
```

Users do not need to know backend IP addresses.

---

# 26. Horizontal Scaling

Without a load balancer:

```text
Users
  ↓
Server 1
```

Server 1 becomes overloaded.

With horizontal scaling:

```text
Users
  ↓
Load Balancer
  ├── Server 1
  ├── Server 2
  ├── Server 3
  └── Server 4
```

Servers can be added or removed dynamically.

---

# 27. Load Balancer + Auto Scaling

Typical AWS architecture:

```text
Internet
   ↓
ALB
   ↓
Target Group
   ↓
Auto Scaling Group
   ├── EC2
   ├── EC2
   └── EC2
```

When load increases:

```text
ASG adds instances
   ↓
Instances register with target group
   ↓
Health checks pass
   ↓
ALB sends traffic
```

---

# 28. Cross-Zone Load Balancing

Suppose:

```text
AZ-A → 2 servers
AZ-B → 5 servers
```

Cross-zone load balancing allows traffic received in one AZ to be distributed across eligible targets in multiple AZs, depending on the load balancer type and configuration.

This helps improve even distribution across targets.

---

# 29. Connection Draining / Deregistration Delay

Suppose Server 1 needs to be removed.

Bad approach:

```text
Server immediately removed
   ↓
Active requests break
```

Better approach:

```text
Stop sending new requests
      ↓
Allow existing requests/connections to finish
      ↓
Remove server
```

This is often called:

```text
Connection draining
Deregistration delay
```

---

# 30. High Availability

A load balancer should itself be highly available.

Cloud-managed load balancers are typically deployed across multiple availability zones.

Example:

```text
             Load Balancer
             /          \
          AZ-A          AZ-B
           ↓              ↓
        Servers        Servers
```

If one AZ fails, traffic can continue through healthy infrastructure in another AZ.

---

# 31. Load Balancer vs Reverse Proxy

They overlap significantly.

A reverse proxy:

```text
Client
  ↓
Reverse Proxy
  ↓
Application
```

A load balancer:

```text
Client
  ↓
Load Balancer
  ├── App1
  ├── App2
  └── App3
```

Products such as:

```text
Nginx
HAProxy
Envoy
```

can function as both.

---

# 32. Load Balancer vs DNS Load Balancing

DNS load balancing:

```text
app.example.com
  ↓
DNS
  ↓
IP A / IP B / IP C
```

The client connects directly to one returned endpoint.

Traditional LB:

```text
Client
  ↓
Single LB endpoint
  ↓
LB chooses backend
```

DNS-based balancing operates at DNS resolution time, while a load balancer makes decisions when traffic reaches it.

---

# 33. Load Balancer Failure Scenario

Suppose:

```text
ALB returns 503
```

Possible reasons:

```text
No healthy targets
Target group empty
Backend health checks failing
Target registration issue
Application unavailable
```

---

# 34. 502 vs 503 vs 504

Very useful interview distinction.

## 502 Bad Gateway

LB/proxy contacted backend but received an invalid/broken response.

Possible causes:

```text
Application crash
Protocol mismatch
Connection reset
Malformed backend response
```

---

## 503 Service Unavailable

LB has no usable backend/service temporarily unavailable.

Possible causes:

```text
No healthy targets
Targets unavailable
Service overloaded
```

---

## 504 Gateway Timeout

LB/proxy waited for backend response but backend took too long.

Possible causes:

```text
Slow application
Slow database
Network delay
Backend timeout
```

---

# 35. Health Check Passing but Application Still Failing

A health check such as:

```text
GET /health
```

may return:

```text
200 OK
```

while:

```text
GET /checkout
```

fails.

Why?

The health endpoint may not validate:

```text
Database connectivity
Dependencies
External API
Application business logic
```

Good health checks should represent actual service readiness appropriately.

---

# 36. Load Balancer Troubleshooting Flow

If users cannot reach an application:

```text
1. DNS resolves?
2. LB reachable?
3. Listener exists?
4. Security rules allow traffic?
5. Correct target group?
6. Targets registered?
7. Targets healthy?
8. Health-check path correct?
9. Backend port correct?
10. Backend application listening?
11. Backend firewall/SG allows LB?
12. Application logs?
```

---

# 37. AWS ALB Example

Architecture:

```text
Internet
   ↓
Route 53
   ↓
ALB :443
   ↓
Target Group :8080
   ├── EC2-A:8080
   ├── EC2-B:8080
   └── EC2-C:8080
```

ALB listener:

```text
HTTPS :443
```

Target group:

```text
HTTP :8080
```

The client does not need to know that applications run internally on port `8080`.

---

# 38. AWS Security Group Flow

Typical setup:

```text
Internet
   ↓
ALB Security Group
Allow TCP 443 from Internet
   ↓
EC2 Security Group
Allow TCP 8080 from ALB Security Group
```

Better than exposing:

```text
EC2:8080
```

directly to the Internet.

---

# 39. One Full Request Example

User opens:

```text
https://app.example.com/api/users
```

Flow:

```text
1. DNS resolves app.example.com → ALB
2. Client establishes TCP connection to ALB:443
3. TLS handshake occurs
4. ALB receives HTTPS request
5. ALB reads HTTP path /api/users
6. Listener rule matches /api/*
7. ALB chooses API target group
8. Healthiest eligible backend chosen
9. Request forwarded to backend
10. Backend processes request
11. Backend responds to ALB
12. ALB returns response to client
```

That is a true **Layer 7 load-balancing flow**.

---

# 40. Important Interview Questions

### Why use a load balancer?

```text
High availability
Traffic distribution
Horizontal scaling
Health checking
Fault tolerance
TLS termination
Application routing
```

### ALB or NLB for HTTP APIs?

Usually:

```text
ALB
```

because you may need:

```text
Path routing
Host routing
HTTP awareness
TLS termination
```

### ALB or NLB for raw TCP?

Usually:

```text
NLB
```

### UDP application?

```text
NLB
```

### Need URL-based routing?

```text
ALB
```

### Need very high-performance Layer-4 traffic?

```text
NLB
```

---

# Quick Revision

```text
Load Balancer
→ distributes traffic across backends

Listener
→ port/protocol accepted by LB

Target Group
→ backend servers

Health Check
→ determines healthy targets

L4 LB
→ TCP/UDP/IP/ports

L7 LB
→ HTTP/HTTPS/host/path/headers

ALB
→ Layer 7

NLB
→ Layer 4

Sticky Session
→ client stays with same backend

TLS Termination
→ LB decrypts HTTPS

Connection Draining
→ finish existing requests before removing backend

502
→ bad backend response

503
→ service/no healthy backend

504
→ backend response timeout
```

## Best Interview Mental Model

```text
DNS
 ↓
Load Balancer Listener
 ↓
Routing Rule
 ↓
Target Group
 ↓
Health Check
 ↓
Selected Backend
 ↓
Application Response
```

Yes — concrete request/response examples make this much clearer.

## 502 Bad Gateway

Client sends:

```http
GET /api/users HTTP/1.1
Host: app.example.com
```

Load balancer forwards to backend:

```text
ALB → 10.0.1.20:8080
```

Backend accepts the connection, but then:

```text
Connection reset
```

or sends a malformed HTTP response.

So ALB returns:

```http
HTTP/1.1 502 Bad Gateway
Content-Type: text/html
```

Meaning:

> The LB reached the backend, but the backend response was invalid or broken.

---

## 503 Service Unavailable

Client sends:

```http
GET /checkout HTTP/1.1
Host: app.example.com
```

ALB checks its target group:

```text
10.0.1.10 → unhealthy
10.0.1.20 → unhealthy
10.0.2.10 → unhealthy
```

There is nowhere to send the request.

ALB responds directly:

```http
HTTP/1.1 503 Service Unavailable
Content-Type: text/html
```

Meaning:

> I received your request, but no healthy service/backend is currently available.

---

## 504 Gateway Timeout

Client sends:

```http
GET /reports/monthly HTTP/1.1
Host: app.example.com
```

ALB forwards:

```text
ALB → 10.0.1.20:8080
```

Backend receives it and maybe runs:

```sql
SELECT ...
```

But the DB query takes too long.

```text
0 sec  → request sent
10 sec → waiting
30 sec → waiting
60 sec → still no response
```

The load balancer timeout is reached.

ALB sends:

```http
HTTP/1.1 504 Gateway Timeout
Content-Type: text/html
```

Meaning:

> I successfully sent the request to the backend, but the backend did not respond in time.

### Side-by-side

```text
502:
Client → LB → Backend → BAD response/reset
Client ← 502

503:
Client → LB → NO healthy backend
Client ← 503

504:
Client → LB → Backend → ........too slow........
Client ← 504
```

The important difference is **where the request fails**:

> **502 = response problem**  
> **503 = availability problem**  
> **504 = timeout problem**

