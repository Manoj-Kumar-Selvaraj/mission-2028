## Linux Network Troubleshooting — Detailed Interview Guide

Use a **layered troubleshooting flow**. Don’t randomly run commands.

```text
1. Interface/IP
2. Routing
3. DNS
4. Reachability
5. Port
6. Listening process
7. Firewall
8. Application/TLS
```

---

# 1. Check Network Interface and IP

```bash
ip addr
```

or shorter:

```bash
ip a
```

You are checking:

- Is the interface UP?
- Does it have the expected IP?
- Is the subnet correct?

Example:

```text
eth0: UP
inet 10.0.1.20/24
```

If there is **no IP**, investigate DHCP/static configuration.

Also:

```bash
ip link
```

Shows interface state.

```text
UP   → interface enabled
DOWN → interface disabled
```

---

# 2. Check Routing

```bash
ip route
```

Example:

```text
10.0.1.0/24 dev eth0
default via 10.0.1.1 dev eth0
```

Meaning:

```text
10.0.1.x → directly reachable
Everything else → 10.0.1.1
```

Very useful command:

```bash
ip route get 8.8.8.8
```

Example output:

```text
8.8.8.8 via 10.0.1.1 dev eth0 src 10.0.1.20
```

This tells you:

> Which interface, gateway and source IP Linux will use.

### Interview scenario

Server cannot reach another network.

Check:

```bash
ip route get <destination-ip>
```

If the route is wrong, the application cannot fix it.

---

# 3. Check Default Gateway Reachability

Suppose:

```text
Gateway = 10.0.1.1
```

Try:

```bash
ping 10.0.1.1
```

If the gateway itself isn't reachable, likely problems include:

- Interface issue
- VLAN/subnet issue
- Local network issue
- Wrong gateway
- Layer-2 issue

---

# 4. Check Destination Reachability

```bash
ping 8.8.8.8
```

This tests basic IP connectivity using **ICMP**.

If this works:

```text
Local interface → routing → external IP connectivity
```

are probably functioning.

But remember:

> **Ping success does NOT mean TCP port 443 is reachable.**

A firewall may allow ICMP and block TCP.

Also, some servers intentionally block ping, so **ping failure alone does not prove the server is down**.

---

# 5. Check DNS

Suppose:

```bash
ping 8.8.8.8
```

works, but:

```bash
curl https://google.com
```

fails with:

```text
Could not resolve host
```

Then suspect DNS.

Commands:

```bash
dig google.com
```

```bash
nslookup google.com
```

Check configured resolver:

```bash
cat /etc/resolv.conf
```

Example:

```text
nameserver 10.0.0.2
```

Modern systems:

```bash
resolvectl status
```

Test another resolver:

```bash
dig @8.8.8.8 google.com
```

If this works while normal `dig google.com` fails:

> Your configured DNS resolver/configuration is likely the problem.

---

# 6. Check TCP/UDP Port Connectivity

This is extremely important.

Suppose application needs:

```text
Database = 10.0.2.20
Port = 5432
```

Test:

```bash
nc -vz 10.0.2.20 5432
```

Possible result:

```text
Connection succeeded
```

Means:

> TCP connection to that port is possible.

### Connection refused

```text
Connection refused
```

Usually means:

> Destination is reachable, but nothing is listening on that port, or the destination actively rejected it.

### Timeout

```text
Connection timed out
```

Often indicates:

- Firewall
- Security Group
- NACL
- Routing issue
- Packet drop
- Return-path issue

This distinction is very useful in interviews.

---

# 7. Check Whether Application Is Listening

On the destination server:

```bash
ss -tulpn
```

Important options:

```text
t → TCP
u → UDP
l → listening
p → process
n → numerical ports/IPs
```

Example:

```text
LISTEN 0 128 0.0.0.0:8080
```

Means service is listening on port `8080` on all IPv4 interfaces.

Another example:

```text
127.0.0.1:8080
```

This is important.

It means:

> Application is listening only on localhost.

Remote machines **cannot connect**.

