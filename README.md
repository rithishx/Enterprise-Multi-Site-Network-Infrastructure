# Enterprise Multi-Site Network Infrastructure

A Cisco Packet Tracer lab that builds the network of a fictional company, EuroConnect Logistics, step by step, starting with its Paris headquarters. Each day adds one layer and documents the configuration commands, the verification output and screenshots.

**Status:** in progress. Days 1 to 3 are complete; Day 4 is next.

## What is built so far

| Day | Topic | Highlights |
|---|---|---|
| [Day 1](Day%201/Day%201.md) | Enterprise LAN and basic switch configuration | Cisco 2960 switch, five workstations and one server, static IPv4 addressing, MAC address learning, ICMP tests |
| [Day 2](Day%202/Day%202.md) | Router-on-a-stick | Cisco 2911 router, five VLANs, 802.1Q trunk, one subinterface gateway per VLAN |
| [Day 3](Day%203/Day%203.md) | Inter-VLAN security | Extended ACL `MGMT_ACCESS_POLICY`: only the IT VLAN can reach the management VLAN, verified with ACL hit counters |

## Paris headquarters addressing

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | Administration | 192.168.10.0/24 | 192.168.10.1 |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Employees | 192.168.30.0/24 | 192.168.30.1 |
| 40 | Testing | 192.168.40.0/24 | 192.168.40.1 |
| 99 | Management | 192.168.99.0/24 | 192.168.99.1 |

## Next

Day 4: Layer 2 security, with port security, DHCP snooping, Dynamic ARP Inspection and STP protection.

## Open the lab

Open `Enterprise Multi-Site Network Infrastructure.pkt` in Cisco Packet Tracer.

## Skills practised

VLANs · 802.1Q trunking · inter-VLAN routing (router-on-a-stick) · extended ACLs · wildcard masks · Cisco IOS CLI · troubleshooting with ping and show commands
