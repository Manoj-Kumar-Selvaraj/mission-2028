## Ports — Basics

A **port identifies a specific application/service running on a machine**.

Example:

```text
Server IP = 10.0.0.20
```

That same server can run:

```text
22   → SSH
80   → HTTP
443  → HTTPS
3306 → MySQL
5432 → PostgreSQL
```

So:

```text
10.0.0.20:443
```

means:

> Connect to machine `10.0.0.20`, specifically to the service listening on port `443`.

Simple rule:

> **IP = which machine**  
> **Port = which service/application on that machine**


Suppose your laptop connects to a web server:

```text
Client IP:  192.168.1.10
Server IP:  10.0.0.20
```

The server listens on:

```text
443 → HTTPS
```

Your laptop also needs a temporary **source port**, for example:

```text
53124
```

So the connection becomes:

```text
192.168.1.10:53124 → 10.0.0.20:443
```

Meaning:

- `53124` = temporary client-side port
- `443` = server-side HTTPS port

That temporary client port is called an **ephemeral port**.

Next we can see **why the client needs its own port at all**.

Your laptop also needs a temporary **source port - Why?**

Because your laptop can open **many connections at the same time**.

Example:

```text
Chrome → amazon.com:443
Chrome → youtube.com:443
Teams  → server:443
```

All of them may use destination port `443`.

So your OS assigns each outgoing connection a temporary source port:

```text
192.168.1.10:53124 → Amazon:443
192.168.1.10:53125 → YouTube:443
192.168.1.10:53126 → Teams:443
```

That way, when responses come back, the OS knows **which connection/application they belong to**.

So:

> **Server port = identifies the service you want to reach.**  
> **Client ephemeral port = identifies your specific outgoing connection.**

Next: **TCP 3-Way Handshake**.

Before TCP sends application data, it establishes a connection:

```text
Client                  Server

SYN -------------------->

     <---------------- SYN-ACK

ACK -------------------->
```

Meaning:

```text
SYN      → Can we start a connection?
SYN-ACK  → Yes, I received you.
ACK      → Good, connection established.
```

After that, application data can flow.

> **TCP handshake = confirms both sides are reachable and ready before data transfer starts.**

## TCP Reliability — Sequence & ACK

After the handshake, TCP sends data in chunks called **segments**.

TCP numbers the data using **sequence numbers** so the receiver knows the correct order.

Example:

```text
Segment 1 → Seq 1
Segment 2 → Seq 2
Segment 3 → Seq 3
```

The receiver sends **ACKs** back to confirm what it received.

If a segment is missing:

```text
1 received
2 missing
3 received
```

TCP retransmits the missing data.

> **Sequence number = ordering**  
> **ACK = confirmation**  
> **Retransmission = resend lost data**

That is a major reason TCP is considered **reliable**.

## TCP Connection Termination

When communication is finished, TCP closes the connection gracefully.

Typical flow:

```text
Client                  Server

FIN -------------------->

     <---------------- ACK

     <---------------- FIN

ACK -------------------->
```

Meaning:

```text
FIN → I have finished sending.
ACK → I received that.
FIN → I have also finished sending.
ACK → Connection closed.
```

So TCP has:

```text
3-way handshake → open connection
4-step FIN/ACK   → close connection
```

SYN → Start a connection
ACK → Acknowledge received data
FIN → Gracefully close a connection
RST → Immediately reset/terminate a connection

## TCP Connection States

Common TCP states:

```text
LISTEN       → Server is waiting for new connections
SYN_SENT     → Client sent SYN, waiting for reply
SYN_RECEIVED → Server received SYN and replied
ESTABLISHED  → Connection is active
FIN_WAIT     → One side started closing
CLOSE_WAIT   → Peer closed; local app hasn't closed yet
TIME_WAIT    → Connection closed, waiting briefly before full cleanup
CLOSED       → No connection
```

Most important for troubleshooting:

> **LISTEN = service is ready**  
> **ESTABLISHED = connection is active**  
> **CLOSE_WAIT = app may not be closing sockets properly**  
> **TIME_WAIT = normal after connection close**


## UDP — Basics

**UDP = User Datagram Protocol**

Unlike TCP, UDP does **not establish a connection first**.

It simply sends data:

```text
Client ───── data ─────> Server
```

There is no:

```text
SYN
SYN-ACK
ACK
```

