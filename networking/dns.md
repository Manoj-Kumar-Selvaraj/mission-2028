# DNS — Domain Name System

## 1. What is DNS?

**DNS = Domain Name System**

DNS converts human-readable domain names into IP addresses.

```text
amazon.com
   ↓
DNS
   ↓
IP address
```

Machines communicate using IP addresses, but humans prefer names such as:

```text
amazon.com
google.com
github.com
```

So DNS acts like the **phone book of the Internet**.

---

# 2. Where does DNS sit in OSI?

DNS is an **Application Layer protocol — Layer 7**.

```text
Layer 7 → DNS
Layer 4 → UDP/TCP
Layer 3 → IP
Layer 2 → Local delivery
Layer 1 → Physical transmission
```

DNS normally uses:

```text
UDP port 53
```

but can also use:

```text
TCP port 53
```

---

# 3. Full DNS Flow — Opening amazon.com

Suppose you type:

```text
https://amazon.com
```

The browser needs Amazon's IP address before it can connect.

The flow starts with caches.

```text
Browser
   ↓
Browser DNS cache
   ↓
Operating System DNS cache
   ↓
Configured DNS Resolver
```

If an IP is already cached and still valid, DNS resolution may stop here.

Otherwise, the DNS resolver performs the lookup.

---

# 4. DNS Resolver

Your computer normally does **not directly contact all DNS servers on the Internet**.

Instead, it sends the query to a **recursive DNS resolver**.

Examples could be:

```text
ISP DNS resolver
8.8.8.8   → Google DNS
1.1.1.1   → Cloudflare DNS
```

Your machine asks:

```text
What is the IP of amazon.com?
```

The resolver does the work on your behalf.

---

# 5. Recursive DNS Resolution

Assume nothing is cached.

The resolver contacts DNS infrastructure in stages:

```text
Your Laptop
    ↓
Recursive Resolver
    ↓
Root DNS Server
    ↓
.com TLD Server
    ↓
amazon.com Authoritative DNS
    ↓
IP returned
```

Let's understand each one.

---

# 6. Root DNS Server

The resolver first asks a **Root DNS server**:

```text
Where can I find amazon.com?
```

The root server generally does **not return Amazon's IP**.

Instead, it says:

```text
I don't know amazon.com directly,
but I know who manages .com.
```

It returns information about the `.com` TLD name servers.

So:

```text
Root
  ↓
.com DNS servers
```

---

# 7. TLD Server

**TLD = Top-Level Domain**

Examples:

```text
.com
.org
.net
.in
.io
```

The resolver now asks the `.com` DNS server:

```text
Who knows about amazon.com?
```

The `.com` server replies with the authoritative name servers responsible for:

```text
amazon.com
```

So:

```text
.com TLD
   ↓
Amazon authoritative DNS
```

---

# 8. Authoritative DNS Server

The resolver then asks Amazon's authoritative DNS server:

```text
What is the IP for amazon.com?
```

This server actually contains the DNS records for the domain.

It returns a result such as:

```text
amazon.com → x.x.x.x
```

The recursive resolver then returns this result to your computer.

---

# 9. Complete DNS Resolution Flow

```text
Browser asks for amazon.com
          ↓
Browser cache
          ↓
OS DNS cache
          ↓
Recursive DNS Resolver
          ↓
Root DNS
"Ask .com"
          ↓
.com TLD DNS
"Ask Amazon DNS"
          ↓
Amazon Authoritative DNS
"amazon.com = x.x.x.x"
          ↓
Recursive Resolver
          ↓
Your Laptop
          ↓
Browser now has the IP
```

After DNS resolution, normal network communication begins:

```text
TCP/QUIC
   ↓
TLS
   ↓
HTTP request
```

---

# 10. Recursive vs Iterative DNS

## Recursive Query

Client says:

```text
Give me the final answer.
```

Example:

```text
Laptop → Recursive Resolver
```

The resolver performs all required lookups.

---

## Iterative Query

A DNS server may respond:

```text
I don't know the final answer,
but ask this server next.
```

Example:

```text
Root DNS → Ask .com server

.com DNS → Ask Amazon authoritative server
```

Easy memory:

```text
Recursive = find the answer for me
Iterative = ask this server next
```

---

# 11. DNS Record Types

## A Record

Maps a hostname to an IPv4 address.

```text
amazon.com → 54.x.x.x
```

```text
A = IPv4
```

---

## AAAA Record

Maps a hostname to an IPv6 address.

```text
example.com → 2606:4700:....
```

```text
AAAA = IPv6
```

---

## CNAME Record

**CNAME = Canonical Name**

Creates an alias from one hostname to another.

Example:

```text
www.example.com
        ↓
example.company-cloud.com
```

