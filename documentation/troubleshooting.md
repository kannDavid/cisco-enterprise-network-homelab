# Cisco Enterprise Homelab – Troubleshooting & Deployment Challenges

This document describes physical, hardware, and network configuration problems encountered while building and configuring my physical Cisco enterprise homelab.

Unlike a fully virtualized lab, working with physical networking equipment introduced additional challenges involving rack installation, cabling, device recovery, Ethernet termination, configuration persistence, and hardware troubleshooting.

The troubleshooting process generally followed the OSI model, beginning with physical connectivity before investigating switching, routing, and network services.

---

# Physical Deployment Challenges

## 1. Damaged Network Rack

### Problem

The network rack arrived damaged and was not properly aligned for equipment installation.

### Investigation

Before mounting the Cisco equipment, I inspected the rack and determined that parts of the frame needed to be realigned.

Installing equipment before correcting the rack could have resulted in improperly mounted or unstable hardware.

### Resolution

I manually adjusted and realigned the rack before installing the routers, switches, patch panel, and cabling.

### Lesson Learned

Physical infrastructure is part of network deployment.

Before configuring network devices, the rack, mounting hardware, power, and cabling infrastructure should be inspected and properly installed.

---

## 2. Ethernet Cable and RJ45 AWG Mismatch

### Problem

I initially purchased Cat6 28-AWG Ethernet cable but had RJ45 connectors designed for thicker 24-AWG conductors.

The cable and connectors were not a good physical match.

### Investigation

While preparing Ethernet cables for the lab, I identified that the conductor size of the cable did not properly match the RJ45 connectors I had purchased.

This could result in unreliable terminations or conductors failing to make proper contact with the connector pins.

### Root Cause

The RJ45 connectors were designed for a different conductor size than the 28-AWG Cat6 cable.

### Resolution

I purchased Cat5e 24-AWG cable that properly matched the RJ45 connectors and used it for the physical network connections.

### Lesson Learned

Ethernet category alone is not enough when selecting cabling components.

Cable conductor size (AWG), connector compatibility, conductor type, and termination quality should also be considered when building physical Ethernet infrastructure.

---

## 3. EtherChannel Link Negotiating at 100 Mbps – Faulty RJ45 Termination

### Problem

While building the four-link LACP EtherChannel between SW1 and SW2, I noticed that one of the physical links was negotiating at 100 Mbps instead of the expected 1 Gbps.

Because the EtherChannel was designed using four Gigabit Ethernet links, I needed to determine whether the problem was caused by the switch configuration or the physical connection.

### Investigation

I checked the affected interface and confirmed that the link was operational, but it was negotiating at only 100 Mbps.

Since the connection was working at a reduced speed rather than being completely down, I investigated the physical layer before changing the LACP or trunk configuration.

I inspected the Ethernet cable and discovered that one conductor was not properly seated inside the RJ45 connector.

### Root Cause

The Ethernet cable had been incorrectly terminated.

One conductor was not making proper contact inside the RJ45 connector, preventing the cable from correctly supporting Gigabit Ethernet.

The problem was therefore a Layer 1 cabling issue rather than an LACP or switch configuration problem.

### Resolution

I corrected the RJ45 termination and reconnected the cable.

After fixing the termination, the interface negotiated at the expected Gigabit speed.

I then verified the four-link EtherChannel using:

`show etherchannel summary`

The final EtherChannel showed:

- `Po1(SU)` – Layer 2 Port-channel in use
- All four physical interfaces in `(P)` state – successfully bundled into the Port-channel

### Lesson Learned

A physical Ethernet connection can appear functional while still having a cabling problem.

The fact that the interface was up did not mean Layer 1 was completely healthy. Checking negotiated speed helped identify that something was wrong with the physical connection.

This reinforced a physical-first troubleshooting process:

1. Check interface status.
2. Check negotiated speed and duplex.
3. Inspect and test the cable.
4. Verify the RJ45 termination.
5. Verify EtherChannel membership.
6. Investigate LACP or trunk configuration only if the physical layer is healthy.


The incident demonstrated how a Layer 1 problem can directly affect a Layer 2 technology such as EtherChannel.
---

# Device Recovery & Initial Configuration

## 4. Recovering Access to Secondhand Cisco Equipment

### Problem

The Cisco routers and switches were purchased secondhand and still contained configuration from their previous environment, including authentication settings that prevented normal administrative access.

### Investigation

Before building the lab, I needed to regain administrative access and remove the previous configuration.

This required using Cisco device recovery procedures rather than simply logging into IOS normally.

### Resolution

I used the Cisco password/configuration recovery process to bypass the existing startup configuration, regain administrative access, erase the previous configuration, and prepare the devices for the homelab.

### Lesson Learned

