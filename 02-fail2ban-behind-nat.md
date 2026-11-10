# One wrong password locked mail for the whole Wi-Fi

## Symptom

After a power cut, nobody on the office Wi-Fi could read e-mail. Sending worked, the web worked, and the same laptops read mail normally when plugged into a cable.

## Narrowing it down

I connected a laptop to the Wi-Fi and tested the mail server port by port:

| Port | Service | Result from Wi-Fi |
|---|---|---|
| 143, 993 | IMAP | no answer (packets dropped) |
| 110, 995 | POP3 | no answer (packets dropped) |
| 465, 587 | mail submission | open |
| 443 | webmail | open |

From a wired computer all of them were open. From the same Wi-Fi, IMAP to an external provider worked, so the Wi-Fi itself was not filtering IMAP. A full-size ping with "do not fragment" passed, so it was not an MTU problem.

Only the ports for *reading* mail are dropped, only for one group of clients, and silently. That is exactly what an intrusion blocker does: Fail2Ban has one jail for failed IMAP and POP logins, and a ban from that jail drops those ports for the banned address.

## Cause

The router translates the addresses of Wi-Fi clients when they talk to the mail server, so the mail server sees the entire Wi-Fi network as a single address: the router's.

One device kept trying to log in with an old password after the power came back. Fail2Ban did its job and banned the address those attempts came from. That address was the whole Wi-Fi.

## Fix

- Removed the ban.
- Added the router's Wi-Fi-side address to Fail2Ban's trusted list, so one misconfigured phone can no longer lock out everyone.
- Tested from the Wi-Fi again: all four ports open, IMAP greeting received.

## The trade-off

Trusting that address means Fail2Ban no longer protects the mail server from password guessing that comes from the office Wi-Fi. For an internal network that is acceptable; for a guest network it would not be, and guests are on a separate, isolated network for that reason.

The cleaner fix is to stop translating internal traffic to the server, so it sees the real address of each client and can ban one device instead of a whole network. That needs a change in the router's NAT rules and in how internal names resolve, and is planned separately.

It is also why I could not say which device had the wrong password: in the mail server's log every attempt came from the same address.

## What I took from it

- A selective failure is a strong hint. "Reading fails, sending works" pointed at one specific jail before I had opened any log.
- Any protection that works per address (bans, rate limits) acts on everything behind a NAT as if it were one client. When adding NAT, check what counts addresses on the other side.
- Test from where the users are. From my wired desk everything was fine.
