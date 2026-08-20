# Routing - Main Elements

## Source IP
IP address of the device that sends the packet.
Example: Your laptop `192.168.1.10`.

## Destination IP
IP address of the final device the packet must reach.
Example: Google DNS `8.8.8.8`.

## Subnet
Defines which IP addresses belong to the same local network.
Used to decide whether traffic can stay local or must go through a router.

## Switch
Connects devices inside the same local network.
Mainly forwards Ethernet frames using MAC addresses.

## Router
Connects different networks together.
Reads the destination IP and forwards packets toward the correct network.

## Routing Table
A list of routing rules stored on a host or router.
It tells the system where traffic for a particular destination network should go.

## Route
A single rule inside the routing table.
Example: `10.0.0.0/16 → Gateway A`.

## Next Hop
The next router/device that should receive the packet.
The sender does not need to know the entire path, only the next hop.

## Default Gateway
The router used when there is no more specific route for the destination.
Commonly used to send traffic from a local network toward the Internet.

## Network Interface
The network connection through which traffic enters or leaves a device.
Examples: Ethernet interface, Wi-Fi interface, AWS ENI.

## Metric / Priority
Used when multiple routes can reach the same destination.
The system generally prefers the route with the better/lower metric.


Take this example:

```text
Laptop: 192.168.1.10
Gateway: 192.168.1.1
Destination: 8.8.8.8
```

When your laptop sends traffic to `8.8.8.8`:

```text
1. Application creates data
2. OS sees destination IP = 8.8.8.8
3. OS checks: is 8.8.8.8 in my local network?
4. No
5. OS checks routing table
6. Routing table says: send outside traffic to 192.168.1.1
7. 192.168.1.1 is the default gateway/router
8. Laptop sends the packet to that router
9. Router checks its own routing table
10. Router forwards the packet toward the next router
11. This repeats until the packet reaches 8.8.8.8
```

So the core flow is:

```text
Application
   ↓
Destination IP
   ↓
Local or remote?
   ↓
Remote
   ↓
Routing table
   ↓
Default gateway
   ↓
Router
   ↓
Next router
   ↓
...
   ↓
Destination
```

The important point is:

> **Your laptop does not know the entire path to 8.8.8.8. It only needs to know the next hop.**

That **next-hop concept** is the heart of routing.

Next: **Next Hop**.

A **next hop** is simply the **next router/device that should receive the packet on its way to the final destination**.

Example:

```text
Destination: 10.20.0.0/16
Next Hop:    192.168.1.1
```

Meaning:

> If traffic is going to `10.20.x.x`, send it first to `192.168.1.1`.

The sender does **not** need to know the full end-to-end path. It only needs to know the **next hop**.

## Default Gateway vs Next Hop

**Next hop** is the general concept: the **next router/device** to which a packet should be sent.

**Default gateway** is a specific next hop used when **no more-specific route matches**.

Example:

```text id="wz4r7k"
10.0.0.0/8  → 192.168.1.2   # specific next hop
0.0.0.0/0   → 192.168.1.1   # default gateway
```

So:

```text id="1cn4l9"
Destination 10.5.5.5
→ Next hop = 192.168.1.2

Destination 8.8.8.8
→ No specific route
→ Default gateway = 192.168.1.1
```

**Simple rule:**

> Every default gateway is a **next hop**, but not every next hop is the **default gateway**.

Next: **How a router forwards a packet from router → router until it reaches the destination.**

## Router → Router Flow

Suppose your laptop wants to reach `8.8.8.8`.

```text id="tfx5x9"
Laptop → Router A → Router B → Router C → 8.8.8.8
```

Each router does basically the same thing:

```text id="yjj11f"
Receive packet
      ↓
Read destination IP
      ↓
Check its routing table
      ↓
Find matching route
      ↓
Send to next hop
```

Important point:

> **Router A doesn't need to know the entire path. It only needs to know where to send the packet next.**

Then Router B makes **its own routing decision**, Router C makes its own, and so on until the destination network is reached.

This is the fundamental idea behind **IP routing**.


## Static vs Dynamic Routing

### Static Routing

A route is **manually configured** by an administrator.

```text
10.20.0.0/16 → Router A
```

Simple and predictable, but if the network topology changes, someone/system must update the route.

### Dynamic Routing

Routers **exchange routing information using routing protocols** and learn/update routes automatically.

Examples:

```text
OSPF
BGP
```

Useful for large networks where manually maintaining every route isn't practical.

### Easy distinction

> **Static = we configure the route.**  
> **Dynamic = routing protocols learn/exchange routes.**

For cloud/DevOps interviews, **BGP is particularly important** because you'll encounter it with AWS Site-to-Site VPN and Direct Connect.

You're right — the missing piece is **how the learning actually happens**.

## BGP: how routes are learned

Take two routers:

```text
Company Router
AS 65001

AWS Router
AS 64512
```

They first form a **BGP neighbor relationship**.

That happens over:

```text
TCP port 179
```

So first:

```text
Company Router ←→ TCP connection ←→ AWS Router
```

Once the BGP session is established, they exchange routing messages.

For example, Company Router sends:

```text
Network I know:
10.0.0.0/16

Reach it through:
me
```

