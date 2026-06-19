# Lab 2: ACLs & DMZ Configuration

**TechNova GmbH — Network Engineering Lab Series**

---

## Scenario

TechNova GmbH requires a public-facing web server to host company resources accessible from the internet. To protect internal systems, the IT team has been tasked with implementing a **DMZ (Demilitarised Zone)** to isolate the web server, and configuring **Extended Named ACLs** to enforce strict traffic policies across all network segments.

---

## Objectives

- Create a DMZ network segment to host a public-facing web server
- Configure Extended Named ACLs to control inter-segment traffic
- Block DMZ-initiated traffic from reaching internal VLANs
- Allow only HTTP/HTTPS traffic from outside into the DMZ
- Permit internal users to access the DMZ web server
- Block all outside traffic from reaching internal VLANs

---

## Tools Used

- Cisco Packet Tracer
- Cisco IOS CLI (Router 2911, Switch 3650-24PS)

---

## Network Topology

> *Insert full topology screenshot here*

### Devices

| Device | Model | Role |
|---|---|---|
| Router-GW | Cisco 2911 | Gateway router, ACL enforcement |
| Core-SW | Cisco 3650-24PS | Layer 3 switch, inter-VLAN routing |
| DMZ-Server | Server-PT | Public-facing web server |
| PC-IT | PC-PT | IT Department client |
| PC-HR | PC-PT | HR Department client |
| PC-Sales | PC-PT | Sales Department client |
| Outside-PC | PC-PT | Simulated internet user |

### Network Segments

| Segment | Network | Purpose |
|---|---|---|
| VLAN 10 – IT | 192.168.10.0/24 | IT Department |
| VLAN 20 – HR | 192.168.20.0/24 | HR Department |
| VLAN 30 – Sales | 192.168.30.0/24 | Sales Department |
| DMZ | 192.168.100.0/24 | Public-facing web server |
| WAN (Outside) | 10.0.0.0/30 | Simulated internet link |
| Uplink | 172.16.0.0/30 | Core-SW to Router-GW |

### IP Address Assignments

| Device | Interface | IP Address | Gateway |
|---|---|---|---|
| Core-SW | VLAN 10 SVI | 192.168.10.1/24 | — |
| Core-SW | VLAN 20 SVI | 192.168.20.1/24 | — |
| Core-SW | VLAN 30 SVI | 192.168.30.1/24 | — |
| Core-SW | Gi1/0/1 (uplink) | 172.16.0.2/30 | 172.16.0.1 |
| Router-GW | Gi0/0 (to Core-SW) | 172.16.0.1/30 | — |
| Router-GW | Gi0/1 (DMZ) | 192.168.100.1/24 | — |
| Router-GW | Gi0/2 (Outside) | 10.0.0.1/30 | — |
| DMZ-Server | NIC | 192.168.100.10/24 | 192.168.100.1 |
| Outside-PC | NIC | 10.0.0.2/30 | 10.0.0.1 |
| PC-IT | NIC | 192.168.10.10/24 | 192.168.10.1 |
| PC-HR | NIC | 192.168.20.10/24 | 192.168.20.1 |
| PC-Sales | NIC | 192.168.30.10/24 | 192.168.30.1 |

---

## ACL Policy Design

| Source | Destination | Protocol | Action |
|---|---|---|---|
| Internal VLANs | DMZ | Any | Permit |
| DMZ | Internal VLANs | Any | Deny |
| Outside | DMZ | TCP 80/443 | Permit |
| Outside | Internal VLANs | Any | Deny |
| Internal VLANs | Each other | Any | Permit |

---

## Configuration

### Core-SW

```
enable
configure terminal
hostname Core-SW
no ip domain-lookup

vlan 10
 name IT
vlan 20
 name HR
vlan 30
 name Sales
exit

interface GigabitEthernet1/0/2
 switchport mode access
 switchport access vlan 10
 no shutdown
 exit

interface GigabitEthernet1/0/3
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit

interface GigabitEthernet1/0/4
 switchport mode access
 switchport access vlan 30
 no shutdown
 exit

ip routing

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit

interface vlan 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
 exit

interface GigabitEthernet1/0/1
 no switchport
 ip address 172.16.0.2 255.255.255.252
 no shutdown
 exit

ip route 0.0.0.0 0.0.0.0 172.16.0.1

end
copy running-config startup-config
```

---

### Router-GW

```
enable
configure terminal
hostname Router-GW
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 172.16.0.1 255.255.255.252
 no shutdown
 exit

interface GigabitEthernet0/1
 ip address 192.168.100.1 255.255.255.0
 no shutdown
 exit

interface GigabitEthernet0/2
 ip address 10.0.0.1 255.255.255.252
 no shutdown
 exit

ip route 192.168.10.0 255.255.255.0 172.16.0.2
ip route 192.168.20.0 255.255.255.0 172.16.0.2
ip route 192.168.30.0 255.255.255.0 172.16.0.2
```

---

### ACL Configuration

```
ip access-list extended DMZ_IN
 permit icmp 192.168.100.0 0.0.0.255 any echo-reply
 permit tcp any any established
 deny ip 192.168.100.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.100.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.100.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip any any
 exit

ip access-list extended OUTSIDE_IN
 permit tcp any 192.168.100.0 0.0.0.255 eq 80
 permit tcp any 192.168.100.0 0.0.0.255 eq 443
 deny ip any any
 exit

interface GigabitEthernet0/1
 ip access-group DMZ_IN in
 exit

interface GigabitEthernet0/2
 ip access-group OUTSIDE_IN in
 exit

end
copy running-config startup-config
```

---

## Verification

### show access-lists


---

## Test Results

| Test | Source | Destination | Protocol | Expected | Result |
|---|---|---|---|---|---|
| Internal to DMZ | PC-IT | DMZ-Server | ICMP | Success | ✅ |
| DMZ to Internal | DMZ-Server | PC-IT | ICMP | Blocked | ✅ |
| Outside to DMZ (ping) | Outside-PC | DMZ-Server | ICMP | Blocked | ✅ |
| Outside to Internal | Outside-PC | PC-IT | ICMP | Blocked | ✅ |
| Outside to DMZ (HTTP) | Outside-PC | DMZ-Server | TCP 80 | Success | ✅ |

### Test Screenshots

> *Insert ping screenshots here*

> *Insert web browser screenshot (http://192.168.100.10) here*

---

## Key Concepts Demonstrated

- **DMZ architecture** isolates public-facing servers from internal network segments
- **Extended Named ACLs** filter traffic by source, destination, and protocol
- **ACL placement** matters — DMZ_IN applied inbound on Gi0/1 blocks DMZ-originated traffic before it enters the router
- **OUTSIDE_IN** applied inbound on Gi0/2 ensures only HTTP/HTTPS reaches the DMZ
- **Stateful-like behaviour** achieved by permitting established TCP and ICMP echo-reply before deny rules
- **Implicit deny** at the end of every ACL blocks all unmatched traffic

---

## Lessons Learned

- ACL rule order is critical — permits for return traffic must come before deny statements
- The `permit icmp any any echo-reply` and `permit tcp any any established` rules allow return traffic without opening inbound access from the DMZ
- Using named ACLs (instead of numbered) makes configuration easier to read and maintain
- Always verify routing works before applying ACLs — troubleshooting is much easier without ACLs in the way

---

*TechNova GmbH Lab Series | Packet Tracer Portfolio | Lab 2 of 2*
