# Linux Network Troubleshooting — Complete Interview Guide

Use a **layered troubleshooting flow**. Don’t randomly run commands.

```text
1. Interface / IP
2. Routing
3. Gateway
4. Reachability
5. DNS
6. TCP / UDP Port
7. Listening Process
8. Existing Connections
9. Firewall
10. HTTP / Application
11. TLS / Certificate
12. Network Path
13. Packet Capture
14. Interface Statistics
```

The key principle is:

> **Use every successful test to eliminate an entire class of possible problems.**

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
- Does it have an IP address?
- Is the subnet/CIDR correct?

Example:

```text
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>

inet 10.0.1.20/24
```

Interpretation:

```text
Interface = eth0
Status    = UP
IP        = 10.0.1.20
Subnet    = /24
```

If there is no expected IP address, investigate:

```text
DHCP
Static IP configuration
Interface configuration
Network manager
```

---

## `ip link`

Use:

```bash
ip link
```

This focuses on interface/link status.

Example:

```text
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
state UP
```

Important values:

```text
UP
→ Interface is administratively enabled

LOWER_UP
→ Underlying link is operational

DOWN
→ Interface is disabled or link unavailable
```

---

# Real Scenario — WSL Interface

Command:

```bash
ip addr
```

Output:

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    link/ether 00:15:5d:06:10:7a
    inet 172.20.211.208/20 brd 172.20.223.255 scope global eth0
```

And:

```bash
ip link
```

Output:

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    mtu 1500
    state UP
```

Interpretation:

```text
Interface = eth0
IPv4      = 172.20.211.208
CIDR      = /20
MAC       = 00:15:5d:06:10:7a
MTU       = 1500
Status    = UP
```

Therefore:

```text
Interface exists ✅
Interface active ✅
IP assigned      ✅
```

---

# 2. Check Routing

Use:

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
10.0.1.0/24
→ Directly connected network

Everything else
→ Send through 10.0.1.1
```

The second route is the:

```text
Default route
```

---

# Real Scenario — WSL Routing Table

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
Connected Network = 172.20.208.0/20

Interface         = eth0

Source IP         = 172.20.211.208

Default Gateway   = 172.20.208.1
```

Meaning:

```text
Traffic for 172.20.208.0/20
→ Directly through eth0

Everything else
→ Send to 172.20.208.1
```

---

# 3. Check Exact Route to a Destination

One of the most useful commands:

```bash
ip route get <destination-ip>
```

Example:

```bash
ip route get 8.8.8.8
```

Generic output:

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

# Real Scenario — Exact Route

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

Packet starts roughly as:

```text
WSL
172.20.211.208
      ↓
172.20.208.1
Default Gateway
      ↓
Windows / WSL virtual networking
      ↓
Internet
      ↓
8.8.8.8
```

### Interview use

If:

```text
Server A cannot reach Server B
```

run:

```bash
ip route get <server-b-ip>
```

This immediately tells you how Linux intends to send the packet.

---

# 4. Check Default Gateway Reachability

Suppose:

```text
Gateway = 10.0.1.1
```

Try:

```bash
ping 10.0.1.1
```

If it fails, possible causes include:

```text
Wrong gateway
Interface problem
Local network issue
VLAN issue
Layer-2 issue
ICMP blocked
```

Important:

> **A gateway does not have to reply to ping in order to forward traffic.**

---

# Real Scenario — Gateway Ping Fails

Command:

```bash
ping 172.20.208.1
```

Result:

```text
14 packets transmitted
0 received
100% packet loss
```

At first this appears to mean:

```text
Gateway unreachable ❌
```

But we then tested:

```bash
ping 8.8.8.8
```

and received:

```text
64 bytes from 8.8.8.8
64 bytes from 8.8.8.8
64 bytes from 8.8.8.8
64 bytes from 8.8.8.8

4 packets transmitted
4 received
0% packet loss
```

Therefore the gateway is clearly forwarding traffic.

The likely explanation is:

> The WSL virtual gateway does not reply to ICMP Echo Requests addressed to itself, but still forwards packets.

So:

```text
Gateway ping ❌
Internet ping ✅
Routing       ✅
Forwarding    ✅
```

### Interview lesson

Never conclude:

