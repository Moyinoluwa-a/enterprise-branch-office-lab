# Enterprise Branch Office Resilient WAN Lab
**Cisco CML Lab Documentation | CCNA-Level Implementation**

---

## Project Overview
**Lab Name:** Enterprise Branch Office Resilient WAN Lab  
**Platform:** Cisco Modeling Labs (CML)  
**Scenario:** Regional branch office requiring high availability, departmental segmentation, and redundant WAN connectivity to headquarters  
**Complexity Level:** CCNA+ with enterprise features  

## Business Use Case
This lab simulates a small branch office (25-30 users across Sales, Admin, and IT departments) using enterprise-grade design patterns that can scale to support 150+ users. The network requires:
- **Zero downtime** for critical business operations
- **Departmental isolation** for security compliance  
- **Automatic failover** if primary WAN link fails
- **Controlled inter-VLAN communication** per company security policy

## Network Architecture
**Design Pattern:** Collapsed Core (Distribution + Core combined)  
**Why This Design:** Cost-effective for branch offices while maintaining redundancy and scalability

### Key Components:
- **2x Edge Routers** (EDGE-R1/R2) - Dual WAN connectivity with automatic failover
- **2x Distribution Switches** (DIST-SW1/SW2) - L3 switching with HSRP gateway redundancy  
- **2x Access Switches** (ACC-1/ACC-2) - Redundant access layer connectivity
- **3x VLANs** - Departmental segmentation (Sales/Admin/IT)

## Technologies Implemented

### 1. OSPF Dynamic Routing
- **Area 0** single-area deployment
- **MD5 Authentication** on all OSPF adjacencies for security
- **Router IDs** manually configured for consistency
- **Passive interfaces** on WAN links to prevent unnecessary LSAs

### 2. HSRP Gateway Redundancy
- **Load balancing approach:** Different VLANs use different primary gateways
  - VLAN 10 (Sales): DIST-SW1 primary (priority 110)
  - VLAN 20 (Admin): DIST-SW1 primary (priority 110) 
  - VLAN 30 (IT): DIST-SW2 primary (priority 110)
- **Preemption enabled** for automatic failback when primary recovers

### 3. WAN Redundancy Strategy
- **Primary Path:** Each edge router has direct route to HQ (AD=1)
- **Backup Path:** Floating static routes through peer edge router (AD=10)
- **Automatic Failover:** If primary WAN fails, traffic routes through secondary edge router

### 4. Security Implementation
**Extended ACL "SALES"** on VLAN 10:
- ✅ **ALLOW:** Sales → Admin (business requirement)
- ✅ **ALLOW:** Sales → Internet (web access)
- ✅ **ALLOW:** IT → Sales (ICMP replies only for troubleshooting)
- ❌ **DENY:** Sales → IT (security isolation)

### 5. VLAN Design
| VLAN | Name | Network | Gateway | Users |
|------|------|---------|---------|-------|
| 10 | SALES | 192.168.10.0/24 | .1 | Sales team |
| 20 | ADMIN | 192.168.20.0/24 | .1 | Admin/Finance |
| 30 | IT | 192.168.30.0/24 | .1 | IT Department |

## Configuration Highlights

### OSPF Authentication Setup
```
router ospf 1
 router-id 1.1.1.1
 network 10.1.1.0 0.0.0.255 area 0

interface g0/1
 ip ospf authentication
 ip ospf message-digest-key 1 md5 cisco
```

### HSRP Load Balancing
```
! DIST-SW1 - Primary for Sales & Admin
interface Vlan10
 standby 10 priority 110
 standby 10 preempt

! DIST-SW2 - Primary for IT  
interface Vlan30
 standby 30 priority 110
 standby 30 preempt
```

### Floating Static Routes
```
! Primary route (AD=1)
ip route 0.0.0.0 0.0.0.0 10.0.100.1

! Backup route (AD=10) - only used if primary fails
ip route 0.0.0.0 0.0.0.0 10.1.1.2 10
```

## Verification Commands Used

### Connectivity Testing
- `ping 8.8.8.8` from each VLAN to verify internet connectivity
- `traceroute 8.8.8.8` to confirm primary/backup path selection

### OSPF Verification  
- `show ip ospf neighbor` - Verify all adjacencies established
- `show ip ospf database` - Confirm LSA propagation
- `show ip route ospf` - Verify learned routes

### HSRP Status
- `show standby` - Confirm active/standby roles per VLAN
- `show standby brief` - Quick status overview

### ACL Verification
- `show access-lists` - Verify hit counts on ACL rules
- Test inter-VLAN communication per security policy

## Lab Testing Results
✅ **WAN Failover:** Simulated primary WAN failure - traffic automatically switched to backup path within 30 seconds  
✅ **Gateway Redundancy:** Shutdown DIST-SW1 - VLAN 10/20 users maintained connectivity via DIST-SW2  
✅ **VLAN Security:** Sales users blocked from accessing IT network as designed  
✅ **OSPF Convergence:** All routes learned correctly, no routing loops detected

## What I'd Add in Version 2.0
- **DHCP services** for automatic IP assignment and easier scalability
- **BGP** for better WAN path control and multihoming
- **QoS** policies for voice/video traffic prioritization  
- **Network automation** using Ansible for config management
- **SNMP monitoring** with network performance dashboards
- **VPN** integration for remote worker access
- **Additional security** features like port security and DHCP snooping

## Skills Demonstrated
- **Enterprise Network Design** - Redundancy and high availability planning
- **Dynamic Routing** - OSPF configuration with authentication  
- **Gateway Redundancy** - HSRP implementation with load balancing
- **Security Policy** - ACL design for departmental isolation
- **Troubleshooting** - Systematic verification of network functionality
- **Documentation** - Professional network documentation practices

---
*This lab represents 15+ hours of design, implementation, and testing - demonstrating hands-on experience with enterprise networking technologies.*
