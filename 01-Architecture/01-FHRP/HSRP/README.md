# HSRP — First Hop Redundancy Protocol

## ENCOR 350-401 | Enterprise Network High Availability

## 1. Overview

This project demonstrates Cisco Hot Standby Router Protocol (HSRP) for first-hop gateway redundancy, gateway load sharing, interface tracking and IP SLA-based upstream reachability monitoring.

The lab progresses from a basic two-router HSRP implementation to a multi-VLAN enterprise scenario with automated failover and recovery.

## 2. Learning Objectives

- Understand HSRP operation and election process.
- Configure HSRP Version 2.
- Implement gateway load sharing across multiple VLANs.
- Configure HSRP priority and preemption.
- Implement interface tracking.
- Configure IP SLA ICMP probes and object tracking.
- Demonstrate automatic gateway failover and recovery.
- Verify redundancy using CLI, ping and traceroute.

---

# PART A — Basic HSRP Gateway Redundancy

## 3. Basic Topology

Two routers provide redundant gateway services to a client on a shared LAN.

### IP Addressing

| Device | Interface | IP Address |
|---|---|---|
| R1 | Gi0/0 | 192.168.10.2/24 |
| R2 | Gi0/0 | 192.168.10.3/24 |
| Virtual Gateway | — | 192.168.10.1 |
| PC | — | 192.168.10.4/24 |

## 4. HSRP Configuration

### R1

```cisco
interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
```

### Failover Test

R1's Gi0/0 interface was shut down.

R2 transitioned from Standby to Active.

Observed syslog:

```text
%HSRP-5-STATECHANGE:
GigabitEthernet0/0 Grp 10 state Standby -> Active
```

### Preemption Test

R1 was restored. Its higher priority of 110 and preemption configuration allowed it to reclaim the Active role.

Final state:

- R1: Active
- R2: Standby

---

# PART B — Advanced HSRP with Multiple VLANs

## 5. Enterprise Lab Topology

The advanced implementation uses:

- CSR1 and CSR2 as redundant branch gateway routers.
- S1 as a Layer 2 switch.
- VLANs 10, 20 and 30.
- Three VPCS client devices.
- CSR3 as a simulated ISP/upstream router.
- OSPF process 10 for routing.

### IP Addressing

| VLAN | CSR1 | CSR2 | HSRP Virtual IP | Client |
|---|---|---|---|---|
| 10 | 10.0.0.2 | 10.0.0.3 | 10.0.0.10 | 10.0.0.100 |
| 20 | 20.0.0.2 | 20.0.0.3 | 20.0.0.10 | 20.0.0.100 |
| 30 | 30.0.0.2 | 30.0.0.3 | 30.0.0.10 | 30.0.0.100 |

All addresses above are /24.

### WAN Addressing

| Connection | Interface | IP Address |
|---|---|---|
| CSR1–CSR3 | CSR1 Gi2 | 10.0.13.1/30 |
| CSR1–CSR3 | CSR3 Gi1 | 10.0.13.2/30 |
| CSR2–CSR3 | CSR2 Gi2 | 10.0.23.1/30 |
| CSR2–CSR3 | CSR3 Gi2 | 10.0.23.2/30 |
| CSR3 Loopback0 | — | 8.8.8.8/32 |
| CSR3 Loopback1 | — | 9.9.9.9/32 |

## 6. HSRP Gateway Load Sharing

HSRP Version 2 was configured across all three VLANs.

| VLAN | Group | Preferred Active Router | Priority |
|---|---:|---|---:|
| 10 | 10 | CSR1 | 110 |
| 20 | 20 | CSR2 | 110 |
| 30 | 30 | CSR1 | 110 |

The other router has priority 100 for each group.

Preemption was enabled to allow the preferred router to reclaim the Active role following recovery.

This distributes gateway responsibilities across VLANs. HSRP does not load-balance individual flows within a single HSRP group.

---

## 7. Interface Tracking

Interface tracking was implemented to detect WAN interface failures.

### CSR1

Tracking object 10 monitors GigabitEthernet2.

```cisco
track 10 interface GigabitEthernet2 line-protocol
```

HSRP groups 10 and 30 use the tracking object:

```cisco
standby 10 track 10 decrement 20
standby 30 track 10 decrement 20
```

### CSR2

Tracking object 20 monitors GigabitEthernet2.

```cisco
track 20 interface GigabitEthernet2 line-protocol
```

