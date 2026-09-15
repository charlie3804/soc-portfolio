# Security Controls & Compliance Assessment — Botium Toys

**Type:** Governance, Risk & Compliance (GRC) assessment | **Framework:** NIST CSF | **Regulations reviewed:** PCI DSS, GDPR, SOC | **Risk score:** 8/10 (High)

## Scenario

Botium Toys is a small, fast-growing toy retailer with both a physical store and an e-commerce site. Rapid growth outpaced the IT department's ability to formalize security controls. This assessment scopes the entire security program, inventories current assets, evaluates existing controls against a least-privilege and defense-in-depth standard, and checks compliance posture against three major regulatory frameworks: PCI DSS (payment card data), GDPR (EU customer data), and SOC (organizational trust criteria).

## Scope & Goals

**Scope:** The entire Botium Toys security program — all assets, plus the internal processes and procedures tied to implementing controls and compliance.

**Goal:** Assess existing assets and complete a controls-and-compliance review to determine what needs to be implemented to improve the company's security posture.

## Asset Inventory

Assets managed by the IT department include on-premises office equipment, employee end-user devices (desktops, laptops, smartphones, remote workstations), storefront and warehouse retail inventory, business systems (accounting, telecom, database, e-commerce, inventory management), internet access, the internal network, data retention/storage, and legacy systems requiring manual monitoring.

## Risk Assessment

**Risk score: 8/10 (High).** Botium Toys has inadequate asset management and does not fully adhere to U.S. and international compliance standards. The IT department cannot currently say with confidence which assets would be impacted by a given incident, which is itself part of the risk — you can't protect, or prioritize recovery of, what you haven't inventoried and classified.

**NIST CSF alignment:** the first function of the NIST Cybersecurity Framework is **Identify**. Before Botium Toys can meaningfully improve protection, detection, or response, it needs to dedicate resources to identifying and classifying its assets and determining the business-continuity impact if each were lost.

## Controls Assessment

| Control | In place? | Notes |
|---|---|---|
| Least privilege | No | All employees currently have access to internally stored data, including cardholder data and customer PII/SPII |
| Separation of duties | No | No enforced division of responsibilities |
| Encryption | No | Credit card data is accepted, processed, transmitted, and stored locally **without encryption** |
| Password policy | Partial | A policy exists but requirements are weak — not aligned with current minimum complexity standards (8+ characters, mixed case, number, special character) |
| Centralized password management | No | No system enforces the policy; resets go through manual IT tickets |
| Intrusion detection system (IDS) | No | Not installed |
| Backups | No | No backups of critical data |
| Disaster recovery plan | No | None in place |
| Manual monitoring of legacy systems | Partial | Systems are monitored, but with no regular schedule or clear intervention process |
| Firewall | Yes | Configured with defined security rules |
| Antivirus software | Yes | Installed and monitored regularly |
| Physical security (locks, CCTV, fire detection) | Yes | Offices, storefront, and warehouse are adequately covered |
| Breach notification process (GDPR 72-hour rule) | Yes | Plan exists to notify EU customers within 72 hours of a breach |

**Read:** the technical/physical basics (firewall, AV, physical security) are covered. The gaps are concentrated exactly where the highest-impact risks live — access control, encryption, and recoverability.

## Compliance Assessment

### PCI DSS (Payment Card Industry Data Security Standard)

| Best practice | Met? |
|---|---|
| Only authorized users can access cardholder data | No |
| Cardholder data stored/processed/transmitted in a secure environment | No |
| Encryption applied to credit card transaction data | No |
| Secure password management policy adopted | No |

**Status: Non-compliant.** This is the single largest exposure — Botium Toys processes credit card data without encryption and without restricted access, which is close to a worst-case PCI DSS posture.

### GDPR (General Data Protection Regulation)

| Best practice | Met? |
|---|---|
| EU customer data kept private/secure | Yes |
| 72-hour breach notification plan in place | Yes |
| Data properly classified and inventoried | No |
| Privacy policies/procedures enforced | Yes |

**Status: Partially compliant.** Process and notification commitments exist, but they're undermined by the same root problem as the controls gap: no formal data classification/inventory.

### SOC (System and Organization Controls)

| Best practice | Met? |
|---|---|
| User access policies established | No |
| Sensitive data (PII/SPII) kept confidential | No |
| Data integrity maintained | Yes |
| Data available to authorized users | Yes |

**Status: Partially compliant.** Availability and integrity are handled; confidentiality and access governance are not.

## Control Categories Reference

Controls fall into three categories, each achieving prevention, detection, correction, or deterrence differently:

| Category | Addresses | Examples |
|---|---|---|
| **Administrative/Managerial** | The human element — policy, roles, responsibilities | Least privilege, password policy, separation of duties, disaster recovery plans |
| **Technical** | Systems and software | Firewall, IDS/IPS, antivirus, encryption, backups |
| **Physical/Operational** | Physical access to assets | Locks, CCTV, badge readers, fire detection |

And by function: **preventative** (stop an incident before it happens — least privilege, firewall), **detective** (determine an incident occurred — IDS, CCTV), **corrective** (restore after an incident — backups, disaster recovery, antivirus), **deterrent** (discourage the attempt — encryption, visible alarm signage, locks).

## Recommendations

1. **Encrypt cardholder data** in transit and at rest — this single control closes the largest PCI DSS gap.
2. **Implement least privilege and separation of duties** so employee access to PII/SPII and cardholder data is scoped to what each role actually needs.
3. **Deploy an IDS** to detect anomalous traffic rather than relying on the firewall alone.
4. **Establish backups and a disaster recovery plan** — currently a single incident could mean unrecoverable data loss.
5. **Strengthen the password policy** to current minimum-complexity standards and adopt a centralized password management system.
6. **Formally inventory and classify data and assets** — this is the prerequisite for nearly every other gap above (GDPR classification requirement, risk-based prioritization, and knowing what's actually at stake if an asset is lost).

## Conclusion

Botium Toys' physical and baseline technical controls (firewall, antivirus, physical security) are solid, but the controls that matter most for the data the company actually handles — customer payment information — are largely missing. The risk score of 8/10 reflects a company with the right instincts on physical security but no formal data governance program, which is precisely the gap the NIST CSF "Identify" function exists to close.

---
*Part of a self-directed cybersecurity training program (Google Cybersecurity Professional Certificate — Botium Toys case study). See the [main portfolio](../README.md) for other projects.*
