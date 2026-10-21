# DHCP Relay and DNS Integration Lab — Windows Server 2019

## 1. Lab Overview

This lab demonstrates centralized DHCP address allocation using Windows Server 2019, Cisco IOS DHCP relay, forward DNS, reverse DNS, and end-to-end client validation.

**Objectives**
- Configure Windows Server 2019 as the centralized DHCP and DNS server.
- Configure a Cisco router to relay DHCP requests from a remote LAN.
- Create forward and reverse DNS records.
- Distribute DNS settings through DHCP.
- Verify DHCP and DNS operation from a Windows client.

## 2. Lab Topology and IP Addressing

| Device | Interface / Role | IP address |
|---|---|---|
| PC3 | DHCP client | `192.168.20.22/24` |
| R1 | LAN gateway | `192.168.20.1/24` |
| R1 | Server-facing interface | `192.168.30.1/30` |
| Windows Server 2019 | DHCP and DNS | `192.168.30.2/30` |

**DNS namespace:** `pnetlab.example.com`

**Server FQDN:** `server.pnetlab.example.com`

## Scenario 1 — Centralized DHCP Using a Cisco DHCP Relay

### Objective
Use Windows Server as the DHCP server for PC3, which is on a different subnet.

### Windows DHCP configuration
- Create a DHCP scope for `192.168.20.0/24`.
- Configure an address range of `192.168.20.21–192.168.20.254`.
- Configure the default gateway as `192.168.20.1`.
- Configure DNS server `192.168.30.2`.
- Configure DNS suffix `pnetlab.example.com`.

### Cisco router configuration

Remove the router's local DHCP pool for this LAN to avoid competing DHCP responses.

Configure the relay on the client-facing interface:

```cisco
interface GigabitEthernet0/1
 description LAN2 - PC3 DHCP CLIENTS
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.30.2
 no shutdown
```

The server-facing interface is addressed as `192.168.30.1/30`.

### How it works
1. PC3 broadcasts a DHCP Discover message.
2. R1 receives the broadcast on its LAN interface.
3. `ip helper-address` relays the request to `192.168.30.2`.
4. R1 sets the relay information (`giaddr`) so Windows DHCP selects the correct scope.
5. Windows Server returns the DHCP offer through the relay.
6. PC3 receives an address, gateway, and DNS configuration.

### Verification

On PC3:

```cmd
ipconfig /all
```

Verified client configuration:
- IPv4 address: `192.168.20.22`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.20.1`
- DHCP enabled: `Yes`
- DNS server: `192.168.30.2`
- Connection-specific DNS suffix: `pnetlab.example.com`

**Result: PASS** — Centralized DHCP and relay operation verified.

## Scenario 2 — Forward DNS Lookup (A Record)

### Objective
Resolve the Windows Server hostname to its IPv4 address.

### DNS configuration
Create a primary forward lookup zone:

`pnetlab.example.com`

Create the following A record:

| Field | Value |
|---|---|
| Host | `server` |
| FQDN | `server.pnetlab.example.com` |
| IPv4 address | `192.168.30.2` |

When creating the record, enter only `server` in the Host field. Windows automatically appends the zone suffix.

### Verification

On Windows Server:

```cmd
nslookup server.pnetlab.example.com 127.0.0.1
```

On PC3:

```cmd
nslookup server.pnetlab.example.com
ping server.pnetlab.example.com
```

Expected resolution:

`server.pnetlab.example.com` → `192.168.30.2`

A successful name resolution with ping does not necessarily mean ICMP replies are permitted; Windows Firewall may block the replies.

**Result: PASS** — Forward lookup verified on the server and client.

## Scenario 3 — Reverse DNS Lookup (PTR Record)

### Objective
Resolve the server's IPv4 address back to its FQDN.

### DNS configuration
Create an IPv4 reverse lookup zone for the `192.168.30.0/24` namespace:

`30.168.192.in-addr.arpa`

The server itself is addressed in a `/30` network, but a `/24` reverse zone was used for this lab.

Create a PTR record mapping:

`192.168.30.2` → `server.pnetlab.example.com`

Use the existing A record through the DNS Manager Browse option when selecting the PTR target. This avoids accidentally appending the zone suffix twice.

### Verification

On Windows Server:

```cmd
nslookup 192.168.30.2 127.0.0.1
```

On PC3:

```cmd
nslookup 192.168.30.2
```

Expected result:

`192.168.30.2` → `server.pnetlab.example.com`

**Result: PASS** — Reverse lookup verified.

## Scenario 4 — End-to-End DHCP and DNS Client Validation

### Objective
Verify that PC3 receives the correct DHCP options and can use the Windows DNS server.

### DHCP scope options

| Option | Value |
|---|---|
| 003 Router | `192.168.20.1` |
| 006 DNS Servers | `192.168.30.2` |
| 015 DNS Domain Name | `pnetlab.example.com` |

### Client verification

On PC3:

```cmd
ipconfig /all
nslookup server.pnetlab.example.com
nslookup 192.168.30.2
ping server.pnetlab.example.com
```

Verified results:
- PC3 received `192.168.20.22/24`.
- The default gateway was `192.168.20.1`.
- The DHCP-provided DNS server was `192.168.30.2`.
- The DNS suffix was `pnetlab.example.com`.
- Forward DNS resolved the server FQDN to `192.168.30.2`.
- Reverse DNS resolved `192.168.30.2` to `server.pnetlab.example.com`.
- Ping resolved the FQDN to the expected IPv4 address.

**Result: PASS** — End-to-end DHCP and DNS integration verified.

## Key Lessons Learned

1. A DHCP relay is required when clients and the DHCP server are on different IP subnets.
2. The router's `ip helper-address` must be configured on the client-facing interface.
3. A competing local DHCP pool can cause clients to receive unintended settings.
4. DHCP Option 006 distributes the DNS server address; Option 015 distributes the DNS suffix.
5. Forward and reverse DNS require their respective A and PTR records.
6. DNS queries must be sent to a server authoritative for the lab zone. Public DNS such as `8.8.8.8` will not resolve this private lab namespace.
7. Verify both the DNS server and client; a successful local lookup alone does not prove end-to-end operation.

## Lab Status

| Component | Status |
|---|---|
| Windows DHCP scope | Verified |
| Cisco DHCP relay | Verified |
| DHCP client options | Verified |
| Forward DNS | Verified |
| Reverse DNS | Verified |
| Client FQDN resolution | Verified |
| NTP integration | Planned — separate next phase |

## Next Phase — NTP

Extend this lab by configuring Windows Server `192.168.30.2` as an NTP source and synchronizing the Cisco router to it. Validate synchronization using:

```cisco
show ntp status
show ntp associations
show clock detail
```

NTP configuration and verification will be documented separately after the lab is completed.
