Topology Summary


Switch0 (Inside/LAN) — PC0, PC1, PC2, PC3, PC4
Switch2 (Outside) — PC5, PC6, XYZ SERVER
Switch1 (DMZ) — WEB SERVER, DNS SERVER, DHCP SERVER, Meraki EMAIL SERVER
ASA1 (5505) — sits between all three, one leg per zone


ASA Interface Mapping

ASA PortConnects to ZoneSecurity LevelEthernet0/0Switch0Inside100Ethernet0/1Switch1DMZ50Ethernet0/2Switch2Outside0

IP Addressing Plan

ZoneSubnetASA Gateway IPInside (Switch0)192.168.10.0/24192.168.10.1DMZ (Switch1)192.168.20.0/24192.168.20.1Outside (Switch2)192.168.30.0/24192.168.30.1

Inside hosts:


PC0 → 192.168.10.10
PC1 → 192.168.10.11
PC2 → 192.168.10.12
PC3 → 192.168.10.13
PC4 → 192.168.10.14
Gateway for all: 192.168.10.1


DMZ hosts:


WEB SERVER → 192.168.20.10
DNS SERVER → 192.168.20.11
DHCP SERVER → 192.168.20.12
EMAIL SERVER (Meraki) → 192.168.20.13
Gateway for all: 192.168.20.1


Outside hosts:


PC5 → 192.168.30.10
PC6 → 192.168.30.11
XYZ SERVER → 192.168.30.12
Gateway for all: 192.168.30.1



Step 1: Configure PC/Server IPs

For each device above, go to Desktop → IP Configuration (PCs) or Config → FastEthernet0 (servers), select Static, and enter the IP/Subnet Mask (255.255.255.0)/Gateway from the tables above.

✅ Checkpoint: Once all 12 devices are set, this step is done. No pings will succeed yet — that's expected, since the ASA interfaces aren't configured.


Step 2: Basic ASA Interface Configuration

Click ASA1 → CLI.


Important — ASA 5505 quirk: unlike bigger ASA models, the 5505's physical Ethernet0/x ports are switchports, not routed interfaces. You cannot run nameif directly on Ethernet0/0. Instead, you create VLAN interfaces (the Layer 3, routed side) and then assign each physical port to a VLAN (the Layer 2, switch side).

There's also a licensing limit: the Base license only allows 2 fully-forwarding named interfaces. A third (DMZ, in our case) must be restricted from forwarding to one of the other two using no forward interface vlan X. This isn't a mistake to fix — it's just how the 5505 Base license works, and conveniently it reinforces our DMZ→Inside isolation goal from Step 8.



ciscoasa> enable
ciscoasa# configure terminal

! --- VLAN 1 = Inside ---
ciscoasa(config)# interface vlan 1
ciscoasa(config-if)# nameif inside
ciscoasa(config-if)# security-level 100
ciscoasa(config-if)# ip address 192.168.10.1 255.255.255.0
ciscoasa(config-if)# no shutdown
ciscoasa(config-if)# exit

! --- VLAN 2 = DMZ (restricted from forwarding to Inside, due to Base license) ---
ciscoasa(config)# interface vlan 2
ciscoasa(config-if)# no forward interface vlan 1
ciscoasa(config-if)# nameif dmz
ciscoasa(config-if)# security-level 50
ciscoasa(config-if)# ip address 192.168.20.1 255.255.255.0
ciscoasa(config-if)# no shutdown
ciscoasa(config-if)# exit

! --- VLAN 3 = Outside ---
ciscoasa(config)# interface vlan 3
ciscoasa(config-if)# nameif outside
ciscoasa(config-if)# security-level 0
ciscoasa(config-if)# ip address 192.168.30.1 255.255.255.0
ciscoasa(config-if)# no shutdown
ciscoasa(config-if)# exit

! --- Now assign each physical switchport to its VLAN ---
ciscoasa(config)# interface ethernet0/0
ciscoasa(config-if)# switchport access vlan 1
ciscoasa(config-if)# no shutdown
ciscoasa(config-if)# exit

