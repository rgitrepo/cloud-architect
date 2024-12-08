
### **BGP Path Selection Algorithm**

When multiple paths to the same destination exist, BGP uses the following decision-making process:

1. **Next Hop Reachability**:
   - If the next hop is unreachable, the route is discarded and not placed in the routing table.

2. **Weight**:
   - Prefer the path with the highest weight (Cisco-specific).

3. **Local Preference**:
   - If weights are equal, prefer the path with the highest local preference.

4. **Locally Originated Route**:
   - If local preferences are equal, prefer the route that originated locally on the router (via `network` or `aggregate` commands).

5. **Shortest AS Path**:
   - If no locally originated route exists, prefer the path with the shortest AS path.

6. **Lowest Origin Code**:
   - If AS paths are equal, prefer the path with the lowest origin type (IGP=0 > EGP=1 > Incomplete=2).

7. **Lowest MED**:
   - If origin codes are equal, prefer the path with the lowest MED (indicates the preferred exit from the AS).

8. **eBGP Over iBGP**:
   - If MEDs are equal, prefer routes learned via eBGP over iBGP.

9. **Shortest Path to Next Hop (IGP Metric)**:
   - If routes are still equal, prefer the path with the shortest IGP distance to the next hop.

10. **Oldest Path**:
    - If the IGP metrics are the same, prefer the path that has been in the BGP table the longest (more stable).

11. **Lowest Router ID**:
    - If all else fails, prefer the path originating from the router with the lowest BGP router ID.

12. **Shortest Cluster List**:
    - If using Route Reflectors, prefer the path with the shortest Cluster List (fewer Route Reflectors).

13. **Lowest Neighbor IP Address**:
    - As the last tie-breaker, prefer the path learned from the neighbor with the lowest IP address.


---

### **Commands for Verification**
1. Check BGP neighbors:
   ```bash
   show ip bgp neighbors
   ```
2. View BGP attributes for a route:
   ```bash
   show ip bgp <prefix>
   ```
3. Debug BGP path selection:
   ```bash

   debug ip bgp
   ```



![0 0](https://github.com/user-attachments/assets/cecc083d-4149-4713-9756-480a3ca0ea82)


![image](https://github.com/user-attachments/assets/5fada295-cad7-43a8-8154-9c23eaad0a87)

