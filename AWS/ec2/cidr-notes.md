# CIDR Block Calculation — AWS Networking Notes

## 1. What is CIDR?

**CIDR = Classless Inter-Domain Routing**

CIDR is used to define a **range of IP addresses**.

Example:

```text
10.0.0.0/24
```

This contains two parts:

```text
10.0.0.0    /24
   │         │
   │         └── Prefix length / Network bits
   └──────────── Network address
```

For IPv4, every IP address contains **32 bits**.

Example:

```text
10.0.0.0/24

Network bits = 24
Host bits    = 32 - 24 = 8
```

---

# 2. CIDR Formula

Total number of IP addresses:

```text
2^(32 - CIDR prefix)
```

Example:

```text
10.0.0.0/24

32 - 24 = 8

2^8 = 256 IP addresses
```

Therefore:

```text
/24 = 256 total IPs
```

---

# 3. Important CIDR Sizes

| CIDR | Host Bits | Total IP Addresses |
|---|---:|---:|
| `/32` | 0 | 1 |
| `/31` | 1 | 2 |
| `/30` | 2 | 4 |
| `/29` | 3 | 8 |
| `/28` | 4 | 16 |
| `/27` | 5 | 32 |
| `/26` | 6 | 64 |
| `/25` | 7 | 128 |
| `/24` | 8 | 256 |
| `/23` | 9 | 512 |
| `/22` | 10 | 1,024 |
| `/21` | 11 | 2,048 |
| `/20` | 12 | 4,096 |
| `/19` | 13 | 8,192 |
| `/18` | 14 | 16,384 |
| `/17` | 15 | 32,768 |
| `/16` | 16 | 65,536 |

### Easy Memory

```text
/24 = 256
/25 = 128
/26 = 64
/27 = 32
/28 = 16
/29 = 8
/30 = 4
/31 = 2
/32 = 1
```

Every time the prefix increases by `1`:

```text
Number of IPs ÷ 2
```

Example:

```text
/24 = 256
/25 = 128
/26 = 64
/27 = 32
```

Every time the prefix decreases by `1`:

```text
Number of IPs × 2
```

---

# 4. Network Bits and Host Bits

Consider:

```text
192.168.1.0/24
```

IPv4 has:

```text
32 total bits
```

CIDR says:

```text
/24
```

Therefore:

```text
Network bits = 24
Host bits    = 8
```

The network portion identifies the network.

The host portion identifies individual IP addresses inside that network.

---

# 5. CIDR and Subnet Mask

CIDR prefixes can also be represented using subnet masks.

| CIDR | Subnet Mask |
|---|---|
| `/8` | `255.0.0.0` |
| `/16` | `255.255.0.0` |
| `/17` | `255.255.128.0` |
| `/18` | `255.255.192.0` |
| `/19` | `255.255.224.0` |
| `/20` | `255.255.240.0` |
| `/21` | `255.255.248.0` |
| `/22` | `255.255.252.0` |
| `/23` | `255.255.254.0` |
| `/24` | `255.255.255.0` |
| `/25` | `255.255.255.128` |
| `/26` | `255.255.255.192` |
| `/27` | `255.255.255.224` |
| `/28` | `255.255.255.240` |
| `/29` | `255.255.255.248` |
| `/30` | `255.255.255.252` |

---

# 6. Block Size Calculation

A useful method for manually calculating subnet ranges is:

```text
Block Size = 256 - Subnet Mask value
```

Example:

```text
192.168.1.0/26
```

`/26` subnet mask:

```text
255.255.255.192
```

Calculate:

```text
256 - 192 = 64
```

Therefore subnet boundaries increment by:

```text
64
```

So the valid subnet boundaries are:

```text
192.168.1.0
192.168.1.64
192.168.1.128
192.168.1.192
```

---

# 7. `/26` Example

Given:

```text
192.168.1.0/26
```

Total addresses:

```text
2^(32 - 26)

2^6 = 64
```

Range:

```text
192.168.1.0
      to
192.168.1.63
```

Next subnet:

```text
192.168.1.64/26
```

Range:

```text
192.168.1.64
      to
192.168.1.127
```

Next:

```text
192.168.1.128/26
```

Range:

```text
192.168.1.128
      to
192.168.1.191
```

Next:

```text
192.168.1.192/26
```

Range:

```text
192.168.1.192
      to
192.168.1.255
```

---

# 8. Splitting a `/24` into Smaller Subnets

Suppose we have:

```text
10.0.0.0/24
```

Total:

```text
256 IPs
```

We need **4 equal subnets**.

Number of subnets required:

```text
4 = 2^2
```

Therefore borrow:

```text
2 host bits
```

Original:

```text
/24
```

New:

```text
/24 + 2 = /26
```

Result:

```text
10.0.0.0/26
10.0.0.64/26
10.0.0.128/26
10.0.0.192/26
```

Each subnet contains:

```text
64 total addresses
```

---

# 9. Splitting `/24` into 2 Subnets

Original:

```text
10.0.0.0/24
```

Need:

```text
2 subnets
```

Since:

```text
2 = 2^1
```

Borrow 1 bit:

```text
/24 + 1 = /25
```

Subnets:

```text
10.0.0.0/25
10.0.0.128/25
```

Each contains:

```text
128 IP addresses
```

---

# 10. Splitting `/24` into 8 Subnets

Need:

```text
8 subnets
```

Since:

```text
8 = 2^3
```

Borrow 3 bits:

```text
/24 + 3 = /27
```

Each `/27` contains:

```text
32 IP addresses
```

Subnets:

```text
10.0.0.0/27
10.0.0.32/27
10.0.0.64/27
10.0.0.96/27
10.0.0.128/27
10.0.0.160/27
10.0.0.192/27
10.0.0.224/27
```

---

# 11. AWS Subnet Reserved IP Addresses

AWS reserves **5 IP addresses in every IPv4 subnet**.

Example:

```text
10.0.1.0/24
```

Total:

```text
256 IPs
```

AWS usable:

```text
256 - 5 = 251 IPs
```

AWS reserves:

```text
10.0.1.0
```

Network address.

```text
10.0.1.1
```

VPC router.

```text
10.0.1.2
```

DNS-related use.

```text
10.0.1.3
```

Reserved by AWS for future use.

```text
10.0.1.255
```

Last address of the subnet, reserved by AWS.

Therefore:

```text
AWS usable IPs = Total IPs - 5
```

---

# 12. AWS Usable IP Examples

| CIDR | Total IPs | AWS Reserved | Usable IPs |
|---|---:|---:|---:|
| `/28` | 16 | 5 | 11 |
| `/27` | 32 | 5 | 27 |
| `/26` | 64 | 5 | 59 |
| `/25` | 128 | 5 | 123 |
| `/24` | 256 | 5 | 251 |
| `/23` | 512 | 5 | 507 |
| `/22` | 1,024 | 5 | 1,019 |
| `/20` | 4,096 | 5 | 4,091 |

---

# 13. `/16` VPC Example

Suppose:

```text
VPC = 10.0.0.0/16
```

Total IPs:

```text
2^(32 - 16)

2^16 = 65,536
```

Suppose we want `/20` subnets.

Difference:

```text
20 - 16 = 4 bits
```

Number of `/20` networks available:

```text
2^4 = 16
```

Each `/20` contains:

```text
2^(32 - 20)

2^12 = 4,096 IP addresses
```

---

# 14. `/20` Subnet Increment

`/20` subnet mask:

```text
255.255.240.0
```

Block size:

```text
256 - 240 = 16
```

Therefore third octet increases by:

```text
16
```

Subnets:

```text
10.0.0.0/20
10.0.16.0/20
10.0.32.0/20
10.0.48.0/20
10.0.64.0/20
10.0.80.0/20
10.0.96.0/20
10.0.112.0/20
...
```

---

# 15. `/19` Example

Suppose:

```text
10.20.32.0/19
```

Subnet mask:

```text
255.255.224.0
```

Block size:

```text
256 - 224 = 32
```

Therefore valid third-octet boundaries are:

```text
0
32
64
96
128
160
192
224
```

Since the subnet starts at:

```text
10.20.32.0
```

the next subnet starts at:

```text
10.20.64.0
```

Therefore this subnet range is:

```text
10.20.32.0
      to
10.20.63.255
```