```text
Ping failed
→ Network is definitely down
```

Ping tests **ICMP response**, not every aspect of network connectivity.

---

# 5. Check Destination Reachability

Use:

```bash
ping 8.8.8.8
```

This tests basic Layer-3 connectivity using:

```text
ICMP
```

If it works, you know that:

```text
Interface
Routing
Gateway forwarding
External IP connectivity
```

are probably functioning.

But remember:

> **Ping success does not mean application connectivity works.**

Example:

```text
ICMP allowed ✅
TCP 443 blocked ❌
```

Also, ping failure does not prove the destination is down because ICMP may be blocked.

---

# 6. Check DNS

Suppose:

```bash
ping 8.8.8.8
```

works, but:

```bash
curl https://google.com
```

returns:

```text
Could not resolve host
```

Then suspect:

```text
DNS
```

Useful commands:

```bash
dig google.com
```

```bash
nslookup google.com
```

```bash
host google.com
```

---

## Check Configured DNS Resolver

```bash
cat /etc/resolv.conf
```

Example:

```text
nameserver 10.0.0.2
```

Modern Linux systems may use:

```bash
resolvectl status
```

---

## Test Another DNS Resolver

Example:

```bash
dig @8.8.8.8 google.com
```

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
Local resolver
Corporate DNS
VPN DNS
/etc/resolv.conf
systemd-resolved
```

---

# Real Scenario — DNS Resolution

Command:

```bash
dig google.com
```

Output:

```text
; <<>> DiG 9.18.39 <<>> google.com

;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2961

;; flags: qr rd ra;
;; QUERY: 1
;; ANSWER: 1
;; AUTHORITY: 0
;; ADDITIONAL: 1

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             233     IN      A       142.251.221.206

;; Query time: 8 msec

;; SERVER: 10.255.255.254#53(10.255.255.254) (UDP)
```

---

## `status: NOERROR`

```text
status: NOERROR
```

Means:

> DNS lookup completed successfully.

---

## Question Section

```text
google.com. IN A
```

Meaning:

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

Meaning:

```text
google.com
   ↓
142.251.221.206
```

DNS resolution succeeded.

---

## TTL

```text
233
```

is the TTL:

```text
TTL = 233 seconds
```

The result can be cached for approximately:

```text
3 minutes 53 seconds
```

before refresh may be required.

---

## DNS Resolver

```text
SERVER: 10.255.255.254#53
```

Means:

```text
DNS Resolver = 10.255.255.254
Port         = 53
```

And:

```text
(UDP)
```

means this DNS query used:

```text
UDP port 53
```

---

## Query Time

```text
Query time: 8 msec
```

Means DNS resolution completed in:

```text
8 milliseconds
```

---

## DNS Flags

```text
qr rd ra
```

Meaning:

```text
qr = Query Response
rd = Recursion Desired
ra = Recursion Available
```

Conceptually:

```text
Linux:
"Please completely resolve google.com for me."

        ↓

Recursive DNS resolver

        ↓

Answer returned
```

---

## DNS Counts

```text
QUERY:      1
ANSWER:     1
AUTHORITY:  0
ADDITIONAL: 1
```

Meaning:

```text
1 DNS question
1 answer
0 authority records in response
1 additional metadata section
```

---

# 7. Check TCP / UDP Port Connectivity

Suppose an application needs:

```text
Database = 10.0.2.20
Port     = 5432
```

Test TCP connectivity with:

```bash
nc -vz 10.0.2.20 5432
```

---

## Successful Connection

Example:

```text
Connection to 10.0.2.20 5432 port [tcp/postgresql] succeeded!
```

Means:

> TCP connection to destination port 5432 can be established.

That tells you:

```text
Routing       ✅
Destination   ✅
TCP handshake ✅
Port reachable ✅
```

---

## Connection Refused

Example:

```text
Connection refused
```

Usually means:

> Destination was reachable, but the TCP connection was actively rejected.

Likely causes:

```text
Application not running
Nothing listening on the port
Wrong port
Application listening only on localhost
Firewall actively rejecting connection
```

---

## Connection Timeout

Example:

```text
Connection timed out
```

Usually points more toward:

```text
Firewall drop
Security Group
NACL
Routing issue
Packet loss
Server unreachable
Return-path problem
```

### Important memory

```text
REFUSED
→ I reached something, but it rejected the connection.

