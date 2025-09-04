---
created: 2025-07-20
---
**Table of Content**

```table-of-contents
```
## Switches

>A **switch** is a network device that operates primarily at Layer 2 (Data Link) of the OSI model, forwarding Ethernet frames based on **MAC addresses**.

>[!note] 
>**Layer 3 switches** combine Layer 2 switching with routing capabilities.

### Collision and broadcast domains

>A **collision domain** is a network segment where data packets can potentially collide if two devices attempt to transmit data simultaneously (i.e., over a shared medium).

- A switch segments a LAN into multiple collision domains, one collision domain for each switch port. 

- With a switch, each single link in a modern LAN is considered its own collision domain, even if no collisions can actually occur in that case.

>A **broadcast domain** is a network segment where all devices receive a copy of every broadcast frame sent.

>[!note] All devices connected to a switch in the LAN share the same broadcast domain.

### MAC address tables and switching logic

A switch is responsible for forwarding Ethernet frames between devices within the same LAN based on the MAC addresses specified in the Ethernet frame headers and the corresponding ports found in the **MAC address table** of that switch. This process is called **switching**.

>A **MAC address table**, aka **CAM table**, is a data structure used by network switches to map MAC (Media Access Control) address of the devices connected to the switch to the corresponding switch ports.

The MAC address table is used to decide out of which port a frame with a given destination MAC address should be forwarded (and whether to forward it at all).  

>[!note]
>The **CAM (Content Addressable Memory)** is a special type of memory used in switches to maintain MAC address tables. It is also known as **associative memory**, and is used in operations that require high-speed search through tables, such as switching decisions.

Each time a switch receives an Ethernet frame, it makes a **switching decision**: either to forward the frame out of certain ports (and what ports exactly), or ignore the frame.

The switching decision scheme looks like this:

1. **Filtering**
	- If the switchport corresponding to the destination address in the frame header is the same as the source port, filter (discard) the frame.
2. **Forwarding**
	- Otherwise, forward the frame on to the destination port.
3. **Flooding**
	- If the destination port is unknown, use **flooding** to discover it.
4. **Broadcasting**
	- If the destination address is the broadcast address (`ff:ff:ff:ff:ff:ff`), send the frame out of every available port on the same broadcast domain, except the source port of the frame.
### MAC address learning

>Switches build the address table by listening for incoming frames and examining the **source MAC address** of the frames.

This is how it works:
1. A frame arrives at a certain switch port. The switch checks its **source MAC address**.
2. If the source MAC address is already in the MAC address table, the switch proceeds to the switching decision.
3. If the source MAC address is not yet is the MAC address table, the switch creates a new entry in the MAC address table that associates this MAC address with the port on which the frame has arrived. After that, the switch proceeds to the switching decision.

When a switch later receives a frame with an already known destination MAC address, the switch looks up through the MAC address table, finds the corresponding physical port, and then sends the frame out of it. 

When a switch receives a frame the destination MAC address of which is not yet in the MAC address table, if uses the **flooding algorithm** to learn this MAC address.

>An **unknown unicast** is the frame whose destination MAC address is unknown to the switch.

>A **known unicast** is the frame whose destination MAC address is already known to the switch, i.e., those destination MAC address is already in the MAC address table of that switch.
### Flooding

Here is how the flooding algorithm works:

1. The switch receives an **unknown unicast frame**. 
2. The switch **broadcasts** (floods) the frame out of **all interfaces except the one the frame was received on**.
3. All devices in the LAN receive and inspect that broadcast frame. 
4. The host those MAC address **matches** the destination MAC address specified in the frame sends another frame to the switch. All other hosts ignore the frame.
5. The switch creates an entry in its MAC address table that associates the source MAC address of the frame with the switchport it has arrived on. 
6. When the discovered address is encountered as a destination address of another frame, the switch forwards the frame only to the associated port rather than flooding the frame again.

>MAC addresses learned by the switch with the flooding algorithm are called **dynamic MAC addresses**.

>When the switch is first plugged in, the MAC address table is empty: no destinations are known.

>[!important] Aging
>MAC entries time out after a period of inactivity. Once an entry has expires, the switch needs to learn the address again.
>This allows the table to remain current and accurate.

>[!important] Arrival time 
>To handle dynamic topologies, the arrival time of each frame is recorded and associated with the source MAC address of this frame. Whenever the switch receives another frame with an already known source MAC address, it updates the arrival time of that address in the MAC address table.

>[!important] By default, dynamic MAC addresses are removed from the MAC address table after **5 minutes (300 seconds)** of inactivity on Cisco switches.

