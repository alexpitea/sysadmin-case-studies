# Mail stopped sending: UDP port exhaustion on a Windows server

## Symptom

In the middle of a working day the business application stopped sending e-mail (offers to customers, notifications). Users saw a generic "failure sending mail". Everything else looked normal: people could send from their mail clients, other systems on the network sent mail through the same mail server, the internet link was up.

The application runs on a Windows server and sends through the company's own mail server.

## Wrong turns

**The mail server.** First guess: the mail server refuses or blocks the application server. Its log showed no attempt at all from that host for the whole afternoon, and the intrusion blocker had not banned it. A sender that never reaches the server cannot be refused by it, so the problem was earlier.

**The internet link.** The external monitoring had reached the public address all afternoon and the router showed no link drops.

**A second address on the network card.** The server had a leftover second IP address with its own gateway, and Windows was choosing it as source address for some traffic. That was a real misconfiguration and I fixed it, but mail still did not go out. It was not the cause.

## Finding the cause

The application's own error, in the Windows event log, was more specific than what the users saw:

```text
SmtpException: Failure sending mail
 ---> SocketException (11001): No such host is known
```

"No such host" is a name resolution failure, not an SMTP failure. On the server:

- `nslookup` to every DNS server timed out
- the same query over TCP worked: `Resolve-DnsName example.com -TcpOnly`

DNS over TCP works, DNS over UDP does not, on one machine only. So the question became: why can this machine not send UDP? Creating a UDP socket by hand gave the answer:

```powershell
New-Object Net.Sockets.UdpClient(0)
# "An operation on a socket could not be performed because the system lacked sufficient buffer space..."
```

That message means there is no free port to give to the socket. The numbers confirmed it:

```powershell
netsh int ipv4 show dynamicport udp      # start 49152, 16384 ports
(Get-NetUDPEndpoint).Count               # 16421

Get-NetUDPEndpoint | Group-Object OwningProcess | Sort-Object Count -Descending |
    Select-Object -First 5 Count, @{ n = 'Process'; e = { (Get-Process -Id $_.Name).ProcessName } }
```

One background service of the business application held 11,319 UDP ports. The DNS service held about 5,000, which is its normal socket pool. Together they filled the dynamic range.

The service had been running since the last restart of the server, two months earlier. That is a leak of roughly 185 ports a day, which reached the limit that morning.

## Fix

- Restarted the leaking service. UDP endpoints dropped from 16,421 to 5,104 and name resolution worked at once. The queued e-mails went out within two minutes.
- Reported the leak to the software vendor.
- Until it is fixed in the product: a scheduled task restarts that one service every three days, at night.

## What I took from it

- Look at where the failure really is before theorising. "The mail server shows no attempt" moved the search from the mail server to the sender in one step.
- Read the exact inner error. The users' message said mail; the inner exception said DNS.
- When DNS fails on one machine only, test UDP and TCP separately.
- Slow leaks show up as sudden outages weeks later, with nothing changed that day. Asking "what changed today?" finds nothing; asking "what has been running the longest?" does.
- Fixing an unrelated misconfiguration along the way is fine, but check afterwards whether the original symptom is gone before calling it solved.
