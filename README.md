# High-Availability Campus Network

This repository contains a reference design, lab topology, configurations and validation artifacts for a High-Availability Campus Network built with HSRP, EtherChannel (LACP), and RPVST+.

## Project overview

This project demonstrates a highly available campus network using:

- HSRP for gateway redundancy (Active/Standby)
- EtherChannel (LACP) for link aggregation and higher bandwidth
- RPVST+ and PortFast/BPDU Guard for fast STP convergence and loop prevention
- Inter-VLAN routing with SVIs and DHCP for end hosts

The repo contains topology diagrams, GNS3/EVE-NG-compatible files, configuration snippets and verification screenshots.

## Goals

- Provide repeatable configuration patterns for campus aggregation and core redundancy
- Demonstrate device-level high availability (HSRP), link redundancy (LACP) and routing resilience (OSPF)
- Include lab-ready configs and verification steps to validate failover and convergence

## Quickstart (lab)

1. Clone the repo and enter the directory:

   git clone https://github.com/kamleshsande85/High-Availability-Campus-Network.git
   cd High-Availability-Campus-Network

2. Open the topology in GNS3 or your preferred emulator (see `GNS File/` and `Screenshots/Topology/`).

3. If using Ansible (optional):

   pip install -r ansible/requirements.txt
   update `ansible/inventory` with device IPs and credentials
   ansible-playbook -i ansible/inventory ansible/site.yml

4. Run validation scripts:

   ./tests/check_reachability.sh
   ./tests/check_vrrp_state.sh

## Key highlights (from Project highlight availibility)

- HSRP configured for gateway redundancy with object tracking and preempt to ensure predictable active/standby behavior
- EtherChannel (LACP) used for inter-core aggregation for redundancy and more bandwidth
- RPVST+ for fast per-VLAN convergence; PortFast + BPDU Guard enabled on access ports
- Sub-second failover validated in lab topology

## IP addressing (summary)

| VLAN | Network | Mask | Virtual Gateway (HSRP) | CORE-SW1 | CORE-SW2 |
|------|---------|------|------------------------|----------|----------|
| VLAN 10 (Mgmt) | 172.16.4.0/25 | 255.255.255.128 | 172.16.4.1 | 172.16.4.2 | 172.16.4.3 |
| VLAN 20 (HR)   | 172.16.4.128/25 | 255.255.255.128 | 172.16.4.129 | 172.16.4.130 | 172.16.4.131 |
| VLAN 30 (Finance) | 172.16.5.0/25 | 255.255.255.128 | 172.16.5.1 | 172.16.5.2 | 172.16.5.3 |
| VLAN 40 (Engg) | 172.16.5.128/25 | 255.255.255.128 | 172.16.5.129 | 172.16.5.130 | 172.16.5.131 |

HSRP priority plan: CORE-SW1 active for VLANs 10 & 20 (higher priority), CORE-SW2 active for VLANs 30 & 40.

## Example configuration snippets

Access switch trunk (example):

```
interface gi3/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
```

Core switch EtherChannel and HSRP (example):

```
! EtherChannel (LACP) to peer
interface range gi0/0 - 1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 channel-group 1 mode active
!
interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
!
interface Vlan10
 ip address 172.16.4.2 255.255.255.128
 standby 10 ip 172.16.4.1
 standby 10 priority 150
 standby 10 preempt
```

## Verification & common commands

- Access switches: `show vlan brief`, `show interface trunk`, `show cdp neighbors`, `show spanning-tree brief`
- Core switches: `show standby brief`, `show ip ospf neighbor`, `show etherchannel summary`, `show spanning-tree root`
- End hosts: `ping <gateway>` (HSRP virtual IP), DHCP request and inter-VLAN ping tests

Screenshots and step-by-step verification are available under `Screenshots/`.

## Lessons learned

- HSRP requires consistent VLAN configuration on peers for reliable operation
- EtherChannel in GNS3/EVE-NG behaves more predictably with LACP (active/passive)
- PortFast + BPDU Guard reduces STP convergence delays on access ports
- Native VLAN mismatches cause CDP/trunking issues — always verify `show interface trunk`

## Future improvements

- Add MLS QoS for traffic prioritization
- Implement DHCP Snooping and other security features
- Extend topology and configurations to support IPv6 and HSRPv6

## Repo structure

High-Availability-Campus-Network/
- README.md
- Project highlight availibility .md
- GNS File/
- Screenshots/
- (Optional) ansible/, configs/, docs/, tests/ — see repository for available directories

## License

MIT License — see LICENSE file if present.

---

If you want, I can:
- Add a short topology diagram (drawio) into docs/ or lab/
- Add sample Ansible inventory and a minimal site.yml to bootstrap the lab
- Add example IOS/NX-OS config files into configs/

Tell me which addition you'd like next and I'll add it.