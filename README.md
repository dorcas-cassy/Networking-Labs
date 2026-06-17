# TechNova GmbH Office Network — Cisco Packet Tracer Lab

## Project Overview
This project demonstrates the design and implementation of a small business network for a fictional company, TechNova GmbH, using Cisco Packet Tracer.

The network was configured to support multiple departments through VLAN segmentation, DHCP services, inter-VLAN routing, and secure network communication. The project simulates a real-world enterprise network environment and showcases practical networking skills relevant to IT Support, Network Administration, and Cloud Infrastructure roles.

## Objectives

Design a multi-department office network
Configure VLANs for network segmentation
Implement inter-VLAN routing
Configure DHCP services for automatic IP assignment
Verify end-to-end connectivity between departments
Apply switch trunking between network devices

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

## IP Addressing Scheme
Device	       IP Address
IT Gateway	   192.168.10.1
HR Gateway	   192.168.20.1
Management     VLAN	192.168.99.1
IT PCs	       DHCP Assigned
HR PCs         DHCP Assigned

## Technologies Used

Cisco Packet Tracer
VLAN Configuration
Inter-VLAN Routing
DHCP
Trunk Links
Layer 2 Switching
Layer 3 Routing
Network Troubleshooting

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

## Lessons Learned

This project provided hands-on experience with enterprise networking concepts including VLAN implementation, DHCP deployment, routing configuration, network troubleshooting, and documentation. It strengthened practical skills in Cisco networking technologies and reinforced best practices for designing scalable office networks.

## Tools Used
- Cisco Packet Tracer 8.x
