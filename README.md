# Built Active-Active Enterprise Core with VRRP, MSTP, & DHCP Failover
### *Deployed Deterministic Traffic Engineering and Multi-Layer Redundancy for Minimal-Downtime Infrastructure.*

> My First Project

---

## The Topology

![Network Topology](https://i.imgur.com/q7hxYZl.png)

**Core Layer:** Dual MikroTik CCR2116 (L3) 
**Access Layer:** Triple MikroTik CRS Series (L2)  
**Failover Stack:** DHCP Split-Scope + MSTP + VRRP  
**Segmentation:** 9-VLAN Enterprise Environment using VLSM (10.10.0.0/22)

---

## The Lore

Okay so here's the scenario:

Most networks run Active-Passive Failover design. One router does all the work. The other sits there collecting dust, waiting for a disaster that might never happen. That's a waste of hardware. For this project, I wanted every device to actually **do something**. No idle backups. No wasted bandwidth.

I engineered an Active-Active infrastructure where:
1. VLANs 10,30,50,70 gets its primary DHCP pool from Core Router 1, the other half DHCP pool gets it from Core Router 2 in the event that Core Router 1 dies
   
2. VLANs 20,40,60,80 gets its primary DHCP pool from Core Router 2, the other half DHCP pool gets it from Core Router 1 in the event that Core Router 2 dies
   
3. Each VLANs gets its own VRRP Gateway which acts as a Virtual Router, so in total we have one gateway per VLAN shared by both Core Routers
   
4. The VRRP interface created on number 3 acts as the DHCP Server per VLAN as mentioned on number 1 and 2
- If either dies, the other takes over in under a second

5. As LACP doesn't do the job for two links which are ongoing from two Core Routers, I used MSTP and segmented the two links
- Link 1: VLANs 10,30,50,70
- Link 2: VLANs 20,40,60,80

So every hardware earns its keep!

---

## Performance Highlights

**Layer 2 Redundancy**  
Synchronized MSTP instances with VRRP priorities to steer 10,30,50,70 VLANS through Core 1 and 20,40,60,80 VLANS through Core 2. Eliminated link blocking and allowing failover

**Automatic DHCP Failover**  
Configured split-scope DHCP architecture with reserved pools. Verified 100% lease continuity during total core-router failure scenarios.

**Layer 3 Redundancy**  
Built VRRP-based virtual gateway system achieving failover convergence in <1 second. Maintained persistent sessions for mission-critical inter-VLAN traffic.

**Hardened Network Segmentation**  
Developed stateful firewall filter rules and address lists to enforce strict per department VLAN isolation. Reduced internal attack surface without impacting line-rate performance.

---

## The Proof

### Test 1: The Gateway Transition (VRRP)

**What I did:** Simulated a total hardware crash on Core 1. 
**What happened:** VRRP state transition to Backup Core completed in under 4 seconds. Only 4 packets dropped during the transition.

https://github.com/user-attachments/assets/aacd9f15-05fa-40ee-8f06-9ed837b44fb1

**My Improvement:** I was able to lessen the downtime into less than 1 second by adjusting the interval to 200ms, keep note this methos is cpu intensive!
Since we have CCR2116 as an absolute powerhouse core we will have no problem with an interval of 200ms 

https://github.com/user-attachments/assets/420fa2db-e2a0-4a94-ac59-e8b4fc57c711

---

### Test 2: The Lease Continuity (DHCP Failover)

**What I did:** Simulated a total hardware crash on Core 1.

**What happened:** Client successfully pulled a secondary lease from Core 2's reserved pool. No network lockout.

https://github.com/user-attachments/assets/40daef2c-028d-4c9a-b16c-ce3bb4daf78d

---

### Test 3: The Path Steering (MSTP)

**What I did:** Simulated a physical interface failure on both the redundant links

**What happened:** Achieved a less than one second failover convergence, only one packet loss!

https://github.com/user-attachments/assets/a82f9ea3-aa2a-4ffe-8ced-cd3dec249ae1

---

### Test 4: The Security Layer (Firewall)

**What I did:** Executed cross-VLAN penetration test from VLAN 10 to VLAN 20.

**What happened:** 100% drop rate verified via real-time packet counters on the Firewall Filter chain.



---

## Engineering Challenges

**Core-to-Core Alignment**

The biggest challenge was preventing L2 loops while keeping L3 gateways active.

**How I solved it:** Tuned MSTP Bridge Priorities to force the L2 "Blocked" ports to coincide with the L3 "Backup" router. The shortest physical path is always the active one.

**VLSM Management**

With many potential hosts across 9 VLANs, I had to ensure the subnetting. No subnet overlap. No route flapping.

**How I solved it:** Mapped every VLAN carefully before touching a single router.

---

## Final Thoughts

Modern networks shouldn't have "idle" hardware.

Active-Active is engineering. It forces you to understand how traffic actually flows, where loops can form, and how to make every device work for its keep.

---

## Project Proposal Scenario

<img width="1643" height="941" alt="image" src="https://github.com/user-attachments/assets/6429f679-e8ec-4345-9601-37850e9af3fb" />

<img width="901" height="673" alt="image" src="https://github.com/user-attachments/assets/321e2f9e-016b-42be-b33b-d52a8bc3c855" />

<img width="991" height="686" alt="image" src="https://github.com/user-attachments/assets/9943e027-5059-4cd0-a29b-ed2ddf44b927" />


**Core Layer:** Dual MikroTik CCR2116 (L3) 
**Access Layer:** Triple MikroTik CRS Series (L2)  
**Failover Stack:** DHCP Split-Scope + MSTP + VRRP  
**Segmentation:** 9-VLAN Enterprise Environment using VLSM (10.10.0.0/22)

---

*First project down. More to come.* 🔥

