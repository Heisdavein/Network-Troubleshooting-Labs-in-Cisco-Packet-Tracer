# Network Troubleshooting Labs in Cisco Packet Tracer

Seven fault-finding scenarios, each with a realistic user complaint, a systematic investigation, a root cause, a targeted fix and a verification. Together they cover IP configuration, ACLs, Spanning Tree (BPDU Guard), inter-VLAN routing with NAT, Port Security, OSPF and AAA with TACACS+.


## Contents

- Overview
- Objectives
- Technologies Used
- Skills Demonstrated
- Environment
- Method
- Lab Summary
- Project Scope
- Repository Structure
- Validation and Testing
- Security Considerations
- Lessons Learned
- Future Improvements
- Author

## Overview

Networking is not only knowing how to configure devices, it is knowing how to think when something is broken. In each of these labs a fault was already present. I had to confirm the symptom, find the scope, work through the OSI layers using Cisco diagnostic commands, apply a targeted fix, and verify that it worked without causing new problems.

The guiding rule throughout: **never guess, always verify**.


 <img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/d6ebb6fc-5630-4eaf-8e21-a7925e7ec299" />
`show access-lists` on the router showing the standard ACL that permits only one host.


## Objectives

- Troubleshoot from evidence, not assumptions.
- Establish scope first: one device, one segment, or the whole network.
- Use the OSI model as a checklist, from Layer 1 upwards.
- Read device logs and `show` output to find root causes.
- Fix the cause and keep security features in place rather than switching them off.
- Prove every fix with a test, including a failover test where relevant.

## Technologies Used

| Area | Technology |
| --- | --- |
| Simulator | Cisco Packet Tracer |
| CLI | Cisco IOS |
| Topics | Static IP and DHCP, APIPA, standard ACLs and implicit deny, STP PortFast and BPDU Guard, 802.1Q trunks, Router-on-a-Stick, NAT overload, Port Security (sticky MAC), OSPF areas, SSH, AAA with TACACS+ |
| Client tools | `ping`, `ipconfig`, `ipconfig /all`, `ssh` |

## Skills Demonstrated

| Skill | Lab |
| --- | --- |
| Layered troubleshooting, reading `ipconfig` and the MAC table | Lab 1 |
| Recognising APIPA, ACL analysis, implicit deny, interface ACL placement | Lab 2 |
| STP protection features, access versus trunk ports, error-disabled recovery | Lab 3 |
| NAT overload with an access list, `show ip nat translations` | Lab 4 |
| Port Security, violation modes, sticky MAC | Lab 5 |
| OSPF neighbour troubleshooting, failover testing | Lab 6 |
| AAA, TACACS+, isolating the failing component | Lab 7 |
| Change management and documentation | Method |

## Environment

| Item | Detail |
| --- | --- |
| Platform | Cisco Packet Tracer |
| Test target | `1.1.1.2`, standing in for the internet |
| Typical LAN | `192.168.1.0/24`, gateway `192.168.1.1` |
| Devices | Routers, switches, PCs and a laptop, plus a TACACS+ server in Lab 7 |

## Method

Every lab followed the same pattern:

```mermaid
flowchart LR
    A["Confirm the symptom"] --> B["Find the scope"]
    B --> C["Work through the OSI layers"]
    C --> D["Collect evidence with show commands"]
    D --> E["Apply a targeted fix"]
    E --> F["Verify and test"]
```

Details and the command toolkit are in Method and the command reference.

## Lab Summary

| Lab | Symptom | Root cause | Fix |
| --- | --- | --- | --- |
| 1 | One laptop cannot reach the internet | Static IP from another office (`10.1.1.x`) | Switch to DHCP |
| 2 | Whole office offline, PCs show `169.254.x.x` | ACL 2 on the LAN interface permitted only `192.168.1.100` | Remove `ip access-group 2 in` and delete the ACL |
| 3 | New switch’s link will not come up | Inter-switch link was an access port with PortFast and BPDU Guard | Configure the link as a trunk, reset the port |
| 4 | New VLAN 2 server reaches its gateway, not the internet | NAT access list only permitted `192.168.1.0/24` | Add `192.168.2.0/24` to access list 1 |
| 5 | New PC gets no IP address | Port Security remembered the old PC’s MAC | Remove old MAC, enable sticky MAC |
| 6 | West building lost connectivity when its main link failed | OSPF area 0 on one side, area 100 on the other | Put North’s link in area 0, test failover |
| 7 | New engineer cannot SSH into the router | User never created on the TACACS+ server | Create the account |

All fix commands are in `configs/fix-commands.txt`.

## Project Scope

**In scope:** the seven scenarios above.

**Not covered in the documentation**

- The original configuration of each topology before the fault
- The router’s AAA and TACACS+ configuration in Lab 7
- Troubleshooting beyond the Packet Tracer simulation
- An eighth lab (the title page refers to labs 1 through 8, but seven are documented)

## Repository Structure

```
.
├── README.md
├── LICENSE
├── .gitignore
├── configs/
│   └── fix-commands.txt            Fix commands for all labs
├── docs/
│   ├── methodology.md
│   ├── command-reference.md
│   ├── lab-1-wrong-static-ip.md
│   ├── lab-2-acl-blocking-traffic.md
│   ├── lab-3-bpdu-guard.md
│   ├── lab-4-nat-access-list.md
│   ├── lab-5-port-security.md
│   ├── lab-6-ospf-area-mismatch.md
│   ├── lab-7-tacacs-missing-user.md
│   └── lessons-learned.md
└── screenshots/
    └── README.md                   Index of expected screenshots
```

## Validation and Testing

Each fix was proven with the same test that had failed.

| Lab | Verification | Result |
| --- | --- | --- |
| 1 | `ping 1.1.1.2` from Laptop Zero | Four replies |
| 2 | `ping 1.1.1.2` from a PC, DHCP renewed | Four replies, addresses received |
| 3 | `show interfaces GigabitEthernet0/1 switchport`, pings from PC5 | Trunk with 802.1Q; gateway and `1.1.1.2` reachable |
| 4 | `ping 1.1.1.2` from the App Server, `show ip nat translations` | Four replies; server’s address now translated |
| 5 | DHCP on PC Zero, `ping 1.1.1.2` | Address received, four replies |
| 6 | `show ip ospf neighbor`, continuous ping while the West-South link was removed | Two neighbours; one packet lost, traffic via West, North, East |
| 7 | `ssh -l tony 192.168.1.1` | Login succeeded |

## Security Considerations

- Security features were kept and configured correctly, not removed: BPDU Guard, Port Security and AAA all worked as designed in these labs.
- Standard ACLs end with an invisible deny; test them on non-production interfaces first.
- Prefer SSH to Telnet because SSH encrypts credentials and traffic.
- Central authentication (TACACS+) needs a local fallback account on the device, and accounts must be created on the server as part of onboarding.
- Changes to production devices need review and testing first (change management).

## Lessons Learned

- Confirm the problem yourself before changing anything.
- Ask whether one device or many are affected, since it removes most causes at once.
- The simplest explanation (wrong IP, interface down, missing account) is often the right one.
- Fix the cause, not the symptom.
- Test redundancy by causing a failure, not by inspecting the topology.
- Always test after a change.

More in Lessons learned.

## Future Improvements

- Add the eighth lab to complete the series.
- Record the original and faulted configurations for each topology.
- Add scenarios for VLAN mismatches, duplex problems and DNS failures.
- Practise the same checks on real equipment.

This repository is part of my portfolio as I build toward an IT support and cybersecurity career.
