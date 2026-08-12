# 🏗️ High-Availability Campus Network

[![GNS3](https://img.shields.io/badge/GNS3-2.2-green)](https://www.gns3.com/)
[![Cisco IOS](https://img.shields.io/badge/Cisco-IOSv15-blue)](https://www.cisco.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A complete **High-Availability Campus Network** design using **HSRP**, **EtherChannel (LACP)**, and **RPVST+** to eliminate single points of failure at the Distribution and Access layers.

---

## 📌 Project Overview

This project demonstrates a **production-ready campus network** with:
- ✅ **HSRP** — Gateway redundancy with Active/Standby failover (sub-second)
- ✅ **EtherChannel (LACP)** — Link aggregation for bandwidth and redundancy
- ✅ **RPVST+** — Fast STP convergence with PortFast + BPDU Guard
- ✅ **Inter-VLAN Routing** — SVIs for routing between VLANs
- ✅ **DHCP** — Automatic IP assignment for end devices

---

## 🎯 Objectives

| Objective | Implementation |
|-----------|----------------|
| **Gateway Redundancy** | HSRP with Active/Standby and Preempt |
| **Link Redundancy** | EtherChannel (LACP) between Core Switches |
| **Fast Convergence** | RPVST+ with PortFast + BPDU Guard |
| **Scalability** | 4 VLANs (Management, HR, Finance, Engineering) |
| **Resilience** | OSPF for dynamic routing to Core Routers |

---

## 🏗️ Network Topology

```
                           ┌───────────────────────────────┐
                           │     CORE Routers              │
                           │  CORE-R1      CORE-R2         │
                           └───────┬──────────┬─────────────┘
                                   │          │
                    ┌──────────────┴──────────┴──────────────┐
                    │                                         │
              Gi0/2 │                                    Gi0/2 │
            ┌───────▼──────────┐                  ┌───────────▼────┐
            │   CORE-SW1       │ ◄─ EtherChannel ► │   CORE-SW2     │
            │   (Active)       │     (LACP)        │   (Standby)    │
            │  HSRP Priority   │                   │  HSRP Priority │
            │  V10: 150        │     Gi0/0-Gi0/1   │  V10: 100      │
            │  V20: 150        │◄─────────────────►│  V20: 100      │
            │  V30: 100        │                   │  V30: 150      │
            │  V40: 100        │                   │  V40: 150      │
            └────┬──────────────┘                  └────┬───────────┘
                 │ Gi0/3 (Trunk)                        │ Gi0/3 (Trunk)
                 │                                      │
         ┌───────┴──────────────────────────────────────┴────────┐
         │                                                        │
    Gi3/2│                                                   Gi3/2│
    ┌────▼─────────┐     ┌──────────────┐     ┌──────────────┐   │
    │   ACC-SW1    │     │   ACC-SW2    │     │   ACC-SW3    │   │
    │  (Access)    │     │  (Access)    │     │  (Access)    │   │
    └─────┬────────┘     └──────┬───────┘     └──────┬───────┘   │
          │                     │                    │            │
    ┌─────┴──────┐        ┌─────┴──────┐       ┌─────┴──────┐    │
    │   PC1      │        │   PC2      │       │   PC3      │    │
    │ VLAN 10    │        │ VLAN 20    │       │ VLAN 30    │    │
    └────────────┘        └────────────┘       └────────────┘    │
                                                                   │
                                        ┌──────────────────┐      │
                                        │   DHCP Server    │      │
                                        │                  │      │
                                        └──────────────────┘      │
```

**[📷 View Detailed Topology Diagram](./Screenshots/Topology/Network-Topology-Main.png)**

### Physical Connections

| Link | Source Device | Source Port | Destination Device | Destination Port | Type |
|------|---------------|-------------|-------------------|------------------|------|
| **EtherChannel (LACP)** | CORE-SW1 | Gi0/0, Gi0/1 | CORE-SW2 | Gi0/0, Gi0/1 | Trunk |
| **Trunk 1** | CORE-SW1 | Gi0/3 | ACC-SW1 | Gi3/2 | Trunk |
| **Trunk 2** | CORE-SW2 | Gi0/3 | ACC-SW2 | Gi3/2 | Trunk |
| **Access 1** | ACC-SW1 | Gi1/1, Gi1/2 | PC1, PC2 | eth0 | Access |
| **Access 2** | ACC-SW2 | Gi1/1, Gi1/2 | PC3, PC4 | eth0 | Access |

---

## 📊 IP Addressing Scheme

| VLAN | Network | Subnet Mask | Usable IPs | Virtual Gateway (HSRP) | CORE-SW1 | CORE-SW2 |
|------|---------|-------------|------------|------------------------|----------|----------|
| **VLAN 10 (Mgmt)** | 172.16.4.0/25 | 255.255.255.128 | 126 | 172.16.4.1 | 172.16.4.2 | 172.16.4.3 |
| **VLAN 20 (HR)** | 172.16.4.128/25 | 255.255.255.128 | 126 | 172.16.4.129 | 172.16.4.130 | 172.16.4.131 |
| **VLAN 30 (Finance)** | 172.16.5.0/25 | 255.255.255.128 | 126 | 172.16.5.1 | 172.16.5.2 | 172.16.5.3 |
| **VLAN 40 (Engg)** | 172.16.5.128/25 | 255.255.255.128 | 126 | 172.16.5.129 | 172.16.5.130 | 172.16.5.131 |

### HSRP Priority Plan

| Switch | VLAN 10 | VLAN 20 | VLAN 30 | VLAN 40 |
|--------|---------|---------|---------|---------|
| **CORE-SW1** | ✅ Active (150) | ✅ Active (150) | ⏳ Standby (100) | ⏳ Standby (100) |
| **CORE-SW2** | ⏳ Standby (100) | ⏳ Standby (100) | ✅ Active (150) | ✅ Active (150) |

---

## ⚙️ Configuration Snippets

### Access Switch Trunk (ACC-SW1)

```ios
interface gi3/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
```

### Core Switch EtherChannel (CORE-SW1)

```ios
! EtherChannel (LACP) to CORE-SW2
interface range gi0/0 - 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 10,20,30,40
 channel-group 1 mode active
 no shutdown

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
```

### Core Switch HSRP (CORE-SW1 — Active for VLAN 10)

```ios
interface vlan 10
 ip address 172.16.4.2 255.255.255.128
 standby 10 ip 172.16.4.1
 standby 10 priority 150
 standby 10 preempt
 no shutdown
```

**[📄 Full Configs](./configs/)**

---

## ✅ Verification & Validation

### 1. Access Switches

| Command | Purpose |
|---------|---------|
| `show vlan brief` | Verify VLAN database and port assignments |
| `show interface trunk` | Verify trunking status and allowed VLANs |
| `show cdp neighbors` | Verify CDP neighbors connected to Core Switches |
| `show spanning-tree brief` | Verify STP status and port states |

**[📷 Screenshots](./Screenshots/ACC-SW1/)**

### 2. Core Switches

| Command | Purpose |
|---------|---------|
| `show standby brief` | Verify HSRP Active/Standby states and Virtual IPs |
| `show ip ospf neighbor` | Verify OSPF neighbor adjacency (should be FULL) |
| `show etherchannel summary` | Verify EtherChannel status |
| `show spanning-tree root` | Verify STP root bridge information |

**[📷 Screenshots](./Screenshots/CORE-SW1/)**

### 3. End Hosts

| Command | Purpose |
|---------|---------|
| `ip dhcp` | Request IP from DHCP server |
| `ping 172.16.4.1` | Ping HSRP Virtual Gateway |
| `ping 172.16.5.11` | Ping host in different VLAN |
| `trace 1.1.1.1` | Traceroute to Core Router |

**[📷 Screenshots](./Screenshots/PC2/)**

---

## 🔬 Live Failover Test

### Step 1: Start Continuous Ping
```cmd
ping -t 172.16.4.1
```

### Step 2: Shut Down Active SVI on CORE-SW1
```ios
CORE-SW1(config)# interface vlan 10
CORE-SW1(config-if)# shutdown
```

### Step 3: Verify Standby Becomes Active
```ios
CORE-SW2# show standby brief
```
✅ **CORE-SW2 transitions to Active — Zero packet loss!**

### Step 4: Restore Interface (Preempt Verification)
```ios
CORE-SW1(config)# interface vlan 10
CORE-SW1(config-if)# no shutdown
```
✅ **CORE-SW1 reclaims Active role (Preempt)**

**[📷 Failover Screenshots](./Screenshots/CORE-SW1/)**

---

## 📚 Lessons Learned

| Challenge | Solution |
|-----------|----------|
| **Native VLAN mismatch** | Always verify `show interface trunk` — mismatch causes CDP errors |
| **EtherChannel in GNS3** | LACP (active/passive) works more reliably than manual mode |
| **HSRP Preempt** | Critical to ensure Active switch reclaims role after restoration |
| **PortFast + BPDU Guard** | Essential to prevent STP loops on access ports |

---

## 🚀 Future Improvements

- [ ] Add **MLS QoS** for traffic prioritization
- [ ] Implement **DHCP Snooping** for security
- [ ] Extend to **IPv6** with HSRPv6
- [ ] Add **Ansible Automation** for deployment
- [ ] Integrate **Syslog & SNMP** for monitoring

---

## 📂 Repository Structure

```
High-Availability-Campus-Network/
├── README.md
├── configs/
│   ├── CORE-SW1.txt
│   ├── CORE-SW2.txt
│   └── ACC-SW1.txt
├── GNS-File/
│   └── Project2.gns3
├── Screenshots/
│   ├── ACC-SW1/
│   ├── ACC-SW2/
│   ├── CORE-SW1/
│   ├── CORE-SW2/
│   ├── PC2/
│   ├── PC5/
│   └── Topology/
│       ├── Network-Topology-Main.png
│       ├── Network_Topology3.png
│       └── Network_topology2.png
├── tests/
│   └── check_reachability.sh
└── LICENSE
```

---

## 🛠️ Quickstart

### Clone the Repository
```bash
git clone https://github.com/kamleshsande85/High-Availability-Campus-Network.git
cd High-Availability-Campus-Network
```

### Open in GNS3
1. Open GNS3 → File → Open Project
2. Select `GNS-File/Project2.gns3`
3. Start all devices

### Verify Connectivity
```bash
./tests/check_reachability.sh
```

---

## 📄 License

MIT License — see [LICENSE](./LICENSE) file for details.

---

## 🤝 Connect

**Kamlesh Kumar** — [LinkedIn](https://www.linkedin.com/in/kamlesh-kumar-9921282bb/) | [GitHub](https://github.com/kamleshsande85)

---

⭐ **If you find this useful, please consider giving it a star!** ⭐
