# Lab 7: The New Engineer Who Could Not Log In

**Scenario:** AAA with TACACS+, user account missing from the authentication server.

## Situation

A new network engineer, Tony, was given a username and password for SSH access to the CE router, but every login failed. He had also tried Telnet, which failed too. Not every network problem is caused by a router, switch or cable: sometimes the fault is in a supporting service.

| Term | Meaning |
| --- | --- |
| SSH | Encrypted remote access to devices (unlike Telnet) |
| AAA | Authentication (who you are), Authorization (what you may do), Accounting (what you did) |
| TACACS+ | A server that stores accounts and verifies logins, so every device does not need its own user list |

| Item | Detail |
| --- | --- |
| User | Tony, new network engineer (computer CPC3) |
| Router | CE router, `192.168.1.1` |
| Authentication server | TACACS+ at `192.168.1.203` |
| Goal | Tony can log in with SSH |

## Investigation

**1. Confirm.** From Tony’s PC, `ping 192.168.1.1` succeeded, so the network path was fine. `ssh -l tony 192.168.1.1` then failed with `Login Invalid`: an authentication problem, not a connectivity problem.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/51c624f1-fc75-40bc-8d7d-e1dcd0852aa1" />
 Tony’s PC with the successful ping and the failed SSH login (`Login Invalid`).


**2. Test another user.** Another engineer, Alice, ran `ssh -l alice 192.168.1.1` and logged in successfully. The router’s SSH service and the network worked, so the problem affected only Tony’s account.

**3. How the router authenticates.** On the router, `show running-config` showed AAA in use, sending login requests to the TACACS+ server at `192.168.1.203`, with a local fallback administrator account in case the server is unavailable. Since Alice could log in, SSH, AAA and the TACACS+ server connection were all working. The problem was probably on the server.

**4. Check the server.** Its user list showed Alice and Sam, but **no Tony**.

<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/4d2f0aeb-95d2-453f-86e2-bc5e0316f0cb" />
 The TACACS+ server’s user list with Alice and Sam and no entry for Tony.

## Root cause

Tony had been given a username and password, but nobody created his account on the TACACS+ server. The router asked the server to verify him and the server had never heard of him.

## Fix

Created Tony’s account on the TACACS+ server with the correct username and password, and saved. No router change was needed.


<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/ee13c3ef-22d8-4a67-9691-dfc992a723a8" />
 Tony’s successful SSH session to the router.


## What I learned

- Routers and switches depend on supporting services such as TACACS+ and RADIUS (authentication), Syslog (logging), NTP (time) and SNMP (monitoring). A fault in one can make healthy devices look faulty.
- Testing with a second account (Alice) quickly proved the router and SSH service were working and pointed at Tony’s account.
- Account creation belongs in the onboarding checklist: access also needs to exist on the authentication server.
