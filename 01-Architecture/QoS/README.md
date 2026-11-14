# Lab 8: QoS Classification, Marking and LLQ

## 1. Objective
To demonstrate Quality of Service (QoS) configuration on Cisco IOS using ACL-based traffic classification, Class Maps, Policy Maps and Service Policies.

The lab focuses on identifying UDP voice traffic, marking it with DSCP EF and applying Low Latency Queuing (LLQ).

## 2. Topology and IP Addressing

Two Ubuntu Linux hosts were connected through Cisco routers to generate and receive UDP traffic across the network.

| Device | Interface | IP Address |
|---|---|---|
| Ubuntu Source | Network | 192.168.10.10/24 |
| R1 (QoS Router) | G0/0 | 192.168.10.1/24 |
| R1 (QoS Router) | G0/2 | 10.10.10.1/30 |
| R2 | G0/0 | 10.10.10.2/30 |
| R2 | G0/1 | 192.168.20.1/24 |
| Ubuntu Destination | Network | 192.168.20.10/24 |

## 3. QoS Configuration

### Step 1: Define the Traffic Using an Extended ACL

Identify UDP traffic destined for ports 16384–32767.

```cisco
ip access-list extended VOICE-TRAFFIC
 permit udp 192.168.10.0 0.0.0.255 any range 16384 32767
```

### Step 2: Configure the Class Map

Create a Class Map to match traffic identified by the ACL.

```cisco
class-map match-all VOICE-CLASS
 match access-group name VOICE-TRAFFIC
```

### Step 3: Configure the Policy Map

Create a policy to mark voice traffic with DSCP EF and allocate priority bandwidth.

```cisco
policy-map QOS-POLICY
 class VOICE-CLASS
  set dscp ef
  priority percent 20
 class class-default
  fair-queue
```

### Step 4: Apply the Service Policy

Apply the QoS policy outbound on R1 GigabitEthernet0/2.

```cisco
interface GigabitEthernet0/2
 service-policy output QOS-POLICY
```

## 4. UDP Traffic Generation Using Python

Since the Ubuntu hosts did not have internet access for installing iperf3, Python's built-in socket library was used to generate UDP traffic.

### UDP Receiver – Ubuntu Destination

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.bind(("0.0.0.0", 20000))

print("UDP receiver listening on port 20000...")

total = 0

while True:
    data, addr = s.recvfrom(65535)
    total += len(data)
    print(f"Received {len(data)} bytes from {addr} | Total: {total}")
```

### UDP Sender – Ubuntu Source

```python
import socket
import time

s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

data = b"Q" * 1200
destination = ("192.168.20.10", 20000)

for i in range(10000):
    s.sendto(data, destination)
    time.sleep(0.001)

print("UDP traffic generation completed.")
```

The sender generated UDP packets towards destination port 20000, which falls within the ACL's configured destination port range.

## 5. Verification

The following command was used on R1 to verify QoS classification and policy operation:

```cisco
show policy-map interface GigabitEthernet0/2
```

The verification output confirmed:

- Increasing packet and byte counters under VOICE-CLASS.
- DSCP EF marking counters increasing.
- Traffic matching the configured ACL.
- The QoS policy successfully attached to the outbound interface.

## 6. Key Learning Outcomes

- Understanding the relationship between ACLs, Class Maps, Policy Maps and Service Policies.
- Implementing ACL-based QoS traffic classification.
- Applying DSCP EF marking for voice traffic.
- Understanding Low Latency Queuing (LLQ) using the priority command.
- Generating UDP traffic using Python socket programming.
- Verifying QoS classification and marking using Cisco IOS show commands.

**ENCOR 350-401 Domain:** Architecture – Quality of Service (QoS).

**Lab Environment:** PNETLab, Cisco IOS routers and Ubuntu 22.04 Linux hosts.