UDP also does not guarantee:

```text
Delivery
Ordering
Retransmission
```

So remember:

> **TCP = connection-oriented + reliable**  
> **UDP = connectionless + lightweight**

Common UDP use cases:

```text
DNS
VoIP
Streaming
Online gaming
DHCP
```

Next, we should understand **why anyone uses UDP if TCP is reliable**.


## Why use UDP if TCP is reliable?

Because reliability adds **overhead and delay**.

TCP does:

```text
Connection setup
Acknowledgements
Retransmissions
Ordering
Flow control
```

UDP skips most of that and just sends the data.

Example: **voice call**

If one tiny audio packet is lost, it is usually better to continue:

```text
Hello, how are you?
       ↓ one packet lost
Hello, h_w are you?
```

rather than pause and retransmit old audio.

So:

> **TCP = correctness/reliability is more important**  
> **UDP = low latency and simplicity are more important**

Next: **TCP vs UDP comparison with real examples.**

## TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable | No guarantee |
| Ordering | Maintains order | No ordering guarantee |
| Retransmission | Yes | No |
| Overhead | Higher | Lower |
| Typical use | HTTP/HTTPS, SSH, DB | DNS, VoIP, gaming, DHCP |

Easy memory:

> **TCP = reliable but heavier**  
> **UDP = lightweight but no delivery guarantee**

It’s decided by the **application protocol being used**, not by the domain name itself.

For `amazon.com`, there are two separate parts:

```text
1. DNS lookup
2. Web connection
```

### 1. DNS lookup

Usually:

```text
DNS → UDP port 53
```

If the DNS response is too large or certain conditions apply, DNS can use TCP.

### 2. Website connection

For traditional HTTPS:

```text
HTTP/1.1 or HTTP/2
        ↓
TLS
        ↓
TCP port 443
```

So:

```text
Browser → TCP connection → amazon.com:443
```

But modern browsers can also use **HTTP/3**, which runs over **QUIC**, and QUIC uses UDP:

```text
HTTP/3
  ↓
QUIC
  ↓
UDP port 443
```

So the browser and server determine which supported HTTP protocol to use.

Simple view:

```text
HTTP/1.1 → TCP
HTTP/2   → TCP
HTTP/3   → QUIC → UDP
```

So **the protocol determines TCP vs UDP**. The application/browser already knows what transport that protocol requires.

## Common Ports — TCP vs UDP

| Protocol | Port | TCP / UDP | Use |
|---|---:|---|---|
| **HTTP** | 80 | TCP | Web traffic |
| **HTTPS** | 443 | TCP | Encrypted web traffic |
| **HTTP/3 / QUIC** | 443 | UDP | Modern web traffic |
| **DNS** | 53 | UDP + TCP | Name → IP resolution |
| **SSH** | 22 | TCP | Remote Linux access |
| **FTP** | 20, 21 | TCP | File transfer |
| **SMTP** | 25 / 587 | TCP | Sending email |
| **IMAP** | 143 / 993 | TCP | Reading email |
| **POP3** | 110 / 995 | TCP | Reading/downloading email |
| **DHCP** | 67, 68 | UDP | Assign IP addresses |
| **NTP** | 123 | UDP | Time synchronization |
| **SNMP** | 161, 162 | UDP mostly | Network monitoring |
| **LDAP** | 389 | TCP/UDP | Directory services |
| **LDAPS** | 636 | TCP | Secure LDAP |
| **SMB** | 445 | TCP | Windows file sharing |
| **RDP** | 3389 | TCP + UDP | Windows remote desktop |
| **MySQL** | 3306 | TCP | MySQL database |
| **PostgreSQL** | 5432 | TCP | PostgreSQL database |
| **Redis** | 6379 | TCP | Redis |
| **MongoDB** | 27017 | TCP | MongoDB |

### Easy ones to remember for interviews

```text
22   → SSH        → TCP
53   → DNS        → UDP/TCP
80   → HTTP       → TCP
443  → HTTPS      → TCP
443  → HTTP/3     → UDP
67/68 → DHCP      → UDP
123  → NTP        → UDP
3306 → MySQL      → TCP
5432 → PostgreSQL → TCP
```

One important point:

> **Ports belong to TCP or UDP namespaces.** The same port number can technically exist for both TCP and UDP.

For example, **TCP 443** and **UDP 443** are different transport endpoints.