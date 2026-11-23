# Lab 4: The Server VLAN That Could Not Reach the Internet

**Scenario:** the NAT access list was missing the new VLAN’s subnet.

## Situation

The company added VLAN 2 for an application server. Office computers in VLAN 1 reached the internet, but the new server could only talk to the local network. The network uses Router-on-a-Stick (one physical router link with a sub-interface per VLAN) and NAT, which translates private addresses (such as `192.168.x.x`) to a public address.

| Item | Detail |
| --- | --- |
| VLAN 1 | Workstations, `192.168.1.0/24` |
| VLAN 2 | Application Server, `192.168.2.0/24` (static address) |
| Router | Router-on-a-Stick with two sub-interfaces |
| Internet test | `1.1.1.2` |
| Problem device | App Server |

## Investigation

**1. Confirm and scope.** From the App Server, `ping 192.168.2.1` (its gateway) worked, but `ping 1.1.1.2` failed. A PC on VLAN 1 reached the internet, so the problem affected only VLAN 2.

**2. Check the configuration.** The App Server was in VLAN 2, and the switch-to-router link was a trunk carrying VLANs 1 and 2. On the router both sub-interfaces were up with the right addresses (`show ip interface brief`), and `show ip route` had a route to the internet. Because VLAN 1 browsed fine, routing was not the problem.

**3. Check NAT.** `show ip nat translations`: traffic from a VLAN 1 PC created entries immediately, but traffic from the App Server created none. NAT was ignoring VLAN 2.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/db5221ae-f0c9-4309-9f16-89dc06ec1aa3" />
 `show ip nat translations` with VLAN 1 entries and none for the App Server after generating traffic from both.


In the running configuration:

```
ip nat inside source list 1 interface <WAN_interface> overload
access-list 1 permit 192.168.1.0 0.0.0.255
```

NAT translated only the networks in access list 1, which listed `192.168.1.0/24`. VLAN 2 was missing.


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/6f123294-8203-45f6-8c0d-08f828e3b1d6" />
The NAT `overload` line and access list 1 in the running configuration.


## Root cause

The router translated traffic from `192.168.1.0/24` and ignored `192.168.2.0/24`. The server’s private address was never translated to a public one, so the internet side rejected its traffic.

## Fix

```
access-list 1 permit 192.168.2.0 0.0.0.255
```

`show access-lists` then listed both networks.

## Verification

`ping 1.1.1.2` from the App Server: four successful replies. `show ip nat translations` now showed the App Server’s address being translated.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/9e20ed1a-e3dd-4d9e-a483-c422041f8b35" />
 `show access-lists` with both networks, the successful ping and NAT translations including the App Server.

## What I learned

- Routing and NAT are separate. When a new VLAN or subnet is added, update NAT as well as routing.
- `show ip nat translations` is one of the quickest checks for internet access problems: a device that reaches its gateway but never appears in the NAT table points to NAT configuration, not routing.

## Next

Lab 5: The new PC that was blocked at the switch