TIMEOUT
→ Packets or responses are disappearing somewhere.
```

---

# 8. Check Whether Application Is Listening

On the destination server:

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

Meaning:

> Service listens on port 8080 on all IPv4 interfaces.

---

## Localhost-Only Binding

Example:

```text
127.0.0.1:8080
```

Means:

> Service accepts connections only from the same machine.

Remote servers cannot connect.

If remote access is required, the application may need to bind to:

```text
0.0.0.0:8080
```

or:

```text
<server-private-ip>:8080
```

depending on design.

---

# 9. Check Which Process Owns a Port

Use:

```bash
ss -ltnp
```

Example:

```text
LISTEN 0 128 0.0.0.0:8080 users:(("java",pid=1234))
```

Meaning:

```text
Port    = 8080
Process = java
PID     = 1234
```

Alternative:

```bash
sudo lsof -i :8080
```

Useful question:

> Which application is actually listening on this port?

---

# 10. Check Existing TCP Connections

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

Means:

> TCP connection is active.

---

## SYN-SENT

```text
SYN-SENT
```

Means:

> Client sent SYN but hasn't completed the TCP handshake.

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

Means:

> Remote side closed the connection, but the local application has not fully closed its socket.

A large number of `CLOSE-WAIT` connections may indicate:

```text
Application socket leak
Application not closing connections properly
```

---

## TIME-WAIT

Normally appears after connections close.

A certain amount of:

```text
TIME-WAIT
```

is normal TCP behavior.

---

# 11. Check HTTP Application

Use:

```bash
curl -v http://server:8080
```

For HTTPS:

```bash
curl -v https://example.com
```

`-v` can reveal:

```text
DNS resolution
Destination IP
TCP connection
TLS negotiation
HTTP request
HTTP response
Headers
```

This is far more useful for application troubleshooting than ping.

---

# Example `curl -v` Flow

Conceptually:

```text
Trying 142.x.x.x:443...
Connected
TLS handshake
Certificate validation
HTTP request sent
HTTP response received
```

If you reach:

```text
Connected to ...
```

then:

```text
TCP connectivity is working.
```

If TLS fails afterward:

```text
The problem is higher than basic TCP connectivity.
```

---

# 12. Check Only HTTP Headers

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
Quick health checks
Status-code checks
Header checks
Redirect checks
```

---

# 13. Check TLS / Certificate

If:

```text
TCP 443 works
```

but:

```text
HTTPS fails
```

investigate TLS.

Use:

```bash
openssl s_client -connect example.com:443
```

Better for hostname-based TLS/SNI:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com
```

Check:

```text
Certificate expiry
Certificate issuer
Certificate chain
Hostname
TLS version
TLS handshake
```

Possible problems:

```text
certificate expired
hostname mismatch
unable to verify certificate
unsupported TLS version
certificate chain incomplete
```

---

# 14. Trace the Network Path

Use:

```bash
traceroute 8.8.8.8
```

Conceptually:

```text
Server
 ↓
Gateway
 ↓
Router
 ↓
ISP
 ↓
Another Router
 ↓
Destination
```

Alternative:

```bash
tracepath 8.8.8.8
```

Useful for identifying where the path changes or stops.

But remember:

> Some routers don't respond to traceroute probes, so missing hops do not automatically prove failure.

---

# 15. Check ARP / Neighbor Table

Use:

```bash
ip neigh
```

Example:

```text
10.0.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

Meaning:

```text
IP 10.0.1.1
→ MAC aa:bb:cc:dd:ee:ff
```

Common states:

```text
REACHABLE
STALE
DELAY
FAILED
INCOMPLETE
```

`FAILED` or `INCOMPLETE` can indicate local Layer-2/neighbor-resolution problems.

---

# 16. Check Firewall

## iptables

```bash
sudo iptables -L -n -v
```

Look for rules that:

```text
ACCEPT
DROP
REJECT
```

traffic.

---

## nftables

```bash
sudo nft list ruleset
```

---

## Ubuntu UFW

```bash
sudo ufw status
```

---

Check rules involving:

