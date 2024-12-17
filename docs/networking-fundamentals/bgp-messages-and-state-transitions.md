### **How BGP Forms Neighbor Relationships**

BGP (Border Gateway Protocol) establishes neighbor (peer) relationships between routers to exchange routing information. This process follows a structured series of states, transitioning predictably based on specific conditions and messages. BGP uses a **finite state machine (FSM)** to manage these transitions.

---

### **BGP Finite State Machine (FSM)**

The BGP FSM operates in defined states, ensuring that the session progresses logically. These states are:

1. **Idle**
2. **Connect**
3. **Active**
4. **OpenSent**
5. **OpenConfirm**
6. **Established**

Each state has specific actions and transitions based on success, failure, or timeout.

---

### **1. Idle State**
- **Description**:
  - The initial state when a BGP process starts.
  - The router does not actively attempt to connect to its peer.
  - Idle for too long indicates a problem (e.g., configuration error, missing IP reachability, or firewall blocking TCP port 179).

- **Actions**:
  - Wait for a `start event` (e.g., configuration of a BGP neighbor).
  - Initialize BGP resources.
  - Listen for TCP connections.

- **Transitions**:
  - **To Connect**: If a start event occurs.
  - **Remain Idle**: If no start event occurs.

---

### **2. Connect State**
- **Description**:
  - The router attempts to establish a TCP connection with the peer (using port 179).
  - If the TCP connection succeeds, BGP progresses to the `OpenSent` state.
  - If the TCP connection fails, BGP moves to the `Active` state.

- **Actions**:
  - Try to establish a TCP connection with the neighbor.

- **Transitions**:
  - **To OpenSent**: If the TCP connection is successful.
  - **To Active**: If the TCP connection fails (e.g., peer unreachable or firewall blocks).

---

### **3. Active State**
- **Description**:
  - The router is actively trying to establish a TCP connection with the peer.
  - If successful, the router transitions to the `OpenSent` state.
  - If the connection repeatedly fails, the router will retry periodically or return to the `Idle` state if resources are exhausted.

- **Actions**:
  - Continuously attempt to establish a TCP connection.

- **Transitions**:
  - **To OpenSent**: If the TCP connection succeeds.
  - **To Idle**: If retries fail or resources are exhausted.

---

### **4. OpenSent State**
- **Description**:
  - The TCP connection is established, and the router sends a BGP **OPEN message** to the peer.
  - The router waits for an **OPEN message** from the peer.

- **Actions**:
  - Exchange OPEN messages with the neighbor.
  - Validate parameters (e.g., BGP version, AS number, hold time).

- **Transitions**:
  - **To OpenConfirm**: If the peer responds with a valid OPEN message.
  - **To Idle**: If the OPEN message is invalid or an error occurs.

---

### **5. OpenConfirm State**
- **Description**:
  - Both peers have exchanged and validated OPEN messages.
  - The router now waits for a **KEEPALIVE message** from the peer.

- **Actions**:
  - Send KEEPALIVE messages to confirm the connection.
  - Monitor for KEEPALIVE from the peer.

- **Transitions**:
  - **To Established**: If KEEPALIVE messages are successfully exchanged.
  - **To Idle**: If errors occur or the timer expires.

---

### **6. Established State**
- **Description**:
  - The BGP session is fully established, and the neighbors can now exchange routing information.
  - This is the final and operational state.

- **Actions**:
  - Exchange UPDATE messages to share routing information.
  - Maintain the session using periodic KEEPALIVE messages.

- **Transitions**:
  - **Remain Established**: As long as the connection is healthy.
  - **To Idle**: If the session fails (e.g., no KEEPALIVE, link failure, or manual termination).

---

### **Summary of FSM States and Transitions**

| **State**         | **Description**                                                                                           | **Next States**                 |
|--------------------|-----------------------------------------------------------------------------------------------------------|----------------------------------|
| **Idle**           | Initial state; waiting for configuration or restart event.                                               | Connect or Remain Idle          |
| **Connect**        | Attempting to establish a TCP connection with the peer.                                                  | OpenSent, Active, or Idle       |
| **Active**         | Actively retrying to establish a TCP connection.                                                         | OpenSent or Idle                |
| **OpenSent**       | TCP connection established; sending and receiving OPEN messages.                                         | OpenConfirm or Idle             |
| **OpenConfirm**    | Exchanged OPEN messages; waiting for KEEPALIVE messages.                                                 | Established or Idle             |
| **Established**    | Full BGP session; exchanging UPDATE and KEEPALIVE messages.                                              | Remain Established or Idle      |

---

### **BGP Messages and State Transitions**

#### **1. OPEN Message**:
- Sent in the `OpenSent` state after a TCP connection is established.
- Contains parameters like BGP version, AS number, hold time, and BGP identifier.

#### **2. KEEPALIVE Message**:
- Sent in the `OpenConfirm` and `Established` states to confirm the connection and keep it alive.
- If KEEPALIVE messages stop, the session transitions to `Idle`.

#### **3. UPDATE Message**:
- Sent in the `Established` state to exchange routing information.
- Advertises new routes or withdraws invalid routes.

#### **4. NOTIFICATION Message**:
- Sent when an error occurs (e.g., invalid parameters in the OPEN message or session timeout).
- The session transitions to `Idle` after sending or receiving a NOTIFICATION message.

---

### **Sharing BGP Tables in the Established State**

- Once in the **Established** state, BGP peers exchange their routing tables.
- Each peer evaluates the received routes based on BGP attributes (e.g., AS_PATH, LOCAL_PREF).
- Updates are sent as changes occur (e.g., a new route is learned or an existing route becomes unavailable).

**Example**:
1. **New Route Advertisement**:
   - Internal router learns a new route (`10.1.1.0/24`) and advertises it to the external router:
   ```plaintext
   UPDATE: Advertise 10.1.1.0/24
   ```

2. **Route Withdrawal**:
   - A previously advertised route (`192.168.1.0/24`) becomes unreachable, so it is withdrawn:
   ```plaintext
   UPDATE: Withdraw 192.168.1.0/24
   ```

---

### **FSM Troubleshooting**

- **Idle for Too Long**:
  - Indicates issues like:
    - Misconfigured neighbor relationships.
    - Firewall blocking TCP port 179.
    - Peer unreachable.

- **Active State Loops**:
  - Occurs when the TCP connection repeatedly fails.
  - Possible reasons:
    - Network reachability issues.
    - Incorrect AS number or IP configuration.

- **Failed to Transition to Established**:
  - Ensure both routers can exchange and validate OPEN and KEEPALIVE messages.

**Commands to Verify BGP State**:
1. Check the current BGP state:
   ```plaintext
   show ip bgp neighbors
   ```
2. Debug BGP transitions:
   ```plaintext
   debug ip bgp
   ```

---

Mark Milovanovic: To help you remember: Idle, Connect, and Active happen during the **TCP handshake**. Then OpenSent, OpenConfirm, and Established are part of BGP session establishment.
