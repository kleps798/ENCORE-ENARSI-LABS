# VRRP — Virtual Router Redundancy Protocol

## ENCOR 350-401 Lab

### Objective

Demonstrate VRRP gateway redundancy, priority, preemption, and failover between two Cisco routers.

---

## Topology

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

Virtual Gateway: 192.168.10.1
IP Addressing
Device	Interface	IP Address
R1	Gi0/0	192.168.10.2/24
R2	Gi0/0	192.168.10.3/24
Virtual Gateway	-	192.168.10.1
PC1	-	192.168.10.4/24


VRRP Configuration
R1
interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 vrrp 10 ip 192.168.10.1
 vrrp 10 priority 110
 vrrp 10 preempt

R2
interface GigabitEthernet0/0
 ip address 192.168.10.3 255.255.255.0
 vrrp 10 ip 192.168.10.1
 vrrp 10 priority 100
 vrrp 10 preempt

Normal Operation
R1 has a higher priority:
R1 = 110
R2 = 100

Therefore:
R1 = MASTER
R2 = BACKUP

The hosts use the virtual IP:
192.168.10.1

as their default gateway.
Verification
Useful commands:
show vrrp
show vrrp brief

VRRP Failover Test
R1 was initially the Master.
R1's Gi0/0 interface was shut down:
interface GigabitEthernet0/0
 shutdown

R2 detected the failure and changed state:
Backup → Master

Observed syslog:
%VRRP-6-STATECHANGE: Gi0/0 Grp 10 state Backup -> Master

This demonstrated successful first-hop gateway failover.
VRRP Priority Test
R1 was restored and configured with a lower priority:
vrrp 10 priority 90

R2 retained priority:
R2 = 100

Therefore:
R1 = 90  → BACKUP
R2 = 100 → MASTER

Observed state transition:
%VRRP-6-STATECHANGE: Gi0/0 Grp 10 state Backup -> Master

This demonstrated that the higher-priority router is preferred for the Master role.
Key Concepts Learned
- VRRP provides first-hop gateway redundancy.
- Hosts use a virtual IP as their default gateway.
- VRRP uses Master/Backup terminology.
- Higher priority determines the preferred Master.
- preempt allows a higher-priority router to reclaim the Master role.
- If the Master fails, the Backup becomes Master.
- The virtual gateway IP remains unchanged during failover.
