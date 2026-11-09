# Lab 04: STP Root Bridge Election – Rapid PVST+

## 1. Lab Overview

This lab demonstrates Spanning Tree Protocol (STP) operation, Root Bridge election and bridge priority manipulation using Cisco Rapid PVST+.

Three Cisco Layer 2 switches were deployed in a redundant topology to observe how STP prevents Layer 2 switching loops while maintaining redundant paths.

The primary objective was to configure a specific switch as the Root Bridge and another switch as the Root Secondary for VLAN 10.

## 2. ENCOR 350-401 Objectives

- Configure and verify Spanning Tree Protocol.
- Explain Root Bridge election.
- Understand Bridge ID and STP priority.
- Configure Rapid PVST+.
- Configure Root Primary and Root Secondary.
- Understand Root Port, Designated Port and Alternate Port roles.
- Interpret STP forwarding and discarding states.

## 3. Lab Topology

Three Cisco Layer 2 switches were connected in a redundant triangle topology.

| Switch | Intended Role |
|---|---|
| SW1 | Root Primary |
| SW2 | Root Secondary |
| SW3 | Non-root switch |

VLAN 10 was used for the STP configuration and verification.

## 4. Initial Configuration

Rapid PVST+ was enabled on all switches.

```cisco
conf t
vlan 10
 name DATA
spanning-tree mode rapid-pvst
end
```

Inter-switch interfaces were configured as trunk links to carry VLAN 10.

Example:

```cisco
interface range gi0/0 - 1
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
```

## 5. Root Bridge Election

Initially, SW2 was elected as the Root Bridge because all switches were using the default bridge priority and SW2 had the lowest Bridge ID.

The initial verification command was:

```cisco
show spanning-tree vlan 10
```

SW2 initially displayed:

```text
Root ID    Priority    32778
           This bridge is the root
```

The effective priority was 32778, calculated from the default bridge priority of 32768 plus the VLAN 10 extended system ID.

## 6. Configure SW1 as Root Primary

SW1 was configured as the preferred Root Bridge.

```cisco
conf t
spanning-tree vlan 10 root primary
end
```

Verification:

```cisco
show spanning-tree vlan 10
```

SW1 successfully became the Root Bridge, displaying:

```text
Root ID    Priority    24586
           This bridge is the root
```

The effective priority was 24586, calculated from the configured bridge priority of 24576 plus the VLAN 10 extended system ID.

## 7. Configure SW2 as Root Secondary

SW2 was configured as the backup Root Bridge.

```cisco
conf t
spanning-tree vlan 10 root secondary
end
```

The secondary configuration provides a higher bridge priority than the primary root, but a lower priority than the default bridge priority.

## 8. Verification Commands

The following commands were used to verify STP operation:

```cisco
show spanning-tree vlan 10
show spanning-tree root
show spanning-tree blockedports
show spanning-tree summary
```

## 9. Key Learning Points

- STP prevents Layer 2 switching loops by logically blocking redundant paths.
- The Root Bridge is elected using the lowest Bridge ID.
- Bridge ID consists of bridge priority, extended system ID and MAC address.
- Lower bridge priority wins the Root Bridge election.
- `spanning-tree vlan 10 root primary` configures a preferred Root Bridge.
- `spanning-tree vlan 10 root secondary` configures a backup Root Bridge.
- Rapid PVST+ maintains an independent spanning-tree instance per VLAN.
- Root Ports provide the best path towards the Root Bridge.
- Designated Ports forward traffic on their respective segments.
- Alternate Ports provide redundant paths and normally remain in the discarding state.

## 10. Conclusion

The lab successfully demonstrated STP Root Bridge election and priority manipulation using Rapid PVST+.

SW1 was successfully elected as Root Primary after its bridge priority was lowered from the default value. SW2 was subsequently configured as Root Secondary.

This practical exercise provided hands-on experience with STP configuration, Root Bridge election, bridge priority and verification using Cisco IOS commands.

## Lab Environment

- Platform: PNETLab
- Devices: Cisco Layer 2 switches
- Protocol: Rapid PVST+
- VLAN: 10
