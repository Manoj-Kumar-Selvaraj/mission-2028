# Linux Network Troubleshooting — Complete Interview Guide with Real WSL2 Scenarios

Use a **layered troubleshooting flow**. Do not randomly run commands.

```text
1. Interface / IP
2. Routing
3. Default Gateway
4. Reachability
5. DNS
6. TCP / UDP Port
7. Listening Process
8. Existing Connections
9. Firewall
10. HTTP / Application
11. TLS / Certificate
12. Network Path
13. ARP / Neighbor Table
14. Packet Capture
15. Interface Statistics
```

The core troubleshooting principle is:

> **Use every successful test to eliminate an entire class of possible problems.**

A good mental model is:

```text
Interface
   ↓
IP address
   ↓
Routing
   ↓
Gateway / Next hop
   ↓
DNS
   ↓
TCP / UDP
   ↓
Firewall
   ↓
Application
   ↓
TLS / HTTP
```

---

# 1. Check Network Interface and IP

Use:

```bash
ip addr
```

or:

```bash
ip a
```

Check:

- Is the interface present?
- Is it `UP`?
- Does it have the expected IP address?
- Is the CIDR/subnet correct?
- Does the interface have a MAC address?

Generic example:

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 10.0.1.20/24
```

Interpretation:

```text
Interface = eth0
Status    = UP
IP        = 10.0.1.20
CIDR      = /24
```

If there is no expected IP address, investigate:

```text
DHCP
Static IP configuration
NetworkManager / systemd-networkd
Virtual networking
Interface configuration
```

---

## `ip link`

Use:

```bash
ip link
```

This focuses on interface/link state.

Important values:

```text
UP
→ Interface is administratively enabled.

LOWER_UP
→ Underlying network link is operational.

DOWN
→ Interface is disabled or link unavailable.
```

---

# Real Scenario — WSL2 Interface

Command:

```bash
ip addr
```

Output:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet 10.255.255.254/32 brd 10.255.255.254 scope global lo
    inet6 ::1/128 scope host

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:06:10:7a brd ff:ff:ff:ff:ff:ff
    inet 172.20.211.208/20 brd 172.20.223.255 scope global eth0
    inet6 fe80::215:5dff:fe06:107a/64 scope link
```

Important part:

```text
eth0
IPv4 = 172.20.211.208/20
MAC  = 00:15:5d:06:10:7a
MTU  = 1500
State = UP
```

### `lo`

`lo` is the loopback interface.

```text
127.0.0.1
```

means:

> The machine communicating with itself.

The IPv6 loopback is:

```text
::1
```

---

### `eth0`

This is the main network interface inside WSL.

```text
inet 172.20.211.208/20
```

means:

```text
IPv4 address = 172.20.211.208
CIDR         = /20
```

The interface is:

```text
UP
LOWER_UP
```

Therefore:

```text
Interface exists     ✅
Interface enabled    ✅
Underlying link up   ✅
IPv4 assigned        ✅
```

---

# 2. Check Routing

Use:

```bash
ip route
```

Generic example:

```text
10.0.1.0/24 dev eth0
default via 10.0.1.1 dev eth0
```

Meaning:

```text
10.0.1.0/24
→ Directly connected network.

Everything else
→ Send to 10.0.1.1.
```

The second rule is the **default route**.

---

# Real Scenario — WSL2 Routing Table

Command:

```bash
ip route
```

Output:

```text
default via 172.20.208.1 dev eth0 proto kernel
172.20.208.0/20 dev eth0 proto kernel scope link src 172.20.211.208
```

Interpretation:

```text
Connected network = 172.20.208.0/20
Interface         = eth0
Source IP         = 172.20.211.208
Default Gateway   = 172.20.208.1
```

Meaning:

```text
Destination inside 172.20.208.0/20
→ Send directly through eth0.

Anything else
→ Send to 172.20.208.1.
```

---

# 3. Check Exact Route to a Destination

Use:

```bash
ip route get <destination-ip>
```

This is one of the most useful Linux networking commands.

Generic example:

```bash
ip route get 8.8.8.8
```

Possible output:

```text
8.8.8.8 via 10.0.1.1 dev eth0 src 10.0.1.20
```

This tells you:

```text
Destination
Next hop
Interface
Source IP
```

---

# Real Scenario — Exact Route to 8.8.8.8

Command:

```bash
ip route get 8.8.8.8
```

Output:

```text
8.8.8.8 via 172.20.208.1 dev eth0 src 172.20.211.208 uid 0
    cache
```

Interpretation:

```text
Destination = 8.8.8.8
Next Hop    = 172.20.208.1
Interface   = eth0
Source IP   = 172.20.211.208
```

