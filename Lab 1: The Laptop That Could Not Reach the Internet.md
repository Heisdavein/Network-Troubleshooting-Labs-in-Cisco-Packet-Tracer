# Lab 1: The Laptop That Could Not Reach the Internet


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/870fa9fd-7b26-432b-9a6b-81cbbb1b9e60" />
incorrect static IP address (wrong network).

## Situation

A user from another office brought Laptop Zero to the head office, and it could not access the internet. Success meant the laptop could ping `1.1.1.2`, which represents the internet.

| Item | Detail |
| --- | --- |
| Network | `192.168.1.0/24` |
| Default gateway | `192.168.1.1` |
| Path | Router, switch, PCs and Laptop Zero |
| Internet test | `1.1.1.2` |
| Problem device | Laptop Zero |

## Investigation

**1. Confirm the problem.** `ping 1.1.1.2` from Laptop Zero: every request timed out.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/bdcf379c-dc6d-4054-8214-40e3278dbd12" />
 Laptop Zero Command Prompt with the failed ping to `1.1.1.2`.


**2. Check the scope.** The same ping from PC1 succeeded, so the router, switch and internet connection were all working. Only Laptop Zero had a problem.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/a9237b0a-244e-47c4-8cb0-f48c05c6781c" />
 PC1 pinging `1.1.1.2` successfully.


**3. OSI layers.**

- **Layer 1:** `show ip interface brief` on the switch showed the laptop’s port (`FastEthernet0/2`) up, so the cable and port were fine.
- **Layer 2:** `show mac address-table` was initially sparse. After another ping from the laptop the switch had learned its MAC address on the correct port.
- **Layer 3:** `ipconfig` on the laptop showed an address beginning `10.1.1.x`, while every other device used `192.168.1.x`. PC1’s `ipconfig` showed a correct `192.168.1.0/24` address.


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/994c550e-9a3c-42bf-a116-9f0d564d03ed" />
`show mac address-table` with the laptop’s MAC address learned on `Fa0/2`.


<img width="1774" height="887" alt="image" src="https://github.com/user-attachments/assets/b0f5f757-8260-4d75-9f46-89ac51886cb7" />
`ipconfig` on Laptop Zero showing a `10.1.1.x` address and PC1 showing `192.168.1.x`.


## Root cause

The laptop still had the static IP address from the user’s regular office (`10.1.1.0` network). It could not communicate with the head-office gateway `192.168.1.1`.

## Fix

Changed the laptop from a static address to **DHCP**. It immediately received an address in `192.168.1.x`. See `configs/fix-commands.txt`.

## Verification
`ping 1.1.1.2` again: four successful replies.


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/1c1d5507-03b1-4c30-81e5-54bef6ed4380" />
Laptop Zero `ipconfig` showing a `192.168.1.x` address and the successful ping.

## What I learned

Do not assume a problem is complicated: start with the basics. This is a common real-world fault, especially when users move between sites, and DHCP avoids it by giving each network the right settings.

## Next

Lab 2: The entire office lost the internet
