# Introduction to Microsoft Defender XDR Threat Protection

> Source: [Introduction to Microsoft Defender XDR threat protection](https://learn.microsoft.com/en-us/training/modules/introduction-microsoft-365-threat-protection/)

---

## Introduction

Microsoft Defender XDR is an integrated threat protection suite with solutions that detect malicious activity across **email, endpoints, applications, and identity**. Together, these solutions provide a complete attack-chain compromise story, giving a full understanding of the threat and enabling remediation and protection against future attacks.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/a7128ab0-7aee-4bfa-9cd4-a753db964563" />

**Scenario:** You're a Security Operations Analyst at a company implementing Microsoft Defender XDR solutions. You need to understand how Extended Detection and Response (XDR) combines signals from:

- Endpoints
- Identity
- Email
- Applications

...to detect and mitigate threats.

---

## Explore Extended Detection & Response (XDR) Response Use Cases

### Detection of Threat

**Scenario:** Microsoft Defender for Endpoint (MDE) detects a malicious payload — regardless of where it came from (personal email, USB drive, etc.).

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/5d18136f-74f4-4f8f-bdba-17e5ef88f85b" />

1. The victim receives a malicious email on a personal account *not* protected by Microsoft Defender for Office 365 (MDO), or plugs in an infected USB drive, and opens the attachment.
2. The malware infects the computer — the user is unaware an attack occurred.
3. **MDE detects the attack**, raises an alert to security operations, and provides threat details to the security team.
4. **Access is disabled:** MDE tells Intune the endpoint's risk level has changed. An Intune Compliance Policy configured with an MDE risk-level severity triggers, marking the device as noncompliant.
5. **Conditional Access** (via Microsoft Entra ID) then blocks the user's access to apps.

### Remediation

- MDE remediates the threat — via automated remediation, analyst-approved automated remediation, or manual analyst investigation.
- MDE also contributes threat intelligence about the attack to **Microsoft Threat Intelligence**, helping remediate the same threat across the broader MDE customer base.

### Share Intelligence and Restore Access

- **Restore access:** Once infected devices are remediated, MDE signals Intune to update the device risk status, and Conditional Access restores access to enterprise resources.
- **Remediate threat variants elsewhere:** Threat signals in Microsoft Threat Intelligence are used by other Microsoft tools — MDO and Microsoft Defender for Cloud use these signals to detect and remediate threats across email, Office collaboration, Azure, and more.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/3dc9c308-0a53-44f3-a0ab-cb808a870e0b" />

#### Access Restricted

While a device is compromised:
- Conditional Access knows about the device risk because MDE notified Intune, which updated the device's compliance status in Microsoft Entra ID.
- The user is blocked from all new resource requests and any existing access that supports Continuous Access Evaluation (CAE).
- The user *can* still do general, non-corporate tasks (e.g., browsing YouTube, Wikipedia) but has no access to corporate resources.

#### Access Restored

Once the threat is remediated and cleaned up:
- MDE triggers Intune to update Microsoft Entra ID.
- Conditional Access restores the user's access to corporate resources.

> This approach mitigates organizational risk (blocking attackers from accessing corporate resources via a compromised device) while minimizing disruption to user productivity.

---

## Understand Microsoft Defender XDR in a Security Operations Center (SOC)

Overview of how Microsoft Defender XDR and Microsoft Sentinel integrate within a modern Security Operations Center (SOC).

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/fdbdb39c-351f-4b3b-ab98-f739b718d7a3" />

### Security Operations Model — Functions and Tools

Security operations are composed of several distinct functions. In larger organizations these are separate teams; in smaller ones, they're often combined into a single role/team (e.g., handled by IT Operations, or temporarily by leadership for incident management).

> **Note**
> Analysts are referred to by team name, not "Tier" numbers — each team has unique specialized skills; the tiers aren't a literal hierarchy of value.

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/d444064e-7076-4f65-aa78-f9c404b14004" />


#### Triage and Automation

- **Automation** — Near real-time resolution of well-known, previously-seen incident types.
- **Triage (aka Tier 1)** — Rapid remediation of high-volume, well-known incidents that still need quick human judgment. Often approves automated remediation workflows and escalates anomalies to Investigation (Tier 2).

**Key learnings for Triage and Automation:**

- **90% true positive** — Recommended quality bar for any alert feed requiring analyst response, to avoid alert fatigue from false positives.
- **Alert ratio** — In Microsoft's Cyber Defense Operations Center, XDR alerts produce most high-quality alerts; the rest come from user reports, classic log-query alerts, and other sources.
- **Automation** empowers triage analysts and reduces manual effort (e.g., automated investigation with a human-review step before approving remediation).
- **Tool integration** — Consolidating XDR tools into a single console (Microsoft Defender XDR) for endpoint, email, identity, etc. was one of the biggest time-savers in Microsoft's CDOC, enabling rapid cleanup of phishing emails, malware, and compromised accounts.
- **Focus** — Triage teams keep a narrow technical focus (e.g., user productivity, email, endpoint AV alerts) rather than covering everything, to maintain speed.

#### Investigation and Incident Management (Tier 2)

- Escalation point for issues from Triage (Tier 1).
- Directly monitors alerts indicating sophisticated attackers — behavioral alerts, business-critical asset alerts, ongoing campaign monitoring.
- Proactively reviews the Triage queue and hunts using XDR tools when time allows.
- Handles deeper investigation of complex, often multi-stage, human-operated attacks.
- Pilots new/unfamiliar alert types (e.g., Microsoft Defender for Cloud alerts on VMs, containers, Kubernetes, SQL databases) to document processes for Triage and automation.
- **Incident Management** — Handles the non-technical side: coordination with communications, legal, leadership, and other stakeholders.

#### Hunt and Incident Management (Tier 3)

A multi-disciplinary team focused on catching attackers who slipped past reactive detection, and handling major business-impacting events.

- **Hunt** — Proactively hunts for undetected threats, assists with escalations and advanced forensics, and refines alerts/automation. Operates in a hypothesis-driven model rather than a reactive one — this is where red/purple teams connect with SecOps.

#### How It Comes Together — Example Incident Lifecycle

1. A **Triage (Tier 1)** analyst claims a malware alert and investigates (e.g., via the Defender XDR console).
2. The analyst determines the malware needs more advanced remediation (e.g., device isolation), and escalates to **Investigation (Tier 2)**, which takes the lead. Triage can stay involved to learn (Investigation may use Microsoft Sentinel or another SIEM for broader context).
3. **Investigation** verifies (or digs deeper into) the conclusions, remediates, and closes the case.
4. Later, **Hunt (Tier 3)** reviews closed incidents looking for:
   - Detections eligible for auto-remediation
   - Multiple similar incidents with a common root cause
   - Other process/tool/alert improvement opportunities

   > **Example:** Tier 3 reviewed a case and found the user had fallen for a tech scam — flagged as higher priority since the scammer had gained admin-level access on the endpoint, indicating higher risk exposure.

#### Threat Intelligence

Threat Intelligence teams provide context and insight to support every other function (often via a Threat Intelligence Platform, or TIP, in larger orgs), including:

- Reactive technical research for active incidents
- Proactive research into attacker groups, attack trends, high-profile attacks, and emerging techniques
- Strategic analysis and insights to inform business/technical priorities
- ...and more

---

## Explore Microsoft Security Graph

Microsoft Graph provides a unified programmability model for accessing data across Microsoft 365, Windows, and Enterprise Mobility + Security — used to build customized organizational apps.

The Microsoft Graph API offers a single endpoint, `https://graph.microsoft.com` (v1.0 or beta). You can use REST APIs or SDKs to build apps supporting Microsoft 365 scenarios. It also includes services managing user/device identity, access, compliance, and security to help protect against data leakage or loss.

### What's in Microsoft Graph?

- **Microsoft 365 core services:** Bookings, Calendar, Excel, Microsoft Lists, Microsoft Purview eDiscovery, Microsoft Search, OneDrive, OneNote, Outlook/Exchange, People (Outlook contacts), Planner, SharePoint, Teams, To Do, Viva Insights
- **Enterprise Mobility + Security services:** Microsoft Defender for Identity, Microsoft Entra ID, Intune
- **Windows services:** Activities, devices, notifications, Universal Print
- **Dynamics 365 Business Central services**

### Microsoft Graph Security API

An intermediary service (broker) providing a single programmatic interface to connect multiple Microsoft Graph **security providers**. Requests are federated to all applicable providers, and results are aggregated and returned in a common schema.

<img width="614" height="720" alt="image" src="https://github.com/user-attachments/assets/537bee9d-1b25-4439-be99-123cd4833bd4" />


Developers can use it to build intelligent security services that:

- Integrate and correlate security alerts from multiple sources
- Stream alerts to SIEM solutions
- Automatically send threat indicators to Microsoft security solutions to trigger alert, block, or allow actions
- Unlock contextual data to inform investigations
- Discover opportunities to learn from data and train security solutions
- Automate SecOps for greater efficiency

### Use the Microsoft Graph Security API

Two versions are available:

- **Microsoft Graph REST API v1.0**
- **Microsoft Graph REST API Beta** — includes new/enhanced APIs still in preview; subject to change and may break existing scenarios without notice.

For Security Operations Analysts, both versions support **advanced hunting** via the `runHuntingQuery` method, using **Kusto Query Language (KQL)**.

**Advanced hunting example (Microsoft Defender XDR):**

```http
POST https://graph.microsoft.com/v1.0/security/runHuntingQuery

{
    "Query": "DeviceProcessEvents | where InitiatingProcessFileName =~ \"powershell.exe\" | project Timestamp, FileName, InitiatingProcessFileName | order by Timestamp desc | limit 2"
}
```

You can use [Graph Explorer](https://developer.microsoft.com/graph/graph-explorer) to run hunting queries interactively.

<img width="2738" height="1487" alt="image" src="https://github.com/user-attachments/assets/121e9bdb-935b-48e3-843b-a7b72ea484f5" />

## Investigate Security Incidents in Microsoft Defender XDR

A cloud guide demonstrates how Microsoft Defender XDR and Microsoft Sentinel work together to investigate a security incident in a **hybrid environment**.

🔗 [Investigate security incidents in a hybrid environment](https://mslearn.cloudguides.com/guides/Investigate%20security%20incidents%20in%20a%20hybrid%20environment%20with%20Azure%20Sentinel)

---

## Summary

- Microsoft Defender XDR unifies detection and response across **endpoints, identity, email, and applications**, giving a full attack-chain story instead of isolated alerts.
- A real-world XDR flow ties together **MDE → Intune → Microsoft Entra Conditional Access** to detect, isolate, remediate, and restore access automatically.
- Modern SOCs are organized into **Triage/Automation (Tier 1)**, **Investigation (Tier 2)**, and **Hunt (Tier 3)**, supported by a dedicated **Threat Intelligence** function.
- **Microsoft Graph** (and specifically the **Microsoft Graph Security API**) provides a single programmatic interface to correlate alerts across security providers and run KQL-based hunting queries via `runHuntingQuery`.
- Microsoft Defender XDR and Microsoft Sentinel together support investigating incidents across **hybrid environments**.