>[!example]- Example: flooding algorithm
>For the clarification of the process of discovering new entries for the MAC address table, below a step-by-step description of how it works.
> Consider the process of sending a frame form a host A to a host B. 
> Let the MAC address of the host A be `0000.0000.000A`, and the MAC address of the host B `0000.0000.000B`, and suppose that the MAC addresses of the hosts are unknown to the switch.
> 
> 1. The host A sends a frame to the host B.
> 
> 2. The switch receives the frame on the `E0/0`interface, and records the source address of the host A, `0000.0000.000A`, to the MAC address table. the MAC address is now associated with the interface `E0/0`.
> 
> 3. The destination MAC address of the host B, `0000.0000.000B`, is unknown to the switch. it broadcasts the frame received from the host A to every available interface, except `E0/0`.
> 
> 4. Each host that receives a frame compares the destination address of the frame to its address. if the match occurs, the host responds to the frame, otherwise ignores the frame.
> 
> 5. The host B finds a match between the destination address of the received frame and its own MAC address, `0000.0000.000B`. it then responds to the host A with another frame.
> 
> 6. The switch receives the frame from the host B on the interface `E0/1` and records its MAC address to the table, associating the address with the interface. then forwards the frame to the host A through the port `E0/0`.
> 
> 7. The addresses of the host A and B are know located in the MAC address table of the switch, and the hosts can establish a point-to-point connection with each other.
> 
> 8. If Host A and Host B don’t communicate to the switch again within a certain amount of time, the switch will flush their entries from the database to keep it as current as possible.

## Switch interfaces

- To show information about switch interfaces:

```toml
SW1# show ip interface brief
```


>[!important]
>**Router** interfaces have the `shutdown` command applied by default.
>This means they will be in the `administratively down/down` state by default.
>---
>**Switch** interfaces do NOT have the `shutdown` command applied by default.
>This means they will be:
>- in the `up/up` state **if connected to another device**
>OR
>- in the `down/down` state **if not connected to another deivce**

- To show interface status:

```toml
SW1# show interface status
```

- To configure multiple interfaces at once:

```toml
SW1(config)# interface range f0/5 - 12
SW1(config-if-range)#

# the interface range may not be contiguous:
SW1(config)# int range f0/5 - 6, f0/9 - 12
```
### Speed

- To manually configure the speed at which the interface operates:

```toml
SW1(config-if)# speed ?
10    Force 10 Mbps operation
100   Force 100 Mbps operation
auto  Enable AUTO speed configuration
```

For example:

```toml
SW1(config-if)# speed 100
```

>[!important] Speed is configured automatically by default (auto-negotiated with the connected interfaces).

For example, in the output of `show interfaces status`, the `Speed` field set to `a-100` means the speed of `100` megabits per second was auto-negotiated.
### Duplex

- To manually set interface duplex:

```toml
SW1(config-if)# duplex ?
auto  Enable AUTO duplex configuration
full  Force full duplex operation
half  Force half-duplex operation
```

For example:

```toml
SW1(config)# duplex full
```

>[!important] Duplex is configured automatically by default (auto-negotiated with the connected interfaces).

For example, in the output of `show interfaces status`, the `Duplex` field set to `a-full` means the `full` duplex was auto-negotiated.

>[!important]
>- **Half-duplex**: the device **can't** send and receive data at the same time. If it is receiving a frame, **it must wait before sending a frame**.
>- **Full-duplex**: the device **can** send and receive data at the same time. It doesn't have to wait.
>
>In modern networks that use switches, all devices can use **full duplex** on their devices.

#### About connection speed

>**Connection speed** is the rate at which data can be transferred over a network or Internet connection.

- It is measured in **bits per second (bps)**, commonly expressed in kilobits (Kbps), megabits (Mbps), or gigabits per second (Gbps).
- Network interfaces, internet connections, and devices support different maximum speeds (e.g., 100 Mbps, 1 Gbps).

>[!important] Speed often refers to **bandwidth**, the maximum capacity of the connection channel.

---

- Most consumer ISPs provide **asymmetric connections**, meaning **download speeds are higher than upload speeds**. This is because a typical user downloads more data (websites, videos) than uploads.
- Upload speed is crucial for applications like:
    - Video conferencing (Zoom, Teams)
    - Online gaming (sending commands)
    - Cloud backups and file sharing
    - Streaming your own video content
- Upload speed tends to be smaller on residential plans but can be symmetric for business or fiber plans.

---

>Connection speed suitable for home networks
- **Basic web browsing, email**: ~1-5 Mbps is enough.
- **HD video streaming (Netflix, YouTube)**: 5-10 Mbps per stream.
- **4K video streaming**: 25 Mbps or more per stream.
- **Online gaming**: Often needs reliable latency more than extreme speed; 3-6 Mbps download and 1-3 Mbps upload is generally sufficient.
- **Video calls (Zoom, Skype)**: Requires upload speeds around 1.5-3 Mbps.
- **Multiple users/devices**: Total bandwidth should be aggregate of all expected traffic; e.g., family of 4 streaming + gaming may need 50+ Mbps.
- **General good home plan**: 100 Mbps download with 10-20 Mbps upload is well-balanced today.

>Connection speed suitable for offices/businesses
- Shared internet: Bandwidth aggregates for all employees.
- Remote working with many video calls: Symmetric upload/download of 50-100 Mbps or more preferred.
- Cloud service access, backups: Fast upload critical; direct fiber with 100 Mbps+ upload common.
- Enterprise LAN speed (switch ports): Typically 1 Gbps or higher, often multiple aggregated links.