Total IPs:

```text
2^(32 - 19)

2^13 = 8,192
```

---

# 16. How to Find the Network Range Quickly

Example:

```text
172.16.20.0/22
```

`/22` mask:

```text
255.255.252.0
```

Block size:

```text
256 - 252 = 4
```

Third-octet boundaries:

```text
0
4
8
12
16
20
24
28
...
```

`20` is a valid boundary.

Therefore:

```text
Network:
172.16.20.0

Next network:
172.16.24.0
```

Current range:

```text
172.16.20.0
      to
172.16.23.255
```

---

# 17. Example Where Given IP Is Not the Network Address

Suppose interviewer gives:

```text
10.10.35.25/20
```

A `/20` has block size:

```text
16
```

Valid third-octet boundaries:

```text
0
16
32
48
64
...
```

`35` falls between:

```text
32 - 47
```

Therefore network is:

```text
10.10.32.0/20
```

Range:

```text
10.10.32.0
      to
10.10.47.255
```

The IP:

```text
10.10.35.25
```

belongs to:

```text
10.10.32.0/20
```

---

# 18. Private IPv4 CIDR Ranges

Private networks commonly use RFC1918 address ranges:

```text
10.0.0.0/8
```

Range:

```text
10.0.0.0 - 10.255.255.255
```

---

```text
172.16.0.0/12
```

Range:

```text
172.16.0.0 - 172.31.255.255
```

---

```text
192.168.0.0/16
```

Range:

```text
192.168.0.0 - 192.168.255.255
```

These are commonly used for:

- VPCs
- corporate networks
- on-premises networks
- VPN connectivity
- private application networks

---

# 19. Avoiding CIDR Overlap

When connecting networks, their CIDRs should generally **not overlap**.

Bad design:

```text
AWS VPC
10.0.0.0/16

On-Premises
10.0.0.0/16
```

Problem:

```text
AWS cannot easily determine whether
10.0.1.20 belongs to AWS or on-prem.
```

This creates routing conflicts.

Better:

```text
AWS VPC:
10.10.0.0/16

On-Prem:
10.20.0.0/16
```

This is particularly important for:

- Site-to-Site VPN
- Direct Connect
- VPC Peering
- Transit Gateway
- hybrid connectivity

---

# 20. Example AWS VPC Design

Suppose:

```text
VPC = 10.0.0.0/16
```

We could allocate:

```text
Public Subnet AZ-A
10.0.0.0/24

Public Subnet AZ-B
10.0.1.0/24

Private App Subnet AZ-A
10.0.10.0/24

Private App Subnet AZ-B
10.0.11.0/24

Private DB Subnet AZ-A
10.0.20.0/24

Private DB Subnet AZ-B
10.0.21.0/24
```

Conceptually:

```text
VPC 10.0.0.0/16
|
├── Public AZ-A  → 10.0.0.0/24
├── Public AZ-B  → 10.0.1.0/24
|
├── App AZ-A     → 10.0.10.0/24
├── App AZ-B     → 10.0.11.0/24
|
├── DB AZ-A      → 10.0.20.0/24
└── DB AZ-B      → 10.0.21.0/24
```

---

# 21. How Many Subnets Can Be Created?

Formula:

```text
Number of Subnets = 2^(New Prefix - Original Prefix)
```

Example:

Split:

```text
/16
```

into:

```text
/20
```

Difference:

```text
20 - 16 = 4
```

Therefore:

```text
2^4 = 16 subnets
```

---

Another example:

Split:

```text
/24
```

into:

```text
/27
```

Difference:

```text
27 - 24 = 3
```

Therefore:

```text
2^3 = 8 subnets
```

---

# 22. How Many IPs Per Subnet?

Formula:

```text
IPs per subnet = 2^(32 - prefix)
```

Example:

```text
/27
```

Therefore:

```text
2^(32 - 27)
= 2^5
= 32 IPs
```

AWS usable:

```text
32 - 5 = 27
```

---

# 23. CIDR Interview Shortcut

For `/24` and smaller:

```text
/24 = 256
/25 = 128
/26 = 64
/27 = 32
/28 = 16
```

Block boundaries:

### `/25`

```text
0
128
```

