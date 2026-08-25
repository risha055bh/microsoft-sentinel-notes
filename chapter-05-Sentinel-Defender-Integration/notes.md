# Integrate Microsoft Sentinel with Microsoft Defender XDR

Microsoft Sentinel is now generally available in the Microsoft Defender portal. This integration simplifies SecOps by reducing the need to manage multiple tools and enhances hunting by letting you query across different data sets from a single portal.

---

## Introduction

**Scenario:** You're a Security Operations Manager at a large multinational corporation, tasked with streamlining your team's operations and enhancing hunting capabilities. Your team currently juggles multiple tools for different tasks, which leads to complexity and errors. You also want to:

- Enable automatic **attack disruption for SAP** applications
- Provide **unified entity pages** for devices, users, IP addresses, and Azure resources

The solution: integrate Microsoft Sentinel into the Microsoft Defender portal.

> **Note**
> If you onboard Microsoft Sentinel to the Defender portal and are licensed for Defender XDR, Microsoft Sentinel is automatically connected to Defender XDR — the Defender XDR data connector is set up for you automatically.

### Topics covered in this module

- Understanding the integration of Microsoft Sentinel and Defender XDR in the Defender portal
- Understanding capability differences between Azure and Defender portals in Microsoft Sentinel
- Onboarding Microsoft Sentinel to Microsoft Defender XDR: prerequisites and steps
- Navigating Microsoft Sentinel features in the Defender portal

### Learning objectives

By the end of this module, you're able to:

- Understand the differences between Microsoft Sentinel capabilities in Azure and Defender portals
- Know the prerequisites for integrating Microsoft Defender XDR with Microsoft Sentinel
- Connect a Microsoft Sentinel workspace to Microsoft Defender XDR

### Prerequisites

- Familiarity with security operations in an organization
- Experience installing and configuring Microsoft Sentinel services
- Experience using and navigating the Microsoft Defender portal

---

## Benefits of Integrating Microsoft Sentinel with Defender XDR

There are two ways to integrate Microsoft Sentinel with Microsoft Defender XDR services:

| Method | What it does | How it's enabled |
|---|---|---|
| **Data ingestion only** | Ingest Defender XDR service data into Microsoft Sentinel and view Sentinel data in the **Azure portal** | Install the Defender XDR connector in Microsoft Sentinel |
| **Unified portal** | View Microsoft Sentinel data directly in the **Microsoft Defender portal**, alongside your Defender incidents, alerts, vulnerabilities, and other security data | Install the Defender XDR connector in Microsoft Sentinel, then onboard Microsoft Sentinel to the unified operations platform in the Defender portal |

### How the integration flows

- Insights from signals across your entire organization feed into **Microsoft Defender XDR** and **Microsoft Defender for Cloud**
- Defender XDR and Defender for Cloud send SIEM log data through **Microsoft Sentinel connectors**
- SecOps teams analyze and respond to threats identified in Sentinel and Defender XDR
- Microsoft Sentinel supports **multicloud environments** and integrates with third-party apps and partners

### Azure portal 

<img width="951" height="556" alt="image" src="https://github.com/user-attachments/assets/a08dcc02-26e0-4ff7-a3ee-74ffdf7c7e8a" />

### Defender Portal 

<img width="951" height="548" alt="image" src="https://github.com/user-attachments/assets/6f59dfb0-527e-4c9f-893a-31305188c7de" />

---

## Capability Differences Between Azure and Defender Portals

Most Microsoft Sentinel capabilities are available in **both** the Azure and Defender portals. In the Defender portal, some Sentinel experiences simply open out to the Azure portal to complete a task.

The table below lists the capabilities that are only available in **one** portal, or that behave significantly differently between the two — excluding experiences that just redirect from Defender to Azure.

| Capability | Availability | Details |
|---|---|---|
| **Advanced hunting using bookmarks** | Azure portal only | Bookmarks aren't supported in advanced hunting in the Defender portal. In Defender, they're supported under *Microsoft Sentinel → Threat management → Hunting*. |
| **Attack disruption for SAP** | Defender portal only | Not available in the Azure portal. |
| **Automation** | Mixed | Some automation procedures differ between workspaces onboarded to Defender vs. those that aren't; others behave the same in both portals. |
| **Data connector visibility (Defender-related connectors)** | Azure portal only | After onboarding, the Defender portal hides these connectors from the Data connectors page: Defender for Cloud Apps, Defender for Endpoint, Defender for Identity, Defender for Office 365 (Preview), Defender XDR, Subscription-based Defender for Cloud (Legacy), Tenant-based Defender for Cloud (Preview). They remain listed in the Azure portal. |
| **Add entities to threat intelligence from incidents** | Azure portal only | Not available in the Defender portal. |
| **Fusion: advanced multistage attack detection** | Azure portal only | The Fusion analytics rule is disabled once Sentinel is onboarded to Defender — Defender XDR's own incident-creation and correlation replaces it. |
| **Adding/removing alerts from incidents** | Defender portal only | Once onboarded, you can no longer add or remove alerts from incidents in the Azure portal. In Defender, you can only "remove" an alert by linking it to another (existing or new) incident. |
| **Editing incident comments** | Azure portal only | Comments can be added in either portal, but not edited afterward. Edits made in Azure don't sync to Defender. |
| **Programmatic / manual incident creation** | Azure portal only | Incidents created via API, Logic App playbook, or manually in the Azure portal don't sync to Defender — but remain fully supported in Azure/API. |
| **Reopening closed incidents** | Azure portal only | In Defender, analytics-rule alert grouping can't be set to reopen closed incidents when new alerts arrive — new alerts trigger new incidents instead. |
| **Incident tasks** | Azure portal only | Unavailable in the Defender portal. |
| **Multiple workspace management** | Split | **Defender portal:** limited to one primary Sentinel workspace. **Azure portal:** centrally manage multiple Sentinel workspaces across tenants. |

