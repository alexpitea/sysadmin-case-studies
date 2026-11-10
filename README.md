# Sysadmin case studies

Short write-ups of problems I diagnosed and changes I rolled out. Cases 1 to 4 come from administering a small office network (about forty users, one site, MikroTik router, Windows and Linux servers). Cases 5 and 6 come from software work on a building-automation product.

They are anonymised: no company or product names, and all addresses are placeholders. What is kept is the part that is useful to someone else: the symptoms, the wrong turns, how the cause was found and what I would do again.

| # | Case | Topic |
|---|---|---|
| 1 | [Mail stopped sending: UDP port exhaustion](01-mail-outage-udp-port-exhaustion.md) | Windows Server, DNS, sockets |
| 2 | [One wrong password locked mail for the whole Wi-Fi](02-fail2ban-behind-nat.md) | Fail2Ban, NAT, IMAP |
| 3 | [Moving an office to a filtering DNS resolver, with a way back](03-dns-filter-rollout.md) | DNS, DHCP, change management |
| 4 | [A monitoring stack for a small network](04-network-monitoring-stack.md) | SNMP, NetFlow, syslog, SIEM |
| 5 | [Halving the CPU use of a Python service on a small ARM board](05-python-service-cpu-profiling.md) | Python, profiling, embedded Linux |
| 6 | [Updating devices in the field without bricking them](06-staged-device-updates.md) | release process, staged rollout |

Related code: [mikrotik-change-tools](https://github.com/alexpitea/mikrotik-change-tools) is the tool used for case 3 and for the router side of case 4.
