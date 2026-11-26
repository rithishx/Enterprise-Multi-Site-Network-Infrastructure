# Day 2 – Router-on-a-Stick Implementation

## Objective
Implement inter-VLAN routing in the Paris enterprise network using Router-on-a-Stick architecture.

## Environment
- Cisco Packet Tracer
- Router: Cisco 2911 (R1-PARIS)
- Switch: SW1-PARIS
- Network: Enterprise Multi-Site Network Infrastructure

## VLANs Configured

| VLAN ID | Name | Gateway |
|---|---|---|
| 10 | ADMINISTRATION | 192.168.10.1 |
| 20 | IT | 192.168.20.1 |
| 30 | EMPLOYEES | 192.168.30.1 |
| 40 | SERVERS | 192.168.40.1 |
| 99 | MANAGEMENT | 192.168.99.1 |

## Commands Used

### Switch – Trunk Configuration

```cisco
enable
configure terminal
interface fa0/7
switchport mode trunk
description TRUNK_TO_R1-PARIS
switchport trunk allowed vlan 10,20,30,40,99
exit
end
write memory
show interfaces trunk
```

### Router – Subinterface Configuration

```cisco
enable
configure terminal

interface g0/0
no shutdown
exit

interface g0/0.10
encapsulation dot1q 10
ip address 192.168.10.1 255.255.255.0
description GATEWAY_ADMINISTRATION
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
description GATEWAY_SERVERS
exit

interface g0/0.99
encapsulation dot1q 99
ip address 192.168.99.1 255.255.255.0
description GATEWAY_MANAGEMENT
exit

end
write memory
```

## Verification Commands

```cisco
show ip interface brief
show interfaces trunk
show vlan brief
```

### PC Connectivity Tests

```cmd
ping 192.168.30.1
ping 192.168.20.10
ping 192.168.40.11
ping 192.168.10.10
ping 192.168.99.1
```

## Results

- Router subinterfaces were configured successfully.
- 802.1Q encapsulation was configured for all five VLANs.
- Switch trunk was configured to carry VLANs 10, 20, 30, 40 and 99.
- Inter-VLAN connectivity was successfully verified.
- Initial ping timeouts occurred during testing, followed by successful replies.

## Conclusion

Successfully implemented Router-on-a-Stick routing for the Paris enterprise network. Devices belonging to different VLANs can communicate through the router using their respective default gateways.