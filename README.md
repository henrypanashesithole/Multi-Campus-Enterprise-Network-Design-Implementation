# Multi-Campus-Enterprise-Network-Design-Implementation
Design and implementation of a scalable multi-campus enterprise network architecture featuring a 3-tier hierarchical design, RIPv2 dynamic routing, Serial DCE WAN links, 802.1Q Inter-VLAN routing (ROAS), router-based DHCP pools with server exclusions, Layer 3 distribution switches, and Cloud edge server integration in Cisco Packet Tracer.

## Project Overview & Rationale
This project showcases the design and implementation of an enterprise-grade multi-campus network infrastructure engineered in Cisco Packet Tracer. The architecture interconnects a Main Campus, a Branch Campus, and a Cloud Data Center / ISP Zone, spanning 10 distinct departmental VLANs and centralized application services.  

Key Technical Concepts & Highlights
3-Tier Hierarchical Model: Structured multi-layer architecture incorporating Access, Distribution (Layer 3 switching), and Core/WAN Routing layers.  Dynamic Routing Core (RIPv2): Configured across WAN edge and cloud routers with Classless Routing enabled (ip classless) to establish multi-site convergence over point-to-point Serial WAN links.  Inter-VLAN Routing (ROAS): 802.1Q trunking paired with router sub-interfaces serving as default gateways across 10 operational VLANs.  Dynamic Host Configuration (DHCP): Router-based DHCP pools providing automated IPv4 addressing, domain parameters, and default gateways, coupled with protected IP exclusions for static servers and gateways.  Layer 3 & Layer 2 Switching: Layer 3 distribution switches managing core trunk aggregation, paired with dedicated Layer 2 access switches enforcing departmental VLAN boundaries.  Cloud & Server Integration: Public-facing Cloud edge router interfacing with hosted enterprise services (Email, Web, and FTP Servers) via static WAN subnets.  Network Architecture & Addressing SchemeDepartmental Subnet & VLAN MatrixLocationDepartment / ZoneVLAN IDSubnet / CIDRDefault Gateway (Sub-interface)Dynamic DHCP PoolMain CampusADMINVLAN 10192.168.1.0/24Gi0/0.10 (192.168.1.1)  ADMIN-POOL  Main CampusHRVLAN 20192.168.2.0/24Gi0/0.20 (192.168.2.1)  HR-POOL  Main CampusFINANCEVLAN 30192.168.3.0/24Gi0/0.30 (192.168.3.1)  FINANCE-POOL  Main CampusBUSINESSVLAN 40192.168.4.0/24Gi0/0.40 (192.168.4.1)  BUSINESS-POOL  Main CampusE&CVLAN 50192.168.5.0/24Gi0/0.50 (192.168.5.1)  E&C-POOL  Main CampusA&DVLAN 60192.168.6.0/24Gi0/0.60 (192.168.6.1)  A&D-POOL  Main CampusM-STUD-LABVLAN 70192.168.7.0/24Gi0/0.70 (192.168.7.1)  M_STUD_LAB-POOL  Main CampusIT-DEPTVLAN 80192.168.8.0/24Gi0/0.80 (192.168.8.1)  IT-POOL  Branch CampusSTAFFVLAN 90192.168.9.0/24Gi0/0.90 (192.168.9.1)  STAFF-POOL  Branch CampusB-STUD-LABVLAN 100192.168.10.0/24Gi0/0.100 (192.168.10.1)  B_STUD_LAB-POOL  Point-to-Point WAN & Cloud InfrastructureLink DescriptionSubnet / CIDRInterface (Device A)Interface (Device B)Clock Rate (DCE)Main ↔ Branch WAN10.10.10.0/30  MAIN (Se0/3/1)  BRANCH (Se0/3/0)  64000 (MAIN)  Main ↔ Cloud WAN10.10.10.4/30  MAIN (Se0/3/0)  CLOUD (Se0/3/0)  64000 (CLOUD)  Cloud Server DMZ20.0.0.0/30  CLOUD (Gi0/0)  Email Server  N/ACore Configuration Snippets1. Main Campus Router (ROAS, DHCP Pools & Exclusions, RIPv2)Cisco CLIhostname Router

! Protected Static Server/Gateway Addresses
ip dhcp excluded-address 192.168.8.253 192.168.8.254

! DHCP Pool Example (IT Department)
ip dhcp pool IT-POOL
 network 192.168.8.0 255.255.255.0
 default-router 192.168.8.1
 dns-server 192.168.8.1
 domain-name IT.com

! Sub-Interface Inter-VLAN Routing (ROAS)
interface GigabitEthernet0/0.80
 description CONNECTION TO VLAN 80
 encapsulation dot1Q 80
 ip address 192.168.8.1 255.255.255.0

! Serial DCE Link Setup
interface Serial0/3/1
 ip address 10.10.10.1 255.255.255.252
 clock rate 64000

! RIPv2 Routing Setup
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

### 2. Main Campus Layer 3 Distribution Switch (Trunk & Access Ports)

```cisco
hostname Switch

! 802.1Q Uplink Trunk to Main Router
interface GigabitEthernet1/0/1
 switchport mode trunk

! VLAN Access Port Assignment Example
interface GigabitEthernet1/0/2
 switchport access vlan 10
 switchport mode access

interface GigabitEthernet1/0/9
 switchport access vlan 80
 switchport mode access


### 3. Layer 2 Access Switch Configuration (Access Layer Example)

```cisco
hostname Switch

! Port Mapping across Access Interfaces
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access

interface GigabitEthernet0/1
 switchport access vlan 10
 switchport mode access


---

## Verification & Testing

### 1. Dynamic Host Addressing (DHCP) Verification
Host endpoints in VLANs auto-acquired parameters including IP address, subnet mask, default gateway, and local DNS settings from their respective campus router pools

### 2. Static Server Provisioning
The external Email Server was provisioned with static IPv4 addressing (`20.0.0.2/30`) targeting the Cloud Gateway (`20.0.0.1`)

### 3. End-to-End Cross-Campus Ping Reachability
ICMP diagnostic tests were conducted across multiple VLAN boundaries, confirming convergence across the RIPv2 core