Packet starts approximately as:

```text
WSL2
172.20.211.208
      ↓
eth0
      ↓
172.20.208.1
WSL/Windows virtual gateway
      ↓
Windows networking
      ↓
Home router
      ↓
ISP
      ↓
Internet
      ↓
8.8.8.8
```

### Interview use

If Server A cannot reach Server B:

```bash
ip route get <server-b-ip>
```

This tells you exactly how Linux plans to send that traffic.

If the selected:

```text
gateway
interface
source IP
```

is wrong, investigate routing before looking at the application.

---

# 4. Check Default Gateway Reachability

Suppose:

```text
Gateway = 10.0.1.1
```

Test:

```bash
ping 10.0.1.1
```

If it fails, possible causes include:

```text
Wrong gateway
Interface issue
Local network issue
VLAN issue
Layer-2 issue
ICMP blocked
Virtual gateway not answering ICMP
```

Important:

> **A gateway does not have to reply to ICMP Echo Requests in order to successfully forward traffic.**

---

# Real Scenario — Gateway Ping Fails but Internet Works

Command:

```bash
ping 172.20.208.1
```

Result:

```text
14 packets transmitted, 0 received, 100% packet loss
```

At first, this looks like:

```text
Gateway unreachable ❌
```

But then:

```bash
ping 8.8.8.8
```

returned:

```text
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=19.9 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=12.9 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=20.6 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=112 time=14.6 ms

4 packets transmitted, 4 received, 0% packet loss
```

Therefore:

```text
Gateway ICMP response      ❌
Gateway packet forwarding  ✅
External connectivity      ✅
```

### Why?

The WSL virtual gateway can forward packets without replying to ICMP Echo Requests directed at its own address.

So:

> **Ping failure alone does not prove that a router or gateway is broken.**

This is an excellent interview scenario.

---

# 5. Check Destination Reachability

Use:

```bash
ping 8.8.8.8
```

Ping uses:

```text
ICMP
```

If it works, it gives evidence that:

```text
Interface
Routing
Gateway forwarding
External Layer-3 connectivity
```

are likely functioning.

But:

> **Ping success does not prove that TCP port 443, 22, 5432, etc. is reachable.**

Example:

```text
ICMP allowed ✅
TCP 443 blocked ❌
```

Similarly:

> **Ping failure does not prove that the destination is down.**

Many devices intentionally block ICMP.

---

# 6. DNS Troubleshooting

DNS converts:

```text
hostname → IP address
```

Example:

```text
google.com → 142.251.x.x
```

Useful tools:

```bash
dig
nslookup
host
```

---

# 7. `dig` — Detailed DNS Troubleshooting

Use:

```bash
dig google.com
```

Real output:

```text
; <<>> DiG 9.18.39-0ubuntu0.22.04.4-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2961
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             233     IN      A       142.251.221.206

;; Query time: 8 msec
;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
;; WHEN: Thu Aug 20 21:14:03 IST 2026
;; MSG SIZE  rcvd: 55
```

---

## `status: NOERROR`

```text
status: NOERROR
```

means:

> DNS lookup completed successfully.

Other common statuses include:

```text
NXDOMAIN
SERVFAIL
REFUSED
```

---

## Question Section

```text
google.com. IN A
```

means:

```text
Domain      = google.com
Record type = A
```

`A` means:

```text
IPv4 address
```

---

## Answer Section

```text
google.com. 233 IN A 142.251.221.206
```

means:

```text
google.com
   ↓
142.251.221.206
```

DNS successfully returned an IPv4 address.

---

## TTL

```text
233
```

is the DNS TTL.

```text
TTL = 233 seconds
```

The answer can be cached for approximately:

```text
3 minutes 53 seconds
```

before that TTL expires.

---

## DNS Resolver

```text
SERVER: 10.255.255.254#53
```

means:

```text
DNS Resolver = 10.255.255.254
Port         = 53
```

The output also says:

```text
(UDP)
```

Therefore this query used:

```text
UDP port 53
```

---

## Query Time

```text
Query time: 8 msec
```

means DNS resolution completed in:

```text
8 milliseconds
```

---

## DNS Flags

```text
qr rd ra
```

means:

```text
qr = Query Response
rd = Recursion Desired
ra = Recursion Available
```

Conceptually:

```text
WSL:
"Resolver, please find the final answer for google.com."

        ↓

Recursive DNS resolver

        ↓

Answer returned
```

---

## Query Counts

```text
QUERY:      1
ANSWER:     1
AUTHORITY:  0
ADDITIONAL: 1
```

means:

```text
1 DNS question
1 answer
0 authority records in this response
1 additional metadata section
```

---

# 8. `nslookup` — Successful Resolution