The second hostname is then resolved normally.

Important:

```text
CNAME → hostname
A     → IPv4 address
AAAA  → IPv6 address
```

---

## MX Record

**MX = Mail Exchange**

Specifies the mail servers responsible for receiving email for a domain.

Example:

```text
example.com
   ↓
mail.example.com
```

Used by SMTP infrastructure.

---

## NS Record

**NS = Name Server**

Specifies the authoritative DNS servers responsible for a domain.

Example:

```text
example.com
   ↓
ns1.provider.com
ns2.provider.com
```

---

## TXT Record

Stores arbitrary text information.

Common uses:

```text
Domain verification
SPF
DKIM-related configuration
Security validation
Cloud service verification
```

---

## PTR Record

Used for **reverse DNS lookup**.

Normal lookup:

```text
hostname → IP
```

Reverse lookup:

```text
IP → hostname
```

PTR records provide the reverse mapping.

---

## SOA Record

**SOA = Start of Authority**

Contains administrative information about a DNS zone.

Typically includes:

```text
Primary name server
Responsible contact
Serial number
Refresh interval
Retry interval
Expiration
Default timing values
```

---

## SRV Record

Used to locate a service.

It can specify:

```text
Service
Protocol
Port
Target hostname
Priority
Weight
```

Common in enterprise/service-discovery environments.

---

# 12. DNS Zones

A **DNS zone** is an administrative section of the DNS namespace.

Example:

```text
example.com
```

A DNS provider may host a zone containing:

```text
A
AAAA
CNAME
MX
TXT
NS
```

records for that domain.

In AWS Route 53, this is called a:

```text
Hosted Zone
```

---

# 13. TTL

**TTL = Time To Live**

TTL determines how long a DNS result may remain cached.

Example:

```text
amazon.com → 1.2.3.4
TTL = 300 seconds
```

The resolver may reuse that result for:

```text
5 minutes
```

without querying the authoritative DNS again.

After TTL expires:

```text
Resolver performs another lookup
```

---

# 14. Why DNS Caching Exists

Without caching:

```text
Every request
   ↓
Root
   ↓
TLD
   ↓
Authoritative DNS
```

That would create enormous traffic.

Caching improves:

```text
Performance
Latency
Scalability
DNS infrastructure load
```

Caches may exist at:

```text
Browser
OS
Local network
ISP resolver
Public DNS resolver
```

---

# 15. DNS Cache Problem

Suppose:

```text
old IP = 10.0.0.10
new IP = 10.0.0.20
```

You update DNS.

But TTL is:

```text
3600 seconds
```

Some clients may continue using:

```text
10.0.0.10
```

for up to an hour because they cached the old record.

This is why TTL matters during migrations.

A common strategy is:

```text
Before migration:
Lower TTL

Perform migration

After things stabilize:
Increase TTL again
```

---

# 16. UDP vs TCP for DNS

DNS normally uses:

```text
UDP 53
```

because UDP has low overhead.

Basic flow:

```text
Client
  ↓ UDP 53
DNS Server
  ↓
Response
```

DNS can also use:

```text
TCP 53
```

Examples where TCP may be used include:

```text
Large responses
Zone transfers
Fallback when required
```

So do not say:

```text
DNS only uses UDP.
```

Correct interview answer:

> DNS commonly uses UDP port 53 for queries, but TCP port 53 is also supported and used when necessary.

---

# 17. DNS Zone Transfer

DNS servers may replicate zone information.

Two common mechanisms:

```text
AXFR → Full zone transfer
IXFR → Incremental zone transfer
```

Zone transfers normally use:

```text
TCP port 53
```

---

# 18. DNS over HTTPS

Traditional DNS:

```text
DNS → UDP/TCP 53
```

Modern DNS may also be encrypted.

## DoH

**DNS over HTTPS**

DNS queries are sent over HTTPS.

Commonly:

```text
TCP 443
```

or modern HTTPS transport mechanisms.

---

# 19. DNS over TLS

**DoT = DNS over TLS**

DNS traffic is encrypted using TLS.

Common port:

```text
853
```

Both DoH and DoT help prevent simple plaintext inspection of DNS queries while in transit.

---

# 20. Forward DNS vs Reverse DNS

## Forward DNS

```text
amazon.com
   ↓
IP address
```

Uses records such as:

```text
A
AAAA
```

---

## Reverse DNS

```text
IP address
   ↓
hostname
```

Uses:

```text
PTR record
```

---

# 21. DNS and Load Balancing

A hostname does not necessarily have only one IP.

Example:

```text
app.example.com
   ↓
10.0.1.10
10.0.2.10
10.0.3.10
```

DNS can return multiple addresses.

DNS-based routing can also influence where clients are sent.

