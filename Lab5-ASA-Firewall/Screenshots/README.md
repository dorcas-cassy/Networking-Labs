Topology

DeviceRoleASA1 (5505)Firewall — Inside/DMZ/Outside segmentationSwitch0 (2960)Inside LAN switchSwitch1 (2960)DMZ switchSwitch2 (2960)Outside network switchPC0–PC4Inside LAN hostsWEB, DNS, DHCP, EMAIL ServersDMZ hostsPC5, PC6, XYZ ServerOutside hosts

Interface / Security Plan

ASA PortVLANZoneSecurity LevelGateway IPEthernet0/0VLAN 1Inside100192.168.10.1Ethernet0/1VLAN 2DMZ50192.168.20.1Ethernet0/2VLAN 3Outside0192.168.30.1

IP Plan

ZoneSubnetHost RangeInside192.168.10.0/24PC0–PC4: .10–.14DMZ192.168.20.0/24Web=.10, DNS=.11, DHCP=.12, Email=.13Outside192.168.30.0/24PC5=.10, PC6=.11, XYZ=.12

Web server exposed to Outside via static NAT: 192.168.30.20 (port 80 only).

Key Configuration Steps


Assigned static IPs and gateways to all 12 hosts.
Created VLAN interfaces 1/2/3 on the ASA and assigned nameif, security levels, and IPs (ASA 5505 requires VLAN interfaces — physical ports are switchports, not routable interfaces directly).
Assigned physical ports to their VLANs with switchport access vlan X.
Applied no forward interface vlan 1 on the DMZ VLAN interface (Base license only permits 2 fully-forwarding named interfaces — this restriction also reinforces DMZ isolation from Inside).
Configured dynamic NAT (Inside→Outside, DMZ→Outside) and static NAT for the web server (DMZ→Outside, port 80 exposure).
Applied an explicit ACL (OUTSIDE-IN) permitting inbound ICMP and TCP/80, since Packet Tracer's ASA simulation didn't reliably honor inspect icmp from the default policy-map.
Verified expected allow/deny behavior in both directions (see Testing below).
Saved configuration with write memory.


Testing & Verification

TestDirectionResultWhyPing gatewayEach zone → own gateway✅ SuccessBasic connectivityPing hostInside → Outside✅ SuccessHigh→low security level allowed by defaultPing hostOutside → Inside❌ Fail (expected)Low→high blocked by defaultHTTP accessOutside → DMZ web server (port 80)✅ SuccessExplicit ACL permits itPing hostDMZ → Inside❌ Fail (expected)DMZ isolated from Inside by design

Skills Demonstrated


ASA 5505 interface and security-level configuration
Static and dynamic NAT/PAT
Access Control Lists on a firewall
Three-zone network segmentation (Inside/DMZ/Outside)
Systematic firewall policy testing (proving both allow AND deny rules)



Troubleshooting Notes (Real Issues Hit During This Build)

1. ASA 5505 interfaces are switchports, not routed interfaces.
Unlike larger ASA models, you cannot run nameif directly on Ethernet0/x. You must create VLAN interfaces (interface vlan 1/2/3) for the Layer 3 side, then assign each physical port to its VLAN with switchport access vlan X.

2. Base license only allows 2 fully-forwarding named interfaces.
Adding a third (DMZ) throws: ERROR: This license does not allow configuring more than 2 interfaces with nameif.... Fixed by adding no forward interface vlan 1 on the DMZ VLAN interface before naming it — this restricts DMZ from forwarding directly to Inside, which conveniently reinforces the DMZ isolation goal rather than fighting against it.

3. Inside → Outside ping initially failed despite correct NAT.
show nat confirmed translate_hits incrementing (proof the request was translating and leaving correctly), but replies never returned. Packet Tracer's ASA simulation does not reliably enforce inspect icmp from the default global_policy the way real ASA hardware does. Fixed with an explicit ACL instead:

access-list OUTSIDE-IN extended permit icmp any any
access-group OUTSIDE-IN in interface outside

This is a good example of adapting to a simulator limitation rather than a real ASA misconfiguration — worth noting for anyone reviewing this project who might try it on real hardware and wonder why the explicit ACL was needed in addition to inspection.

4. Confirmed both "should work" and "should fail" cases explicitly, rather than only testing successful paths — this is the difference between "I can ping stuff" and "I understand how security levels enforce policy."

