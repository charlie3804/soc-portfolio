# DNS Service Outage — Traffic Analysis & Incident Report

**Type:** Incident report | **Protocols:** DNS, UDP, ICMP | **Tool:** tcpdump

## Scenario

Multiple users reported being unable to reach `www.yummyrecipesforme.com`, seeing a **"destination port unreachable"** error after the page failed to load. Reproducing the issue confirmed the same error. `tcpdump` was used to capture the traffic generated while attempting to load the page, to determine which protocol and service were actually failing.

## Background: how the page load was supposed to work

1. The browser sends a DNS query over **UDP** to a DNS server to resolve `yummyrecipesforme.com` to an IP address.
2. Once resolved, the browser uses that IP to send an **HTTPS** request to the web server to load the page.

The failure happened at step 1 — the page never got far enough to attempt the HTTPS request.

## Reading the tcpdump log

```
13:24:32.192571 IP 192.51.100.15.51820 > 203.0.113.2.domain: 35084+ A? www.yummyrecipesforme.com. (43)
13:24:32.212455 IP 203.0.113.2 > 192.51.100.15: ICMP 203.0.113.2 udp port 53 unreachable, length 79
[... two more identical request/response pairs ...]
```

| Field | Value | Meaning |
|---|---|---|
| Timestamp | `13:24:32.192571` | 13:24 and 32.192571 seconds — when the request left the host |
| Source → Destination | `192.51.100.15 > 203.0.113.2.domain` | The local host sending a query to the DNS server (`.domain` = port 53) |
| Query ID + flags | `35084+ A?` | Query ID `35084`; `+` means recursion desired; `A?` is a request for an A record (domain name → IPv4 address) |
| Response protocol | `ICMP` | Not a DNS answer — an ICMP error message instead |
| Response source → destination | `203.0.113.2 > 192.51.100.15` | The DNS server's host responding back to the client |
| Error message | `udp port 53 unreachable` | The most important line: something at `203.0.113.2` is telling the network "nothing is listening on UDP port 53" |

The same request/response pair repeats two more times in the log, each with an identical outcome — ruling out a one-off blip in favor of a sustained outage.

## Protocol & service identification

- **Protocol affected:** UDP, specifically the traffic destined for **port 53 — DNS**.
- **What ICMP is doing here:** ICMP isn't the protocol that failed — it's the protocol reporting the failure. When a UDP packet arrives at a host but nothing is listening on the destination port, the receiving host (or a router along the path) sends back an ICMP "destination unreachable / port unreachable" message. ICMP is the network's error-reporting mechanism, not the service that broke.
- **Conclusion:** the DNS service itself — not the network path, not HTTPS, not the web server — is what's down or misconfigured at `203.0.113.2`.

## Impact

Because the DNS query never received a valid answer, the browser had no IP address to send the HTTPS request to. The web server itself may have been perfectly healthy — the entire outage was caused by a single upstream dependency (DNS resolution) failing, which is a useful reminder that "the website is down" and "the DNS service is down" are very different root causes that look identical to an end user.

## Likely Cause

The DNS service on `203.0.113.2` was either stopped, crashed, or misconfigured to not bind to port 53 — or a firewall rule was blocking/redirecting UDP port 53 traffic before it reached the service. An ICMP "port unreachable" specifically (rather than a timeout with no response at all) points toward the host itself actively rejecting the connection, which is more consistent with "no process listening on that port" than with a fully dropped/blackholed connection.

## Recommended Actions

1. Escalate to the network/infrastructure team to verify the DNS service is running and bound to UDP port 53 on `203.0.113.2`.
2. Check firewall/security group rules for that host to rule out an unintended block of inbound UDP 53.
3. Rule out denial-of-service activity or a recent configuration change around the time of the first reports (13:24).
4. Once restored, re-run the same `tcpdump` capture to confirm DNS responses are returning normally before closing the incident.

## Glossary

- **tcpdump:** A command-line packet capture tool — the Linux equivalent of watching Wireshark's packet list in real time, without the GUI.
- **DNS (Domain Name System):** Translates human-readable domain names into IP addresses so browsers know where to send requests.
- **ICMP (Internet Control Message Protocol):** Used for network diagnostics and error reporting — not for carrying application data. A "destination unreachable" message is ICMP telling a sender that something failed to reach its target.
- **A record:** A DNS record type that maps a domain name directly to an IPv4 address.
- **Port unreachable:** A specific ICMP error meaning a packet reached the target host, but no service was listening on the requested port.

---
*Part of a self-directed cybersecurity training program (Google Cybersecurity Professional Certificate). See the [main portfolio](../README.md) for other projects.*