AWS Route 53 supports routing policies such as:

```text
Simple
Weighted
Latency-based
Failover
Geolocation
Geoproximity
Multi-value
```

---

# 22. DNS Failover Concept

Suppose:

```text
Primary server   → healthy
Secondary server → standby
```

DNS can normally return:

```text
Primary IP
```

If health checks detect failure, DNS may start returning:

```text
Secondary IP
```

This provides DNS-level failover.

Remember that DNS caching and TTL mean failover is not always instantaneous for every client.

---

# 23. DNS and AWS Route 53

**Amazon Route 53** is AWS's managed DNS service.

It provides:

```text
Domain registration
DNS hosting
Hosted zones
DNS records
Health checks
Traffic routing
Private DNS
```

---

# 24. Public Hosted Zone

A **public hosted zone** contains DNS records accessible through public DNS.

Example:

```text
mycompany.com
```

Records could include:

```text
www.mycompany.com
api.mycompany.com
```

---

# 25. Private Hosted Zone

A **private hosted zone** provides DNS resolution inside associated VPCs.

Example:

```text
database.internal
```

might resolve only from inside the company's AWS environment.

Useful for internal applications.

---

# 26. Route 53 Alias Record

AWS has a special concept called an **Alias record**.

It can point DNS names to AWS resources such as:

```text
Application Load Balancer
CloudFront
S3 website endpoints
Other Route 53 records
```

Conceptually:

```text
app.example.com
      ↓
ALB
```

Alias records are especially useful at the zone apex where traditional CNAME usage may be restricted.

---

# 27. DNS Search Flow on Linux

The system has DNS resolver configuration.

Depending on the Linux setup, useful commands include:

```bash
cat /etc/resolv.conf
```

This may show:

```text
nameserver 8.8.8.8
```

Meaning DNS queries are sent toward that configured resolver.

Modern distributions may use:

```bash
resolvectl status
```

---

# 28. DNS Troubleshooting Commands

## nslookup

```bash
nslookup amazon.com
```

Useful for simple name resolution testing.

---

## dig

```bash
dig amazon.com
```

Provides detailed DNS information.

Important fields include:

```text
QUESTION
ANSWER
AUTHORITY
ADDITIONAL
TTL
Query time
DNS server
```

---

## Query specific record

```bash
dig amazon.com A
```

```bash
dig amazon.com AAAA
```

```bash
dig amazon.com MX
```

```bash
dig amazon.com NS
```

---

## Query a specific DNS server

```bash
dig @8.8.8.8 amazon.com
```

Meaning:

```text
Ask Google's DNS server directly.
```

---

## Trace DNS resolution

```bash
dig +trace amazon.com
```

This helps show the hierarchy:

```text
Root
 ↓
TLD
 ↓
Authoritative DNS
```

Very useful for understanding DNS resolution.

---

## host

```bash
host amazon.com
```

Simple DNS lookup command.

---

# 29. DNS Troubleshooting Flow

Suppose:

```text
curl https://myapp.com
```

fails with:

```text
Could not resolve host
```

Start with DNS.

### Step 1

```bash
nslookup myapp.com
```

or:

```bash
dig myapp.com
```

### Step 2

Check configured DNS resolver:

```bash
cat /etc/resolv.conf
```

### Step 3

Try another resolver:

```bash
dig @8.8.8.8 myapp.com
```

### Step 4

Check authoritative records:

```bash
dig myapp.com NS
```

### Step 5

Check TTL / stale cache.

---

# 30. Common DNS Problems

## Wrong DNS Record

Example:

```text
app.example.com → old IP
```

Result:

```text
Users reach wrong server.
```

---

## DNS Record Missing

```text
app.example.com
```

has no A/AAAA/CNAME record.

Result:

```text
NXDOMAIN
```

---

## DNS Resolver Unreachable

Client cannot reach:

```text
DNS server:53
```

Result:

```text
DNS queries timeout.
```

---

## Firewall Blocking DNS

If:

```text
UDP 53 blocked
```

DNS may fail.

TCP 53 may also need to be permitted depending on the environment.

---

## Expired/Stale Cache

DNS record changed but old result remains cached because TTL has not expired.

---

## Incorrect CNAME

Example:

```text
app.example.com
    ↓
wrong.example.net
```

Traffic ends up at the wrong destination.

---

## Authoritative DNS Problem

The authoritative DNS server may be:

```text
Unavailable
Misconfigured
Missing records
```

---

# 31. DNS Response Codes

Some useful response types:

```text
NOERROR  → Query succeeded
NXDOMAIN → Domain/name does not exist
SERVFAIL → DNS server failed to answer properly
REFUSED  → DNS server refused the query
```

