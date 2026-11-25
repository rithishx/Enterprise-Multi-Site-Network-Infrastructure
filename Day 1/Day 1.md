# Day 1 – Enterprise LAN Setup and Basic Switch Configuration

## 1. Overview

**Project:** Enterprise Multi-Site Network Infrastructure  
**Company:** EuroConnect Logistics (Fictional)  
**Location:** Paris Headquarters  
**Lab Tool:** Cisco Packet Tracer  
**Device:** Cisco Catalyst 2960 Switch  
**Day:** Week 1 – Day 1

## 2. Objective

The objective of Day 1 was to build a basic Local Area Network (LAN) for the Paris headquarters of EuroConnect Logistics.

The practical focused on:

- Creating an enterprise LAN topology.
- Connecting multiple end devices through a Cisco switch.
- Configuring static IPv4 addresses.
- Performing basic Cisco switch configuration using CLI.
- Understanding MAC address learning.
- Testing connectivity between network devices using ICMP ping.
- Saving the switch configuration.

## 3. Network Topology

The network consists of:

| Device | Quantity | Description |
|---|---:|---|
| Cisco 2960 Switch | 1 | Central Layer 2 switch |
| PC-PT | 5 | Department workstations |
| Server-PT | 1 | Headquarters server |
| Copper Straight-Through | 6 | Network connections |

### Device Naming

| Device | Name |
|---|---|
| Switch | SW1-PARIS |
| PC 1 | PC-ADMIN |
| PC 2 | PC-IT |
| PC 3 | PC-EMP |
| PC 4 | PC-MGMT |
| PC 5 | PC-TEST |
| Server | SRV-PARIS |

## 4. IP Addressing Plan

A single IPv4 subnet was used for the initial LAN configuration.

**Network:** 192.168.1.0/24  
**Subnet Mask:** 255.255.255.0

| Device | IP Address | Subnet Mask |
|---|---|---|
| PC-ADMIN | 192.168.1.10 | 255.255.255.0 |
| PC-IT | 192.168.1.11 | 255.255.255.0 |
| PC-EMP | 192.168.1.12 | 255.255.255.0 |
| PC-MGMT | 192.168.1.13 | 255.255.255.0 |
| PC-TEST | 192.168.1.14 | 255.255.255.0 |
| SRV-PARIS | 192.168.1.100 | 255.255.255.0 |

All addresses were configured manually using the Desktop → IP Configuration interface in Cisco Packet Tracer.

No default gateway was configured because all devices were communicating within the same subnet.

## 5. Switch Port Allocation

Each end device was connected to a dedicated FastEthernet interface.

| Switch Interface | Connected Device |
|---|---|
| Fa0/1 | PC-ADMIN |
| Fa0/2 | PC-IT |
| Fa0/3 | PC-EMP |
| Fa0/4 | PC-MGMT |
| Fa0/5 | PC-TEST |
| Fa0/6 | SRV-PARIS |

All connections used Copper Straight-Through cables.

## 6. Commands Used

### Step 1 – Enter Privileged EXEC Mode

```bash
enable
```

Provides access to privileged switch commands.

### Step 2 – Enter Global Configuration Mode

```bash
configure terminal
```

Enters global configuration mode to modify switch settings.

### Step 3 – Configure Switch Hostname

```bash
hostname SW1-PARIS
```

Changes the switch hostname to SW1-PARIS.

### Step 4 – Disable DNS Lookup

```bash
no ip domain-lookup
```

Prevents unnecessary DNS lookup attempts when an incorrect command is entered.

### Step 5 – Configure Interface Description

```bash
interface fastethernet 0/1
description PC-ADMIN
exit
```

Adds a description to interface Fa0/1 to identify the connected device.

### Step 6 – Configure Multiple Interface Descriptions

```bash
interface range fastethernet 0/2-6
description END-DEVICES
exit
```

Applies the same interface description to ports Fa0/2 through Fa0/6.

### Step 7 – Exit Configuration Mode

```bash
end
```

Returns to privileged EXEC mode.

### Step 8 – Verify Interface Status

```bash
show interfaces status
```

Displays interface information, including connection status and VLAN assignment.

### Step 9 – Verify MAC Address Learning

```bash
show mac address-table
```

Displays dynamically learned MAC addresses and their associated switch ports.

### Step 10 – Save Configuration

```bash
copy running-config startup-config
```

Saves the current running configuration to startup configuration so it is retained after a restart.

## 7. Connectivity Testing

Connectivity was verified using the Command Prompt available on PC-ADMIN.

### Test 1 – PC-to-PC Connectivity

```bash
ping 192.168.1.11
```

**Destination:** PC-IT

### Test 2 – PC-to-PC Connectivity

```bash
ping 192.168.1.12
```

**Destination:** PC-EMP

### Test 3 – PC-to-Server Connectivity

```bash
ping 192.168.1.100
```

**Destination:** SRV-PARIS

### Ping Test Result

The ping test to PC-IT returned:

| Parameter | Result |
|---|---|
| Packets Sent | 4 |
| Packets Received | 4 |
| Packets Lost | 0 |
| Packet Loss | 0% |
| Average Response Time | 0 ms |

**Result:** Successful connectivity was established between PC-ADMIN and PC-IT.

## 8. MAC Address Table Verification

The following command was used:

```bash
show mac address-table
```

The switch successfully learned six dynamic MAC addresses.

| VLAN | Interface | MAC Learning Type |
|---|---|---|
| 1 | Fa0/1 | Dynamic |
| 1 | Fa0/2 | Dynamic |
| 1 | Fa0/3 | Dynamic |
| 1 | Fa0/4 | Dynamic |
| 1 | Fa0/5 | Dynamic |
| 1 | Fa0/6 | Dynamic |

All six connected devices were learned under the default VLAN 1.

This verified that the switch was successfully learning device MAC addresses and associating them with the correct interfaces.

## 9. Final Results

The following objectives were successfully completed:

- Created a basic enterprise LAN topology.
- Connected five PCs and one server to a Cisco 2960 switch.
- Configured static IPv4 addresses.
- Configured the switch hostname.
- Added interface descriptions.
- Verified switch interface status.
- Observed dynamic MAC address learning.
- Successfully tested PC-to-PC connectivity.
- Saved the switch configuration.

**Final Outcome:** A functional single-subnet LAN was successfully implemented and tested using Cisco Packet Tracer.

## 10. Technologies and Concepts Learned

| Technology / Concept | Practical Application |
|---|---|
| IPv4 Addressing | Static IP configuration |
| Subnetting | /24 network configuration |
| Layer 2 Switching | Communication between end devices |
| MAC Address Learning | Switch MAC address table |
| Cisco IOS CLI | Basic switch administration |
| ICMP | Connectivity testing using ping |
| Ethernet | Physical LAN connectivity |
| Network Troubleshooting | Verification of connectivity and interfaces |

## 11. Next Steps – Day 2

The existing flat LAN will be upgraded using VLAN-based network segmentation.

Planned activities:

1. Create VLAN 10 – Administration.
2. Create VLAN 20 – IT.
3. Create VLAN 30 – Employees.
4. Create VLAN 40 – Servers.
5. Create VLAN 99 – Management.
6. Assign switch interfaces to the appropriate VLANs.
7. Verify VLAN configuration using `show vlan brief`.
8. Test communication between devices in the same and different VLANs.

**Expected outcome:** A logically segmented enterprise LAN with separate broadcast domains for each department.

---

**Project Status:** Week 1 – Day 1 Completed  
**Next Milestone:** Week 1 – Day 2: VLAN Configuration and Network Segmentation