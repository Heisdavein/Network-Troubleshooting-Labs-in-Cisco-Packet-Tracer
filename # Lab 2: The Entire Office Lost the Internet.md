# Lab 2: The Entire Office Lost the Internet

**Scenario:** a misconfigured ACL blocking all traffic.

## Situation

After lunch the entire office, including the CEO, could not reach websites. A junior administrator had been changing the router earlier while preparing a new server. Success meant users could ping `1.1.1.2`.

## Investigation

**1. Confirm and scope.** From PC0, `ping 1.1.1.2` timed out, and `ping 192.168.1.1` (the default gateway) also timed out. The computers could not even reach the router, so the fault was inside the office network.

**2. Check the PC.** `ipconfig` showed `169.254.x.x`. That is an **APIPA** address, which Windows assigns itself when it cannot reach a DHCP server, so the PCs were not receiving addresses.


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/25c6b50e-86ea-4ee1-a2cb-005024cfd41c" />
`ipconfig` on PC0 showing a `169.254.x.x` address.

**3. Is the router online?** `ping 1.1.1.2` from the router succeeded, so the ISP connection was fine. `show ip interface brief` showed `GigabitEthernet0/0` up, and `show interfaces gig0/0` showed no errors, no dropped packets and no CRC errors, so the physical link was healthy.

**4. Check DHCP.** `show ip dhcp binding` showed no bindings. `show ip dhcp pool` and `show running-config` showed the pool, network, mask and gateway all correct. DHCP was not the problem.

**5. Check ACLs.** `show access-lists` showed two ACLs. The first looked normal. **Access list 2 permitted only `192.168.1.100`.**

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/9d5141f7-bac4-4fb6-afa3-1db9fc556f7b" />
  `show access-lists` showing the standard ACL 2 with a single permitted host.


Every Cisco ACL ends with an invisible **implicit deny**. With only one host permitted, every other device was blocked. `show running-config` showed where it was applied:

```
interface GigabitEthernet0/0
 ip access-group 2 in
```

That checks every packet entering the LAN interface against ACL 2.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/0921541b-33f8-4fbc-92f3-e1dbe8b07e7a" />
 The `GigabitEthernet0/0` section of the running configuration with `ip access-group 2 in`.


## Root cause

An ACL meant for a new server had been applied inbound on the live LAN interface without allowing the rest of the office. DHCP requests, gateway traffic and internet traffic were all dropped.

## Fix

```
interface GigabitEthernet0/0
 no ip access-group 2 in
exit
no ip access-list standard 2
end
write memory
```

The PCs then renewed their network settings and immediately received valid addresses.

## Verification

`ping 1.1.1.2`: all replies successful.

<img width="1671" height="941" alt="image" src="https://github.com/user-attachments/assets/61ce3ef2-01e0-473f-9c0c-986b20f1528f" />
  A PC with a `192.168.1.x` address and a successful ping to `1.1.1.2`.


## What I learned

- A single configuration mistake can stop an entire network.
- Every ACL has an implicit “deny all” at the end.
- Verify what changed. **Change management** (planning, reviewing, testing and approving changes) exists to prevent exactly this.