Command:

```bash
nslookup google.com
```

Real output:

```text
Server:         10.255.255.254
Address:        10.255.255.254#53

Non-authoritative answer:
Name:   google.com
Address: 142.251.221.206
Name:   google.com
Address: 2404:6800:4007:831::200e
```

---

## DNS Server

```text
Server: 10.255.255.254
Address: 10.255.255.254#53
```

means:

```text
Configured DNS resolver = 10.255.255.254
DNS port                = 53
```

This matches the resolver shown by `dig`.

---

## Non-authoritative Answer

```text
Non-authoritative answer:
```

means:

> The DNS server replying to you is not Google's authoritative DNS server.

It is a recursive/caching resolver giving you the answer it retrieved or cached.

Conceptually:

```text
Your WSL
   ↓
10.255.255.254
Recursive Resolver
   ↓
DNS hierarchy / cache
   ↓
Google authoritative DNS
```

The resolver returns the final answer to you.

---

## IPv4 Result

```text
Address: 142.251.221.206
```

is the:

```text
A record → IPv4
```

---

## IPv6 Result

```text
Address: 2404:6800:4007:831::200e
```

is the:

```text
AAAA record → IPv6
```

So this output proves:

```text
DNS resolver reachable  ✅
IPv4 lookup              ✅
IPv6 lookup              ✅
google.com exists        ✅
```

---

# 9. `nslookup` — NXDOMAIN Example

Command:

```bash
nslookup applie-manoj.com
```

Output:

```text
Server:         10.255.255.254
Address:        10.255.255.254#53

** server can't find applie-manoj.com: NXDOMAIN
```

`NXDOMAIN` means:

```text
Non-Existent Domain
```

The DNS system is effectively saying:

> I successfully processed your DNS request, but that domain name does not exist in DNS.

This is very different from:

```text
DNS timeout
Network failure
Resolver unreachable
```

So:

```text
DNS communication ✅
DNS query processed ✅
Requested domain ❌ does not exist
```

### Interview takeaway

If you see:

```text
NXDOMAIN
```

do **not** immediately troubleshoot TCP, firewall, HTTP, or application ports.

The problem happens earlier:

```text
Hostname cannot be resolved.
```

Possible causes:

```text
Typo in hostname
DNS record not created
Wrong DNS zone
Record deleted
Wrong search domain
```

---

# 10. Another NXDOMAIN Example

Command:

```bash
nslookup manoj-tech-solution.com
```

Output:

```text
Server:         10.255.255.254
Address:        10.255.255.254#53

** server can't find manoj-tech-solution.com: NXDOMAIN
```

Again:

```text
Resolver working ✅
Domain record    ❌
```

The DNS resolver was reachable and returned a valid DNS error response.

---

# 11. Important — `nslookup` Takes a Hostname, Not a URL

This command was attempted:

```bash
nslookup http://manoj-tech-solution.com/
```

That is not the correct input format.

`nslookup` expects:

```text
hostname
```

not:

```text
URL
```

Correct:

```bash
nslookup manoj-tech-solution.com
```

Incorrect:

```bash
nslookup http://manoj-tech-solution.com/
```

Why?

A URL contains multiple parts:

```text
http://manoj-tech-solution.com/
│      │
│      └── hostname
└── scheme/protocol
```

DNS only resolves:

```text
manoj-tech-solution.com
```

DNS does not resolve:

```text
http://
https://
/path
:port
```

---

# 12. DNS Timeout vs NXDOMAIN

These mean very different things.

## NXDOMAIN

```text
server can't find example.com: NXDOMAIN
```

means:

```text
DNS server reachable       ✅
Query processed             ✅
Domain does not exist       ❌
```

---

## Timeout

Example:

```text
communications error to 10.255.255.254#53: timed out
```

means:

> A DNS request was sent but no valid response arrived before the timeout.

Possible causes include:

```text
Temporary resolver issue
Packet loss
DNS server slow/unavailable
Firewall
Malformed/unusual query
Network issue
```

One timeout alone is not enough to conclude the DNS server is completely unavailable.

Always retry with a valid hostname.

---

# 13. Check `/etc/resolv.conf`

Use:

```bash
cat /etc/resolv.conf
```

Typical output:

```text
nameserver 10.255.255.254
```

This tells Linux which resolver it should normally use.

Modern Linux systems may also use:

```bash
resolvectl status
```

---

# 14. Test a Specific Resolver

Use:

```bash
dig @8.8.8.8 google.com
```

This bypasses the normally configured DNS resolver and asks:

```text
8.8.8.8
```

directly.

If:

```bash
dig google.com
```

fails but:

```bash
dig @8.8.8.8 google.com
```

works, suspect:

