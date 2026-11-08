# Lab 7: VRF + GRE Tunnel + Static Routing

## 1. Lab Overview

This lab demonstrates the implementation of Virtual Routing and Forwarding (VRF), Generic Routing Encapsulation (GRE) tunnelling, and static routing to establish isolated network connectivity between two geographically separated sites across an ISP network.

The objective was to create a dedicated Finance VRF, establish a GRE tunnel between R11 and R22, and enable communication between the two Finance networks using VRF-specific static routes.

## 2. Technologies Implemented

- **VRF (Virtual Routing and Forwarding):** Created a separate routing instance named `Finance` to isolate routing information from the global routing table.
- **GRE Tunnel:** Established a point-to-point GRE tunnel between R11 and R22 across the ISP network.
- **Static Routing:** Configured global static routes for ISP reachability and VRF-specific static routes for Finance network connectivity.
- **Layer 3 Switching:** Configured SW11 and SW22 with routed interfaces, VLAN interfaces (SVIs), and VRF membership.

## 3. Network Architecture

The topology consists of:

- ISP: Provides the underlay IP connectivity between the two sites.
- R11 and R22: Edge routers responsible for GRE tunnel termination and VRF routing.
- SW11 and SW22: Layer 3 switches hosting the respective Finance VLANs.

The GRE tunnel uses the ISP-facing interfaces as tunnel source and destination addresses, while the tunnel interfaces operate within the Finance VRF.

## 4. Configuration Summary

**VRF Configuration**
- Created the Finance VRF on the relevant routers and Layer 3 switches.
- Enabled IPv4 address-family support where required.
- Assigned the appropriate routed interfaces and tunnel interfaces to the Finance VRF.

**GRE Configuration**
- Configured Tunnel 10 between R11 and R22.
- Used the ISP-facing interface addresses as GRE tunnel endpoints.
- Assigned tunnel IP addresses from the `10.10.100.0/30` subnet.

**Static Routing**
- Configured default routes in the global routing table for ISP reachability.
- Configured VRF-specific static routes to provide reachability between the remote Finance networks.
- Configured default routes on the Layer 3 switches pointing towards their respective edge routers.

## 5. Verification and Testing

The following commands were used to verify the implementation:

| Command | Purpose |
|---|---|
| `show vrf` | Verify VRF instances |
| `show ip interface brief` | Verify interface addressing and status |
| `show interface tunnel 10` | Verify GRE tunnel status |
| `show ip route` | Inspect the global routing table |
| `show ip route vrf Finance` | Inspect the Finance VRF routing table |
| `ping vrf Finance <destination>` | Test connectivity within the Finance VRF |
| `show ip interface tunnel 10` | Verify tunnel interface details |

## 6. Key Learning Outcomes

- Understanding VRF-based routing isolation.
- Configuring and assigning interfaces to VRF instances.
- Understanding the relationship between GRE tunnels and underlay IP connectivity.
- Implementing VRF-aware static routing.
- Understanding the separation between global and VRF routing tables.
- Troubleshooting routing and tunnel connectivity across isolated networks.

## 7. Lab Environment

- **Platform:** PNETLab
- **Devices:** Cisco IOS routers and Layer 3 switches
- **Routing:** Static Routing
- **Virtualization:** VRF
- **Tunnelling:** GRE

---

**Lab Status:** Completed

**ENCOR Domain:** Domain 2 – Virtualization

**Topics:** VRF, GRE Tunnelling and Static Routing.
