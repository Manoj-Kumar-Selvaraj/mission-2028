# Firewall Concepts

## 1. What is a Firewall?

A **firewall filters network traffic** based on defined rules.

It decides:

```text
Allow
Deny
Drop
Reject
```

based on traffic properties.

---

## 2. What Can a Firewall Check?

Common fields:

```text
Source IP
Destination IP
Source Port
Destination Port
Protocol → TCP / UDP / ICMP
Direction → Inbound / Outbound
Connection State
```

Example:

```text
Allow TCP 443 from 0.0.0.0/0
Allow TCP 22 from 10.10.10.0/24
Deny everything else
```

---

## 3. Inbound vs Outbound

### Inbound

Traffic entering a machine/network.

```text
Internet → Web Server:443
```

### Outbound

Traffic leaving a machine/network.

```text
Web Server → Database:5432
```

---

# 4. Stateful Firewall

A **stateful firewall remembers active connections**.

Example:

```text
Client → Server:443
```

If outbound connection is allowed, the firewall remembers it.

Return traffic:

```text
Server → Client
```

is automatically allowed because it belongs to an existing connection.

### Example

```text
Client → TCP 443 → Server
          ALLOWED

Server → Response → Client
          ALLOWED automatically
```

### AWS Example

```text
Security Group = Stateful
```

---

# 5. Stateless Firewall

A **stateless firewall does not remember connections**.

Inbound and outbound traffic are evaluated independently.

If you allow:

```text
Client → Server:443
```

you must also ensure the return traffic is allowed.

### AWS Example

```text
Network ACL = Stateless
```

---

# 6. Stateful vs Stateless

| Feature | Stateful | Stateless |
|---|---|---|
| Tracks connections | Yes | No |
| Return traffic automatically allowed | Yes | No |
| Rules evaluated | Connection-aware | Every packet separately |
| AWS example | Security Group | NACL |

Remember:

```text
SG   → Stateful
NACL → Stateless
```

---

# 7. Allow vs Deny vs Drop vs Reject

### Allow

Traffic is permitted.

### Deny / Drop

Packet is silently discarded.

Client normally sees:

```text
Timeout
```

### Reject

Packet is blocked, but the sender receives an error/reset.

Client gets a faster failure instead of waiting for timeout.

---

# 8. Default-Allow vs Default-Deny

## Default Allow

Everything is allowed unless explicitly blocked.

```text
Allow everything
Deny selected traffic
```

Less secure.

## Default Deny

Everything is blocked unless explicitly allowed.

```text
Deny everything
Allow only required traffic
```

Preferred security model.

> **Allow only what is required.**

---

# 9. Firewall Rule Example

Web server:

```text
Inbound:

TCP 443
Source: 0.0.0.0/0
Action: Allow

TCP 22
Source: 10.10.10.0/24
Action: Allow
```

Meaning:

```text
Anyone → HTTPS allowed
Only admin network → SSH allowed
```

---

# 10. Source and Destination Rules

Example:

```text
Source:
10.0.1.20

Destination:
10.0.2.30

Protocol:
TCP

Port:
5432
```

Means:

> Allow machine `10.0.1.20` to connect to PostgreSQL on `10.0.2.30`.

---

# 11. Host Firewall vs Network Firewall

## Host Firewall

Runs directly on a server.

Examples:

```text
iptables
nftables
ufw
Windows Defender Firewall
```

Protects that specific machine.

---

## Network Firewall

Traffic passes through a dedicated firewall before reaching another network.

```text
Internet
   ↓
Firewall
   ↓
Internal Network
```

Examples:

```text
Palo Alto
Fortinet
Cisco
AWS Network Firewall
```

---

# 12. L3 / L4 Firewall

Filters mainly using:

```text
IP addresses
Protocols
Ports
Connection state
```

Example:

```text
Allow TCP 443 from 10.0.0.0/8
```

---

# 13. Layer 7 Firewall

Understands application-level traffic.

Can inspect things such as:

```text
HTTP path
HTTP method
Headers
Hostnames
Application signatures
```

Example:

```text
Allow:
GET /api/users

Block:
POST /admin/delete
```

---

# 14. WAF vs Traditional Firewall

**WAF = Web Application Firewall**

Traditional firewall:

```text
IP + Port + Protocol
```

WAF:

```text
HTTP/HTTPS content
```

WAF can block:

```text
SQL Injection
Cross-Site Scripting
Malicious HTTP requests
Bad bots
```

AWS example:

```text
AWS WAF
```

---

# 15. Firewall and TCP Handshake

Suppose TCP 443 is blocked.

Client sends:

```text
SYN →
```

If firewall drops it:

```text
SYN →
     X

No SYN-ACK
```

Client eventually gets:

```text
Connection timeout
```

This is useful when troubleshooting.

---

# 16. Firewall vs Routing

Very important:

```text
Routing
→ Where should traffic go?

Firewall
→ Should traffic be allowed?
```

A route can exist but firewall can still block traffic.

Example:

```text
Route = correct
Firewall = deny TCP 443
```

Result:

```text
Connection fails
```

---

# 17. Firewall vs NAT

```text
NAT
→ Changes IP/port information

Firewall
→ Allows or blocks traffic
```

They are different concepts, although the same device may perform both.

---

# 18. Firewall vs Proxy

```text
Firewall
→ Filters traffic

Proxy
→ Terminates/forwards traffic on behalf of another system
```

A proxy may also include firewall-like security capabilities.

---

# 19. Security Group vs NACL — AWS

## Security Group

Attached to:

```text
ENI / EC2 interface
```

Properties:

```text
Stateful
Allow rules only
Return traffic automatically allowed
```

---

## Network ACL

Applied at:

```text
Subnet boundary
```

Properties:

```text
Stateless
Allow + Deny rules
Inbound and outbound evaluated separately
Rule order matters
```

---

# 20. Common Troubleshooting Flow

Application cannot reach:

```text
10.0.2.20:443
```

Check:

```text
1. DNS
2. Routing
3. Source firewall
4. Security Group / NACL
5. Intermediate firewall
6. Destination firewall
7. Is port 443 listening?
```

Useful commands:

```bash
nc -vz 10.0.2.20 443

curl -v https://10.0.2.20

ss -tulpn

iptables -L

nft list ruleset
```

---

# 21. Interview Scenario

### Question

Route is correct, ping works, but port `443` fails.

Possible causes:

```text
Firewall blocking TCP 443
Security Group issue
NACL issue
Application not listening on 443
```

So:

> **Ping working does not mean application ports are open.**

---

# 22. Quick Revision

```text
Firewall → Allow/block traffic

Stateful
→ remembers connections
→ return traffic automatically allowed
→ AWS Security Group

Stateless
→ checks every packet independently
→ inbound + outbound rules required
→ AWS NACL

L3/L4 firewall
→ IP, protocol, port

L7 firewall
→ application-level traffic

WAF
→ protects HTTP/HTTPS applications

Routing
→ where traffic goes

Firewall
→ whether traffic is allowed

NAT
→ changes IP/port information
```