```text
Configured DNS resolver
/etc/resolv.conf
systemd-resolved
VPN DNS
Corporate DNS
Local DNS forwarding
```

---

# 15. `host` Command

Command:

```bash
host google.com
```

Real output:

```text
google.com has address 142.251.221.206
google.com has IPv6 address 2404:6800:4007:831::200e
google.com mail is handled by 10 smtp.google.com.
```

`host` is a simple DNS lookup utility.

---

## IPv4

```text
google.com has address 142.251.221.206
```

means:

```text
A record
google.com → 142.251.221.206
```

---

## IPv6

```text
google.com has IPv6 address 2404:6800:4007:831::200e
```

means:

```text
AAAA record
google.com → IPv6 address
```

---

## MX / Mail Server

```text
google.com mail is handled by 10 smtp.google.com.
```

This is an:

```text
MX record
```

The value:

```text
10
```

is the MX priority/preference.

Generally, a lower MX preference value is preferred over a higher one.

So `host google.com` quickly showed:

```text
A     → IPv4
AAAA  → IPv6
MX    → Mail server
```

---

# 16. DNS Troubleshooting Decision Flow

```text
Can I ping an external IP such as 8.8.8.8?
        │
        ├── NO → investigate interface/routing/connectivity
        │
        └── YES
              ↓
Can I resolve google.com?
              │
              ├── NO
              │    ↓
              │ Check dig/nslookup/resolv.conf
              │
              └── YES
                   ↓
DNS is probably not the issue.
Continue to TCP / TLS / HTTP.
```

---

# 17. Check TCP Port Connectivity

Suppose an application needs:

```text
Database = 10.0.2.20
Port     = 5432
```

Use:

```bash
nc -vz 10.0.2.20 5432
```

For HTTPS:

```bash
nc -vz google.com 443
```

This checks:

> Can a TCP connection be established to the destination port?

---

## Successful Connection

Example:

```text
Connection to google.com 443 port [tcp/https] succeeded!
```

Evidence:

```text
DNS resolution   ✅
Routing          ✅
Destination path ✅
TCP handshake    ✅
Port 443         ✅
```

It does **not** yet prove that TLS or HTTP works.

---

## Connection Refused

```text
Connection refused
```

Usually means:

> Network path reached the destination, but the connection was actively rejected.

Likely causes:

```text
Application not running
Nothing listening on the port
Wrong port
Service bound only to localhost
Firewall configured to REJECT
```

---

## Connection Timeout

```text
Connection timed out
```

Think more about:

```text
Firewall DROP
Security Group
NACL
Routing
Server unreachable
Return path
Packet loss
```

Easy memory:

```text
REFUSED
→ I reached something, but it rejected me.

TIMEOUT
→ My request or its response disappeared somewhere.
```

---

# 18. Check Whether the Application Is Listening

Use:

```bash
ss -tulpn
```

Options:

```text
t → TCP
u → UDP
l → Listening
p → Process
n → Numeric addresses/ports
```

Example:

```text
LISTEN 0 128 0.0.0.0:8080
```

means:

> Service is listening on TCP/8080 on all IPv4 interfaces.

---

## Localhost-Only Binding

Example:

```text
127.0.0.1:8080
```

means:

> The application is accepting connections only from the same machine.

Remote systems cannot connect to it through the server's network IP.

It may need to bind to:

```text
0.0.0.0:8080
```

or to a specific private/interface IP.

---

# 19. Check Which Process Owns a Port

Use:

```bash
ss -ltnp
```

Example:

```text
LISTEN 0 128 0.0.0.0:8080 users:(("java",pid=1234))
```

Interpretation:

```text
Port    = 8080
Process = java
PID     = 1234
```

Alternative:

```bash
sudo lsof -i :8080
```

---

# 20. Check Existing TCP Connections

Use:

```bash
ss -tan
```

Common states:

```text
LISTEN
SYN-SENT
SYN-RECV
ESTABLISHED
FIN-WAIT
TIME-WAIT
CLOSE-WAIT
```

---

## ESTABLISHED

```text
ESTABLISHED
```

means:

> TCP connection is active.

---

## SYN-SENT

```text
SYN-SENT
```

means:

> Client sent SYN but the TCP handshake has not completed.

Possible causes:

```text
Firewall
Routing
Server unavailable
Return-path issue
```

---

## CLOSE-WAIT

```text
CLOSE-WAIT
```

means:

> Remote side closed the connection, but the local application has not finished closing its socket.

Large numbers of `CLOSE-WAIT` may indicate:

```text
Application socket leak
Application not closing connections correctly
```

---

## TIME-WAIT

`TIME-WAIT` is normally seen after TCP connections close.

