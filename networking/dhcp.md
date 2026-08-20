Next: **DHCP**.

## DHCP — What is it?

**DHCP = Dynamic Host Configuration Protocol**

Its job is to automatically give a device:

- IP address
- Subnet mask
- Default gateway
- DNS server

Example:

```text
Laptop connects to Wi-Fi
        ↓
DHCP Server
        ↓
IP:      192.168.1.20
Gateway: 192.168.1.1
DNS:     8.8.8.8
```

Without DHCP, you would have to configure these values manually.

> **DHCP = automatically gives network configuration to devices.**

Next we can see **how DHCP actually assigns the IP using DORA**.

## DHCP DORA Flow

When a laptop joins a network and has no IP yet:

```text
1. Discover
2. Offer
3. Request
4. Acknowledge
```

### 1. Discover
Laptop broadcasts:

```text
Is there any DHCP server?
```

### 2. Offer
DHCP server replies:

```text
I can give you 192.168.1.20
```

### 3. Request
Laptop says:

```text
I want 192.168.1.20
```

### 4. Acknowledge
DHCP server confirms:

```text
192.168.1.20 is assigned to you
```

Easy memory:

> **DORA = Discover → Offer → Request → Acknowledge**

DHCP usually uses **UDP 67 and 68**.

Next: **Firewall + stateful vs stateless**.

