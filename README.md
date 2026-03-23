# 🏢 Enterprise Branch Network — GNS3 Simulation

> **Multi-site enterprise network with OSPF dynamic routing, VLAN segmentation, NAT/PAT, DMZ architecture, and pfSense perimeter firewall — fully simulated in GNS3 with real VM integration.**

---

## 📋 Project Overview

This project simulates a real-world enterprise network for a fictional company (**TechBrew Coffee**) connecting a headquarters office to a remote branch office. The topology covers all major CCNA-level technologies: dynamic routing, VLAN segmentation, inter-VLAN routing, DHCP, NAT/PAT, DMZ isolation, named ACLs, and perimeter firewall security.

All configurations were built from scratch, tested end-to-end, and documented to professional standards.

---

## 🗺️ Network Topology

```
Internet (Cloud1)
      |
   HQ Router (C7200) — 203.0.113.0/30
      |
   BR2 Router (C7200) — NAT/PAT, DMZ gateway
      |        \
   BR1 Router   DMZSW (IOU L2) — VLAN 100
   (Inter-VLAN    |
    routing)    SERVER (192.168.100.10)
      |
   LANSW (IOU L2)
   ├── VLAN 10 — HR Dept (PC1, PC2) — 192.168.10.0/24
   ├── VLAN 20 — Sales Dept (PC3, PC4) — 192.168.20.0/24
   └── VLAN 99 — Management — 192.168.99.0/24
```

**pfSense Firewall** sits at the perimeter of the physical lab — all internet-bound traffic from GNS3 virtual devices, Ubuntu Server, and DC01 passes through pfSense for NAT, inspection, and routing.

---

## 🛠️ Technologies & Protocols

| Category | Technology |
|---|---|
| Routing Protocol | OSPF Area 0 (3 routers, unique Router IDs) |
| Switching | 802.1Q Trunking, VLANs 10/20/99/100, STP PortFast |
| Inter-VLAN Routing | Router-on-a-stick (subinterfaces on BR1) |
| IP Services | DHCP Server (2 pools), NAT/PAT Overload |
| Security | Named Extended ACLs, DMZ Architecture |
| Perimeter Firewall | pfSense CE 2.8.1 (WAN/LAN/OPT1, Hybrid NAT) |
| Simulation Platform | GNS3 2.2+, Cisco C7200 IOS, IOU L2 Switch |
| VM Integration | Ubuntu Server 24.04, Windows Server 2022 DC01 |

---

## 📡 IP Addressing Scheme

| Network | Subnet | Purpose |
|---|---|---|
| VLAN 10 (HR) | 192.168.10.0/24 | HR Department workstations |
| VLAN 20 (Sales) | 192.168.20.0/24 | Sales Department workstations |
| VLAN 99 (MGMT) | 192.168.99.0/24 | Management / switch access |
| VLAN 100 (DMZ) | 192.168.100.0/24 | Public-facing server |
| HQ ↔ BR2 WAN | 10.0.0.0/30 | WAN link |
| BR2 ↔ BR1 WAN | 10.0.12.0/30 | WAN link |
| HQ ↔ Internet | 203.0.113.0/30 | Simulated ISP link |

---

## ⚙️ Key Configurations

### OSPF Dynamic Routing
- **3 routers** in Area 0: BR1 (1.1.1.1), BR2 (2.2.2.2), HQ (3.3.3.3)
- Passive interfaces on all LAN-facing subinterfaces
- `default-information originate` at HQ propagates default route to all routers

### VLAN Segmentation & Trunking
- VLANs 10, 20, 99 on LANSW — 802.1Q trunk to BR1
- VLAN 100 (DMZ) on DMZSW — 802.1Q trunk to BR2
- `switchport nonegotiate` to disable DTP on all trunk ports

### NAT/PAT
- BR2 performs PAT overload: VLAN 10 + VLAN 20 + DMZ → single public IP
- ACL `NAT_INSIDE` explicitly permits all internal subnets

### Named ACLs
- `BLOCK_VLAN10_TO_DMZ` — denies HR department (VLAN 10) from accessing DMZ server (applied inbound on BR1 fa0/1.10)
- `DMZ_PROTECT` — permits only HTTP (80), HTTPS (443), ICMP to DMZ; denies all other inbound traffic (applied inbound on BR2 fa2/0.100)

### pfSense Perimeter Firewall
- **WAN** (em0): 192.168.58.140 — VMware NAT, internet-facing
- **LAN** (em2): 10.10.10.2 — lab network gateway (VMnet4)
- **OPT1** (em1): 192.168.10.254 — management GUI (VMnet1)
- Hybrid Outbound NAT covering all internal subnets
- Static routes to GNS3 topology via HQ router (203.0.113.1)
- HTTP port forward: WAN:80 → Ubuntu Server (192.168.100.10:80)

---

## ✅ Testing Results

| Test | From | To | Result |
|---|---|---|---|
| Gateway connectivity | PC1 | 192.168.10.1 | ✅ Pass |
| Inter-VLAN routing | PC1 | PC3 (VLAN 20) | ✅ Pass |
| DHCP lease assignment | PC1/PC3 | BR1 DHCP pool | ✅ Pass |
| OSPF neighbor adjacency | BR1 | BR2 | ✅ FULL state |
| NAT/PAT to internet | PC1 | 203.0.113.1 (HQ) | ✅ Pass |
| ACL block VLAN10 → DMZ | PC1 | SERVER (DMZ) | ✅ Blocked |
| ACL allow VLAN20 → DMZ | PC3 | SERVER (DMZ) | ✅ Pass |
| DMZ isolation | Internet | Internal LAN | ✅ Blocked |
| Trunk functionality | LANSW | BR1 | ✅ VLANs passing |

---

## 📁 Project Files

```
enterprise-branch-network/
├── ENTERPRISE_BRANCH_NETWORK.gns3       # GNS3 project file (import to run)
├── configs/
│   ├── BR1_startup-config.txt
│   ├── BR2_startup-config.txt
│   ├── HQ_startup-config.txt
│   ├── LANSW_startup-config.txt
│   └── DMZSW_startup-config.txt
├── docs/
│   └── Enterprise_Branch_Network_Documentation.pdf
└── README.md
```

---

## 🚀 How to Run This Lab

1. Install **GNS3 2.2+** with VMware integration
2. Import Cisco C7200 IOS image and IOU L2 switch image
3. Open `ENTERPRISE_BRANCH_NETWORK.gns3` in GNS3
4. Start all devices — apply startup configs from `/configs/`
5. Verify with `show ip ospf neighbor`, `show ip route`, `show ip nat translations`

---

## 📚 CCNA Topics Demonstrated

`VLANs` `802.1Q Trunking` `Inter-VLAN Routing` `OSPF Area 0` `DHCP Server` `NAT/PAT` `Extended ACLs` `DMZ Architecture` `Subnetting/VLSM` `Static Routing` `Passive Interfaces` `STP PortFast` `Management SVI` `Router IDs`

---

## 👨‍💻 Author

**Vasanth Kumar**
BCA — Vivekananda Institute of Management, Bengaluru, Karnataka
📧 vasanthkumarvk2855@gmail.com
🔗 [LinkedIn](https://linkedin.com/in/vasanth-kumar-vk55) | [GitHub](https://github.com/vasanth-kumar-vk)

---

*This project is part of a hands-on IT infrastructure portfolio built to demonstrate entry-level Network Administrator skills.*
