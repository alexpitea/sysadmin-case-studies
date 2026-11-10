# Moving an office to a filtering DNS resolver, with a way back

## Goal

Put every office computer behind a filtering DNS resolver (ad, tracker and malware domains blocked, with a query log), without an outage and with a quick way to undo it. DNS is the one service where a mistake stops everything for everybody at once.

## Starting point

The router handed out itself and two public resolvers as DNS servers through DHCP. Three networks: wired office, office Wi-Fi and an isolated guest Wi-Fi.

## Plan

1. **One pilot laptop.** A script sets the new resolver as fixed DNS on a single machine, and a second script puts it back on DHCP. A few days of normal work on that laptop showed which blocklists were too aggressive.
2. **Keep a second server.** DHCP hands out the filter first and the router second. If the filter is down, clients fall back to the router after a short delay instead of losing DNS. During that time nothing is filtered; I accepted that in exchange for not depending on one virtual machine.
3. **One network at a time.** Wired office first, office Wi-Fi a few days later. Guests keep public resolvers: they are isolated from the office and their queries do not belong in the office log.
4. **Backup, snapshot, apply, snapshot.** Before every change the router saves a backup and the current DNS and DHCP configuration is written to a file. The change itself is one command, and so is the rollback. The tool is published as [mikrotik-change-tools](https://github.com/alexpitea/mikrotik-change-tools).
5. **Short leases.** The DHCP lease time is 30 minutes, so a rollback reaches every client within half an hour without touching them. `ipconfig /renew` makes it immediate on one machine.

## Details that mattered

- **Internal names.** Some company host names must resolve to internal addresses from inside and public ones from outside. The filter got the same local records the router had, otherwise internal services would have broken the moment clients switched.
- **Who may ask.** The resolver accepts queries only from the two office networks.
- **Names in the log.** Client names are taken from the router's DHCP leases, so the query log shows a computer name instead of only an address.
- **A manual way back.** Next to the snapshots there is a text file with the two router commands that undo the change by hand, in case the administrator's computer is the one without DNS.
- **No forced redirect.** I tried redirecting all outgoing DNS to the filter with a NAT rule, so that devices with hard-coded resolvers would be covered too. It caused more confusion than it solved and was removed; clients use the filter because DHCP tells them to.

## Result

Both office networks were moved with no reported interruption. The rollback was tested on the pilot before the first real change and has not been needed since.

## What I would repeat

- Decide how to go back before deciding how to go forward, and test the way back first.
- Change the smallest unit that proves the idea (one laptop, then one network).
- Write down the old values in the same place as the new ones. The rollback is then the same operation with the other column.
