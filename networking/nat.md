
## NAT — Basics

**NAT = Network Address Translation**

It changes IP address information in packets while traffic passes through a router/firewall.

Most common example:

```text
Laptop private IP: 192.168.1.10
Router public IP:  49.x.x.x
```

When the laptop accesses the Internet:

```text
192.168.1.10 → Router/NAT → 49.x.x.x → Internet
```

The Internet does not see your private IP. It sees the router’s **public IP**.

So remember:

NAT by itself is mainly outbound + return traffic. New inbound connections need explicit forwarding/DNAT.

> **NAT lets private IP devices communicate with external networks using another IP, usually a public IP.**

## SNAT

**SNAT = Source Network Address Translation**

It changes the **source IP address** of an outgoing packet.

Example:

```text
Before NAT:
Source      = 192.168.1.10
Destination = 8.8.8.8
```

After SNAT:

```text
Source      = 49.x.x.x
Destination = 8.8.8.8
```

So:

> **SNAT = change the source IP, usually for outbound traffic.**

Think of **SNAT** as:

> **Change who the packet appears to come from.**

Example:

```text
Laptop private IP: 192.168.1.10
Router public IP:  49.10.20.30
Destination:       8.8.8.8
```

Laptop sends:

```text
From: 192.168.1.10
To:   8.8.8.8
```

Before sending it to the Internet, NAT changes only the **source**:

```text
From: 49.10.20.30
To:   8.8.8.8
```

So `8.8.8.8` sees the request as coming from:

```text
49.10.20.30
```

not from:

```text
192.168.1.10
```

That is **Source NAT**, or **SNAT**.

Simple memory:

```text
S in SNAT = Source IP changes
```


Your laptop has a private IP:

```text
192.168.1.10
```

Your router has a public IP:

```text
49.10.20.30
```

When your laptop sends traffic to Google:

```text
192.168.1.10 → Router → Google
```

The router changes this:

```text
From: 192.168.1.10
To:   8.8.8.8
```

into this:

```text
From: 49.10.20.30
To:   8.8.8.8
```

Why?

Because `192.168.1.10` is a **private IP** and cannot be used on the public Internet.

So the router replaces the laptop’s private source IP with its own public IP.

That specific action is called:

> **SNAT = changing the source IP**

And:

> **NAT = the general concept of changing IP addresses**

So SNAT is just **one type of NAT**.

**SNAT is just an action/type of NAT** where the **source IP gets changed**.

Next is **DNAT**.

**DNAT = Destination Network Address Translation**

This changes the **destination IP** of an incoming packet.

Example:

```text
Internet user sends to:

49.10.20.30:8080
```

DNAT can change that destination to:

```text
192.168.1.10:80
```

So:

```text
Before DNAT:
Destination = 49.10.20.30

After DNAT:
Destination = 192.168.1.10
```

Think:

> **SNAT = change source**  
> **DNAT = change destination**

DNAT is commonly used for **inbound traffic / port forwarding**.

## PAT — Port Address Translation

PAT lets **many private devices share one public IP** by using different source ports.

Example:

```text
Laptop A → 192.168.1.10:50001
Laptop B → 192.168.1.20:50002
```

After PAT:

```text
49.10.20.30:60001
49.10.20.30:60002
```

Same public IP, different ports.

> **PAT = many private IPs share one public IP using ports.**

This is what most home routers actually use for Internet access.

Next: **Port Forwarding**.

Port forwarding is an inbound rule that sends traffic arriving on a public IP/port to a specific internal IP/port.

Example:

```text
Public: 49.10.20.30:8080
            ↓
Forward to
            ↓
Private: 192.168.1.10:80
```

So an external user connecting to:

```text
49.10.20.30:8080
```

actually reaches:

```text
192.168.1.10:80
```

This is usually implemented using **DNAT**.

> **Port forwarding = expose a specific internal service through a public IP/port.**