```text
Source IP
Destination IP
Protocol
Source Port
Destination Port
```

Example:

```text
DROP tcp -- 10.0.1.0/24 0.0.0.0/0 tcp dpt:443
```

Meaning:

> TCP/443 traffic from `10.0.1.0/24` is being dropped.

---

# 17. Packet Capture with tcpdump

When higher-level commands don't explain the issue, inspect actual packets.

Basic capture:

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

TCP-specific:

```bash
sudo tcpdump -i eth0 'tcp port 443'
```

---

## Scenario — SYN Repeated, No Response

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

> Client is transmitting connection requests, but no response is returning.

Possible causes:

```text
Firewall drop
Security Group
NACL
Routing issue
Return-path issue
Server unavailable
```

---

## Scenario — RST Returned

You see:

```text
SYN →
← RST
```

Meaning:

> Destination is reachable, but the connection is actively rejected/reset.

Often:

```text
No service listening
Application rejected connection
Firewall sent reject/reset
```

---

# 18. Check Network Statistics

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

This can indicate:

```text
Interface problems
Driver problems
Network congestion
Buffer pressure
Physical/link issues
```

---

## TCP Statistics

```bash
ss -s
```

or:

```bash
netstat -s
```

Useful for high-level connection and protocol statistics.

---

# 19. Check Interface Speed / Link

For physical servers:

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

suspect:

```text
Cable
NIC
Switch port
Physical connectivity
```

---

# 20. Scenario — Website Not Working

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
Is route correct?

3. ping <IP>
        ↓
Basic ICMP reachability?

4. nc -vz <IP> 443
        ↓
Can TCP 443 connect?

5. curl -v https://app.company.com
        ↓
Does TLS/HTTP work?

6. openssl s_client ...
        ↓
TLS/certificate issue?

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

# 21. Scenario — App Server Cannot Reach Database

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

Verify route.

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

On database server:

```bash
ss -ltnp | grep 5432
```

If:

```text
127.0.0.1:5432
```

then PostgreSQL is listening only locally.

It may need:

```text
0.0.0.0:5432
```

or an appropriate private IP binding.

---

# 22. Scenario — IP Works but Hostname Doesn't

This works:

```bash
curl http://10.0.1.20
```

But this fails:

```bash
curl http://app.internal
```

Likely problem:

```text
DNS
```

Check:

```bash
dig app.internal
```

and:

```bash
cat /etc/resolv.conf
```

Interpretation:

```text
IP connectivity ✅
Name resolution ❌
```

---

# 23. Scenario — Ping Works but TCP Port Doesn't

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

# 24. Scenario — Connection Refused

Command:

```bash
curl http://10.0.2.20:8080
```

Result:

```text
Connection refused
```

Likely meaning:

> Network path reached the destination, but connection was rejected.

Possible causes:

```text
App not running
Wrong port
Service listening only on localhost
Firewall configured to reject
```

Check:

```bash
ss -ltnp | grep 8080
```

---

# 25. Scenario — Connection Timeout

Result:

```text
Connection timed out
```

Think:

```text
Firewall drop
Routing
Security Group
NACL
Destination unavailable
Return-path issue
```

Easy interview memory:

```text
REFUSED
→ Destination was reached, but rejected connection.

TIMEOUT
→ Traffic disappeared or no response returned.
```

---

# 26. Scenario — DNS Works but Website Doesn't

Suppose:

```bash
dig app.company.com
```

returns an IP successfully.

But:

```bash
curl https://app.company.com
```

fails.

Then DNS has already done its job.

Continue with:

```bash
ip route get <resolved-ip>

nc -vz <resolved-ip> 443

curl -v https://app.company.com
```

Possible issue now lies in:

```text
Routing
Firewall
TCP
TLS
Load Balancer
Application
```

---

# 27. Scenario — `nc` Works but HTTPS Doesn't

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

Likely issues:

```text
Expired certificate
Hostname mismatch
Broken certificate chain
TLS version mismatch
Application/proxy problem
```

---

# 28. Scenario — Application Listening Only on Localhost

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

Remote connection:

```bash
nc -vz <server-ip> 8080
```

fails.

Reason:

> Service listens only on localhost and not on the network interface.

---

