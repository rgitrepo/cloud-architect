

### **BGP and Its Relationship with TCP**

**1. BGP Overview:**
- **Exterior Gateway Protocol (EGP):** BGP is designed to communicate routing information between different autonomous systems (AS), typically across the internet.
- **Routing Protocol:** It exchanges network-layer reachability information (NLRI) to determine the best paths for traffic.
- **Path Vector Protocol:** BGP uses the sequence of AS numbers traversed to reach a destination as part of its path selection process.

**2. TCP-Based Communication:**
- BGP operates over **TCP (Transmission Control Protocol)** to ensure reliable delivery of routing information.
- TCP port **179** is specifically reserved for BGP sessions.

---

### **Why TCP Port 179 Matters**

When configuring BGP, port **179** needs to be open in firewalls to allow communication between BGP peers. Here's why:

1. **Firewall and Stateful Inspection:**
   - A stateful firewall tracks the state of a connection.
   - If an internal router initiates a BGP session to an external router, the firewall allows the response because it tracks the connection's sequence numbers.

2. **Unidirectional Initiation:**
   - If an **external router** tries to initiate a connection to an **internal router**, the firewall will block it unless explicitly allowed, as the firewall doesn't know the connection's state.

---

### **TCP Three-Way Handshake and Its Role in BGP**

The **TCP three-way handshake** establishes a reliable connection between two devices (e.g., two BGP routers). Here's how it works step by step:

1. **SYN (Synchronization):**
   - The initiating router (Router A) sends a **SYN** message to the responding router (Router B).
   - The SYN contains a **sequence number** (e.g., Sequence 1) that starts the communication.

2. **SYN-ACK (Synchronization + Acknowledgment):**
   - Router B responds with a **SYN-ACK**, which:
     - Acknowledges Router A's **SYN** (Sequence 1).
     - Sends its own **SYN** with a new sequence number (e.g., Sequence 2).

3. **ACK (Acknowledgment):**
   - Router A sends an **ACK** to confirm Router B's sequence (Sequence 2) and moves to Sequence 3.

**Example Sequence:**
| **Step**   | **Message**       | **Sequence Number** | **Acknowledgment Number** |
|------------|-------------------|----------------------|---------------------------|
| Step 1     | SYN               | 1                   | -                         |
| Step 2     | SYN-ACK           | 2                   | 1                         |
| Step 3     | ACK               | 3                   | 2                         |

This ensures both routers are ready to exchange data. Once the handshake is complete, the BGP session can exchange routing updates reliably.

---

### **TCP Connection Teardown (Two Two-Way Handshakes)**

When a TCP session is terminated, it uses a **two two-way handshake** process:

1. **FIN (Finish):**
   - Router A sends a **FIN** to Router B, indicating it wants to terminate the connection.
2. **ACK (Acknowledge):**
   - Router B acknowledges Router A's **FIN** and continues to use the session if needed.
3. **FIN (Finish):**
   - Router B sends its own **FIN** to Router A to confirm termination.
4. **ACK (Acknowledge):**
   - Router A acknowledges Router B's **FIN** and closes the connection.

---

### **Firewalls and Access Control Lists (ACLs)**

1. **Stateful Firewalls:**
   - **Track Connection State:** Stateful firewalls monitor the sequences and states of TCP sessions.
   - **Allow Responses:** If an internal router initiates a BGP session, the firewall allows the external router's responses because it tracks the state.

2. **Stateless ACLs:**
   - **No State Awareness:** ACLs do not track connection states, so they must explicitly allow both inbound and outbound traffic.
   - Example:
     - Outbound rule: Allow TCP traffic to port 179.
     - Inbound rule: Allow TCP traffic from port 179.

**Combination of Firewall and ACL:**
- Firewalls handle stateful traffic tracking.
- ACLs are used for finer-grained control, such as restricting access to specific IP addresses.

---

### **Why Stateful Inspection and ACLs Matter for BGP**

- When using a firewall:
  - Only the initiating router needs to open the session.
  - The firewall allows responses because it tracks the TCP state.

- When using stateless ACLs:
  - Both inbound and outbound rules must be explicitly defined because the connection state is unknown.

---

### **Practical Example**

**Scenario: Internal Router Configuring BGP with External Router**

1. **Firewall Rule:**
   - Allow **TCP port 179** for outbound traffic.
   - Stateful inspection will allow the inbound response automatically.

2. **Stateless ACL Example:**
   ```plaintext
   access-list 100 permit tcp any any eq 179 outbound
   access-list 101 permit tcp any eq 179 any inbound
   ```

3. **Router Configuration for BGP:**
   ```plaintext
   router bgp 65001
    neighbor 192.168.1.2 remote-as 65002
   ```

4. **Verification:**
   ```plaintext
   show ip bgp summary
   show tcp brief
   ```

