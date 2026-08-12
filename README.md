# High-Availability-Campus-Network

A reference design and configuration repository for building a highly available campus network. This project contains topology diagrams, configuration examples, automation playbooks, and validation tests to help deploy resilient Layer 2/3 campus networks with redundant distribution/core, high-availability services, and monitoring.

## Goals

- Provide repeatable, well-documented configuration patterns for campus aggregation and core redundancy
- Demonstrate device-level high-availability (VRRP/HSRP), link redundancy (LACP, MLAG), and routing resilience (OSPF/BGP)
- Include automation examples (Ansible) to provision and validate configurations
- Offer testing and troubleshooting guidance for common failure scenarios

## Repository layout

- docs/ — design notes, diagrams, and topology drawings
- configs/ — example device configuration templates (IOS, NX-OS, Junos)
- ansible/ — playbooks and inventory for automated deployment and validation
- tests/ — scripts and test cases for reachability, failover, and performance checks
- lab/ — sample topology definitions for network emulators (EVE-NG, GNS3)

(If any of these directories are missing in this repository, they are intended locations for future content.)

## Key concepts

- Control-plane redundancy: using OSPF or BGP with graceful restart and proper timers to speed convergence
- Device HA: VRRP or HSRP for gateway redundancy; use object tracking for active/standby failover
- Link redundancy: LACP across uplinks, MLAG (or vPC) where supported to provide active-active layer-2
- Loop prevention & segmentation: implement VLAN design with SVI and private VLANs where required
- Automation & testing: codify configurations as templates and verify idempotency with playbooks

## Example quickstart (lab)

1. Clone the repository:

   git clone https://github.com/kamleshsande85/High-Availability-Campus-Network.git
   cd High-Availability-Campus-Network

2. Review the lab topology in lab/topology.yml (or docs/topology.drawio) and start your emulator (EVE-NG/GNS3).

3. Install Ansible and required collections:

   pip install -r ansible/requirements.txt

4. Update `ansible/inventory` with lab device IPs and credentials.

5. Run the playbook to deploy base configurations:

   ansible-playbook -i ansible/inventory ansible/site.yml

6. Validate basic connectivity and HA state with provided tests:

   ./tests/check_reachability.sh
   ./tests/check_vrrp_state.sh

## Example VRRP snippet (concept)

Configure VRRP on distribution switches for a VLAN SVI (replace variables accordingly):

interface Vlan100
  ip address 10.0.100.2/24
  vrrp 100 ip 10.0.100.1
  vrrp 100 priority 110
  vrrp 100 preempt

On the peer, set priority lower (e.g., 100) so the higher-priority switch becomes active.

## Monitoring & Observability

- Export telemetry via SNMP/NETCONF/gNMI or stream to a time-series platform (Prometheus/Grafana)
- Use syslog aggregation and flow telemetry (NetFlow/sFlow) to analyze traffic patterns
- Monitor HA state (VRRP/HSRP), interface counters, and routing adjacency for early warnings

## Testing & Validation

- Simulate link and device failures in the lab and verify traffic continuity and convergence time
- Run ECMP and path validation tests for routing symmetry and blackhole detection
- Validate control-plane and management-plane isolation so that user traffic and protocols do not interfere

## Troubleshooting tips

- When HA failovers behave unexpectedly: check object-tracking, interface priorities, and preempt settings
- For slow routing convergence: tune OSPF/BGP timers and verify no excessive SPF/LSA churn
- For L2 loops: verify MLAG/vPC consistency, VLAN allowed lists, and STP configuration

## Contributing

Contributions are welcome. Please:

1. Open an issue to discuss major changes or proposals.
2. Fork the repository, make changes in a feature branch, and submit a pull request with a clear description and testing notes.

## License

This repository is provided under the MIT License unless otherwise specified in individual files.

---

If you'd like, I can also:

- Add a topology diagram file (drawio) and a sample Ansible inventory and playbook to bootstrap a lab.
- Create example configuration files for IOS/NX-OS/Junos for the most common topology (2-core, 2-distribution, access layer).

Tell me which additions you'd like and I'll add them next.