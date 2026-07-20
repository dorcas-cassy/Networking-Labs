# ASA Firewall with 3-Zone Segmentation

## TechNova GmbH — Lab 5

## Topology

| Device | Role |
|---|---|
| ASA1 (5505) | Firewall — Inside/DMZ/Outside segmentation |
| Switch0 (2960) | Inside LAN switch |
| Switch1 (2960) | DMZ switch |
| Switch2 (2960) | Outside network switch |
| PC0–PC4 | Inside LAN hosts |
| WEB, DNS, DHCP, EMAIL Servers | DMZ hosts |
| PC5, PC6, XYZ Server | Outside hosts |

## Interface / Security Plan

| ASA Port | VLAN | Zone | Security Level | Gateway IP |
|---|---|---|---|---|
| Ethernet0/0 | VLAN 1 | Inside | 100 | 192.168.10.1 |
| Ethernet0/1 | VLAN 2 | DMZ | 50 | 192.168.20.1 |
| Ethernet0/2 | VLAN 3 | Outside | 0 | 192.168.30.1 |

## IP Plan

| Zone | Subnet | Host Range |
|---|---|---|
| Inside | 192.168.10.0/24 | PC0–PC4: .10–.14 |
| DMZ | 192.168.20.0/24 | Web=.10, DNS=.11, DHCP=.12, Email=.13 |
| Outside | 192.168.30.0/24 | PC5=.10, PC6=.11, XYZ=.12 |

Web server exposed to Outside via static NAT: `192.168.30.20` (port 80 only).

## Key Configuration Steps

1. Assigned static IPs and gateways to all 12 hosts.
2. Created VLAN interfaces 1/2/3 on the ASA and assigned `nameif`, security levels, and IPs (ASA 5505 requires VLAN interfaces — physical ports are switchports, not routable interfaces directly).
3. Assigned physical ports to their VLANs with `switchport access vlan X`.
4. Applied `no forward interface vlan 1` on the DMZ VLAN interface (Base license only permits 2 fully-forwarding named interfaces — this restriction also reinforces DMZ isolation from Inside).
5. Configured dynamic NAT (Inside→Outside, DMZ→Outside) and static NAT for the web server (DMZ→Outside, port 80 exposure).
6. Applied an explicit ACL (`OUTSIDE-IN`) permitting inbound ICMP and TCP/80, since Packet Tracer's ASA simulation didn't reliably honor `inspect icmp` from the default policy-map.
7. Verified expected allow/deny behavior in both directions (see Testing below).
8. Saved configuration with `write memory`.

## Testing & Verification

| Test | Direction | Result | Why |
|---|---|---|---|
| Ping gateway | Each zone → own gateway | ✅ Success | Basic connectivity |
| Ping host | Inside → Outside | ✅ Success | High→low security level allowed by default |
| Ping host | Outside → Inside | ❌ Fail (expected) | Low→high blocked by default |
| HTTP access | Outside → DMZ web server (port 80) | ✅ Success | Explicit ACL permits it |
| Ping host | DMZ → Inside | ❌ Fail (expected) | DMZ isolated from Inside by design |

## Skills Demonstrated
- ASA 5505 VLAN-based interface configuration
- Security levels and default traffic policy
- Static and dynamic NAT/PAT
- Access Control Lists (ACLs)
- Firewall policy verification — proving both allow and deny cases

---
*TechNova GmbH — Lab 5: ASA Firewall*