HSRP Group 20 uses the tracking object during the interface-tracking test:

```cisco
standby 20 track 20 decrement 20
```

### Observed Result

When the WAN interface on the Active router was shut down:

- The tracking object changed state.
- HSRP priority decreased by 20.
- The peer router assumed the Active role.
- Client connectivity was restored through the surviving router.

---

## 8. IP SLA-Based Upstream Reachability Tracking

The second phase introduced IP SLA to detect upstream reachability failures even when the local WAN interface remained operational.

CSR3 Loopback1 (`9.9.9.9/32`) was configured as the monitoring destination and advertised through OSPF.

### CSR2 IP SLA Configuration

```cisco
ip sla 21
 icmp-echo 9.9.9.9 source-interface GigabitEthernet2
 frequency 5
exit

ip sla schedule 21 life forever start-time now

track 21 ip sla 21 reachability
```

### HSRP Integration

```cisco
interface GigabitEthernet1.20
 standby 20 track 21 decrement 20
```

This links the IP SLA tracking object to HSRP Group 20.

### Normal Operation

- IP SLA destination reachable.
- Track 21 UP.
- CSR2 HSRP priority 110.
- CSR2 Active for VLAN 20.

### Failure Simulation

CSR3 Loopback1 was shut down to simulate loss of the monitored upstream destination.

Observed results:

- CSR2 GigabitEthernet2 remained UP/UP.
- IP SLA returned `Timeout`.
- Track 21 changed to DOWN.
- CSR2 HSRP priority decreased from 110 to 90.
- CSR1 became Active for VLAN 20.

### Client-Side Verification

PC2 successfully reached the HSRP virtual gateway and the external test destination.

Traceroute:

```text
Tracing route to 8.8.8.8

1    20.0.0.2
2    8.8.8.8
```

The first hop confirmed that traffic was now passing through CSR1.

---

## 9. Automatic Recovery and Preemption

CSR3 Loopback1 was restored.

Observed results:

- IP SLA return code returned to `OK`.
- Track 21 returned to UP.
- CSR2 priority recovered from 90 to 110.
- CSR2 reclaimed the Active role through HSRP preemption.

Final VLAN 20 state:

```text
Gi1.20  Grp20  Pri110  P Active
```

---

## 10. Verification Commands

```cisco
show standby
show standby brief
show track
show track 21
show ip sla statistics 21
show ip sla configuration 21
show ip route 9.9.9.9
show ip ospf neighbor
show ip interface brief
```

Client verification:

```text
ping 20.0.0.10
ping 8.8.8.8
tracert 8.8.8.8
```

---

## 11. Key Concepts Learned

1. HSRP provides first-hop gateway redundancy.
2. HSRP virtual IP addresses allow hosts to retain a consistent default gateway.
3. Priority influences Active router election.
4. Preemption allows a preferred router to reclaim the Active role.
5. Gateway load sharing can be implemented across multiple VLANs.
6. Interface tracking detects local interface failures.
7. IP SLA provides end-to-end reachability monitoring.
8. Object tracking integrates IP SLA results with HSRP.
9. Priority decrement enables automatic failover.
10. Traceroute helps verify the actual forwarding path following failover.

## 12. Lab Results

| Test | Result |
|---|---|
| Basic HSRP configuration | Passed |
| Active/Standby election | Passed |
| HSRP preemption | Passed |
| Multi-VLAN gateway load sharing | Passed |
| Interface tracking | Passed |
| IP SLA configuration | Passed |
| Object tracking | Passed |
| Automatic gateway failover | Passed |
| Client-side connectivity verification | Passed |
| Automatic recovery | Passed |

## 13. Enterprise Application

In enterprise branch networks, the default gateway may remain operational even when upstream connectivity is lost.

Interface tracking alone may not detect this condition.

IP SLA provides a mechanism to monitor an upstream destination and trigger HSRP priority changes when reachability is lost.

This improves network resilience by allowing the standby gateway to assume responsibility when the preferred gateway can no longer reach the monitored destination.

**Production design consideration:** Monitoring targets should be selected carefully to represent the actual services and paths requiring protection. A single probe destination can fail independently of the wider network.

---

**Lab Environment:** Cisco CSR1000v, Cisco IOS-XE, PNETLab, VPCS.

**Certification Alignment:** Cisco CCNP Enterprise ENCOR 350-401.
