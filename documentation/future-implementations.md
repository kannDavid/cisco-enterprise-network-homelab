# Future Homelab Implementations

The Cisco Enterprise Homelab is an ongoing project. The current network provides the physical networking foundation for additional server, virtualization, Linux, security, and infrastructure projects.

This document outlines planned implementations that will expand the lab beyond the current Cisco routing and switching environment.

---

# 1. Homelab Server

## Goal

Add a dedicated physical server to the existing network infrastructure.

The server will provide the compute resources needed to run multiple virtual machines and services without relying entirely on my primary computer.

## Planned Uses

- Virtual machine hosting
- Linux server administration
- Windows Server experimentation
- Network services
- Centralized logging and monitoring
- Security testing
- Infrastructure troubleshooting

The server will connect to the existing Cisco network and allow virtual systems to communicate through the VLAN and routing infrastructure already implemented in the lab.

---

# 2. VMware Virtualization

## Goal

Deploy VMware virtualization on the homelab server to create and manage multiple virtual machines.

Virtualization will allow several operating systems and server roles to run simultaneously on a single physical system.

## Skills I Plan to Practice

- Creating and managing virtual machines
- Allocating CPU, memory, and storage
- Virtual networking
- Virtual switches
- VM network adapters
- Connecting virtual machines to physical VLANs
- VM snapshots
- Resource management
- Virtual infrastructure troubleshooting

## Network Integration

The goal is to integrate the VMware environment with the existing Cisco network rather than operate it as an isolated virtualization lab.

Virtual machines will eventually be placed into different network segments depending on their purpose.

For example:

- User/client systems → VLAN 10
- Server systems → VLAN 20
- Infrastructure management → VLAN 99

This will allow the physical Cisco infrastructure and virtual server environment to operate as one lab.

---

# 3. Linux Server Administration

## Goal

Deploy Linux virtual machines to develop practical Linux administration and troubleshooting skills.

## Skills I Plan to Practice

- Linux installation and configuration
- User and group management
- File permissions
- SSH administration
- Package management
- Service management
- Process monitoring
- Disk and memory monitoring
- Network configuration
- DNS troubleshooting
- Routing and connectivity troubleshooting
- Firewall configuration
- Log analysis

## Networking Tools

I also plan to gain hands-on experience with Linux networking and troubleshooting commands such as:

`ip addr`

`ip route`

`ping`

`traceroute`

`ss`

`dig`

`nslookup`

`curl`

`systemctl`

`journalctl`

These systems will provide Linux endpoints and servers that can interact with the existing Cisco network.

---

# 4. Windows Server and Active Directory

## Goal

Integrate a Windows Server environment with the physical network.

This would extend my existing Active Directory practice into the physical homelab infrastructure.

## Planned Services

- Active Directory Domain Services
- DNS
- DHCP experimentation
- Organizational Units
- Users and groups
- Group Policy
- Domain-joined Windows clients
- Account lockout and password reset troubleshooting
- File sharing and permissions

The goal is to connect domain-joined systems through the Cisco network and practice troubleshooting across both network and system infrastructure.

---

# 5. Network Monitoring and Logging

## Goal

Add centralized monitoring to gain visibility into the physical and virtual infrastructure.

## Areas to Explore

- Syslog
- SNMP
- Network monitoring platforms
- Linux system logs
- Interface utilization
- Device availability
- CPU and memory utilization
- Network event troubleshooting

Cisco routers and switches could forward information to a monitoring or logging server running as a virtual machine.

---

# 6. Network Security Expansion

## Goal

Expand the security configuration of the lab as additional endpoints and servers are introduced.

## Areas to Explore

- Inter-VLAN ACLs
- Management-plane restrictions
- SSH hardening
- DHCP Snooping
- Dynamic ARP Inspection
- Port Security
- BPDU Guard
- Firewall implementation
- Network segmentation
- Server access policies

These controls can be tested against actual Linux and Windows endpoints rather than only between networking devices.

---

# 7. Future Architecture

The long-term goal is to evolve the current Cisco routing and switching lab into a small enterprise-style infrastructure environment.

Planned architecture:

Internet / Home Network
        |
        |
   Cisco Edge Router
        |
        |
   Cisco Routing Layer
   OSPF / HSRP
        |
        |
   Cisco Switching Layer
   VLANs / LACP / STP
        |
        |
   Physical Server
        |
      VMware
        |
   -------------------------
   |          |            |
 Linux VM  Windows VM   Monitoring VM
   |          |            |
 VLAN 20    VLAN 20      VLAN 99
   |
Additional Clients / Services

This architecture will allow networking, virtualization, operating systems, security, and infrastructure troubleshooting to be practiced within the same environment.

---

# Project Direction

The objective of these future implementations is not simply to add more technologies to the lab.

The goal is to understand how different infrastructure components interact with each other and to gain experience troubleshooting problems across multiple layers:

Physical Infrastructure  
↓  
Cisco Switching  
↓  
Routing  
↓  
Network Security  
↓  
Virtualization  
↓  
Operating Systems  
↓  
Network Services  
↓  
Monitoring and Troubleshooting

As the lab expands, completed implementations will be documented separately with configurations, diagrams, verification commands, and troubleshooting examples.
