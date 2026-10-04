# HSRP — First Hop Redundancy Protocol

## ENCOR 350-401 Lab

### Objective

Demonstrate HSRP gateway redundancy, priority, preemption, and failover between two Cisco routers.

---

## Topology

![HSRP Topology](topology.png)

### IP Addressing

| Device | Interface | IP Address |
|---|---|---|
| R1 | Gi0/0 | 192.168.10.2/24 |
| R2 | Gi0/0 | 192.168.10.3/24 |
| Virtual Gateway | - | 192.168.10.1 |
| PC | - | 192.168.10.4/24 |

---

## HSRP Configuration

### R1

```cisco
interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt

Failover Test
R1's Gi0/0 interface was shut down.
R2 changed state:
Standby → Active

Observed syslog:
%HSRP-5-STATECHANGE: GigabitEthernet0/0 Grp 10 state Standby -> Active

R2 successfully assumed the Active role.
Preemption Test
R1 was restored.
Because R1 had:
Priority: 110
Preempt: enabled

R1 reclaimed the Active role.
Final state:
R1 = Active
R2 = Standby

Key Concepts Learned
- HSRP provides first-hop gateway redundancy.
- Hosts use a virtual IP as their default gateway.
- Higher priority influences the Active router election.
- preempt allows the preferred router to reclaim the Active role.
- HSRP uses Active/Standby terminology.
- Gateway availability is maintained when the Active router fails.
Verification Commands
show standby
show standby brief

Lab Result
✅ HSRP configured successfully
✅ Active/Standby roles verified
✅ Gateway failover tested
✅ Preemption tested
✅ HSRP state-change syslog observed


---

# Step 3 — Upload your screenshot

Now go back to the repository main page.

Click:

**Add file → Upload files**

Then either:

- click **choose your files**, or
- drag your screenshot into the browser.

GitHub supports uploading existing files directly through the web interface. Browser uploads are limited to **25 MiB per file**, so your screenshot is well within the limit. :chatgpt-content-reference{index="2"}

Rename your screenshot on your laptop first to:

```text
topology.png