Some `TIME-WAIT` is normal TCP behavior.

---

# 21. Check HTTP / HTTPS

Use:

```bash
curl -v http://server:8080
```

For HTTPS:

```bash
curl -v https://google.com
```

`-v` can show:

```text
DNS resolution
Selected destination IP
TCP connection
TLS negotiation
HTTP request
HTTP response
Headers
```

This is much more useful than ping for application troubleshooting.

---

## Example Layer Interpretation

If `curl -v` reaches:

```text
Connected to google.com ...
```

then:

```text
DNS        ✅
Routing    ✅
TCP        ✅
```

If it fails during certificate negotiation:

```text
TLS        ❌
```

If TLS succeeds but the server returns:

```text
HTTP 500
```

then the network is mostly working and the issue is likely at the application layer.

---

# 22. Check Only HTTP Headers

Use:

```bash
curl -I https://example.com
```

Example:

```text
HTTP/2 200
content-type: text/html
server: nginx
```

Useful for:

```text
Health check
HTTP status
Redirect
Headers
Server response
```

---

# 23. Check TLS / Certificate

If TCP/443 works but HTTPS fails:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Check:

```text
Certificate
Expiry date
Issuer
Certificate chain
Hostname
TLS version
TLS handshake
```

Possible problems:

```text
Expired certificate
Hostname mismatch
Incomplete certificate chain
Unsupported TLS version
Certificate verification failure
```

---

# 24. Trace the Network Path

Use:

```bash
traceroute google.com
```

Real output:

```text
traceroute to google.com (142.251.221.206), 30 hops max, 60 byte packets

1  LAPTOP-HOO65F3F.mshome.net (172.20.208.1)  7.450 ms  7.440 ms  0.320 ms
2  192.168.0.1 (192.168.0.1)  15.012 ms  14.857 ms  14.735 ms
3  * * *
4  * * *
5  * * *
6  * * *
7  * * *
8  49.205.72.39.actcorp.in (49.205.72.39)  28.356 ms  19.030 ms  34.172 ms
9  72.14.243.242 (72.14.243.242)  10.863 ms  7.731 ms  7.899 ms
10 * * *
11 74.125.253.12 (74.125.253.12) 24.547 ms
   142.251.55.216 (142.251.55.216) 16.808 ms
   74.125.253.12 (74.125.253.12) 24.132 ms
12 142.251.55.245 (142.251.55.245) 17.333 ms
   142.251.55.245 (142.251.55.245) 17.069 ms
   142.250.239.228 (142.250.239.228) 16.287 ms
13 pnmaaa-ba-in-f14.1e100.net (142.251.221.206) 14.717 ms
   142.251.230.53 (142.251.230.53) 17.134 ms
   pnmaaa-ba-in-f14.1e100.net (142.251.221.206) 15.058 ms
```

---

# 25. Understanding `traceroute`

Traceroute shows the routers/hops between your machine and the destination.

Conceptually:

```text
Your machine
   ↓
Router 1
   ↓
Router 2
   ↓
ISP routers
   ↓
Destination network
   ↓
Destination
```

---

## Hop 1

```text
1 LAPTOP-HOO65F3F.mshome.net (172.20.208.1)
```

This is your WSL/Windows virtual gateway.

Earlier:

```bash
ping 172.20.208.1
```

did not receive replies.

Yet traceroute shows the gateway as the first hop.

This reinforces an important lesson:

> A device may not answer normal ICMP ping but may still participate in routing and traceroute.

---

## Hop 2

```text
2 192.168.0.1
```

This is likely the LAN/home router.

So the path starts roughly:

```text
WSL
172.20.211.208
      ↓
WSL/Windows Gateway
172.20.208.1
      ↓
Home Router
192.168.0.1
```

---

## Hops with `* * *`

Example:

```text
3 * * *
4 * * *
```

This does **not automatically mean packet loss or broken routing**.

It usually means that the router at that hop:

```text
Did not reply to the traceroute probe
Blocks TTL-expired ICMP responses
Rate-limits traceroute
Is configured not to expose itself
```

Since later hops and the final destination respond, traffic clearly continued through the network.

Important:

> **Traceroute stars are not automatically failures.**

---

## Hop 8 — ISP Network

```text
49.205.72.39.actcorp.in
```

This indicates traffic is traversing the ACT ISP network.

So roughly:

```text
Your device
   ↓
Home router
   ↓
ACT ISP
```

---

## Hop 9 and Later — Google Network

```text
72.14.243.242
74.125.x.x
142.251.x.x
```

These addresses are part of the path through Google's network infrastructure.

The route has moved from:

```text
Local network
   ↓
ISP
   ↓
Google network
```

---

