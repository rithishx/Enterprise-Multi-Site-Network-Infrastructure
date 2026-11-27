# Day 04 — Layer 2 Security

## Project: Enterprise Multi-Site Network Infrastructure
**Site:** Paris | **Platform:** Cisco Packet Tracer

## 1. Objectives
- Restrict unauthorized endpoints using sticky MAC Port Security.
- Deploy DHCP and protect clients against unauthorized DHCP servers.
- Configure Dynamic ARP Inspection (DAI).
- Protect edge interfaces using PortFast and BPDU Guard.
- Isolate and disable unused interfaces.
- Verify existing routing and management ACL functionality.

## 2. Network inventory
| Port | Endpoint | VLAN | IPv4 address |
|---|---|---:|---|
| Fa0/1 | PC-ADMIN | 10 | 192.168.10.10 |
| Fa0/2 | PC-IT | 20 | 192.168.20.10 |
| Fa0/3 | PC-EMP | 30 | 192.168.30.50 (DHCP) |
| Fa0/4 | PC-MGMT | 99 | 192.168.99.10 |
| Fa0/5 | PC-TEST | 30 | 192.168.30.51 (DHCP) |
| Fa0/6 | SRV-PARIS | 40 | 192.168.40.11 |
| Fa0/7 | R1-PARIS | Trunk | — |
| Fa0/8–24, Gi0/1–2 | Unused | 999 | — |

## 3. Port Security
```cisco
configure terminal
interface range fa0/1 - 6
switchport mode access
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation shutdown
end
```
- `switchport mode access`: forces access mode.
- `switchport port-security`: enables MAC-based access restrictions.
- `maximum 1`: permits one secure MAC per port.
- `mac-address sticky`: learns and records secure MAC addresses in the running configuration.
- `violation shutdown`: err-disables a port upon a violation.

Verification: `show port-security`, `show port-security address`, `show port-security interface fa0/1`, `show interfaces status`.

### Unauthorized endpoint test
Fa0/1 learned 00E0.8F16.7429. A replacement endpoint with MAC 0001.6354.75CD triggered one violation, secure-shutdown, err-disabled status and 100% ping loss. After reconnecting the legitimate endpoint, `shutdown` followed by `no shutdown` recovered the interface. Ping succeeded and the port returned to Secure-up.

## 4. DHCP server — R1-PARIS
```cisco
configure terminal
service dhcp
ip dhcp excluded-address 192.168.30.1 192.168.30.49
ip dhcp pool EMPLOYEES_POOL
network 192.168.30.0 255.255.255.0
default-router 192.168.30.1
end
```
- `service dhcp`: enables DHCP service.
- `ip dhcp excluded-address`: reserves addresses from allocation.
- `ip dhcp pool`: creates a named pool.
- `network`: specifies the client subnet and mask.
- `default-router`: supplies the gateway.

Allocation range: 192.168.30.50–254 (205 addresses). Other VLANs retain static addressing. No DNS server was configured.

Verification: `show ip dhcp pool`, `show ip dhcp binding`, `show running-config`. The exclusion counter displayed 1 despite the configured range; actual allocations started at .50. An accidental pool named EMPLOYEES_POOLend was identified; cleanup command: `no ip dhcp pool EMPLOYEES_POOLend`.

## 5. DHCP Snooping — SW1-PARIS
```cisco
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 30
no ip dhcp snooping information option
interface fa0/7
ip dhcp snooping trust
exit
interface fa0/3
ip dhcp snooping limit rate 15
exit
interface fa0/5
ip dhcp snooping limit rate 15
end
```
- Global snooping enables the feature; VLAN configuration activates protection on VLAN 30.
- Option 82 insertion is disabled for this simple lab configuration.
- Trust permits server DHCP messages arriving from the router trunk.
- Rate limiting restricts incoming DHCP traffic on client ports to 15 packets per second.

### Verified bindings
| IP | MAC | VLAN | Interface |
|---|---|---:|---|
| 192.168.30.50 | 00D0.BCC2.650C | 30 | Fa0/3 |
| 192.168.30.51 | 0003.E48A.1E1D | 30 | Fa0/5 |

Verification: `show ip dhcp snooping` checks trust, VLANs and limits; `show ip dhcp snooping binding` displays learned associations. Lease seconds displayed zero in the simulator.

### Rogue DHCP test
A temporary server at 192.168.30.200 was attached to untrusted Fa0/8 in VLAN 30. PC-TEST renewed its legitimate .51 address and .1 gateway twice. Packet captures showed an unauthorized response at the switch, a dropped packet, and a legitimate response at the client. This supports DHCP Snooping protection, although the exact DHCP message type and explicit drop reason were not conclusively established. Temporary equipment was removed and the test port reused later. The temporary DHCP exclusion can be removed with `no ip dhcp excluded-address 192.168.30.200`.

## 6. Dynamic ARP Inspection
```cisco
configure terminal
interface fa0/7
ip arp inspection trust
exit
ip arp inspection vlan 30
ip arp inspection validate src-mac dst-mac ip
end
```
- `ip arp inspection trust`: bypasses DAI checks for incoming ARP on the router-facing interface.
- `ip arp inspection vlan 30`: enables ARP inspection on VLAN 30 using authorized bindings.
- `validate src-mac dst-mac ip`: enables additional Ethernet/ARP MAC consistency and IP validity checks.

