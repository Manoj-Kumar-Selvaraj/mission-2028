# IP Address Basics

- An **IP (Internet Protocol) address** is a logical address assigned to a network interface/device for communication over an IP network.
- There are two versions: **IPv4** and **IPv6**.

## IPv4

- An **IPv4 address is 32 bits** long.
- It is divided into **4 octets**, with each octet containing **8 bits**.

```text
8 bits . 8 bits . 8 bits . 8 bits
  ↓        ↓        ↓        ↓
192   .   168   .    1   .   10
```

Binary representation:

```text
192        168        1          10
11000000   10101000   00000001   00001010
```

### Why does an octet range from 0–255?

An octet contains **8 bits**:

```text
128  64  32  16  8  4  2  1
```

The maximum value occurs when all 8 bits are `1`:

```text
11111111
```

```text
128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255
```

Therefore:

```text
Minimum = 00000000 = 0
Maximum = 11111111 = 255
```

**Key point:** IPv4 is **32 bits = 4 bytes**, and each octet can represent a value from **0–255**.

### IPv6 Basics

IPv6 was introduced mainly because **IPv4 addresses are limited**.

An **IPv6 address is 128 bits** long.

```text
IPv4 = 32 bits
IPv6 = 128 bits
```

IPv6 is written in **hexadecimal** instead of normal decimal.

Example:

```text
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

It has **8 groups**, and each group contains **16 bits**:

```text
16 × 8 = 128 bits
```

Each group has **4 hexadecimal digits**:

```text
2001 : 0db8 : 85a3 : 0000 : 0000 : 8a2e : 0370 : 7334
```

Hexadecimal uses:

```text
0-9 and A-F
```

For example:

```text
A = 10
B = 11
C = 12
D = 13
E = 14
F = 15
```

For now, remember just this:

> **IPv6 = 128-bit IP address, written as 8 hexadecimal groups.**

Next concept should be **why hexadecimal is used and how one IPv6 group equals 16 bits**.

IPv4 = 32 bits  → 2^32  ≈ 4.3 billion IPs
IPv6 = 128 bits → 2^128 ≈ 3.4 × 10^38 IPs

Next: **Public IP vs Private IP**.

This is important before subnetting, NAT, VPCs, and routing.

### Public IP
A **public IP** is globally routable on the Internet.

Example:

```text
8.8.8.8
```

### Private IP
A **private IP** is used inside internal networks and is **not directly routable on the public Internet**.

IPv4 private ranges:

```text
10.0.0.0      - 10.255.255.255      → 10.0.0.0/8
172.16.0.0    - 172.31.255.255      → 172.16.0.0/12
192.168.0.0   - 192.168.255.255     → 192.168.0.0/16
```

Typical example:

```text
Laptop → 192.168.1.10   (private IP)
Router → public IP from ISP
```

To access the Internet, the router usually performs **NAT**.

For now remember:

> **Private IP = internal network**  
> **Public IP = Internet-routable**

Think of an IP address as having **two pieces**:

```text
Network part + Host part
```

Example:

```text
192.168.1.10/24
```

With `/24`:

```text
192.168.1 | 10
----------|---
 Network  | Host
```

Here:

- `192.168.1` identifies **which network**
- `10` identifies **which device inside that network**

Similar to:

```text
Street name + House number
```

Example:

```text
Network = 192.168.1.0/24

192.168.1.10  → device 10
192.168.1.20  → device 20
192.168.1.50  → device 50
```

All of them belong to the **same network** because their network portion is the same:

```text
192.168.1
```

The last part changes to identify different hosts.

So the simplest meaning is:

> **Network part = identifies the network**  
> **Host part = identifies a device inside that network**

### Subnet Mask — Basics

A **subnet mask tells us which part of an IPv4 address is the network part and which part is the host part**.

Example:

```text
IP:          192.168.1.10
Subnet Mask: 255.255.255.0
```

This is commonly written as:

```text
192.168.1.10/24
```

Why `/24`?

Because:

```text
255 = 11111111
255 = 11111111
255 = 11111111
0   = 00000000
```

So the subnet mask is:

```text
11111111.11111111.11111111.00000000
```

There are **24 ones**, so:

```text
/24
```

For now, remember:

> `/24` means the first 24 bits are the **network portion**, and the remaining 8 bits are for **hosts**.