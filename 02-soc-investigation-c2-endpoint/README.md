# SOC Investigation Report — Endpoint Compromise / Command & Control (C2)

**Ticket ID:** INC-2026-0714-DESKTOPJ4K2 | **Severity:** High | **Classification:** Endpoint compromise / C2 | **Status:** Contained — pending eradication and closure

Simulated Level 1 SOC investigation combining PCAP traffic analysis and host log analysis to confirm and scope a malware infection with active Command & Control communication.

## 1. Executive Summary

Anomalous activity was detected on endpoint **DESKTOP-J4K2** (internal IP 10.4.4.101). Combined PCAP and host log analysis confirmed a compromise: a malicious email attachment triggered the download of an executable from the domain `update-cdn-svc.top`, which then established encrypted (TLS) communication with an external C2 IP, exhibiting periodic beaconing (~60 seconds). The malicious process (PID 5521) also created a scheduled task to maintain persistence across reboots.

Correlating network evidence (PCAP) with host evidence (logs) via the shared PID confirmed the incident end-to-end — from initial delivery to the persistence mechanism — without relying on a single evidence source.

## 2. Scope & Methodology

- **Network traffic analysis (PCAP):** display filters, DNS, HTTP, TCP stream reconstruction, object extraction, beaconing detection
- **Host log analysis:** `/var/log` (syslog, auth.log), advanced `journalctl` (`--since`/`--until`, `-u`, `-p`)
- **Evidence correlation:** PID used as the common identifier linking network and host evidence
- **Tools:** Wireshark (Statistics > Conversations, Follow TCP Stream, Export Objects, display filters), journalctl / Linux system logs

## 3. Incident Timeline

*(Simulated timestamps, 24h format, for training purposes)*

| Time | Event | Source |
|---|---|---|
| 09:14 | User receives and opens a malicious attachment (dropper pattern: doc + exe) | Email / EDR (simulated) |
| 09:15 | Endpoint resolves `update-cdn-svc.top` via DNS | PCAP — DNS |
| 09:15 | Malicious executable downloaded from `update-cdn-svc.top` | PCAP — HTTP / Export Objects |
| 09:16 | Binary executes; process created (PID 5521) | Host log — process creation |
| 09:16 | TLS session established to C2 IP 185.220.101.45 | PCAP — TLS handshake |
| 09:17+ | Periodic beaconing (~60s) to 185.220.101.45 | PCAP — Conversations/TCP |
| 09:19 | Scheduled task created for persistence, spawned by PID 5521 | Host log — command audit |
| 09:31 | Correlation confirmed: same PID in network connection and scheduled task creation | PCAP + log correlation |

## 4. Network Evidence (PCAP)

**DNS:** The endpoint resolves `update-cdn-svc.top` immediately before the binary download. The domain was not previously seen in the environment; the pattern (generic "cdn"-style name, no reputation) is consistent with low-cost malware delivery infrastructure.

**HTTP / Object Extraction:** Using *File > Export Objects > HTTP*, the transferred files are extracted. The pattern matches a classic dropper: a decoy document (e.g., a fake invoice) and an executable served from the same host. Extracted files were not executed — in a real case, their SHA256 hash would be checked against VirusTotal/threat intel before any further action.

**TCP Streams:** *Follow TCP Stream* reconstructs the full conversation. Traffic to `update-cdn-svc.top` during the download is plaintext HTTP (readable); subsequent traffic to the C2 is TLS/HTTPS (metadata visible — SNI, certificate — but payload encrypted).

**Beaconing (C2 detection):** In *Statistics > Conversations > TCP*, sorted by absolute start time, a regular reconnection pattern (~60s, with jitter) to `185.220.101.45` is visible. That regularity, combined with the IP's lack of known reputation, is the key beaconing indicator — unlike periodic traffic to well-known, reputable services.

## 5. Host Evidence (Logs)

