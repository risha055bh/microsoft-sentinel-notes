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

# Exercise — Connect Microsoft Sentinel to Microsoft Defender XDR

![Platform](https://img.shields.io/badge/Platform-Microsoft%20Sentinel%20%7C%20Defender%20XDR-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)
![Type](https://img.shields.io/badge/Type-Hands--On%20Exercise-blue?style=flat-square)
![Duration](https://img.shields.io/badge/Duration-25%20min-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

> **TL;DR** — A guided, simulation-based exercise covering how to onboard both a new and an existing Microsoft Sentinel workspace to Microsoft Defender XDR, and an overview of what Sentinel capabilities become available inside the unified Defender portal once connected.

---
## Table of Contents

- [Scenario](#scenario)
- [Objectives](#objectives)
- [Environment Note](#environment-note)
- [Task 1 — Connect a New Sentinel Workspace to Defender XDR](#task-1--connect-a-new-sentinel-workspace-to-defender-xdr)
- [Task 2 — Connect an Existing Sentinel Workspace to Defender XDR](#task-2--connect-an-existing-sentinel-workspace-to-defender-xdr)
- [Exploring Sentinel Capabilities in the Defender Portal](#exploring-sentinel-capabilities-in-the-defender-portal)
- [Key Takeaways](#key-takeaways)
- [References](#references)

---

## Scenario

You're a Security Operations Analyst working at a company that has deployed both **Microsoft Defender XDR** and **Microsoft Sentinel**. To bring both platforms into a single pane of glass, you need to prepare Microsoft Sentinel for use within the Microsoft Defender portal by connecting it to Defender XDR.

---

## Objectives

In this exercise, Interactive Guides were used to simulate the following tasks:

- Connect a **new** Microsoft Sentinel workspace to Microsoft Defender XDR
- Connect an **existing** Microsoft Sentinel workspace to Microsoft Defender XDR
- Explore the Microsoft Sentinel capabilities available within the Microsoft Defender XDR portal

---

## Environment Note

> ⚠️ The environment for this exercise is a **simulation** generated from the actual product. As a limited simulation:
> - Some links on the page may not be enabled
> - Text-based inputs outside the expected script may not be supported
> - A pop-up may appear stating: *"This feature isn't available within the simulation."* — select **OK** and continue with the exercise steps

---

## Task 1 — Connect a New Sentinel Workspace to Defender XDR

**Duration:** ~10 minutes

In this Interactive Guide, a new Microsoft Sentinel workspace is onboarded to Microsoft Defender XDR. This walks through the initial connection flow an analyst would follow the first time Sentinel is linked to the unified Defender portal for a workspace that has not previously been onboarded.

---

## Task 2 — Connect an Existing Sentinel Workspace to Defender XDR

**Duration:** ~10 minutes

In this Interactive Guide, an **already-existing** Microsoft Sentinel workspace is connected to Microsoft Defender XDR. This covers the onboarding flow for organizations that already have Sentinel deployed and are integrating it into Defender XDR after the fact, rather than starting from a fresh workspace.

---

## Exploring Sentinel Capabilities in the Defender Portal

After completing both onboarding simulations, Sentinel capabilities become available directly inside the Microsoft Defender XDR portal, allowing incidents, hunting, and analytics from Sentinel to be reviewed alongside Defender XDR's native detections in a single interface.

> **Note:** Since this is a simulation, exploration of Sentinel capabilities within the Defender portal is limited. In a real production environment, the full set of Microsoft Sentinel capabilities — including analytics rules, hunting queries, workbooks, and incident management — would be explorable directly from within the Defender portal.

---

## Key Takeaways

| # | Observation | Why It Matters |
|---|-------------|-----------------|
| 1 | **Two distinct onboarding paths exist** | Connecting a brand-new workspace vs. connecting an already-deployed workspace follow different flows — both are relevant depending on an organization's existing maturity with Sentinel. |
| 2 | **Onboarding unifies SecOps into a single portal** | Once connected, Sentinel data and capabilities surface directly inside Defender XDR, reducing the need for analysts to context-switch between two portals. |
| 3 | **Some Sentinel-only capabilities may behave differently post-onboarding** | As covered in related notes on capability differences between the Azure and Defender portals, certain Sentinel features remain Azure-portal-only even after onboarding — this exercise is the connection step, not the full behavioral picture. |
| 4 | **Simulation-based learning mirrors real portal flows** | Despite being a simulation, the interactive guide closely mirrors the real Azure/Defender onboarding experience, making it a reasonable proxy for hands-on familiarity. |

---

## References

- [Connect Microsoft Sentinel to Microsoft Defender XDR](https://learn.microsoft.com/en-us/azure/sentinel/microsoft-365-defender-sentinel-integration)
- [Microsoft Sentinel Documentation](https://learn.microsoft.com/en-us/azure/sentinel/overview)
- [Microsoft Defender XDR Documentation](https://learn.microsoft.com/en-us/microsoft-365/security/defender/microsoft-365-defender)
- [Microsoft Sentinel in the Microsoft Defender Portal](https://learn.microsoft.com/en-us/azure/sentinel/microsoft-sentinel-defender-portal)

---

<div align="center">

**Category:** SIEM / XDR Integration &nbsp;|&nbsp; **Platform:** Microsoft Sentinel & Defender XDR &nbsp;|&nbsp; **Status:** Completed

</div>
---

## Summary

- Microsoft Sentinel can be integrated with Defender XDR either as a **data feed into Sentinel** (Azure portal) or as a **fully unified portal experience** (Defender portal)
- Most capabilities work identically in both portals, but a handful — like Fusion detection, incident tasks, bookmarks in hunting, and multi-workspace management — differ or are portal-exclusive
- Onboarding requires the right RBAC roles, the Defender XDR solution/connector installed, and a single Log Analytics workspace with Sentinel enabled
- Only **one** primary workspace can be connected to the Defender portal per tenant at a time — switching means offboarding the current one first
