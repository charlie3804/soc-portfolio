# Network Reconnaissance & Asset Discovery — Nmap

**Type:** Network security assessment | **Tool:** Nmap | **Focus:** Service discovery, exposure risk, hardening recommendations

## Scenario

A small business network (subnet `192.168.10.0/24`) had never been formally inventoried. Before onboarding a centralized logging/SIEM pipeline, the security team needed a baseline: which hosts are alive, what services are exposed, and whether any of them present immediate risk. This is a standard defensive use case for Nmap — not offensive reconnaissance, but the same technique used to build an asset inventory and catch configuration drift.

## Methodology

```
nmap -sn 192.168.10.0/24                     # host discovery (ping sweep)
nmap -sV -sC -p- 192.168.10.15,22,40         # full port range + service/version detection + default scripts
```

- `-sn`: discovers which hosts in the range are up, without scanning ports (fast first pass).
- `-sV`: probes open ports to identify the service and version running behind them — critical for spotting outdated software.
- `-sC`: runs Nmap's default script set (banner grabbing, basic vulnerability checks).
- `-p-`: scans all 65535 ports instead of the default top 1000, to avoid missing services on non-standard ports.

## Findings

| Host | Port | Service | Version | Risk |
|---|---|---|---|---|
| 192.168.10.15 | 21/tcp | FTP | vsftpd 2.3.4 | **Critical** — this specific version has a publicly known backdoor (CVE-2011-2523) allowing remote command execution |
| 192.168.10.22 | 3389/tcp | RDP | Microsoft Terminal Services | **High** — Remote Desktop exposed to the entire internal subnet with no access restriction or MFA observed |
| 192.168.10.40 | 445/tcp | SMB | SMBv1 enabled | **High** — SMBv1 is the protocol version exploited by EternalBlue (MS17-010) / WannaCry |
| 192.168.10.40 | 139/tcp | NetBIOS | — | Medium — legacy protocol, unnecessary exposure, should be disabled if unused |
| 192.168.10.8 | 80/tcp, 443/tcp | HTTP/HTTPS | nginx 1.24 | Low — current version, no known CVEs at scan time |
| 192.168.10.5 | 22/tcp | SSH | OpenSSH 9.2 | Low — current version; confirm password auth is disabled in favor of key-based auth |

## Risk Assessment

The two critical/high findings — the vsftpd 2.3.4 backdoor and unrestricted SMBv1/RDP exposure — are both the kind of finding a SOC analyst is trained to recognize instantly by version number alone, because they map to some of the most exploited vulnerabilities in recent history. Left unaddressed, either one gives an attacker a direct path to remote code execution without needing to phish anyone first.

## Recommendations

1. Patch or replace the vsftpd 2.3.4 service immediately; if FTP is required, migrate to SFTP/FTPS.
2. Disable SMBv1 across the network and enforce SMBv3 with signing.
3. Restrict RDP (3389) to a VPN or jump host — never expose it flat across the internal subnet, and enable MFA.
4. Disable NetBIOS (139) on hosts that don't require legacy Windows file-sharing compatibility.
5. Re-run the scan after remediation to confirm the exposure is closed, and add these ports/services to recurring scan monitoring so drift is caught early next time.

## Conclusion

A baseline Nmap sweep of a previously un-inventoried subnet surfaced two high-severity exposures (a backdoored FTP version and unrestricted legacy protocols) that would otherwise have gone unnoticed until exploited. This is the core value of asset discovery in a SOC context: most of what gets exploited isn't a zero-day, it's a known, patchable issue nobody had visibility into.

## Glossary

- **Nmap:** Network Mapper — the standard tool for discovering hosts and services on a network by sending crafted packets and interpreting the responses.
- **CVE:** Common Vulnerabilities and Exposures — a public, numbered catalog of known security flaws, used as a shared reference between vendors, researchers, and defenders.
- **EternalBlue:** An exploit targeting a flaw in SMBv1 (MS17-010), used by the WannaCry and NotPetya ransomware outbreaks — one of the most cited examples of why disabling SMBv1 matters.
- **Service/version detection:** Identifying not just that a port is open, but exactly what software and version is listening on it — the detail that turns "port 21 is open" into "port 21 is running a version with a known backdoor."

---
*Part of a self-directed 90-day SOC Analyst training plan (Phase 1: Networking + Nmap). See the [main portfolio](../README.md) for other projects.*