You may need:

```text
0.0.0.0:8080
```

or the server's actual interface IP.

---

# 8. Check Existing Connections

```bash
ss -tan
```

You might see:

```text
ESTABLISHED
SYN-SENT
SYN-RECV
TIME-WAIT
CLOSE-WAIT
```

### Useful interpretation

```text
ESTABLISHED
→ TCP connection working
```

```text
SYN-SENT
→ Client sent SYN but hasn't completed handshake
```

Possible:

- Firewall
- Routing
- Server unavailable
- Return-path problem

```text
CLOSE-WAIT
→ Remote side closed; local application hasn't closed socket yet
```

Lots of `CLOSE-WAIT` may indicate an application/socket handling issue.

---

# 9. Check HTTP Application

Use:

```bash
curl -v http://server:8080
```

For HTTPS:

```bash
curl -v https://example.com
```

`-v` shows useful stages:

```text
DNS resolution
Connection
TCP establishment
TLS
HTTP headers
HTTP response
```

Example:

```text
Connected to example.com
TLS handshake...
HTTP/1.1 200 OK
```

This tells you much more than `ping`.

---

# 10. Check Only HTTP Headers

```bash
curl -I https://example.com
```

Example:

```text
HTTP/2 200
content-type: text/html
```

Useful for quick web health checks.

---

# 11. Check TLS / Certificate

If TCP 443 works but HTTPS fails, investigate TLS.

```bash
openssl s_client -connect example.com:443
```

For hostname/SNI:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Check:

- Certificate
- Expiry
- Issuer
- TLS handshake
- Certificate chain

Example problems:

```text
certificate expired
hostname mismatch
unable to verify certificate
```

---

# 12. Trace the Network Path

```bash
traceroute 8.8.8.8
```

Shows intermediate network hops.

Conceptually:

```text
Server
 ↓
Gateway
 ↓
Router 2
 ↓
ISP
 ↓
Destination
```

If it stops at a specific hop, that gives a clue where connectivity may be failing.

On some systems:

```bash
tracepath 8.8.8.8
```

can also be useful.

---

# 13. Check ARP / Neighbor Table

You don't need deep ARP knowledge, but know this command:

```bash
ip neigh
```

Example:

```text
10.0.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

This tells Linux which MAC is associated with a local IP.

States like:

```text
REACHABLE
STALE
FAILED
INCOMPLETE
```

can help diagnose local network problems.

---

# 14. Check Firewall

For traditional iptables:

```bash
sudo iptables -L -n -v
```

For nftables:

```bash
sudo nft list ruleset
```

For Ubuntu UFW:

```bash
sudo ufw status
```

You're looking for rules blocking:

```text
Source IP
Destination IP
Protocol
Port
```

Example:

```text
DROP tcp -- 10.0.1.0/24 0.0.0.0/0 tcp dpt:443
```

---

# 15. Packet Capture with tcpdump

This is one of the strongest troubleshooting tools.

Example:

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

TCP handshake:

```bash
sudo tcpdump -i eth0 'tcp port 443'
```

You might observe:

```text
SYN →
SYN →
SYN →
```

but no SYN-ACK.

That tells you:

> Client is sending packets, but no response is coming back.

Possible:

- Firewall/drop
- Routing/return path
- Server unreachable

If you see:

```text
SYN →
← RST
```

then:

> Destination is reachable but connection is actively rejected.

---

# 16. Check Network Statistics

```bash
ip -s link
```

Shows:

```text
RX packets
TX packets
Dropped packets
Errors
```

Example:

```text
RX errors: 5000
Dropped: 2000
```

Could indicate interface/network problems.

Another command:

```bash
netstat -s
```

or:

```bash
ss -s
```

for connection statistics.

---

# 17. Check Interface Speed / Link

For physical Linux servers:

```bash
ethtool eth0
```

Can show:

```text
Speed: 1000Mb/s
Duplex: Full
Link detected: yes
```

If:

```text
Link detected: no
```

you have a physical/link-level problem.

---

# 18. Check Process Owning a Port

```bash
ss -ltnp
```

Example:

```text
LISTEN 0 128 0.0.0.0:8080 users:(("java",pid=1234))
```

Or:

```bash
sudo lsof -i :8080
```

This tells you:

> Which process owns port 8080?

---

# 19. Common Scenario: Website Not Working

Suppose:

```text
https://app.company.com
```

fails.

Use this order:

```text
1. dig app.company.com
        ↓
