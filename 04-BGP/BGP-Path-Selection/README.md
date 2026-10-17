
# BGP Path Selection Lab – Weight, LOCAL_PREF and MED

## Overview

This lab demonstrates Border Gateway Protocol (BGP) path selection using a four-router topology in PNETLab. It explores how BGP attributes influence the best-path decision when multiple routes to the same destination are available.

The lab uses Cisco IOS routers configured with eBGP across multiple autonomous systems.

## Lab Objectives

- Establish eBGP peering between four routers.
- Advertise and verify loopback prefixes across multiple ASes.
- Examine BGP AS-PATH and understand AS-PATH loop prevention.
- Apply inbound prefix filtering and outbound route filtering.
- Compare BGP path selection using Weight, LOCAL_PREF and MED.
- Observe how a preferred path changes when a link fails and recovers.
- Verify BGP best-path decisions using Cisco IOS show commands.

## Topology

Four routers are connected in the following arrangement:

```text
                 AS 65001
                    R1
                     |
                  eBGP
                     |
                 AS 65002
                    R2
                   /  \
                eBGP  eBGP
                 /      \
          AS 65003      AS 65004
              R3 -------- R4
                   eBGP
```

**Router and AS assignments**

| Router | Autonomous System |
|---|---:|
| R1 | 65001 |
| R2 | 65002 |
| R3 | 65003 |
| R4 | 65004 |

R2 provides multiple paths toward the loopback network advertised by R3, allowing BGP path selection to be tested.

## Key Concepts Tested

### 1. eBGP Peering and Route Advertisement

Established eBGP neighbor relationships between the routers and verified route propagation across the topology.

### 2. AS-PATH and Loop Prevention

Examined AS-PATH information to understand how BGP records the autonomous systems traversed by a route and prevents routing loops.

### 3. Prefix Filtering

Applied prefix lists and a route map to control which routes were advertised to and accepted from a BGP neighbor.

### 4. LOCAL_PREF

Modified LOCAL_PREF to influence the preferred exit path within an autonomous system.

**Observation:** The path with the higher LOCAL_PREF was preferred, even when it had a longer AS-PATH.

### 5. MED

Configured different MED values on routes advertised by R3 and R4.

- R3 advertised a MED of 50.
- R4 advertised a MED of 150.
- R2 was configured with `bgp always-compare-med` to compare MED values across paths from different neighboring ASes.

**Observation:** With earlier best-path attributes tied, the path with the lower MED was preferred.

### 6. Cisco BGP Weight

Configured a higher Weight on R2 for routes received from R4.

**Observation:** The R4 path became the best path despite having a longer AS-PATH and a higher MED.

This demonstrated that Cisco Weight is evaluated before LOCAL_PREF, AS-PATH and MED in the BGP best-path decision process.

Weight is Cisco-proprietary and is local to the router; it is not advertised to BGP peers.

### 7. Failover Testing

Simulated a link failure by shutting down an interface, observed the BGP neighbor and route changes, and restored connectivity to verify path recovery.

## Verification Commands

The following Cisco IOS commands were used to inspect BGP operation and troubleshoot the lab:

```cisco
show ip bgp summary
show ip bgp
show ip bgp 3.3.3.3
show ip route bgp
show running-config | section router bgp
show ip bgp neighbors
```

## Key Learnings

- BGP selects a best path using a sequence of attributes and tie-breakers.
- Weight is Cisco-specific, local to the router, and higher is preferred.
- LOCAL_PREF is used to influence outbound path selection within an AS; higher is preferred.
- AS-PATH length influences route selection; a shorter path is generally preferred when earlier attributes are equal.
- MED can influence how a neighboring AS selects an entry point; lower is preferred when the paths are comparable.
- MED comparison across paths from different neighboring ASes depends on the BGP implementation and configuration.
- Prefix lists and route maps provide granular control over route advertisement and acceptance.
- BGP failover behavior can be tested by simulating link failures and observing route reconvergence.

## Lab Environment

- **Platform:** PNETLab
- **Routing protocol:** BGP
- **Peering type:** eBGP
- **Devices:** Four Cisco IOS routers
- **Focus:** BGP path selection, route filtering, failover and troubleshooting

## Conclusion

This lab provided hands-on experience with eBGP route propagation and path selection. By changing Weight, LOCAL_PREF and MED, I verified how BGP attributes affect the best-path decision and gained practical experience troubleshooting routing behavior in a multi-AS environment.

*This is a virtual lab created for learning and professional development.*