Secondhand enterprise networking equipment may retain configuration and credentials from its previous environment.

Understanding boot behavior, startup configuration, and password recovery procedures is important when repurposing used network equipment.

---

## 5. Router ROMMON / Break Signal Problem

### Problem

During recovery of the Cisco routers, I attempted to interrupt the normal boot process and enter ROMMON mode.

The expected break sequence from the PuTTY console session was not being detected by the routers.

### Investigation

The console connection itself was functioning, but the router continued through its normal boot process instead of entering ROMMON.

This indicated that the problem was not basic console connectivity but the boot interruption process.

### Resolution

I used the router's removable CompactFlash/boot storage behavior to prevent a normal IOS boot and force the router into ROMMON.

After entering ROMMON, I reinserted the storage and continued the recovery procedure so that I could regain access and clear the previous configuration.

### Lesson Learned

Device recovery sometimes requires understanding the boot process below the normal IOS CLI.

This provided hands-on experience with:

- Cisco ROMMON
- Boot behavior
- Console access
- Startup configuration recovery
- Physical device storage

---

## 6. VLAN Database Persisted After Configuration Reset

### Problem

After clearing the previous switch configuration, old VLAN information was still present when I began creating the VLANs for my homelab.

### Investigation

I initially expected clearing the startup configuration to return the switch completely to a clean state.

When configuring the new VLAN environment, I discovered that VLAN information from the previous configuration had persisted.

### Root Cause

The VLAN database is stored separately in the `vlan.dat` file.

Clearing the startup configuration did not remove this file.

### Resolution

I deleted the existing `vlan.dat` file and reset/reloaded the switch before rebuilding the VLAN configuration.

### Lesson Learned

On Cisco switches, the startup configuration and VLAN database are separate configuration components.

When completely resetting a switch, both the startup configuration and existing VLAN database should be considered.

---

# Network Configuration Troubleshooting

---

## 7. Incorrect DNS Setting in DHCP Configuration

### Problem

A client could receive network configuration through DHCP, but DNS-related connectivity did not behave as expected.

### Investigation

Instead of assuming the entire network connection had failed, I separated IP connectivity from name resolution.

I reviewed the DHCP configuration being provided to clients and inspected the DNS server value.

### Root Cause

An incorrect DNS server value had been configured in the DHCP pool.

### Resolution

I corrected the DNS server configuration in the DHCP pool and renewed/retested the client configuration.

### Lesson Learned

Successful DHCP assignment does not guarantee that every DHCP option is correct.

When a client has an IP address and gateway but cannot resolve names, DNS configuration should be investigated separately from general IP connectivity.

---

## 8. ACL Affecting ICMP / Connectivity Testing

### Problem

During connectivity testing, expected ICMP traffic was not reaching its destination.

### Investigation

I verified addressing and routing before reviewing traffic filtering.

Because the route existed but the traffic was still being blocked, I inspected the ACL configuration along the traffic path.

### Root Cause

A `deny ip any any` entry had accidentally been placed above the ACL entry intended to permit the traffic.

Because Cisco ACLs are processed from top to bottom and stop at the first matching entry, the `deny ip any any` statement matched the ICMP traffic before the router could reach the permit statement.

### Resolution

I corrected the ACL entry order so that the required traffic was permitted before the deny statement.

I then repeated the ICMP connectivity test and confirmed that the traffic was successfully permitted.

### Lesson Learned

ACL entry order is critical because Cisco processes access control entries sequentially from top to bottom and stops at the first match.

A broad deny statement placed too early in an ACL can unintentionally block legitimate traffic, even when a permit statement exists later in the list.

---

# Troubleshooting Methodology

The homelab reinforced a layered troubleshooting process:

Physical Layer
↓
Cabling / Link Status / Speed
↓
VLAN & Switching
↓
Trunks / EtherChannel
↓
IP Addressing
↓
Routing / OSPF
↓
Gateway Redundancy / HSRP
↓
ACLs / Security
↓
DHCP / DNS
↓
Application Connectivity

Instead of changing multiple configurations at once, I learned to identify the failing layer, collect evidence, make a targeted change, and repeat the original test to confirm the resolution.

---

# Useful Troubleshooting Commands

## Interfaces

`show ip interface brief`

`show interfaces`

## VLANs and Trunks

`show vlan brief`

`show interfaces trunk`

## EtherChannel

`show etherchannel summary`

## Routing

`show ip route`

`show ip ospf neighbor`

## HSRP

`show standby brief`

## DHCP

`show ip dhcp binding`

## NAT

`show ip nat translations`

## Spanning Tree

`show spanning-tree vlan 10`

`show spanning-tree vlan 20`

`show spanning-tree vlan 99`

These commands were used throughout the lab to verify device state and isolate problems across different network layers.