> **Note**
> Only **one** primary Microsoft Sentinel workspace can be connected per tenant in the Microsoft Defender portal.

---

## Onboarding Microsoft Sentinel to Microsoft Defender XDR

### Prerequisites

The Defender portal supports a single Microsoft Entra tenant, connected to one primary workspace plus multiple secondary workspaces. (A "workspace" here = a Log Analytics workspace with Microsoft Sentinel enabled.)

To onboard and use Microsoft Sentinel in the Defender portal, you need:

- A Log Analytics workspace with Microsoft Sentinel enabled
- The **Microsoft Defender XDR** data connector (formerly *Microsoft 365 Defender*) enabled in Sentinel for incidents and alerts
  <img width="1898" height="871" alt="image" src="https://github.com/user-attachments/assets/c11beb71-c0a3-44cf-8dba-dfe1b1119bea" />
- Access to Microsoft Defender XDR in the Defender portal
- Microsoft Defender XDR onboarded to the Microsoft Entra tenant
- An Azure account with the right roles (see below)

#### Required roles

| Task | Azure built-in role required | Scope |
|---|---|---|
| Connect/disconnect a Sentinel-enabled workspace | *Owner* **or** *User Access Administrator* + *Microsoft Sentinel Contributor* | Subscription (Owner/UAA); subscription, resource group, or workspace (Sentinel Contributor) |
| View Microsoft Sentinel in the Defender portal | *Microsoft Sentinel Reader* | Subscription, resource group, or workspace |
| Query Sentinel data tables / view incidents | *Microsoft Sentinel Reader* or custom role with workspace read/query and incident/comment/relation/task read actions | Subscription, resource group, or workspace |
| Take investigative actions on incidents | *Microsoft Sentinel Contributor* or custom role with incident/comment/relation/task read+write actions | Subscription, resource group, or workspace |
| Create a support request | *Owner*, *Contributor*, *Support Request Contributor*, or custom role with `Microsoft.Support/*` | Subscription |

> **Note**
> Once Sentinel is connected to the Defender portal, your existing Azure RBAC permissions carry over. Continue managing roles from the **Azure portal** — changes reflect automatically in Defender.

### Onboard Microsoft Sentinel

> **Important**
> Before connecting a workspace, install the **Microsoft Defender XDR solution** for Microsoft Sentinel from the Content hub, and enable the **Microsoft Defender XDR data connector** to collect incidents and alerts.

1. Go to the **Microsoft Defender portal** and sign in
2. In Defender XDR, select **Home (Overview)**
3. In the *"Get your SIEM and XDR in one place"* banner, select **Connect a workspace**
4. Choose the workspace to connect, then select **Next**
5. Review the product changes this will cause:
   - Log tables, queries, and functions from the Sentinel workspace become available in **advanced hunting** within Defender XDR
   - The **Microsoft Sentinel Contributor** role is assigned to the *Microsoft Threat Protection* and *WindowsDefenderATP* apps within the subscription
   - Active Microsoft security **incident creation rules are deactivated** to avoid duplicates (applies only to Microsoft-alert incident creation rules, not other analytics rules)
   - All Defender XDR product alerts stream directly from the main Defender XDR data connector for consistency — make sure incidents/alerts from this connector are turned on
6. Select **Connect**

Once connected, the Home page banner confirms your unified SIEM + XDR setup is ready, and new sections appear with Sentinel metrics (data connectors, automation rules, etc.).

<img width="1856" height="917" alt="image" src="https://github.com/user-attachments/assets/f8444c4e-e07b-4041-99a4-e5ca28dbb2b7" />

### Offboard Microsoft Sentinel

Only one workspace can be connected to the Defender portal at a time. To switch workspaces, disconnect the current one first.

1. Go to the **Microsoft Defender portal** and sign in
2. Under **System**, select **Settings → Microsoft Sentinel**
3. On the **Workspaces** page, select the connected workspace → **Disconnect workspace**
4. Provide a reason for disconnecting
5. Confirm