Does DNS resolve?

2. ip route get <IP>
        ↓
Is routing correct?

3. ping <IP>
        ↓
Basic reachability?

4. nc -vz <IP> 443
        ↓
Can TCP 443 connect?

5. curl -v https://app.company.com
        ↓
TLS/HTTP working?

6. openssl s_client ...
        ↓
Certificate/TLS?

7. Server: ss -ltnp
        ↓
Is application listening?

8. Firewall / SG / NACL
```

---

# 20. Scenario: App Server Cannot Reach Database

```text
Application:
10.0.1.20

Database:
10.0.2.30:5432
```

Start:

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
SG/NACL
Firewall
Return path
```

On DB:

```bash
ss -ltnp | grep 5432
```

If DB shows:

```text
127.0.0.1:5432
```

then PostgreSQL is only listening locally.

It may need to listen on:

```text
0.0.0.0:5432
```

or the appropriate private IP.

---

# 21. Scenario: IP Works but Hostname Doesn't

```bash
curl http://10.0.1.20
```

works.

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

---

# 22. Scenario: Ping Works, Port Doesn't

```bash
ping 10.0.2.20
```

works.

But:

```bash
nc -vz 10.0.2.20 443
```

times out.

Interpretation:

```text
L3 connectivity → likely okay
L4 TCP/443      → failing
```

Check:

- Firewall
- Security group
- NACL
- Service listening
- Port configuration

---

# 23. Scenario: Connection Refused

```bash
curl http://10.0.2.20:8080
```

returns:

```text
Connection refused
```

This usually tells you something important:

> Network path reached the destination.

Likely:

```text
Application not running
Wrong port
Service only listening on localhost
Destination firewall actively rejected connection
```

Check:

```bash
ss -ltnp | grep 8080
```

---

# 24. Scenario: Connection Timeout

```text
Connection timed out
```

Think more about the **network path**:

```text
Firewall drop
Routing issue
Security group
NACL
Server unreachable
Return-path problem
```

Timeout and refused are not the same.

### Interview memory

```text
REFUSED
→ I reached something, but it rejected me.

TIMEOUT
→ My packets or responses are disappearing somewhere.
```

---

# 25. Important Commands to Memorize

```bash
ip a
```

> IP/interface

```bash
ip route
```

> Routing table

```bash
ip route get <IP>
```

> Exact route Linux will use

```bash
ping <IP>
```

> Basic ICMP reachability

```bash
dig <hostname>
```

> DNS

```bash
nc -vz <host> <port>
```

> TCP port connectivity

```bash
ss -tulpn
```

> Listening ports/processes

```bash
curl -v <URL>
```

> HTTP/TLS/application troubleshooting

```bash
traceroute <IP>
```

> Network path

```bash
tcpdump
```

> Actual packets

```bash
iptables -L -n -v
```

> Firewall

```bash
ip neigh
```

> Local neighbor/MAC information

---

# Interview Troubleshooting Framework

If they ask:

> **Server A cannot connect to Server B. How will you troubleshoot?**

Give this answer:

```text
1. Verify source interface and IP
2. Check route to destination
3. Verify DNS if hostname is used
4. Test basic reachability
5. Test required TCP/UDP port
6. Check service is listening on destination
7. Check host/network firewalls
8. Check SG/NACL if cloud
9. Use traceroute if path issue suspected
10. Use tcpdump to identify where packets stop
```

That is a strong, structured troubleshooting answer.

### One line to remember

> **DNS tells me the IP → routing tells me the path → firewall decides whether traffic is allowed → TCP verifies the port → application must actually be listening.**