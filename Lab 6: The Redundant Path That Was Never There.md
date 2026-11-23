# Lab 6: The Redundant Path That Was Never There

**Scenario:** an OSPF area mismatch preventing routers from becoming neighbours.

## Situation

Earlier in the day the West-South link failed, and the entire West building lost connectivity even though a backup path through the North router should have taken over. By the time I started, the link had been repaired. The task was to find out why the backup never worked and make sure it would next time.

OSPF is a dynamic routing protocol. Routers must become **neighbours** before they exchange routing information.

| Item | Detail |
| --- | --- |
| Routers | West, South, North, East |
| Primary path | West to South |
| Backup path | West to North to East |
| Test device | West PC1 |
| Destination | Server `172.16.1.100` |

## Investigation

**1. Current state.** `ping 172.16.1.100` from West PC1 succeeded, confirming the main link was back. The question was why the backup failed.

**2. Neighbours.** `show ip ospf neighbor` on the West router listed only one neighbour. West saw South but not North, even though a cable connected West and North. Without a neighbour relationship there is no routing exchange, so the backup path had never been available.


<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/d35bef7b-c9cb-46f5-ad81-8462ccd28ea6" />
 `show ip ospf neighbor` on West with only the South neighbour.


**3. Logs.** `show logging` on the North router repeatedly reported an **OSPF area mismatch**.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/33755dba-796e-4910-89f1-a5a576dba6b6" />
 `show logging` on North with the area mismatch messages.


**4. Compare configurations.**

```
West:   network 10.1.1.0 0.0.0.3 area 0
North:  network 10.1.1.0 0.0.0.3 area 100
```

Routers on the same link must use the same area number. Area 0 on one side and area 100 on the other means they never become neighbours.

## Root cause

The two routers disagreed on the OSPF area for their shared link. The backup path had been broken from the start and went unnoticed because the main path worked.

## Fix

On the North router:

```
configure terminal
router ospf 1
 no network 10.1.1.0 0.0.0.3 area 100
 network 10.1.1.0 0.0.0.3 area 0
end
```

The routers formed a neighbour relationship almost immediately. `show ip ospf neighbor` on West then listed both South and North.

<img width="1671" height="941" alt="image" src="https://github.com/user-attachments/assets/3d7159b3-fab2-47d4-95a4-1f1313bcbee2" />
`show ip ospf neighbor` on West with South and North both listed.

## Test the backup, not just the fix

I started a continuous ping from West PC1 and, while it ran, disconnected the West-to-South link:

```
ping 172.16.1.100 -t
```

Only one ping packet was lost, then traffic continued through **West, North, East**.

<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/830008fb-7b68-4b3b-a257-e12ea848e41f" />
 The continuous ping with a single timeout followed by replies, with the removed link visible in the topology.


## What I learned

- Redundancy only works when it is configured correctly: a physical backup link is useless if the routers never become neighbours.
- An area mismatch is easy to miss while the main path is healthy.
- Test failover, not just current connectivity.

## Next

Lab 7: The new engineer who could not log in
