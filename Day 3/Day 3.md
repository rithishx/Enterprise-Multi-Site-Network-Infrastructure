# Day 3 – Router-on-a-Stick & Inter-VLAN Security

## 1. Objective

Implement Router-on-a-Stick architecture to enable inter-VLAN routing and secure access to the Management VLAN using Extended Access Control Lists (ACLs).

## 2. Network Topology

**Devices**
- R1-PARIS – Cisco 2911 Router
- SW1-PARIS – Layer 2 Switch
- PC-ADMIN – VLAN 10
- PC-IT – VLAN 20
- PC-EMP – VLAN 30
- PC-TEST – VLAN 40
- SRV-PARIS – VLAN 99

## 3. VLAN and IP Addressing

| VLAN | Department | Network | Gateway |
|---|---|---|---|
| 10 | Administration | 192.168.10.0/24 | 192.168.10.1 |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Employees | 192.168.30.0/24 | 192.168.30.1 |
| 40 | Testing | 192.168.40.0/24 | 192.168.40.1 |
| 99 | Management | 192.168.99.0/24 | 192.168.99.1 |

Management server: `192.168.99.10`

## 4. Switch Configuration

The switch port connected to R1-PARIS was configured as an 802.1Q trunk.

```cisco
enable
configure terminal

interface fa0/7
 description TRUNK_TO_R1-PARIS
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99
 exit

end
write memory
```

## 5. Router-on-a-Stick Configuration

The router's physical interface G0/0 connects to the switch trunk.

Subinterfaces were created for each VLAN.

```cisco
enable
configure terminal

interface g0/0
 no shutdown
 exit

interface g0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
 description GATEWAY_ADMIN
 exit

interface g0/0.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
 description GATEWAY_IT
 exit

interface g0/0.30
 encapsulation dot1q 30
 ip address 192.168.30.1 255.255.255.0
 description GATEWAY_EMPLOYEES
 exit

interface g0/0.40
 encapsulation dot1q 40
 ip address 192.168.40.1 255.255.255.0
 description GATEWAY_TESTING
 exit

interface g0/0.99
 encapsulation dot1q 99
 ip address 192.168.99.1 255.255.255.0
 description GATEWAY_MANAGEMENT
 exit

end
write memory
```

## 6. Management VLAN Security – Extended ACL

An Extended ACL named `MGMT_ACCESS_POLICY` was created to control access to the Management VLAN.

### Security policy

| Source VLAN | Destination | Action |
|---|---|---|
| VLAN 20 – IT | VLAN 99 | Permit |
| VLAN 10 – Administration | VLAN 99 | Deny |
| VLAN 30 – Employees | VLAN 99 | Deny |
| VLAN 40 – Testing | VLAN 99 | Deny |
| Other traffic | Any | Permit |

### ACL Configuration

```cisco
enable
configure terminal

ip access-list extended MGMT_ACCESS_POLICY
 permit ip 192.168.20.0 0.0.0.255 192.168.99.0 0.0.0.255
 deny ip 192.168.10.0 0.0.0.255 192.168.99.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.99.0 0.0.0.255
 permit ip any any
 exit

interface g0/0.99
 ip access-group MGMT_ACCESS_POLICY out
 exit

end
write memory
```

## 7. Verification and Testing

### Router interface verification

Command:

```cisco
show ip interface brief
```

All five subinterfaces were verified as operational:

- G0/0.10 – UP/UP
- G0/0.20 – UP/UP
- G0/0.30 – UP/UP
- G0/0.40 – UP/UP
- G0/0.99 – UP/UP

### Connectivity testing

The following connectivity tests were performed:

| Test | Result |
|---|---|
| PC-EMP → VLAN 30 Gateway | Successful |
| PC-EMP → VLAN 20 Host | Successful after initial timeout |
| PC-EMP → VLAN 40 Host | Initial requests timed out |
| PC-ADMIN → VLAN 99 Server | Blocked |
| PC-IT → VLAN 99 Server | Permitted |
| VLAN 30 → VLAN 99 | Blocked |
| VLAN 40 → VLAN 99 | Blocked |

### ACL verification

Command:

```cisco
show access-lists MGMT_ACCESS_POLICY
```

Final observed ACL counters:

| ACL Rule | Match Count |
|---|---:|
| Permit VLAN 20 | 8 |
| Deny VLAN 10 | 12 |
| Deny VLAN 30 | 8 |
| Deny VLAN 40 | 4 |

The ACL counters confirmed that traffic was matching the intended security rules.

## 8. Key Concepts Learned

1. Router-on-a-Stick architecture.
2. IEEE 802.1Q VLAN tagging.
3. Router subinterfaces.
4. Inter-VLAN routing.
5. Extended Access Control Lists.
6. ACL direction and placement.
7. Wildcard masks.
8. ACL hit-counter verification.
9. Management VLAN access control.
10. Network troubleshooting using ping and IOS commands.

## 9. Final Outcome

Router-on-a-Stick was successfully implemented on R1-PARIS.

Inter-VLAN routing was established across five VLANs. An Extended ACL was configured on the Management VLAN gateway to permit IT access while restricting Administration, Employees and Testing VLANs.

The configuration was saved, and ACL counters verified the expected policy matches.

**Day 3 Status: COMPLETED**

**Next:** Day 4 – Layer 2 Network Security (Port Security, DHCP Snooping, Dynamic ARP Inspection and STP Protection).
