# Multi-Campus-Enterprise-Network-Design-Implementation
Design and implementation of a scalable multi-campus enterprise network architecture featuring a 3-tier hierarchical design, RIPv2 dynamic routing, Serial DCE WAN links, 802.1Q Inter-VLAN routing (ROAS), router-based DHCP pools with server exclusions, Layer 3 distribution switches, and Cloud edge server integration in Cisco Packet Tracer.

## Project Overview & Rationale
This project showcases the design and implementation of an enterprise-grade multi-campus network infrastructure engineered in Cisco Packet Tracer. The architecture interconnects a Main Campus, a Branch Campus, and a Cloud Data Center / ISP Zone, spanning 10 distinct departmental VLANs and centralized application services.  

## Technical Highlights:
3-Tier Hierarchical Model: Structured architecture incorporating Access Layer 2 switches, Distribution Layer 3 switches, and Core/WAN Edge routers.
  - Dynamic Routing Core (RIPv2): Configured across WAN edge and cloud routers with Classless Routing enabled (ip classless) to establish multi-site convergence over point-      to-point Serial WAN links.
  - Inter-VLAN Routing & Sub-interfaces: Router-on-a-Stick (ROAS) 802.1Q encapsulation paired with sub-interfaces serving as departmental default gateways.
  - Router-Based DHCP & Exclusions: Router-configured DHCP pools delivering IP addressing, DNS, and domain parameters, with protected static exclusions for servers and           interfaces.
  - Distribution & Access Layer Switching: Layer 3 switches managing trunk aggregation alongside dedicated Layer 2 access switches for departmental isolation.
  - Cloud Edge & Server Hosting: External Cloud Router interfacing with static DMZ subnets hosting production Email, Web, and FTP enterprise servers.

## Network Architecture & Addressing Scheme
Subnet & Department Addressing Matrix

Location,Department / Zone,VLAN,Subnet / CIDR,Router Sub-Interface,Dynamic DHCP Pool
Main Campus,ADMIN,VLAN 10,192.168.1.0/24,Gi0/0.10 (192.168.1.1),ADMIN-POOL
Main Campus,HR,VLAN 20,192.168.2.0/24,Gi0/0.20 (192.168.2.1),HR-POOL
Main Campus,FINANCE,VLAN 30,192.168.3.0/24,Gi0/0.30 (192.168.3.1),FINANCE-POOL
Main Campus,BUSINESS,VLAN 40,192.168.4.0/24,Gi0/0.40 (192.168.4.1),BUSINESS-POOL
Main Campus,E&C,VLAN 50,192.168.5.0/24,Gi0/0.50 (192.168.5.1),E&C-POOL
Main Campus,A&D,VLAN 60,192.168.6.0/24,Gi0/0.60 (192.168.6.1),A&D-POOL
Main Campus,M-STUD-LAB,VLAN 70,192.168.7.0/24,Gi0/0.70 (192.168.7.1),M_STUD_LAB-POOL
Main Campus,IT-DEPT,VLAN 80,192.168.8.0/24,Gi0/0.80 (192.168.8.1),IT-POOL
Branch Campus,STAFF,VLAN 90,192.168.9.0/24,Gi0/0.90 (192.168.9.1),STAFF-POOL
Branch Campus,B-STUD-LAB,VLAN 100,192.168.10.0/24,Gi0/0.100 (192.168.10.1),B_STUD_LAB-POOL

## Point-to-Point WAN Links & Cloud DMZ

Link Connection,Subnet / CIDR,Device A Interface,Device B Interface,Clock Rate (DCE)
MAIN-ROUTER ↔ BRANCH-ROUTER,10.10.10.0/30,Main (Se0/3/1),Branch (Se0/3/0),64000 (MAIN)
MAIN-ROUTER ↔ CLOUD-ROUTER,10.10.10.4/30,Main (Se0/3/0),Cloud (Se0/3/0),64000 (CLOUD)
CLOUD-ROUTER ↔ EMAIL-SERVER,20.0.0.0/30,Cloud (Gi0/0),Server (20.0.0.2),N/A

## Core Configuration Highlights
1. Main Campus Router (DHCP, ROAS & RIPv2 Setup)

hostname Router

! IP Address Exclusions
ip dhcp excluded-address 192.168.8.253 192.168.8.254

! DHCP Pool Setup Example (IT Department)
ip dhcp pool IT-POOL
 network 192.168.8.0 255.255.255.0
 default-router 192.168.8.1
 dns-server 192.168.8.1
 domain-name IT.com

! 802.1Q Inter-VLAN Router Sub-Interface
interface GigabitEthernet0/0.80
 description CONNECTION TO VLAN 80
 encapsulation dot1Q 80
 ip address 192.168.8.1 255.255.255.0

! Serial DCE Link Configuration
interface Serial0/3/1
 ip address 10.10.10.1 255.255.255.252
 clock rate 64000

! Dynamic Routing Protocol Setup
router rip
 version 2
 network 10.0.0.0
 network 192.168.1.0
 network 192.168.2.0
 network 192.168.3.0
 network 192.168.4.0
 network 192.168.5.0
 network 192.168.6.0
 network 192.168.7.0
 network 192.168.8.0

2. Main Campus Layer 3 Switch Trunking & Access

hostname Switch

! Uplink Trunk Port to Gateway Router
interface GigabitEthernet1/0/1
 switchport mode trunk

! Access Port Assignments
interface GigabitEthernet1/0/2
 switchport access vlan 10
 switchport mode access

interface GigabitEthernet1/0/9
 switchport access vlan 80
 switchport mode access

 3. Layer 2 Access Switch (Department VLAN Enforcement)

hostname Switch

! FastEthernet & Gigabit Access Assignment
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access

interface GigabitEthernet0/1
 switchport access vlan 10
 switchport mode access

 ## Verification & Testing
 1. Dynamic Host Addressing (DHCP): End-user PCs and hosts across all departmental VLANs successfully received IP addresses, default gateways, and DNS settings dynamically.
 2. Static DMZ Configuration: Cloud Email Server configured statically with IPv4 address 20.0.0.2/30 pointing to default gateway
 3. Cross-Subnet & WAN Reachability: ICMP ping tests verified successful convergence across the RIPv2 core network:
     - PC0 (192.168.1.3) pinging Default Gateway 192.168.4.1: 0% Packet Loss.
     - PC0 (192.168.1.3) pinging Branch Staff Gateway 192.168.9.1: 0% Packet Loss
     - PC0 (192.168.1.3) pinging Branch Student Lab Gateway 192.168.10.1: 0% Packet Loss.
 
