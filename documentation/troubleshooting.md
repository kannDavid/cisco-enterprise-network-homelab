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

## 3. Ethernet Link Negotiating at 100 Mbps

### Problem

One Ethernet connection negotiated at 100 Mbps instead of the expected 1 Gbps.

### Investigation

Because the connection was operational but running below the expected speed, I investigated the physical layer rather than immediately changing switch configuration.

I inspected the Ethernet termination and discovered that one conductor was not properly seated in the RJ45 connector.

### Root Cause

An improperly seated conductor prevented the cable from providing all of the wire pairs required for Gigabit Ethernet operation.

### Resolution

I corrected the RJ45 termination and retested the connection.

The link was then able to negotiate at the expected Gigabit speed.

### Lesson Learned

A cable can appear functional while still having a physical defect.

A successful link does not necessarily mean the physical layer is completely healthy. Link speed, duplex, interface counters, and cable termination should also be checked.

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

## 7. LACP EtherChannel – Faulty Cable Termination

### Problem

While configuring the four-link LACP EtherChannel between SW1 and SW2, one of the physical links was not operating correctly.

The EtherChannel uses four physical Gigabit Ethernet connections between the two switches.

### Investigation

Because the EtherChannel configuration was otherwise functioning, I investigated the physical links participating in the bundle.

I checked the individual interfaces and inspected the Ethernet cabling.

One of the Ethernet cables had been incorrectly terminated. A conductor inside the RJ45 connector was not properly seated, preventing that physical link from operating correctly.

### Root Cause

The problem was not an LACP or switch configuration mismatch.

The root cause was a faulty RJ45 cable termination on one of the physical links participating in the EtherChannel.

### Resolution

I corrected the Ethernet cable termination and reconnected the link.

I then verified the EtherChannel using:

`show etherchannel summary`

After correcting the cable, the EtherChannel showed:

- `Po1(SU)` – Port-channel operating as a Layer 2 EtherChannel and in use
- All four member interfaces in `(P)` state – successfully bundled in the Port-channel

### Lesson Learned

Not every EtherChannel problem is caused by switch configuration.

Because EtherChannel depends on multiple physical links, Layer 1 problems can affect the bundle even when the LACP and trunk configuration is correct.

This reinforced the importance of troubleshooting from the physical layer upward:

1. Check interface/link status.
2. Inspect and test physical cabling.
3. Verify speed and duplex.
4. Verify EtherChannel membership.
5. Only then investigate LACP or trunk configuration if necessary.

It also demonstrated why commands such as `show etherchannel summary` and interface status commands are useful for identifying whether individual physical links are successfully participating in the bundle.

---

## 8. Incorrect DNS Setting in DHCP Configuration

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

## 9. ACL Affecting ICMP / Connectivity Testing

### Problem

During connectivity testing, expected ICMP traffic was not reaching its destination.

### Investigation

I verified addressing and routing before reviewing traffic filtering.

Because the route existed but the traffic was still being blocked, I inspected the ACL configuration along the traffic path.

### Root Cause

An ACL rule was preventing the expected ICMP traffic.

### Resolution

I reviewed and corrected the ACL behavior and then repeated the original connectivity test.

### Lesson Learned

Routing determines where traffic should go, while ACLs determine whether that traffic is permitted.

A valid route does not guarantee successful connectivity if a security policy blocks the traffic.

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