- **Process creation:** The host command audit log records execution of the downloaded binary (PID 5521) at a time consistent with the PCAP download (09:16) — functionally equivalent to Windows Event ID 4688 or a Linux `auditd`/`journalctl` process-creation entry.
- **Persistence:** Minutes later, the same PID 5521 is associated with the creation of a scheduled task — ensuring the malware re-executes even after a reboot or if the original process is killed.
- **journalctl filters used:** `--since`/`--until` (narrow the time window around the event), `-u <service>` / `_COMM=<process>` (filter by the suspect process/service), `-p warning` (prioritize higher-severity events), `--no-pager` (required when chaining journalctl in scripts/pipelines).

## 6. Evidence Correlation

Neither the PCAP nor the host log alone proves the full incident: the PCAP shows a suspicious network connection; the log shows a scheduled task created by a process. **PID 5521 is the link** that ties both facts into one confirmed story — the same process that connected to the C2 also installed persistence. This mirrors real SOC correlation logic (e.g., Sysmon Event ID 1 [process creation] + Event ID 3 [network connection] correlated by PID/process GUID in a SIEM).

## 7. Indicators of Compromise (IOC)

| Type | Value | Context |
|---|---|---|
| Delivery domain | `update-cdn-svc.top` | Hosted the malicious executable delivered via decoy document |
| C2 IP | `185.220.101.45` | Destination of TLS handshake and periodic beaconing |
| Affected host | DESKTOP-J4K2 (10.4.4.101) | Endpoint where the binary executed |
| PID | 5521 | Process connecting to C2 and creating the persistence task |
| Network pattern | ~60s beaconing to 185.220.101.45 | Indicator of active C2 communication |
| Persistence mechanism | Scheduled task created by PID 5521 | Ensures reinfection after reboot |

## 8. Impact Assessment

- **Confidentiality:** At risk — an active C2 channel enables data exfiltration.
- **Integrity:** At risk — the attacker has code execution on the endpoint and can modify files/configuration.
- **Availability:** No direct impact observed (not a denial-of-service attack).
- **Scope:** Limited to one endpoint (DESKTOP-J4K2) at time of detection; no evidence of lateral movement in this case.

## 9. Containment & Remediation

1. Isolate DESKTOP-J4K2 from the network (quarantine) to cut C2 communication.
2. Block `update-cdn-svc.top` and `185.220.101.45` at firewall/proxy.
3. Remove the persistence scheduled task tied to PID 5521; check for other persistence mechanisms (registry, services, other tasks).
4. Hash the extracted binary and check it against VirusTotal/threat intel before further analysis.
5. Review whether the endpoint's user reuses credentials elsewhere; force a password reset as a precaution.
6. Hunt for the same IOCs (domain/IP/hash) across the rest of the environment to rule out additional compromised hosts.

## 10. Lessons Learned

- Correlating PCAP and host logs is more reliable than either source alone — document both whenever available.
- The "document + executable from the same host" pattern is a strong dropper signal and should trigger an automated SIEM alert.
- Beaconing is easier to spot via aggregated conversation views (Statistics > Conversations) than packet-by-packet review.
- For production environments, this manual analysis should be translated into Splunk detection rules to avoid depending on manual PCAP review.

## Glossary

- **C2 (Command and Control):** Infrastructure an attacker uses to remotely control a compromised machine — the malware's "remote control."
- **Beaconing:** Regular, periodic connections from a compromised host to its C2, like a repeated "check-in" for new instructions.
- **Dropper:** A program whose only job is to download/install the real malware — the delivery courier, not the payload itself.
- **Persistence:** Techniques ensuring malware stays active after a reboot or process kill — like hiding a spare key to get back in even if the lock changes.
- **PID (Process ID):** A unique identifier the OS assigns to each running process.
- **TLS handshake:** The initial exchange that establishes an encrypted connection between two systems.
- **IOC (Indicator of Compromise):** A concrete artifact (IP, domain, hash, pattern) indicating a system was compromised.

---
*Part of a self-directed 90-day SOC Analyst training plan (Phase 2: Wireshark + Linux Logs). See the [main portfolio](../README.md) for other projects.*
