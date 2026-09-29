# CCNP Enterprise Network Project (GNS3)

A comprehensive CCNP-level enterprise network project covering design, implementation, and verification in a GNS3 environment.

![CCNP](https://img.shields.io/badge/CCNP-200--301-blue)
![GNS3](https://img.shields.io/badge/GNS3-Latest-green)

## 📋 Table of Contents
- [Project Overview](#project-overview)
- [Network Topology](#network-topology)
- [IP Addressing Scheme](#ip-addressing-scheme)
- [Technologies Used](#technologies-used)
- [Device Configuration](#device-configuration)
- [Verification & Testing](#verification--testing)
- [Lessons Learned](#lessons-learned)
- [References](#references)

---

## 🎯 Project Overview

This project simulates a Hub-and-Spoke enterprise network architecture consisting of a Headquarters (HQ) and two branch offices (Branch-1 and Branch-2). The primary objective is to practically implement core CCNA concepts—including VLANs, OSPF, NAT, and ACLs—within a GNS3 environment.

### Project Objectives
- Design a 3-site enterprise network topology.
- Implement dynamic routing using OSPF (Single-Area).
- Implement NAT/PAT for Internet connectivity.
- Implement Extended ACLs for inter-branch access control.
- Provide complete, reproducible documentation.

---

## 🏗️ Network Topology

![Topology](assets/topology.png)

### Architecture
- **R1 (Edge Router):** Connects to the simulated Internet (NAT Node).
- **HQ (Core Router):** The central hub for all branch connections.
- **Branch-1 / Branch-2:** Branch routers serving local clients.
- **SW-1 / SW-2:** Layer 2 switches connecting local PCs.

### Device Inventory
| Device | Role | Model | GNS3 Image |
|--------|------|-------|-----------|
| R1 | Edge Router | Cisco 7200 | c7200-adventerprisek9 |
| HQ | Core Router | Cisco 7200 | c7200-adventerprisek9 |
| Branch-1 | Branch Router | Cisco 7200 | c7200-adventerprisek9 |
| Branch-2 | Branch Router | Cisco 7200 | c7200-adventerprisek9 |
| SW-1 | L2 Switch | IOSvL2 | vios_l2-adventerprisek9 |
| SW-2 | L2 Switch | IOSvL2 | vios_l2-adventerprisek9 |

---

## 📊 IP Addressing Scheme

### WAN Links (Point-to-Point)
| Link | Subnet | R-Side | Other-Side |
|------|-------|--------|------------|
| R1 ↔ HQ | 10.0.0.0/30 | 10.0.0.1 | 10.0.0.2 |
| HQ ↔ Branch-1 | 10.0.1.0/30 | 10.0.1.1 | 10.0.1.2 |
| HQ ↔ Branch-2 | 10.0.2.0/30 | 10.0.2.1 | 10.0.2.2 |

### LAN Networks
| Subnet | Gateway | Description |
|-------|---------|-------------|
| 192.168.20.0/24 | 192.168.20.1 | Branch-1 Network (PC1: .10) |
| 192.168.30.0/24 | 192.168.30.1 | Branch-2 Network (PC2: .10) |

### Loopback Interfaces (Router ID)
| Router | Loopback |
|--------|----------|
| R1 | 1.1.1.1/32 |
| HQ | 2.2.2.2/32 |
| Branch-1 | 3.3.3.3/32 |
| Branch-2 | 4.4.4.4/32 |

---

## 🛠️ Technologies Used

| Technology | Purpose | Implementation Site |
|----------|---------|---------------------|
| **OSPF (Single-Area)** | Dynamic routing between routers | All routers |
| **NAT/PAT (Overload)** | Internal network Internet access | R1 |
| **Extended ACL** | Inter-branch access control | Branch-1 |
| **Default Route** | Default route to the Internet | R1 |
| **Loopback Interface** | Stable Router ID for OSPF | All routers |

---

## ⚙️ Device Configuration

Full configuration files are available in the [`configs/`](configs/) directory.

### 1️⃣ IP Configuration

**R1 (Edge):**
```cisco
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown
interface GigabitEthernet1/0
 ip address dhcp
 no shutdown
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
```

HQ (Core):

```cisco
interface GigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.252
interface GigabitEthernet1/0
 ip address 10.0.1.1 255.255.255.252
interface GigabitEthernet2/0
 ip address 10.0.2.1 255.255.255.252
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
```

2️⃣ OSPF Configuration

R1:

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
 default-information originate
```

HQ:

```cisco
router ospf 1
 router-id 2.2.2.2
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.1.0 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.3 area 0
 network 2.2.2.2 0.0.0.0 area 0
```

3️⃣ NAT/PAT Configuration on R1

```cisco
! Define Inside/Outside interfaces
interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet1/0
 ip nat outside

! Traffic allowed for NAT
access-list 1 permit 10.0.0.0 0.0.0.3
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255

! Enable PAT (Overload)
ip nat inside source list 1 interface GigabitEthernet1/0 overload

! Default Route
ip route 0.0.0.0 0.0.0.0 GigabitEthernet1/0 dhcp
```

4️⃣ Extended ACL Configuration on Branch-1

```cisco
! Block traffic from Branch-1 to Branch-2
ip access-list extended BLOCK_TO_BRANCH2
 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip any any

! Apply on LAN interface (inbound from PC1)
interface GigabitEthernet1/0
 ip access-group BLOCK_TO_BRANCH2 in
```

(Note: The ACL was applied to Gi0/1 or Gi1/0 depending on the physical port used for the LAN connection).

---

✅ Verification & Testing

1. OSPF Neighbor Check

```
HQ# show ip ospf neighbor
```

Expected output: 3 neighbors in FULL state.

2. Routing Table Check

```
HQ# show ip route ospf
```

You should see 192.168.20.0/24 and 192.168.30.0/24 marked with O.

3. NAT Translations Check

```
R1# show ip nat translations
```

Active translations from internal IPs to the public IP should be visible.

4. ACL Testing

| Test | Command | Expected Result |
| :--- | :--- | :--- |
| PC1 → PC2 | `ping 192.168.30.10` | ❌ Timeout (Blocked by ACL) |
| PC1 → Internet | `ping 8.8.8.8` | ✅ Success (Permitted) |
| PC2 → PC1 | `ping 192.168.20.10` | ✅ Success (ACL is one-way) |
| PC2 → Internet | `ping 8.8.8.8` | ✅ Success |

5. ACL Hit Counters

```
Branch-1# show ip access-lists BLOCK_TO_BRANCH2
```

The deny counter should increment with every PC1→PC2 ping attempt.

---

📚 Lessons Learned

1. Stable Router ID: Using Loopback interfaces for OSPF Router IDs ensures protocol stability.
2. ACL Placement: Extended ACLs must be placed close to the Source. Since PC1 enters Branch-1 via the LAN interface, the ACL was applied to the LAN port (e.g., Gi0/1), NOT the WAN port (Gi0/0).
3. NAT Boundaries: Precise configuration of ip nat inside and ip nat outside is critical—NAT will not function without it.
4. Default Route Propagation: default-information originate in OSPF allows all routers to learn the Internet route.
5. virbr0 Issue in GNS3: Using the NAT Node requires enabling the default network in libvirt (virsh net-start default).
6. /30 Masks for P2P Links: Using /30 instead of /24 conserves IP addresses on point-to-point WAN links.

---

🔗 References

· Cisco CCNA 200-301 Official Cert Guide
· GNS3 Documentation
· Cisco OSPF Configuration Guide

---

👤 Author

[Your Name] — [LinkedIn/GitHub Link]

```
```