### `/26`

```text
0
64
128
192
```

### `/27`

```text
0
32
64
96
128
160
192
224
```

### `/28`

```text
0
16
32
48
64
80
96
112
128
144
160
176
192
208
224
240
```

---

# 24. Important Interview Scenario

### Question

You have:

```text
VPC = 10.0.0.0/16
```

You need **4 equal-sized subnets**.

### Solution

Need 4 subnets:

```text
4 = 2^2
```

Borrow 2 bits:

```text
/16 + 2 = /18
```

Each `/18` contains:

```text
2^(32 - 18)
= 16,384 IPs
```

Subnets:

```text
10.0.0.0/18

10.0.64.0/18

10.0.128.0/18

10.0.192.0/18
```

---

# 25. Important Interview Scenario

### Question

How many IPs are available in:

```text
10.0.1.0/27
```

Calculation:

```text
32 - 27 = 5

2^5 = 32 total IPs
```

AWS reserves:

```text
5
```

Therefore:

```text
32 - 5 = 27 usable IPs
```

---

# 26. Important Interview Scenario

### Question

What CIDR would you select if approximately **50 EC2/private IPs** are required?

Check:

```text
/27 = 32 total
```

Not enough.

```text
/26 = 64 total
```

AWS usable:

```text
64 - 5 = 59
```

Therefore:

```text
/26
```

can accommodate approximately 50 AWS resources.

However, in a real production design, leave additional capacity for:

- Auto Scaling
- future expansion
- load balancers
- EKS ENIs/pods
- migrations
- temporary resources

So we may intentionally select a larger subnet.

---

# 27. Important Interview Scenario — EKS

EKS can consume many subnet IP addresses because:

```text
Worker Node
   |
   └── ENI
        |
        ├── Pod IP
        ├── Pod IP
        ├── Pod IP
        └── Pod IP
```

Therefore, when designing EKS VPCs:

> Do not create very small subnets just because the number of worker nodes is small.

Pod IP requirements must also be considered.

---

# 28. Important Interview Scenario — Hybrid Connectivity

Suppose:

```text
AWS VPC:
10.0.0.0/16

On-Prem:
10.0.0.0/8
```

These networks overlap.

That can cause routing ambiguity.

Before implementing:

```text
Site-to-Site VPN
Direct Connect
Transit Gateway
VPC Peering
```

always evaluate:

```text
CIDR allocation
+
CIDR overlap
+
future growth
```

---

# 29. Quick CIDR Calculation Process

When interviewer gives:

```text
10.20.35.10/19
```

Follow these steps:

### Step 1 — Find subnet mask

```text
/19 = 255.255.224.0
```

### Step 2 — Find block size

```text
256 - 224 = 32
```

### Step 3 — Find boundaries

```text
0
32
64
96
128
160
192
224
```

### Step 4 — Locate the IP

Third octet:

```text
35
```

falls between:

```text
32 - 63
```

### Step 5 — Network

```text
10.20.32.0/19
```

### Step 6 — Range

```text
10.20.32.0
      to
10.20.63.255
```

---

# Quick Memory Notes

- **CIDR** defines an IP address range.
- IPv4 contains **32 bits**.
- `/24` means **24 network bits + 8 host bits**.
- **Total IPs = `2^(32-prefix)`**.
- Larger `/` number = **smaller network**.
- Smaller `/` number = **larger network**.
- `/24 = 256 IPs`.
- `/26 = 64 IPs`.
- `/28 = 16 IPs`.
- AWS reserves **5 IPv4 addresses per subnet**.
- **Usable AWS IPs = Total IPs - 5**.
- **Block size = 256 - subnet-mask octet**.
- Avoid **overlapping CIDRs** when connecting VPCs/on-prem networks.
- Size AWS subnets for **current + future capacity**, especially for EKS and Auto Scaling.

## Essential Memory Table

```text
/16 = 65,536
/17 = 32,768
/18 = 16,384
/19 = 8,192
/20 = 4,096
/21 = 2,048
/22 = 1,024
/23 = 512
/24 = 256
/25 = 128
/26 = 64
/27 = 32
/28 = 16
/29 = 8
/30 = 4
/31 = 2
/32 = 1
```