DAI trust is independent of DHCP Snooping trust. Other VLANs were excluded because their endpoints use static addressing.

Verification: `show ip arp inspection`, `show ip arp inspection interfaces`, `show ip arp inspection statistics`.

### Enforcement test and limitation
Legitimate gateway and peer pings succeeded. PC-TEST was temporarily assigned 192.168.30.25, outside the DHCP allocation range. Its snooping binding disappeared, while Port Security remained Secure-up with zero violations. ICMP replies still reached the endpoint, and DAI counters remained zero. No fresh unauthorized ARP drop was conclusively captured. Enforcement is therefore **inconclusive**, possibly due to cached ARP or simulator limitations. DAI filters ARP rather than general IP traffic; IP Source Guard is a separate control. PC-TEST was restored to DHCP and both bindings were verified again.

## 7. STP PortFast and BPDU Guard
Baseline: PVST, SW1-PARIS root for all six original active VLANs, bridge MAC 00E0.B010.82BD, base priority 32768, hello 2 s, max age 20 s, forward delay 15 s. FastEthernet short path cost 19. All active port/VLAN instances were forwarding.

```cisco
configure terminal
interface range fa0/1 - 6
spanning-tree portfast
spanning-tree bpduguard enable
end
```
- `spanning-tree portfast`: skips listening and learning delays on edge ports; does not disable STP.
- `spanning-tree bpduguard enable`: err-disables a port receiving a BPDU.

Fa0/7 trunk was excluded. Global defaults remain disabled because configuration is per interface.

Verification: `show spanning-tree summary`, `show spanning-tree vlan 30`, `show spanning-tree interface fa0/1 detail`, `show running-config`. PortFast was directly confirmed on Fa0/1 and Fa0/5; BPDU Guard was directly confirmed in Fa0/5 running configuration.

### Rogue switch test
A temporary switch was connected to Fa0/8, configured as an access port with PortFast and BPDU Guard, without Port Security to isolate the test. The operator confirmed successful verification. Raw violation logs were not supplied. The temporary switch was removed and the port recovered using shutdown/no shutdown.

## 8. Unused port hardening
```cisco
configure terminal
vlan 999
name PARKING_LOT
exit
interface range fa0/8 - 24
 description UNUSED_DISABLED
 switchport mode access
 switchport access vlan 999
 switchport nonegotiate
 shutdown
exit
interface range gi0/1 - 2
 description UNUSED_DISABLED
 switchport mode access
 switchport access vlan 999
 switchport nonegotiate
 shutdown
end
```
- `vlan 999`: creates an isolated parking VLAN.
- `switchport mode access`: prevents negotiated trunk operation.
- `switchport access vlan 999`: assigns unused ports to the parking VLAN.
- `switchport nonegotiate`: disables DTP negotiation where supported.
- `shutdown`: administratively disables the ports.

19 unused ports were hardened. VLAN 999 is excluded from the router trunk and has no routed gateway. Operator confirmed verification; raw final interface output was not supplied.

## 9. Final regression testing
| Source | Destination | Expected result |
|---|---|---|
| PC-ADMIN | Gateway | Allowed |
| PC-ADMIN | PC-MGMT | Blocked |
| PC-IT | PC-MGMT | Allowed |
| PC-EMP | Gateway | Allowed |
| PC-EMP | PC-TEST | Allowed |
| PC-EMP | SRV-PARIS | Allowed |
| PC-EMP | PC-MGMT | Blocked |
| SRV-PARIS | PC-MGMT | Blocked |

Operator confirmed all final checks were okay. Individual final ping transcripts were not supplied.

Final switch audit: `show vlan brief`, `show interfaces status`, `show interfaces trunk`, `show port-security`, `show port-security address`, `show ip dhcp snooping`, `show ip dhcp snooping binding`, `show ip arp inspection`, `show ip arp inspection statistics`, `show spanning-tree summary`.

Final router audit: `show ip interface brief`, `show ip route`, `show ip dhcp pool`, `show ip dhcp binding`, `show access-lists`, `show ip interface gigabitEthernet0/0.99`.

Save on both devices using `copy running-config startup-config` (copies active RAM configuration to startup storage), then save the existing cumulative Packet Tracer project.

## 10. Evidence checklist
- Port Security sticky MAC table, violation and recovery outputs.
- Router DHCP pool and switch DHCP Snooping bindings.
- Rogue and legitimate DHCP packet captures.
- DAI configuration, zero counters and unauthorized static-IP test.
- STP baseline and PortFast/BPDU Guard interface configuration.
- BPDU Guard violation output and final parking VLAN/interface status.
- Final connectivity and management ACL tests.

## 11. Conclusion
Day 4 configuration and regression testing completed. Port Security, DHCP allocation and binding learning were directly demonstrated. Rogue DHCP evidence supports filtering but does not conclusively identify the drop reason. BPDU Guard and unused port hardening were confirmed by the operator. DAI enforcement remains unverified and should be revisited on a more complete IOS lab platform.
