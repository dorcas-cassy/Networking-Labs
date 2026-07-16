# DHCP Server Configuration with Inter-VLAN Routing

## TechNova GmbH — Network Infrastructure Lab

### Overview
This lab simulates a segmented corporate network for TechNova GmbH, where three departments (Sales, IT, and HR) are isolated into separate VLANs on a single switch. A router performs inter-VLAN routing using Router-on-a-Stick (802.1Q sub-interfaces), and a centralized DHCP server automatically assigns IP addresses to each department's subnet via DHCP relay (IP helper).

### Objective
To design and implement a multi-VLAN network where:
- Devices in different departments are logically separated for security and traffic management
- Hosts automatically receive IP configuration from a central DHCP server, regardless of which VLAN they belong to
- Routing between VLANs is enabled through router sub-interfaces

### Topology
| Device | Role |
|---|---|
| Router0 (2911) | Inter-VLAN routing (Router-on-a-Stick) |
| Switch0 (2960) | VLAN segmentation, trunking |
| Server0 | DHCP server |
| PC0, PC1, PC2 | End devices — one per VLAN |

### VLAN / Subnet Plan
| VLAN ID | Name | Subnet | Gateway | DHCP Pool Start |
|---|---|---|---|---|
| 10 | Sales | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.10 |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.10 |
| 30 | HR | 192.168.30.0/24 | 192.168.30.1 | 192.168.30.10 |

DHCP server static IP: `192.168.10.2` (VLAN 10), gateway `192.168.10.1`.

### Key Configuration Steps
1. Created VLANs 10, 20, and 30 on the switch and named them Sales, IT, and HR.
2. Assigned switch access ports to their respective VLANs.
3. Configured the switch port facing the router as a trunk (`switchport mode trunk`, allowed VLAN all).
4. Configured router sub-interfaces (`Gi0/0.10`, `Gi0/0.20`, `Gi0/0.30`) with 802.1Q encapsulation and gateway IPs for each VLAN.
5. Enabled the physical `Gi0/0` interface with `no shutdown`.
6. Configured the DHCP server with a static IP and created a DHCP pool per VLAN (gateway, start address, subnet mask, max users).
7. Configured `ip helper-address 192.168.10.2` on the VLAN 20 and 30 sub-interfaces so DHCP requests from those VLANs are relayed to the server (VLAN 10 reaches it directly, no relay needed there).
8. Set each PC's IP configuration to DHCP and verified assigned addresses with `ipconfig`.
9. Tested inter-VLAN connectivity with `ping` between hosts on different VLANs.

### Verification
- `ipconfig` on each PC confirms an IP address, subnet mask, and default gateway matching its VLAN's pool.
- Successful `ping` between PCs on different VLANs (e.g., VLAN 10 → VLAN 20) confirms inter-VLAN routing is working.
- `show vlan brief` on the switch confirms correct port-to-VLAN assignment.
- `show ip interface brief` on the router confirms all sub-interfaces are up/up.

### Skills Demonstrated
- VLAN creation and port assignment
- 802.1Q trunking
- Router-on-a-Stick inter-VLAN routing
- DHCP server pool configuration
- DHCP relay (IP helper-address)
- Network verification and troubleshooting (ipconfig, ping, show commands)

### Files in this Repository
- `DHCP_Inter-VLAN_Routing.pkt` — Packet Tracer project file
- `README.md` — this documentation
- `/screenshots` — supporting evidence (see below)

---
*Part of the TechNova GmbH networking lab series.*