AWS receives that advertisement and stores it in its **BGP table**.

Conceptually:

```text
BGP table on AWS:

10.0.0.0/16
Next hop → Company Router
Learned via → BGP
```

AWS may then install that route into its actual **routing table**, if it is the best valid route.

So the flow is:

```text
1. Routers establish BGP session
2. Router A advertises a network prefix
3. Router B receives the advertisement
4. Router B stores it in BGP routing information
5. BGP evaluates whether it is the best path
6. Best route gets installed into the routing table
7. Traffic can now use that route
```

### What exactly is being sent?

Not the actual traffic.

BGP exchanges **routing information** like:

```text
Prefix:    10.0.0.0/16
Next Hop:  192.168.100.1
AS Path:   65001
```

So AWS effectively learns:

> To reach `10.0.0.0/16`, send packets toward this BGP neighbor.

That is what "BGP learns a route" really means.

Next, the important thing to understand is **AS Path and how BGP decides which advertised route is better**.

## AS and AS Path

First, **AS = Autonomous System**.

An AS is basically a network, or group of networks, managed under one routing administration.

For example:

```text id="4pjc84"
Company A → AS 65001
ISP       → AS 65002
Company B → AS 65003
```

Each AS gets an **ASN (Autonomous System Number)**.

Now imagine:

```text id="pqy8re"
Company A      ISP        Company B
 AS65001  →  AS65002  →   AS65003
```

Company B owns:

```text id="59m2fj"
10.20.0.0/16
```

Company B advertises:

```text id="0fpg2i"
10.20.0.0/16
AS Path: 65003
```

ISP receives it and advertises it onward:

```text id="guc6jd"
10.20.0.0/16
AS Path: 65002 65003
```

Company A now learns:

```text id="f3rb7k"
Destination: 10.20.0.0/16
AS Path:     65002 → 65003
```

Meaning:

```text id="9i46ph"
To reach 10.20.0.0/16:

Company A
   ↓
AS65002
   ↓
AS65003
   ↓
10.20.0.0/16
```

### Why keep the AS Path?

One reason is **path selection**.

Suppose Company A learns two routes:

```text id="1c0m8c"
Path 1: 65002 → 65003
Path 2: 65004 → 65005 → 65003
```

All else being equal, BGP generally prefers the **shorter AS path**:

```text id="x85vzl"
Path 1 → 2 AS hops
Path 2 → 3 AS hops

→ Prefer Path 1
```

Another major reason is **loop prevention**.

If AS65003 receives an advertisement containing:

```text id="7e7ldh"
AS Path: 65002 → 65003
```

it sees **its own ASN (`65003`) already in the path** and rejects that route.

So remember:

> **AS = independently administered routing domain.**  
> **ASN = its number.**  
> **AS Path = list of autonomous systems a BGP route advertisement has passed through.**

Route A:
10.20.0.0/16 → ISP-A
AS Path: 65002 → 65003

Route B:
10.20.0.0/16 → ISP-B
AS Path: 65004 → 65005 → 65003


BGP cannot install both as the single preferred path in the ordinary case, so it runs its best-path selection algorithm.


If you run **BGP privately**, such as between your company router and AWS over a private setup, you can use a **private ASN**. You don’t have to globally register it.

Private ASN ranges include:

```text
64512–65534
4200000000–4294967294
```

If you want to participate in **public Internet BGP**, then yes, you generally need a **public ASN** assigned through the appropriate Regional Internet Registry, such as APNIC for much of Asia-Pacific.

So:

> **Private BGP → private ASN is enough**  
> **Public Internet BGP → public ASN required**

Now that **interface** is clear, the three route types become easier:

### 1. Connected Route

A router automatically knows networks that are directly attached to its interfaces.

```text
Router Interface 1: 192.168.1.1/24
```

So the router automatically knows:

```text
192.168.1.0/24 → directly connected
```

No manual route is needed.

---

### 2. Static Route

A route manually added by an administrator.

Example:

```text
10.20.0.0/16 → next hop 192.168.1.2
```

Meaning:

> To reach `10.20.0.0/16`, send traffic to router `192.168.1.2`.

---

### 3. Default Route

Used when no specific route matches.

```text
0.0.0.0/0 → 192.168.1.1
```

Meaning:

> If I don't know a better route, send the packet to `192.168.1.1`.

### Simple memory

```text
Connected → I am directly attached to that network
Static    → Someone manually told me the route
Default   → Use this when nothing else matches
```

## Next: Network Interface & Packet Forwarding

A router normally has **multiple network interfaces**, connecting it to different networks.

Example:

```text
Network A                         Network B
192.168.1.0/24                   10.0.0.0/24
      |                                |
      | 192.168.1.1        10.0.0.1   |
      +---- Interface 1  Router  Interface 2 ----+
```

Suppose:

```text
192.168.1.10 → wants to reach → 10.0.0.20
```

Router receives the packet on **Interface 1**, checks its routing table, and sees:

```text
10.0.0.0/24 → Interface 2
```

So it forwards the packet out through **Interface 2**.

That's **packet forwarding**:

> **Packet comes into one interface → router checks destination IP/routing table → packet leaves through the appropriate interface.**
