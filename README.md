# Cisco Packet Tracer networking labs demonstrating VLANs, routing, DHCP, and network security configurations

# TechNova GmbH Office Network — Cisco Packet Tracer Lab

## Project Overview
Designed and configured a multi-VLAN office network for a fictional 
company (TechNova GmbH) simulating a real-world small business 
network environment.

## Network Topology
- 1x Cisco ISR4331 Router
- 1x Cisco 3650-24PS Core Switch (Multilayer)
- 2x Cisco 2960-24TT Access Switches
- 1x DHCP Server
- 4x End User PCs

## VLANs Configured
| VLAN | Name | Subnet |
|------|------|--------|
| 10 | IT Department | 192.168.10.0/24 |
| 20 | HR Department | 192.168.20.0/24 |
| 99 | Management | 192.168.99.0/24 |

## Key Concepts Demonstrated
- VLAN creation and segmentation
- Router-on-a-stick inter-VLAN routing
- DHCP server with multiple pools
- DHCP relay (ip helper-address)
- Trunk and access port configuration
- End-to-end connectivity testing

## Test Results
- IT-PC1 → Gateway: 0% packet loss
- IT-PC1 → HR-PC1 (cross-VLAN): Successful

## Tools Used
- Cisco Packet Tracer 8.x
