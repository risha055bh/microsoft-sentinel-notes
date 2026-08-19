# Microsoft Sentinel Automation: Automation Rules & Playbooks

Microsoft Sentinel isn't just a SIEM-it's also a **SOAR** (Security Orchestration, Automation, and Response) platform. Its automation capabilities let SOC teams eliminate repetitive, predictable triage and response work, freeing analysts to focus on real threats.

This post covers the two core building blocks of automation in Sentinel: **Automation Rules** and **Playbooks**, and how to create/manage them across the Azure and Defender portals.

---

## Why Automate?

A common scenario: an analytics rule generates incidents that are consistently **Benign Positive**. Instead of manually closing each one, an automation rule can close them automatically the moment they're created — saving analyst time and keeping the incident queue clean.

---

## Automation Rules vs. Playbooks

| | Automation Rules | Playbooks |
|---|---|---|
| **Purpose** | Simple, centralized incident management (assign, tag, close, change status) | Complex, multi-step response and remediation logic |
| **Built on** | Native Sentinel engine | Azure Logic Apps (Standard or Consumption) |
| **Can call the other?** | ✅ Can trigger playbooks | ❌ Cannot trigger automation rules |
| **Integrates with external systems** | ❌ No | ✅ Yes |
| **Best for** | Triage, tagging, ownership, closing noisy incidents | Enrichment, notifications, ticketing, cross-platform response |

In short: use **automation rules** for quick, no-code incident housekeeping, and **playbooks** when you need to integrate with external systems or run advanced logic. Automation rules can even invoke playbooks as one of their actions — combining both.

---

## Where to Create Automation Rules

Sentinel gives you three entry points, depending on scope:

| Location | Best for |
|---|---|
| **Automation page** (`Configuration → Automation`) | Rules that apply broadly across multiple/future analytics rules. Full configuration control. |
| **Analytics Rule Wizard** (`Automated response` tab) | Rules scoped to the specific analytics rule you're creating/editing. |
| **Incidents blade** (`Create automation rule`) | Quick rule for a single recurring incident — auto-populates conditions/entities from that incident. Ideal for suppression rules. |

> **Note:** Navigation differs by portal. In the **Azure portal**: Sentinel workspace → *Automation* (under Configuration). In the **Defender portal**: *Microsoft Sentinel* → *Automation*.

---

## Anatomy of an Automation Rule

**1. Trigger** — when the rule fires:
- `Incident created` — most common; used for initial triage/assignment/tagging
- `Incident updated` — fires on status, severity, or owner changes; useful for escalation
- `Alert created` — fires on Scheduled/NRT rule alerts; used when incidents aren't created, or when Defender XDR manages incident creation

> In the Defender portal, incident-trigger rules apply to both Sentinel and Defender XDR incidents. Alert-trigger rules apply to Sentinel alerts only.

**2. Conditions** — govern when actions run, using `AND` / `OR` / `NOT` / `CONTAINS` logic based on incident/entity attributes.

**3. Actions** — what the rule does:
- Change incident status (with closing reason + comment)
- Change severity
- Assign an owner
- Add a tag
- Run a playbook (multiple playbooks/actions can be chained in order)

**4. Expiration date** — auto-disables the rule after a set date. Useful for temporary noise suppression (e.g., during penetration testing).

**5. Order** — rules run sequentially; each rule evaluates the incident's state *after* prior rules have acted on it.

> **Example:** If Rule 1 downgrades severity from Medium → Low, and Rule 2 only runs on Medium+ severity, Rule 2 will **not** trigger on that incident.

---

## Quick Reference: Closing Benign Positive Incidents

To auto-close incidents from a known noisy/benign analytics rule:

1. Go to the **Incidents** blade → select a sample incident → **Create automation rule**
2. Confirm the pre-filled analytics rule condition
3. Set action → **Change status** → `Closed`, closing reason `Benign Positive`
4. (Optional) Add a comment and expiration date
5. Save

---

## Summary

- **Automation rules** = centralized, no-code incident management (status, severity, ownership, tagging, order of execution)
- **Playbooks** = Logic Apps-powered workflows for advanced, cross-system response
- Create rules from the **Automation page**, the **Analytics rule wizard**, or the **Incidents blade** — pick the scope that fits your use case
- Combine both for maximum flexibility: automation rules for triage, playbooks (called from rules) for deep remediation
