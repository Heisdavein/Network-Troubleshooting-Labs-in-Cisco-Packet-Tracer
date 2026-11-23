# Lab 5: The New PC That Was Blocked at the Switch

**Scenario:** Port Security violation with the wrong MAC address configured.

## Situation

PC Zero was installed that morning and never worked, and monitoring showed it offline. The rest of the office was fine.

Port Security is a Layer 2 feature that remembers the MAC address of an approved device on a port and blocks different devices.

| Item | Detail |
| --- | --- |
| Network | Office LAN through one switch |
| Problem device | PC Zero on `FastEthernet0/2` |
| IP assignment | DHCP |
| Goal | Restore internet access |

## Investigation

**1. Confirm.** PC1 pinged `1.1.1.2` successfully, so the router, switch and internet were fine. PC Zero had no IP address although it was set to DHCP, and reseating the cable did not help.

**2. Logs.** `show logging` on the switch showed a **Port Security violation on FastEthernet0/2**: the switch had seen an unknown device and disabled the port.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/40d5749f-311c-4fc4-a85e-64ff9c329d44" />
`show logging` with the Port Security violation message for `Fa0/2`.

**3. Port Security status.** `show port-security interface FastEthernet0/2` showed Port Security enabled, violation mode `shutdown`, and the port in a secure shutdown state.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/26071e55-042d-4d21-a362-e5deefef3c8e" />
 `show port-security interface FastEthernet0/2` output.

**4. Compare MAC addresses.** The running configuration contained `switchport port-security mac-address <MAC Address>`, one specific address. `ipconfig /all` on PC Zero showed a different MAC address.


<img width="1671" height="941" alt="image" src="https://github.com/user-attachments/assets/83dae830-f763-491c-8d4e-13bf8ddbb0c7" />
 The configured secure MAC in the running configuration beside PC Zero’s `ipconfig /all` Physical Address.

## Root cause

The switch still held the old computer’s MAC address. When the new PC was connected, it looked unauthorised, and in `shutdown` mode the switch disabled the port.

## Fix

I used **sticky MAC** learning instead of typing the new address:

```
configure terminal
interface FastEthernet0/2
 no switchport port-security mac-address <old_MAC>
 switchport port-security mac-address sticky
 shutdown
 no shutdown
end
copy running-config startup-config
```

| Command | Purpose |
| --- | --- |
| `no switchport port-security mac-address <old_MAC>` | Removes the old address |
| `switchport port-security mac-address sticky` | Switch learns and keeps the connected device’s address |
| `shutdown` / `no shutdown` | Clears the violation shutdown |
| `copy running-config startup-config` | Saves the change |

## Verification

PC Zero immediately received an address by DHCP and `ping 1.1.1.2` returned four replies.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/052bd00b-5a3f-4244-b894-55cca2ad3c6e" />
 PC Zero with a DHCP address and a successful ping to `1.1.1.2`.

## What I learned

- Port Security protects the network but blocks legitimate devices if it is not updated after hardware changes.
- Sticky MAC is easier to manage than typing addresses and still stops unknown devices.
- `show logging` identified the exact port and reason without guessing.

## Next

Lab 6: The redundant path that was never there
