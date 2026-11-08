# Lab 02: OSPF Multi-Area Configuration and ABR Route Summarisation

## 1. Lab Overview

This lab demonstrates the configuration and verification of multi-area OSPF, with a focus on Area Border Router (ABR) functionality and inter-area route summarisation.

Three Cisco routers were configured in a multi-area topology. R1 operates in Area 40, R3 operates in Area 0, while R2 acts as the ABR connecting the two areas.

The objective was to advertise multiple networks from Area 40 and summarise them into a single route before advertising them into Area 0.

## 2. Lab Objectives

- Configure OSPF in a multi-area environment.
- Establish OSPF neighbour relationships.
- Understand the role of an ABR.
- Advertise loopback networks into OSPF.
- Configure inter-area route summarisation.
- Verify routing-table changes before and after summarisation.

## 3. Network Topology

Three routers were used:

- **R1:** Internal router in Area 40.
- **R2:** ABR connecting Area 40 and Area 0.
- **R3:** Backbone router in Area 0.

## 4. IP Addressing

| Router | Interface | IP Address | OSPF Area |
|---|---|---|---|
| R1 | G0/0 | 10.12.12.1/30 | 40 |
| R2 | G0/0 | 10.12.12.2/30 | 40 |
| R2 | G0/1 | 10.23.23.2/30 | 0 |
| R3 | G0/0 | 10.23.23.3/30 | 0 |
| R1 | Lo10 | 10.40.10.1/24 | 40 |
| R1 | Lo20 | 10.40.50.1/24 | 40 |
| R3 | Lo10 | 172.16.30.1/24 | 0 |

## 5. OSPF Configuration

OSPF process ID 1 was configured on all three routers.

R1 advertised its transit and loopback networks into Area 40.

R2 participated in both Area 40 and Area 0, making it the ABR.

R3 advertised its transit and loopback networks into Area 0.

Loopback interfaces on R1 were configured with the OSPF point-to-point network type to ensure their configured /24 prefixes were advertised instead of the default /32 host routes.

Example:

```cisco
interface loopback10
 ip ospf network point-to-point
```

## 6. OSPF Route Summarisation

Before summarisation, R3 received the following individual inter-area routes:

- 10.40.10.0/24
- 10.40.50.0/24

The objective was to summarise these networks into:

**10.40.0.0/17**

The summary address was calculated using the subnet mask 255.255.128.0.

The following command was configured on R2:

```cisco
router ospf 1
 area 40 range 10.40.0.0 255.255.128.0
```

This instructed the ABR to advertise the summarised prefix into Area 0 instead of advertising the individual component routes.

## 7. Verification

The following commands were used:

```cisco
show ip ospf neighbor
show ip ospf
show ip route ospf
show ip ospf database summary
```

### Before summarisation

R3 received the individual Area 40 prefixes:

```text
O IA 10.40.10.0/24
O IA 10.40.50.0/24
```

### After summarisation

R3 received a single inter-area summary route:

```text
O IA 10.40.0.0/17
```

R2 continued to maintain the individual connected /24 routes in its routing table.

## 8. Key Learning Points

- An ABR connects multiple OSPF areas and exchanges inter-area routing information.
- Area summarisation is configured on the ABR using the `area range` command.
- OSPF uses the subnet mask, not the wildcard mask, in the `area range` command.
- OSPF loopback interfaces advertise /32 routes by default unless their network type is changed.
- Summarisation reduces the number of inter-area routes advertised into other areas.
- The `O IA` routing-table code identifies OSPF inter-area routes.

## 9. Lab Environment

- Cisco routers
- PNETLab
- OSPF Process ID: 1
- OSPF Areas: 0 and 40

## 10. Conclusion

The lab successfully demonstrated multi-area OSPF configuration and inter-area route summarisation using an ABR.

The individual Area 40 prefixes were successfully summarised into 10.40.0.0/17 and advertised into Area 0.

This exercise provided practical experience in OSPF area design, ABR operation, route summarisation and routing-table verification.
