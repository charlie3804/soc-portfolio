# Network Attack Analysis — SYN Flood (DoS)

**Type:** Incident report | **Attack:** SYN Flood (Denial of Service) | **Tool referenced:** Packet capture / Wireshark

## Scenario

A company's web server began failing during business hours. An automated monitoring alert flagged connectivity problems, and users attempting to reach the site received connection timeout errors, indicating a possible service disruption.

## Traffic Analysis

Packet capture analysis between clients and the web server revealed a large volume of TCP SYN requests originating from a single unknown IP address, with no completed three-way handshakes. This pattern is characteristic of a **SYN flood** — a Denial of Service (DoS) attack.

## Attack Breakdown: SYN Flood

**What it is:** A SYN flood abuses the TCP three-way handshake. The attacker sends a high volume of SYN requests but never completes the connection (no final ACK).

**Effect on the server:** Each incomplete SYN request occupies a slot in the server's connection table. Once that table fills with bogus half-open connections, the server can no longer accept legitimate new connections.

**Result:** The server becomes saturated and stops responding to legitimate traffic, causing a service outage.

## Impact

| Area | Impact |
|---|---|
| Availability | Website inaccessible to customers and staff |
| Operations | Employees unable to access the sales platform |
| Reputation | Customers may perceive the site as unreliable |
| Revenue | Lost sales opportunities during downtime |

## Response Actions Taken

1. Temporarily took the server offline to prevent further degradation and allow recovery.
2. Blocked the attacking IP address at the firewall.
3. Performed forensic traffic analysis to confirm the attack type.

## Recommendations

- Enable **SYN cookies** on the server so incomplete connections don't consume the connection table.
- Implement **rate limiting** to cap incoming requests from a single source IP.
- Deploy an **IDS/IPS** to detect and automatically respond to anomalous traffic patterns.
- Consider a **cloud-based DoS mitigation service** for large-scale attacks.
- Add continuous traffic monitoring with earlier-warning alerting thresholds.
- Segment the network to isolate critical servers from general traffic.

## Conclusion

The website was the target of a SYN flood (DoS) attack that disrupted service for both customers and staff. Immediate containment (firewall block, temporary server disconnection) restored the system, but stronger preventive controls (SYN cookies, rate limiting, IDS/IPS) are needed to reduce the risk of recurrence.

---
*Part of a self-directed 90-day SOC Analyst training plan. See the [main portfolio](../README.md) for other projects.*
