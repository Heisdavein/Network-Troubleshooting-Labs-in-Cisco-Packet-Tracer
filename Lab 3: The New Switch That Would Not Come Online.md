# Lab 3: The New Switch That Would Not Come Online

**Scenario:** BPDU Guard shutting down the switch connection.

## Situation

The company added Switch 2 to the existing Switch 1. As soon as the cable was connected, the link between the switches refused to come up, and every computer on Switch 2 failed to get an IP address or reach the network.

## Investigation

**1. Confirm and scope.** PC5, PC6 and PC7 on Switch 2 had no automatic address. PC0 on Switch 1 reached its gateway and the network normally. The fault was limited to the new switch or the link between them.

**2. Interfaces.** `show ip interface brief` on Switch 2: the PC ports were up, but the uplink `GigabitEthernet0/1` to Switch 1 was down. Re-checking the configuration, reseating the cable and trying another port changed nothing.


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/0c292126-77bb-40ca-ab51-f38bae4e5064" />
 `show ip interface brief` on Switch 2 with `Gi0/1` down.


**3. Logs.** On Switch 1, warning messages appeared, and `show logging` showed the switch had received a BPDU and **BPDU Guard had automatically shut down the port**.


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/d88d1127-d069-41bb-8051-7d82e3c3d9af" />
 `show logging` on Switch 1 with the BPDU Guard message naming the port.


**4. Configuration.** The configuration contained:

```
spanning-tree portfast bpduguard default
```

## Root cause

| Term | Meaning |
| --- | --- |
| BPDU | Message switches send each other for Spanning Tree |
| PortFast | Lets a port to an end device forward immediately |
| BPDU Guard | Shuts down a PortFast port that receives BPDUs, assuming an unexpected switch was connected |
| Access port | Carries one VLAN, for computers |
| Trunk port | Carries multiple VLANs, for switch-to-switch links |

The link between Switch 1 and Switch 2 had been configured as an **access port** with PortFast and BPDU Guard. When Switch 2 was plugged in and sent BPDUs, BPDU Guard shut the port down (error-disabled). The security feature worked exactly as designed; the port type was wrong for the connection.

## Fix

I kept BPDU Guard and corrected the link by making it a trunk. On Switch 1, then the same on Switch 2:

```
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 1
 shutdown
 no shutdown
end
write
```

| Command | Purpose |
| --- | --- |
| `switchport mode trunk` | Carries traffic between switches |
| `switchport trunk native vlan 1` | VLAN 1 for untagged traffic |
| `switchport trunk allowed vlan 1` | VLAN 1 may use the trunk |
| `shutdown` / `no shutdown` | Resets the port after BPDU Guard error-disabled it |

## Verification

`show ip interface brief` showed `GigabitEthernet0/1` up on both switches. `show interfaces GigabitEthernet0/1 switchport` showed the port operating as a trunk using 802.1Q. PC5 received an address by DHCP, `ping 192.168.1.1` succeeded, and so did `ping 1.1.1.2`.


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/4c5f6758-1d38-447c-a523-39953207263a" />
`show interfaces GigabitEthernet0/1 switchport` showing trunk mode and 802.1Q.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/c821b860-86f9-4127-8a0f-4aceebda0c8c" />
 PC5 with a DHCP address and successful pings to `192.168.1.1` and `1.1.1.2`.

## What I learned

- Do not remove a security feature just because it blocked something. Understand why it triggered, then configure the network correctly.
- PortFast belongs on end-device ports only.
- Switches connect to switches with trunk ports.

## Next

Lab 4: The server VLAN that could not reach the internet