## Multiple IPs on the Same Hop

For example:

```text
11 74.125.253.12
   142.251.55.216
   74.125.253.12
```

Traceroute normally sends multiple probes per hop.

Different probes can sometimes take different equal-cost paths.

This can occur because of:

```text
ECMP
Load balancing
Routing differences
Different network interfaces/routers replying
```

So seeing multiple IPs for one hop is not necessarily a problem.

---

## Final Hop

```text
pnmaaa-ba-in-f14.1e100.net (142.251.221.206)
```

This is the destination IP returned earlier by DNS:

```text
google.com → 142.251.221.206
```

Therefore the complete test ties together:

```text
DNS returned 142.251.221.206
        ↓
Routing selected gateway
        ↓
Traceroute followed routers
        ↓
Reached 142.251.221.206
```

---

# 26. Why Traceroute Works

Traceroute manipulates packet TTL values.

Conceptually:

```text
Probe 1 → TTL 1
Router 1 decrements TTL to 0
Router 1 responds
```

Then:

```text
Probe 2 → TTL 2
Router 1 forwards
Router 2 decrements TTL to 0
Router 2 responds
```

Then:

```text
Probe 3 → TTL 3
...
```

By increasing TTL one hop at a time, traceroute discovers the path.

---

# 27. `traceroute` Not Installed

Initial output:

```text
Command 'traceroute' not found
```

Ubuntu suggested:

```bash
apt install traceroute
```

After:

```bash
apt install traceroute
```

the command became available.

This is a useful Linux administration reminder:

```text
Command missing
→ Identify package
→ Install package
→ Retry
```

---

# 28. Check ARP / Neighbor Table

Use:

```bash
ip neigh
```

Example:

```text
10.0.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

This maps:

```text
Local IP → MAC address
```

Useful neighbor states:

```text
REACHABLE
STALE
DELAY
FAILED
INCOMPLETE
```

If a local destination/gateway is:

```text
FAILED
INCOMPLETE
```

suspect a local Layer-2/neighbor-resolution problem.

---

# 29. Check Firewall

## iptables

```bash
sudo iptables -L -n -v
```

Look for:

```text
ACCEPT
DROP
REJECT
```

---

## nftables

```bash
sudo nft list ruleset
```

---

## UFW

```bash
sudo ufw status
```

Check rules involving:

```text
Source IP
Destination IP
Protocol
Source port
Destination port
```

Example:

```text
DROP tcp -- 10.0.1.0/24 0.0.0.0/0 tcp dpt:443
```

means:

> TCP/443 traffic matching that rule is dropped.

---

# 30. Packet Capture with `tcpdump`

When higher-level tests are not enough, inspect actual packets.

Basic:

```bash
sudo tcpdump -i eth0
```

Filter by host:

```bash
sudo tcpdump -i eth0 host 10.0.2.20
```

Filter by port:

```bash
sudo tcpdump -i eth0 port 443
```

TCP only:

```bash
sudo tcpdump -i eth0 'tcp port 443'
```

---

## Scenario — Repeated SYN, No SYN-ACK

You see:

```text
SYN →
SYN →
SYN →
```

but no:

```text
SYN-ACK
```

Meaning:

```text
Client created traffic ✅
Client transmitted SYN  ✅
Response received       ❌
```

Possible causes:

```text
Firewall DROP
Security Group
NACL
Routing
Return path
Destination unavailable
```

---

## Scenario — RST Returned

```text
SYN →
← RST
```

Meaning:

> Destination is reachable, but the TCP connection was actively rejected/reset.

Often:

```text
Nothing listening on port
Application rejected connection
Firewall REJECT
```

---

# 31. Check Interface Statistics

Use:

```bash
ip -s link
```

Shows:

```text
RX packets
TX packets
RX errors
TX errors
Dropped packets
```

Example:

```text
RX errors: 5000
Dropped: 2000
```

Possible causes:

```text
Driver issue
Interface issue
Congestion
Buffer pressure
Physical/link issue
```

Also:

```bash
ss -s
```

or:

```bash
netstat -s
```

can provide transport/network statistics.

---

# 32. Check Physical Interface Speed / Link

For physical Linux servers:

```bash
ethtool eth0
```

Example:

```text
Speed: 1000Mb/s
Duplex: Full
Link detected: yes
```

If:

```text
Link detected: no
```

investigate:

```text
Cable
NIC
Switch port
Physical connectivity
```

---

# 33. Scenario — Website Not Working

Suppose:

```text
https://app.company.com
```

fails.

Troubleshoot:

```text
1. dig app.company.com
        ↓
Does DNS resolve?

2. ip route get <resolved-ip>
        ↓
Is route correct?

3. ping <resolved-ip>
        ↓