Once disconnected, the *Microsoft Sentinel* section disappears from the Defender portal's left navigation, and Sentinel data no longer appears on the Overview page. To connect a different workspace, go back to the **Workspaces** page and select **Connect a workspace**.

# Connecting Microsoft Sentinel to Microsoft Defender XDR

Microsoft Sentinel and Microsoft Defender XDR can be connected so that both platforms operate from a **single, unified portal**. Once connected, Sentinel incidents, analytics, and hunting capabilities surface directly inside the Defender portal alongside Defender XDR's native detections — reducing the need for analysts to switch between two separate portals during investigations.

This post covers how to connect both a **new** and an **existing** Sentinel workspace to Defender XDR, based on a hands-on Microsoft Learn simulation exercise.

---

## Scenario

You're a Security Operations Analyst at a company that has deployed both Microsoft Defender XDR and Microsoft Sentinel. To bring both into a unified SOC experience, Sentinel needs to be prepared for use inside the Microsoft Defender portal by connecting it to Defender XDR.

---

## Why Connect Sentinel to Defender XDR?

Running Sentinel and Defender XDR side by side without connecting them means analysts have to work across two separate portals — one for Sentinel's SIEM capabilities, and one for Defender XDR's native XDR detections. Connecting the two brings incident management, alerts, and select Sentinel capabilities into the same Defender portal experience, streamlining triage and investigation.

There are two onboarding paths, depending on where an organization currently stands:

| Scenario | What It Covers |
|---|---|
| **New Sentinel workspace** | Onboarding a workspace that has not been connected to anything before |
| **Existing Sentinel workspace** | Connecting a workspace that's already deployed and actively used in the Azure portal |

---

## Task 1: Connect a New Sentinel Workspace to Defender XDR

This flow covers onboarding a **brand-new** Sentinel workspace directly into Defender XDR. It's the path used when Sentinel hasn't previously existed in the environment, or the workspace being connected has never been onboarded before.

**Approximate time:** 10 minutes

---

## Task 2: Connect an Existing Sentinel Workspace to Defender XDR

This flow covers connecting a Sentinel workspace that is **already deployed and in use** — the more common real-world scenario, since most organizations adopt Sentinel before later integrating it with Defender XDR.

**Approximate time:** 10 minutes

---

## Exploring Sentinel Capabilities in the Defender Portal

Once onboarding is complete for either path, Sentinel's capabilities — including incidents, alerts, and select analytics — become accessible directly from within the Microsoft Defender portal.

> **Note:** Not all Sentinel capabilities behave identically once inside the Defender portal. As covered separately in *"Capability Differences Between Azure and Defender Portals,"* certain Sentinel experiences (e.g., advanced hunting with bookmarks, programmatic incident creation, editing incident comments) remain Azure-portal-only or behave differently post-onboarding.

---

## Environment Note

> ⚠️ This exercise is based on a **simulation** environment generated from the actual product:
> - Some links on the page may not be enabled
> - Text-based inputs outside the expected script may not be supported
> - A pop-up may appear stating: *"This feature isn't available within the simulation."* — select **OK** and continue

---

## Summary

- Sentinel can be connected to Defender XDR through **two onboarding paths**: new workspace or existing workspace
- Once connected, Sentinel incidents and select capabilities surface directly inside the **unified Defender portal**
- Connecting the two reduces portal-switching for analysts and centralizes SOC operations
- Not every Sentinel capability behaves identically after onboarding — some remain Azure-portal-only (see capability differences notes)
- This walkthrough was completed via Microsoft Learn's Interactive Guide simulation, which closely mirrors the real onboarding experience in both portals

---

## References

- [Connect Microsoft Sentinel to Microsoft Defender XDR](https://learn.microsoft.com/en-us/azure/sentinel/microsoft-365-defender-sentinel-integration)
- [Microsoft Sentinel in the Microsoft Defender Portal](https://learn.microsoft.com/en-us/azure/sentinel/microsoft-sentinel-defender-portal)
- [Microsoft Sentinel Documentation](https://learn.microsoft.com/en-us/azure/sentinel/overview)
- [Microsoft Defender XDR Documentation](https://learn.microsoft.com/en-us/microsoft-365/security/defender/microsoft-365-defender)

## Summary

- Microsoft Sentinel can be integrated with Defender XDR either as a **data feed into Sentinel** (Azure portal) or as a **fully unified portal experience** (Defender portal)
- Most capabilities work identically in both portals, but a handful — like Fusion detection, incident tasks, bookmarks in hunting, and multi-workspace management — differ or are portal-exclusive
- Onboarding requires the right RBAC roles, the Defender XDR solution/connector installed, and a single Log Analytics workspace with Sentinel enabled
- Only **one** primary workspace can be connected to the Defender portal per tenant at a time — switching means offboarding the current one first
