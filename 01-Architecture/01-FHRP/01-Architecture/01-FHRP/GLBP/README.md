# GLBP – Gateway Load Balancing Protocol Lab

## ENCOR 350-401 – Enterprise Network Design / High Availability

This lab demonstrates Cisco Gateway Load Balancing Protocol (GLBP), including:

- Active Virtual Gateway (AVG)
- Active Virtual Forwarders (AVFs)
- GLBP priority and election
- Virtual gateway IP
- Virtual MAC addresses
- Gateway load balancing
- Failover
- Preemption
- Verification using `show glbp brief`

---

## 1. Topology

```text
             R1                  R2
        192.168.10.2        192.168.10.3
             \                  /
              \                /
               \              /
                 LAN / VLAN 10
                       |
                      PC1
                192.168.10.4/24

              Virtual Gateway
                192.168.10.1

2. GLBP Configuration
R1
interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 glbp 10 ip 192.168.10.1
 glbp 10 priority 110
 glbp 10 preempt

R2
interface GigabitEthernet0/0
 ip address 192.168.10.3 255.255.255.0
 glbp 10 ip 192.168.10.1
 glbp 10 priority 100
 glbp 10 preempt

3. Initial GLBP State
R1
R1#show glbp brief

Interface   Grp  Fwd Pri State    Address         Active router   Standby router
Gi0/0       10   -   110 Active   192.168.10.1    local           192.168.10.3
Gi0/0       10   1   -   Active   0007.b400.0a01  local           -
Gi0/0       10   2   -   Listen   0007.b400.0a02  192.168.10.3    -

Interpretation
R1 became the Active Virtual Gateway (AVG) because it has the higher priority of 110.
R1 was also acting as Active Virtual Forwarder (AVF) 1.
R2 was acting as AVF 2.
Therefore, both routers were capable of forwarding traffic through GLBP.
R2 – Initial State
R2#show glbp brief

Interface   Grp  Fwd Pri State    Address         Active router   Standby router
Gi0/0       10   -   100 Standby  192.168.10.1    192.168.10.2    local
Gi0/0       10   1   -   Listen   0007.b400.0a01  192.168.10.2    -
Gi0/0       10   2   -   Active   0007.b400.0a02  local           -

R2 was the Standby AVG, while also acting as AVF 2.
4. GLBP Virtual MAC Addresses
The lab showed the following virtual MAC addresses:
AVF 1:
0007.b400.0a01

AVF 2:
0007.b400.0a02

GLBP can assign different virtual MAC addresses to different active forwarders.
This allows hosts to use the same virtual gateway IP while their traffic can be forwarded through different physical routers.
5. Failover Test
R1 was shut down:
R1#interface GigabitEthernet0/0
R1(config-if)#shutdown

R2 was then checked:
R2#show glbp brief

Interface   Grp  Fwd Pri State    Address         Active router   Standby router
Gi0/0       10   -   100 Active   192.168.10.1    local           unknown
Gi0/0       10   1   -   Active   0007.b400.0a01  local           -
Gi0/0       10   2   -   Active   0007.b400.0a02  local           -

Result
R2 successfully became the Active Virtual Gateway (AVG).
R2 also became responsible for both active forwarders:
AVF 1 → R2
AVF 2 → R2

The virtual gateway IP remained unchanged:
192.168.10.1

This demonstrates GLBP gateway redundancy and failover.
6. Key GLBP Concepts
AVG – Active Virtual Gateway
The AVG is responsible for:
- Managing the GLBP group
- Responding for the virtual gateway
- Assigning virtual MAC addresses
- Controlling the active forwarders
AVF – Active Virtual Forwarder
An AVF is a router that actively forwards traffic for a GLBP virtual MAC address.
Unlike HSRP and VRRP, GLBP can have multiple active forwarders simultaneously.