ciscoasa(config)# interface ethernet0/1
ciscoasa(config-if)# switchport access vlan 2
ciscoasa(config-if)# no shutdown
ciscoasa(config-if)# exit

ciscoasa(config)# interface ethernet0/2
ciscoasa(config-if)# switchport access vlan 3
ciscoasa(config-if)# no shutdown
ciscoasa(config-if)# exit

✅ Checkpoint — verify before continuing:

ciscoasa# show interface ip brief
ciscoasa# show switch vlan

All three VLAN interfaces should read up/up with the correct IPs, and show switch vlan should confirm Ethernet0/0→VLAN1, Ethernet0/1→VLAN2, Ethernet0/2→VLAN3. Screenshot both outputs.


Step 3: Test Basic Connectivity (Same-Zone Only)

From PC0: ping 192.168.10.1 — should succeed (own gateway).
From WEB SERVER: ping 192.168.20.1 — should succeed.
From PC5: ping 192.168.30.1 — should succeed.

✅ Checkpoint: All three gateway pings succeed before moving on. If any fail, recheck that device's static IP/gateway and the ASA interface's no shutdown status.


Step 4: Configure NAT (Inside → Outside, DMZ → Outside)

ciscoasa(config)# object network inside-net
ciscoasa(config-network-object)# subnet 192.168.10.0 255.255.255.0
ciscoasa(config-network-object)# nat (inside,outside) dynamic interface
ciscoasa(config-network-object)# exit

ciscoasa(config)# object network dmz-net
ciscoasa(config-network-object)# subnet 192.168.20.0 255.255.255.0
ciscoasa(config-network-object)# nat (dmz,outside) dynamic interface
ciscoasa(config-network-object)# exit

✅ Checkpoint:

ciscoasa# show nat

Screenshot this, confirming both rules appear.


Step 5: Test Inside → Outside (Should Succeed)

From PC0: ping 192.168.30.10 (PC5's IP)

This should succeed — Inside (100) → Outside (0) is allowed by default, and NAT handles the translation.

✅ Checkpoint — screenshot this successful ping.


Step 6: Test Outside → Inside (Should FAIL)

From PC5: ping 192.168.10.10 (PC0's IP)

This should fail — Outside (0) → Inside (100) is blocked by default. This is the firewall working correctly.

✅ Checkpoint — screenshot this failed ping, and label it "expected failure — security level enforcement" in your README.


Step 7: Expose the Web Server to Outside (Static NAT + ACL)

This lets outside users reach your DMZ web server on port 80, while everything else stays blocked.

ciscoasa(config)# object network web-server
ciscoasa(config-network-object)# host 192.168.20.10
ciscoasa(config-network-object)# nat (dmz,outside) static 192.168.30.20
ciscoasa(config-network-object)# exit

ciscoasa(config)# access-list OUTSIDE-IN extended permit tcp any host 192.168.30.20 eq 80
ciscoasa(config)# access-group OUTSIDE-IN in interface outside

✅ Checkpoint:

ciscoasa# show access-list OUTSIDE-IN

Screenshot this. Then, from PC5, try opening http://192.168.30.20 in the web browser app (if WEB SERVER has HTTP service enabled under its Services tab) — it should load. A ping to 192.168.30.20 will still fail (ICMP isn't permitted, only TCP/80) — that's correct and expected.


Step 8: Confirm DMZ Cannot Reach Inside

From WEB SERVER: ping 192.168.10.10 (PC0)

Should fail — DMZ (50) → Inside (100) is blocked by default. Confirms a compromised DMZ server can't pivot into your internal LAN.

✅ Checkpoint — screenshot this failed ping.


Step 9: Save Configuration

ciscoasa# write memory


Final Screenshot Checklist


Full topology (already have this)
show interface ip brief — all 3 up/up
Gateway pings from each zone (3 screenshots or 1 combined)
show nat output
PC0 → PC5 ping (success)
PC5 → PC0 ping (fail — expected)
show access-list OUTSIDE-IN
PC5 → Web Server HTTP access (success)
Web Server → PC0 ping (fail — expected)
show running-config (full reference)



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