Does ICMP/basic IP connectivity work?

4. nc -vz <resolved-ip> 443
        ↓
Can TCP/443 connect?

5. curl -v https://app.company.com
        ↓
Does TLS/HTTP work?

6. openssl s_client ...
        ↓
Certificate/TLS issue?

7. Server: ss -ltnp
        ↓
Is application listening?

8. Firewall / SG / NACL
        ↓
Is traffic permitted?

9. tcpdump
        ↓
Where do packets stop?
```

---

# 34. Scenario — App Server Cannot Reach Database

Application:

```text
10.0.1.20
```

Database:

```text
10.0.2.30:5432
```

First:

```bash
ip route get 10.0.2.30
```

Then:

```bash
nc -vz 10.0.2.30 5432
```

If timeout:

```text
Check routing
Security Group
NACL
Host firewall
Return path
```

On DB:

```bash
ss -ltnp | grep 5432
```

If it shows:

```text
127.0.0.1:5432
```

PostgreSQL is listening only locally.

It may need:

```text
0.0.0.0:5432
```

or the correct private interface address.

---

# 35. Scenario — IP Works but Hostname Doesn't

This works:

```bash
curl http://10.0.1.20
```

But:

```bash
curl http://app.internal
```

fails.

Likely:

```text
DNS issue
```

Check:

```bash
dig app.internal
cat /etc/resolv.conf
```

Interpretation:

```text
IP connectivity ✅
Name resolution ❌
```

---

# 36. Scenario — Ping Works but Port Doesn't

This works:

```bash
ping 10.0.2.20
```

But:

```bash
nc -vz 10.0.2.20 443
```

times out.

Interpretation:

```text
Layer 3 / ICMP → likely working
Layer 4 / TCP  → failing
```

Check:

```text
Firewall
Security Group
NACL
Service listening
Port configuration
Return path
```

---

# 37. Scenario — Connection Refused

```bash
curl http://10.0.2.20:8080
```

returns:

```text
Connection refused
```

Likely meaning:

> Network path reached the destination, but the destination rejected the connection.

Possible:

```text
App not running
Wrong port
Service bound to localhost
Firewall REJECT
```

Check:

```bash
ss -ltnp | grep 8080
```

---

# 38. Scenario — Connection Timeout

```text
Connection timed out
```

Think:

```text
Firewall DROP
Routing issue
Security Group
NACL
Destination unavailable
Return path
```

Memory:

```text
REFUSED
→ Destination was reached but rejected connection.

TIMEOUT
→ Traffic or response disappeared somewhere.
```

---

# 39. Scenario — DNS Works but Website Doesn't

Suppose:

```bash
dig app.company.com
```

returns the expected IP.

But:

```bash
curl https://app.company.com
```

fails.

DNS has done its job.

Continue with:

```bash
ip route get <resolved-ip>
nc -vz <resolved-ip> 443
curl -v https://app.company.com
```

Potential issue is now:

```text
Routing
Firewall
TCP
TLS
Load balancer
Application
```

---

# 40. Scenario — `nc` Works but HTTPS Doesn't

Suppose:

```bash
nc -vz app.company.com 443
```

succeeds.

But:

```bash
curl https://app.company.com
```

fails.

Interpretation:

```text
DNS            ✅
Routing        ✅
TCP connection ✅
Port 443       ✅
TLS / HTTP     ❌
```

Check:

```bash
openssl s_client \
  -connect app.company.com:443 \
  -servername app.company.com
