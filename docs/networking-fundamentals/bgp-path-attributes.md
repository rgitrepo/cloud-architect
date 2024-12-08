
### **What is NEXT_HOP?**
1. **Definition**: The `NEXT_HOP` attribute in BGP (Border Gateway Protocol) tells the router which IP address should be used as the next stop to forward packets to the destination.

2. **Purpose**: It ensures that routers know the next step in the journey of a packet through the network.

3. **Key Rule**: This is a **mandatory attribute** that must always be set.

---

### **How is NEXT_HOP Determined?**
BGP determines the `NEXT_HOP` value differently depending on the type of connection (internal vs. external peers) and how far the destination is.

---

| **No.** | **Original Text** | **Explanation with Multiple Practical Examples** |
|---------|--------------------|--------------------------------------------------|
| **1**   | When sending a message to an internal peer, if the route is not locally originated, the BGP speaker SHOULD NOT modify the NEXT_HOP attribute unless it has been explicitly configured to announce its own IP address as the NEXT_HOP. | **1. Internal Peer (Within the Same Network)** <br> - **Scenario**: When a router sends a route update to another router within the same organization or network. <br> **Rules**: <br> **1.1.** If the route is not **locally originated**: <br> - Do **not** change the `NEXT_HOP` unless explicitly configured to do so. <br> **Practical Examples**: <br> **Example 1:** Router A learns about a route `192.168.1.0/24` from Router B (internal) with `NEXT_HOP = 172.16.0.1`. Router A advertises this route to Router C (internal). The `NEXT_HOP` remains `172.16.0.1` unless Router A is explicitly configured to override it. <br> **Example 2:** Router A is configured to keep the `NEXT_HOP` of incoming routes unchanged. If Router B advertises `10.0.0.0/8` with `NEXT_HOP = 172.20.0.1`, Router A passes the same `NEXT_HOP` to Router C when sending this route. <br> **Example 3:** If Router A learns the route from an internal Router B and advertises it to Router C without modifying the `NEXT_HOP`, Router C knows that it needs to use Router B as the next hop to reach the destination. |
| **1.2** | When announcing a locally-originated route to an internal peer, the BGP speaker SHOULD use the interface address of the router through which the announced network is reachable for the speaker as the NEXT_HOP. | **1. Internal Peer (Within the Same Network)** <br> **1.2.** If the route is **locally originated**: <br> - Use the router's own interface address that is used to reach the destination as the `NEXT_HOP`. <br> **Practical Examples**: <br> **Example 1:** Router A originates a static route for `192.168.2.0/24` and advertises it to Router B. Router A sets its own interface address (`172.16.0.1`) as the `NEXT_HOP`. Router B knows to use Router A to forward traffic to this destination. <br> **Example 2:** If Router A is directly connected to network `10.0.1.0/24` and announces it to Router B, Router A’s interface IP (`192.168.0.1`) is set as the `NEXT_HOP`. <br> **Example 3:** A company’s data center router (Router A) originates a route to its internal network `10.10.10.0/24`. It advertises this to another internal router (Router B) with `NEXT_HOP` set to Router A’s interface IP. |
| **1.3** | If the route is directly connected to the speaker, or if the interface address of the router through which the announced network is reachable for the speaker is the internal peer's address, then the BGP speaker SHOULD use its own IP address for the NEXT_HOP attribute (the address of the interface that is used to reach the peer). | **1. Internal Peer (Within the Same Network)** <br> **1.3.** If the route is directly connected to the router or the peer's address is used to reach the network: <br> - The router sets its own IP as the `NEXT_HOP`. <br> **Practical Examples**: <br> **Example 1:** Router A directly connects to the network `172.16.2.0/24`. It advertises this route to Router B (internal) with `NEXT_HOP = 172.16.2.1` (Router A’s IP). <br> **Example 2:** Router A connects directly to `192.168.3.0/24` and advertises the route to Router C. Router A’s interface IP `192.168.3.1` is used as the `NEXT_HOP`. <br> **Example 3:** Router A connects to an internal subnet used for application servers (`10.20.30.0/24`). When advertising this route to Router B, Router A uses `NEXT_HOP = 10.20.30.1`. |
| **2**   | When sending a message to an external peer, X, and the peer is one IP hop away from the speaker: | **2. External Peer (One Hop Away)** <br> - **Scenario**: When a router sends a route update to a router outside its network. <br> **Rules**: Determine `NEXT_HOP` based on the origin of the route. |
| **2.1** | If the route being announced was learned from an internal peer or is locally originated, the BGP speaker can use an interface address of the internal peer router (or the internal router) through which the announced network is reachable for the speaker for the NEXT_HOP attribute, provided that peer X shares a common subnet with this address. | **2. External Peer (One Hop Away)** <br> **2.1.** If the route came from an internal peer or is locally originated: <br> - Use the IP of the internal peer/router, **if the external peer shares a subnet with this address**. <br> **Practical Examples**: <br> **Example 1:** Router A (internal) learns the route `10.10.10.0/24` from Router B (internal) with `NEXT_HOP = 172.16.0.1`. Router A advertises this to Router C (external), keeping `NEXT_HOP = 172.16.0.1` since Router C shares the same subnet. <br> **Example 2:** Router A advertises a locally originated route `192.168.5.0/24` to Router B (internal) with `NEXT_HOP = 172.20.0.1`. Router B forwards it to an external Router C, keeping `NEXT_HOP = 172.20.0.1` as they share a subnet. |
| **2.2** | Otherwise, if the route being announced was learned from an external peer, the speaker can use an IP address of any adjacent router (known from the received NEXT_HOP attribute) that the speaker itself uses for local route calculation in the NEXT_HOP attribute, provided that peer X shares a common subnet with this address. | **2. External Peer (One Hop Away)** <br> **2.2.** If the route came from an external peer: <br> - Use the IP of the adjacent external router, **if the external peer shares a common subnet with this address**. <br> **Practical Examples**: <br> **Example 1:** Router A (internal) learns a route `192.168.6.0/24` from External Router B with `NEXT_HOP = 10.0.0.1`. Router A advertises this to External Router C with `NEXT_HOP = 10.0.0.1`, provided Router B and Router C share a subnet. |
| **3.2** | By default, the BGP speaker SHOULD use the IP address of the interface that the speaker uses in the NEXT_HOP attribute to establish the BGP connection to peer X. | **3. External Peer (Multiple Hops Away)** <br> **3.2.** **Default**: <br> - Use the IP of the interface used to establish the BGP connection. <br> **Practical Examples**: <br> **Example 1:** Router A establishes a BGP session with Router B using interface `192.168.1.1`. Any route advertised by Router A to Router B has `NEXT_HOP = 192.168.1.1`. |

---


### **Key Principles of NEXT_HOP**
1. **Shortest Path**: The `NEXT_HOP` is typically chosen to ensure the shortest path is taken.
2. **No Self-Reference**:
   - A router cannot use its own IP as the `NEXT_HOP`.
3. **No Peer’s Address**:
   - A route cannot be advertised to a peer using that peer’s own address as the `NEXT_HOP`.

---

### **How is the NEXT_HOP Used?**
1. **Route Lookup**: The `NEXT_HOP` is used to look up the Routing Table.
   - The Routing Table determines:
     - The **outbound interface** (e.g., Ethernet0, Serial1).
     - The **immediate next-hop address** (e.g., `192.168.1.2`).

2. **Example**:
   - Routing Table Entry: 
     - Destination: `10.0.0.0/24`
     - NEXT_HOP: `192.168.1.2`
     - Outbound Interface: `Ethernet0`

   When sending a packet to `10.0.0.5`, the router forwards it to `192.168.1.2` via `Ethernet0`.

---


