## VPN — Basics

**VPN = Virtual Private Network**

It creates an encrypted tunnel between two networks or devices over an untrusted network like the Internet.

Example:

```text
On-prem Network
10.0.0.0/16
      ↓
   VPN Tunnel
      ↓
AWS VPC
172.16.0.0/16
```

Traffic between the two networks travels through that tunnel securely.

Common uses:

- On-prem ↔ AWS connectivity
- Employee remote access
- Site-to-site connectivity
- Secure communication over public Internet

> **VPN = secure tunnel over another network.**

Next: **Site-to-Site VPN vs Client VPN**.

## Site-to-Site VPN vs Client VPN

### Site-to-Site VPN

Connects **one network to another network**.

```text
On-prem office network
10.0.0.0/16
      ↓
VPN tunnel
      ↓
AWS VPC
172.16.0.0/16
```

Used when:

- Data center must talk to AWS
- Branch office must talk to cloud
- Entire networks need private connectivity

> **Site-to-Site = network ↔ network**

---

### Client VPN

Connects **one user/device to a private network**.

```text
Employee Laptop
      ↓
VPN client
      ↓
Company / AWS private network
```

Used for:

- Work from home
- Admin access
- Private application access

> **Client VPN = user/device ↔ network**
