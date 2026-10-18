# DHCP Server and DHCP Relay Lab

## 1. Overview

This lab demonstrates DHCP operation using Cisco IOS and Microsoft Windows Server 2019. Cisco IOS provides DHCP services for LAN1, while Windows Server provides DHCP services for LAN2 through a Cisco IOS DHCP relay agent.

The lab was built and tested in PNETLab.

## 2. Lab Objectives

- Configure a Cisco IOS DHCP pool.
- Install and configure the DHCP Server role on Windows Server 2019.
- Configure a DHCP scope for a remote client subnet.
- Configure `ip helper-address` on a Cisco router.
- Verify DHCP address allocation and relay operation.
- Troubleshoot DHCP connectivity and client address assignment.

## 3. Topology and IP Addressing

| Device / Interface | IP Address | Purpose |
|---|---|---|
| R1 Gi0/0 | 192.168.10.1/24 | LAN1 gateway |
| R1 Gi0/1 | 192.168.20.1/24 | LAN2 gateway and DHCP relay |
| R1 Gi0/2 | 192.168.30.1/30 | Windows Server network |
| Windows Server 2019 | 192.168.30.2/30 | DHCP server |
| PC1 and PC2 | DHCP | LAN1 clients |
| PC3 | DHCP | LAN2 client |

## 4. DHCP Design

### LAN1 — Cisco IOS DHCP

R1 provides DHCP services to clients in `192.168.10.0/24`.

Example configuration:

```cisco
ip dhcp excluded-address 192.168.10.1 192.168.10.20

ip dhcp pool DHCP-LAN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
```

### LAN2 — Windows Server DHCP

Windows Server 2019 provides DHCP services to clients in `192.168.20.0/24`.

Scope configuration:

- Scope network: `192.168.20.0/24`
- Address range: `192.168.20.21–192.168.20.254`
- Default gateway: `192.168.20.1`
- DNS server: `8.8.8.8`

The LAN2 DHCP pool was removed from R1 so that Windows Server could assign LAN2 addresses.

## 5. DHCP Relay Configuration

R1 relays DHCP requests from LAN2 to Windows Server at `192.168.30.2`.

```cisco
interface GigabitEthernet0/1
 description DHCP-RELAY-CLIENT-LAN
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.30.2
```

The `ip helper-address` command enables the router to forward DHCP client requests to the remote DHCP server.

## 6. DHCP Operation

The DHCP process follows DORA:

1. **Discover** — The client broadcasts a DHCP discovery message.
2. **Offer** — The DHCP server offers an IP address.
3. **Request** — The client requests the offered address.
4. **Acknowledgement (ACK)** — The server confirms the lease.

DHCP broadcasts do not normally cross routers. R1 relays the client's request to Windows Server. The relay agent uses the gateway address, `192.168.20.1`, in the DHCP relay information (`giaddr`) so the server can select the correct LAN2 scope.

## 7. Verification

### Cisco IOS

Check the relay configuration:

```cisco
show running-config interface GigabitEthernet0/1
```

Check DHCP bindings served by R1:

```cisco
show ip dhcp binding
```

Check interface status and addressing:

```cisco
show ip interface brief
```

Check ARP entries:

```cisco
show ip arp
```

### Windows Server 2019

- Confirm the DHCP Server role is installed.
- Confirm the LAN2 scope is active.
- Inspect the Address Leases list for assigned client addresses.
- Verify that PC3 receives an address in `192.168.20.0/24`, with gateway `192.168.20.1`.

### Client

On a Windows client, use:

```powershell
ipconfig /all
ipconfig /release
ipconfig /renew
```

## 8. Key Learning Outcomes

- DHCP server configuration on Cisco IOS and Windows Server.
- DHCP relay operation across routed networks.
- The role of `ip helper-address` and `giaddr`.
- DHCP scope and default-gateway configuration.
- Verification and troubleshooting using router CLI, the Windows DHCP console, and client commands.

## 9. Lab Environment

- PNETLab
- Cisco IOS router
- Windows Server 2019
- Windows client VMs