```

Possible causes:

```text
Expired certificate
Hostname mismatch
Broken certificate chain
TLS mismatch
Proxy/application issue
```

---

# 41. Scenario — Application Listening Only on Localhost

Server:

```bash
ss -ltnp
```

shows:

```text
127.0.0.1:8080
```

Local:

```bash
curl http://127.0.0.1:8080
```

works.

Remote:

```bash
nc -vz <server-ip> 8080
```

fails.

Reason:

> Service is bound only to loopback.

---

# 42. Important Commands to Memorize

## Interface

```bash
ip a
```

> IP/interface information.

```bash
ip link
```

> Interface/link state.

---

## Routing

```bash
ip route
```

> Routing table.

```bash
ip route get <IP>
```

> Exact route, next hop, source IP and interface Linux will use.

---

## Reachability

```bash
ping <IP>
```

> ICMP/basic IP reachability.

---

## DNS

```bash
dig <hostname>
```

> Detailed DNS response.

```bash
nslookup <hostname>
```

> Simple resolver lookup.

```bash
host <hostname>
```

> Quick A/AAAA/MX-style lookup.

```bash
dig @8.8.8.8 <hostname>
```

> Ask a specific DNS resolver.

```bash
cat /etc/resolv.conf
```

> View configured DNS resolver.

---

## TCP Port

```bash
nc -vz <host> <port>
```

> Test TCP connectivity to a port.

---

## Listening Services

```bash
ss -tulpn
```

> Listening TCP/UDP ports and processes.

```bash
lsof -i :<port>
```

> Process using a specific port.

---

## Connections

```bash
ss -tan
```

> TCP connections and states.

---

## HTTP / HTTPS

```bash
curl -v <URL>
```

> DNS + TCP + TLS + HTTP troubleshooting.

```bash
curl -I <URL>
```

> HTTP headers/status only.

---

## TLS

```bash
openssl s_client -connect <host>:443 -servername <host>
```

> Certificate/TLS troubleshooting.

---

## Path

```bash
traceroute <host>
```

> Network hops/path.

```bash
tracepath <host>
```

> Alternative path/MSS/MTU-related troubleshooting tool.

---

## Neighbor / ARP

```bash
ip neigh
```

> Local IP-to-MAC neighbor table.

---

## Firewall

```bash
iptables -L -n -v
```

```bash
nft list ruleset
```

```bash
ufw status
```

---

## Packet Capture

```bash
tcpdump
```

> Actual packet inspection.

---

## Statistics

```bash
ip -s link
```

> RX/TX packets, drops and errors.

---

# 43. Real WSL2 Troubleshooting Summary

From the real tests:

```text
Interface eth0              ✅
IPv4 172.20.211.208/20      ✅
Interface UP                ✅

Connected route             ✅
Default route               ✅
Next hop 172.20.208.1       ✅

Gateway ping                ❌
Gateway forwarding          ✅

Internet ping 8.8.8.8       ✅

DNS resolver 10.255.255.254 ✅
UDP/53 DNS query            ✅
google.com A record         ✅
google.com AAAA record      ✅
google.com MX record        ✅

Invalid/nonexistent domains
→ NXDOMAIN                  ✅ valid DNS response

Traceroute path             ✅
WSL gateway                 ✅
Home router                 ✅
ISP                         ✅
Google network              ✅
Final destination           ✅
```

---

# 44. Complete Real Packet Path

Based on the actual outputs:

```text
WSL2
172.20.211.208
      ↓
eth0
      ↓
172.20.208.1
WSL / Windows virtual gateway
      ↓
192.168.0.1
Home router
      ↓
ACT ISP
      ↓
Google network
      ↓
142.251.221.206
google.com
```

DNS path:

```text
WSL2
   ↓ UDP/53
10.255.255.254
Recursive DNS resolver
   ↓
DNS hierarchy/cache
   ↓
google.com records
   ↓
A    → 142.251.221.206
AAAA → 2404:6800:4007:831::200e
MX   → smtp.google.com
```

---

# 45. Interview Troubleshooting Framework

If asked:

> **Server A cannot connect to Server B. How will you troubleshoot?**

Answer:

```text
1. Verify source interface and IP.
2. Verify destination IP.
3. Check route using ip route get.
4. Verify DNS if a hostname is used.
5. Test basic reachability.
6. Test the required TCP/UDP port.
7. Check whether the destination service is listening.
8. Check source/destination host firewalls.
9. Check SG/NACL/network firewall if cloud.
10. Verify the return path.
11. Use traceroute when path troubleshooting is needed.
12. Use tcpdump for packet-level confirmation.
```

---

# 46. Best Interview Mental Model

```text
Does the interface work?
        ↓
Do I have the correct IP?
        ↓
Is the route correct?
        ↓
Can I reach the destination network?
        ↓
Does DNS resolve the hostname?
        ↓
Can TCP/UDP reach the required port?
        ↓
Is the application listening?
        ↓
Does firewall allow it?
        ↓
Does TLS work?
        ↓
Does HTTP/application respond?
        ↓
Still unclear?
        ↓
Use tcpdump.
```

---

# Final Memory Lines

> **DNS tells me the destination IP → routing tells me where to send the packet → firewall decides whether traffic is allowed → TCP/UDP reaches the service → the application must actually be listening.**

> **Ping tests ICMP, not the entire network stack. A failed ping does not automatically mean routing is broken.**

> **NXDOMAIN means DNS worked but the requested domain does not exist. A timeout means the resolver did not return a response in time.**

> **`nslookup` resolves hostnames, not full URLs. Use `manoj-tech-solution.com`, not `http://manoj-tech-solution.com/`.**

> **Traceroute `* * *` does not automatically mean failure; intermediate routers may simply refuse or rate-limit traceroute responses.**

> **Never troubleshoot networking with only one command. Move layer by layer and use every successful test to narrow the failure domain.**