These are useful during troubleshooting.

---

# 32. DNS Does Not Create the Web Connection

Important interview distinction:

DNS only gives you:

```text
Hostname → IP
```

After that:

```text
DNS complete
   ↓
Routing
   ↓
TCP/QUIC
   ↓
TLS
   ↓
HTTP
```

So if:

```text
nslookup amazon.com
```

works, that only proves:

> DNS resolution works.

It does **not prove port 443 or HTTP is working**.

---

# 33. Real Example — amazon.com

```text
User enters:
https://amazon.com
```

### DNS Stage

```text
amazon.com
   ↓
DNS resolver
   ↓
Amazon IP
```

### Networking Stage

```text
Laptop checks routing
   ↓
Router
   ↓
Internet
   ↓
Amazon network
```

### Transport Stage

Traditional HTTPS:

```text
TCP connection → port 443
```

### Security

```text
TLS handshake
```

### Application

```text
HTTP request
```

---

# 34. Interview Scenario 1

### Question

User can ping `8.8.8.8`, but:

```text
ping google.com
```

fails.

### Likely area

```text
IP connectivity works
DNS resolution probably broken
```

Check:

```bash
dig google.com
cat /etc/resolv.conf
```

---

# 35. Interview Scenario 2

### Question

`nslookup app.company.com` returns the correct IP, but the website doesn't open.

DNS is probably **not the issue**.

Next investigate:

```text
Routing
Firewall
TCP 443
TLS
Application
```

Try:

```bash
nc -vz app.company.com 443
curl -v https://app.company.com
```

---

# 36. Interview Scenario 3

### Question

You changed a DNS record, but some users still reach the old server.

Likely cause:

```text
DNS caching / TTL
```

Some resolvers still have the old response cached.

---

# 37. Interview Scenario 4

### Question

Internal AWS users can resolve:

```text
database.internal
```

but users on the Internet cannot.

Possible explanation:

```text
Route 53 Private Hosted Zone
```

This is expected if the DNS record exists only inside a private hosted zone.

---

# 38. Interview Scenario 5

### Question

DNS works with:

```bash
dig @8.8.8.8 example.com
```

but normal:

```bash
dig example.com
```

fails.

Likely problem:

```text
Configured local DNS resolver
/etc/resolv.conf
systemd-resolved
Corporate DNS
VPN DNS configuration
```

---

# 39. Interview Scenario 6

### Question

Why can DNS use both UDP and TCP?

Answer:

> Most ordinary DNS queries use UDP because it is lightweight. DNS also supports TCP, including for cases such as larger responses and zone transfers.

---

# 40. Interview Scenario 7

### Question

What is the difference between A and CNAME?

```text
A:
hostname → IPv4
```

```text
CNAME:
hostname → another hostname
```

Example:

```text
app.example.com → 10.0.0.20
```

is an A record.

```text
app.example.com → loadbalancer.example.net
```

is a CNAME-style relationship.

---

# 41. Interview Scenario 8

### Question

What happens if DNS is unavailable?

Even if:

```text
Server
Network
TCP
Application
```

are healthy, users may not be able to reach:

```text
myapp.com
```

because they cannot discover its IP.

However, direct communication using a known IP may still work depending on the application and TLS/virtual-host requirements.

---

# 42. DNS Mental Model

Remember this:

```text
Name
 ↓
DNS
 ↓
IP
 ↓
Routing
 ↓
TCP/UDP
 ↓
Application
```

And the DNS hierarchy:

```text
Client
 ↓
Recursive Resolver
 ↓
Root
 ↓
TLD
 ↓
Authoritative DNS
 ↓
Answer
```

---

# 43. Quick Revision

```text
DNS = Domain Name System
OSI Layer = 7
Normal port = UDP 53
Also supports = TCP 53

A     → IPv4
AAAA  → IPv6
CNAME → hostname alias
MX    → mail server
NS    → authoritative DNS server
TXT   → text/verification/security
PTR   → reverse DNS
SOA   → zone authority metadata

TTL → how long DNS data can be cached

Root → points toward TLD
TLD → points toward authoritative DNS
Authoritative DNS → holds final domain records
Recursive resolver → performs lookup for client
```

## Most Important Interview Answer

If asked:

**What happens when DNS resolves amazon.com?**

Answer:

> The client first checks its local caches. If the record is not available, it sends the request to its configured recursive resolver. If the resolver also does not have the answer cached, it follows the DNS hierarchy by querying a root server, then the appropriate TLD server such as `.com`, and finally Amazon's authoritative name server. The authoritative server returns the DNS record, the recursive resolver caches it according to its TTL and returns the result to the client. The client can then use the returned IP address to begin the actual TCP/QUIC, TLS and HTTP communication.