# 29. Scenario — SYN Leaves but Nothing Returns

Packet capture:

```bash
sudo tcpdump -i eth0 'tcp port 443'
```

shows:

```text
SYN →
SYN →
SYN →
```

No SYN-ACK.

Interpretation:

```text
Client generated packet ✅
Client interface sent it ✅
Return response           ❌
```

Investigate:

```text
Remote firewall
Security Group
NACL
Route
Return path
Destination service/network
```

---

# 30. Important Commands to Memorize

## Interface

```bash
ip a
```

> IP address and interface details

```bash
ip link
```

> Interface/link state

---

## Routing

```bash
ip route
```

> Routing table

```bash
ip route get <IP>
```

> Exact route Linux will use

---

## Reachability

```bash
ping <IP>
```

> ICMP connectivity

---

## DNS

```bash
dig <hostname>
```

> DNS resolution

```bash
dig @8.8.8.8 <hostname>
```

> Test a specific DNS resolver

```bash
cat /etc/resolv.conf
```

> Configured resolver

---

## TCP Port

```bash
nc -vz <host> <port>
```

> Test TCP port connectivity

---

## Listening Services

```bash
ss -tulpn
```

> Listening TCP/UDP ports and processes

```bash
lsof -i :<port>
```

> Process using a specific port

---

## Connections

```bash
ss -tan
```

> TCP connections and states

---

## HTTP / HTTPS

```bash
curl -v <URL>
```

> DNS + TCP + TLS + HTTP troubleshooting

```bash
curl -I <URL>
```

> HTTP headers only

---

## TLS

```bash
openssl s_client -connect <host>:443 -servername <host>
```

> TLS/certificate troubleshooting

---

## Path

```bash
traceroute <IP>
```

> Network-hop path

---

## Neighbor / ARP

```bash
ip neigh
```

> Local IP-to-MAC neighbor information

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

> Inspect actual packets

---

## Interface Statistics

```bash
ip -s link
```

> RX/TX/errors/drops

---

# 31. Interview Troubleshooting Framework

If interviewer asks:

> **Server A cannot connect to Server B. How will you troubleshoot?**

Answer:

```text
1. Verify source interface and IP
2. Verify destination IP
3. Check routing using ip route get
4. Check DNS if hostname is being used
5. Test basic reachability
6. Test required TCP/UDP port
7. Check whether destination application is listening
8. Check source and destination host firewall
9. Check SG/NACL/network firewall if cloud
10. Check return path
11. Use traceroute if path issue is suspected
12. Use tcpdump when packet-level confirmation is required
```

---

# 32. Real WSL Troubleshooting Summary

Our tests showed:

```text
Interface eth0              ✅
IP 172.20.211.208/20        ✅
Interface state UP          ✅

Connected route             ✅
Default gateway             ✅
Route to 8.8.8.8            ✅

Gateway ping                ❌
Gateway packet forwarding   ✅

Internet ICMP               ✅

DNS resolver                ✅
UDP port 53                 ✅
DNS A record resolution     ✅
```

Current packet path:

```text
WSL
172.20.211.208
      ↓
eth0
      ↓
172.20.208.1
Virtual Gateway
      ↓
Windows Networking
      ↓
Internet
```

DNS:

```text
WSL
  ↓ UDP/53
10.255.255.254
  ↓
DNS resolution
  ↓
google.com
  ↓
142.251.221.206
```

---

# 33. Complete Layered Mental Model

When troubleshooting:

```text
Does interface exist?
        ↓
Does it have an IP?
        ↓
Is routing correct?
        ↓
Can destination IP be reached?
        ↓
Does DNS resolve?
        ↓
Can required TCP/UDP port connect?
        ↓
Is application listening?
        ↓
Does firewall allow traffic?
        ↓
Does TLS work?
        ↓
Does application respond correctly?
        ↓
Still unclear?
        ↓
Capture packets with tcpdump
```

---

# Final Memory Line

> **Interface gives connectivity → routing chooses the path → DNS gives the destination IP → firewall allows or blocks traffic → TCP/UDP reaches the service → application must be listening → TLS/HTTP handle the higher-level communication.**

And:

> **Never troubleshoot networking with only one command. Move layer by layer and use each successful result to narrow the failure domain.**