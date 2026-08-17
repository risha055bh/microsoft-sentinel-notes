# Deploying and Managing a Microsoft Sentinel Workspace

*Workspace architecture, provisioning, cross-tenant management, and access control*

## Introduction

Deploying Microsoft Sentinel isn't just clicking "create" — it means designing a workspace configuration that satisfies your security, compliance, and cost requirements up front, because some of these decisions are hard to reverse later.

Picture this scenario: you're a Security Operations Analyst tasked with standing up Sentinel for your company. The mandate is clear — minimize cost, meet compliance regulations, and keep the environment manageable for the team that'll live in it every day. Before touching the Azure portal, you need to understand the workspace architecture options, decide which one fits, and only then provision your first workspace.

By the end of this post, you'll be able to:

- Describe Microsoft Sentinel workspace architecture
- Onboard a Microsoft Sentinel workspace to Microsoft Defender
- Manage a Microsoft Sentinel workspace in Microsoft Defender
- Understand how roles and permissions control access to it all

## Planning the Workspace

Sentinel itself is installed *into* a Log Analytics workspace — so most of your architectural decisions are really Log Analytics workspace decisions. The single biggest one: **region**, since that determines where your log data physically lives, and workspaces can't be moved between regions after creation.

There are three broad implementation patterns to choose from.

### 1. Single-Tenant, Single Workspace
One central Sentinel workspace ingests logs from every resource across every region in the tenant.

| Pros | Cons |
|---|---|
| Single pane of glass | May not satisfy data governance requirements |
| Consolidates all security logs | Cross-region bandwidth costs |
| Simple to query everything in one place | |
| Azure Log Analytics RBAC for data access | |
| Microsoft Sentinel RBAC for service access | |

### 2. Single-Tenant, Regional Workspaces
Multiple Sentinel/Log Analytics workspaces, one (or more) per region.

| Pros | Cons |
|---|---|
| No cross-region bandwidth cost | No single pane of glass |
| Helps meet data governance requirements | Analytics rules, workbooks, etc. must be deployed to each workspace separately |
| Granular data access control | |
| Granular retention settings per workspace | |
| Billing is split by workspace | |

To query across workspaces in this model, use the `workspace()` function:

```kusto
TableName
| union workspace("WorkspaceName").TableName
```

### 3. Multiple Tenants
If you need to manage a Sentinel workspace that lives outside your own tenant — common for MSSPs — you implement this using **Azure Lighthouse**, which grants cross-tenant access. Within each tenant, the regional-vs-single-workspace decision still applies.

### Sharing a Workspace with Microsoft Defender for Cloud
It's common — and recommended — to use the same Log Analytics workspace for both Sentinel and Microsoft Defender for Cloud, since all of Defender for Cloud's logs become usable by Sentinel too. One catch: **the default workspace Defender for Cloud auto-creates cannot be used for Sentinel.** You need to manually create a workspace first, then repoint Defender for Cloud's tier to use it.

## Creating the Workspace

Once you've settled on an architecture, provisioning is straightforward:

1. In the Azure portal, search for **Sentinel** and select **Microsoft Sentinel**.
2. On the Workspaces list, select **+ Add**.
3. On the "Add Microsoft Sentinel to a workspace" screen, select **+ Create a new workspace**.
4. Fill in the Basics tab:

| Option | What it controls |
|---|---|
| Subscription | Which subscription hosts the workspace |
| Resource Group | New or existing resource group |
| Name | Name of the Log Analytics workspace — this also becomes the Sentinel workspace name by default |
| Region | Where log data is physically stored — **cannot be changed later** |

5. Select **Review + Create**, then **Create**.
6. Back on the "Add Microsoft Sentinel to Workspace" screen, wait for your new workspace to appear (may take a few minutes), select it, and click **Add**.

You'll need **Contributor** permissions on the subscription to enable Sentinel, and **Contributor or Reader** on the resource group to use it day to day.

Once created, Sentinel's left-hand navigation is organized into four areas: **General**, **Threat management**, **Content management**, and **Configuration** — with the Overview tab giving you a dashboard of ingestion, alerts, and incidents at a glance.

## Managing Workspaces Across Tenants

If your responsibilities span multiple workspaces or tenants, you have two tools available:

- **Microsoft Sentinel Workspace Manager** — lets you designate a central workspace that pushes content (analytics rules, workbooks, etc.) out to member workspaces at scale, across one or more tenants. Enabled under Configuration settings.
- **Azure Lighthouse** — grants delegated access into another tenant entirely. Once onboarded, you switch into those subscriptions using the directory + subscription selector in the Azure portal. This is especially useful for service providers managing multiple customer environments without juggling separate logins.

## Managing Workspace Settings

Sentinel settings live in two places: within Sentinel itself (Settings blade — Pricing, Settings, Workspace Settings tabs) and within the underlying Log Analytics workspace, since many configuration tasks (like specific data connector setup) actually redirect you there.

### Log Retention
Retention is configurable from **30 to 730 days (2 years)** at the workspace level, unless you're on the legacy Free tier. To adjust it: go to Workspace Settings in Sentinel → this drops you into the Log Analytics portal → **Usage and estimated costs** → **Retention** button at the top.

## Configuring Logs: Plans and Tiers

This is where cost planning gets real. You need to think in terms of two data states, three table plans, and (currently) three data tiers.

### Two Data States
- **Analytics retention** — "hot" data, available for real-time monitoring, troubleshooting, and analytics
- **Long-term retention** — a low-cost "cold" state (not available on every table plan), accessible via search jobs and restores

### Three Table Plans
| Plan | Best for | Retention window |
|---|---|---|
| **Analytics** | Continuous monitoring, real-time detection, interactive multi-table queries | 30 days – 2 years |
| **Basic** | Troubleshooting and incident response, single-table queries | 30 days |
| **Auxiliary** | Low-touch verbose logs, audit/compliance data | 30 days |

### Three Data Tiers
- **Analytics tier** — powers alerting, hunting, workbooks, and every Sentinel feature. Data lives here "hot" for 30 days by default (extendable to 2 years; Sentinel solution tables extend to 90 days free), and is mirrored into the data lake for up to **12 years of total retention** at low cost.
- **Data lake tier** — a low-cost "cold" tier. Not usable for real-time alerting or hunting, but queryable on demand via KQL jobs, scheduled Spark/KQL jobs, and summary rules.
- **XDR default tier** — Defender XDR's own 30-day hunting retention, included in the XDR license, and not ingested into the analytics or data lake tiers unless you explicitly extend it.

> **Note:** Microsoft is positioning the Sentinel **data lake** as the go-forward option for secondary and long-term storage, offering more scalability and integration. It's currently in public preview, so it's worth tracking as it moves toward general availability.

### Managing Tables in the Defender Portal
For workspaces connected to Defender, table tiering and retention are managed from the Defender portal's table management experience (**Microsoft Sentinel → Configuration → Tables**). A few things to know:

- If you have Basic-tier logs, you must convert them to Analytics tier via the Log Analytics Tables experience before you can manage tiering from Defender.
- Selecting a table opens a details panel showing description, tier, and retention — from there, **Manage table** lets you adjust Analytics retention (30 days–2 years), Total retention (up to 12 years), or switch tiers entirely.
- Not every table can change tiers — XDR and Sentinel solution tables must stay in the Analytics tier since core security features depend on near-real-time access to them.
- Dropping a table from Analytics to Data Lake tier will disable features that depend on it: alerting, advanced hunting, analytics rules, and custom detection rules. Read the warning prompts before confirming.

### Basic Logs and KQL Limits
Not every table supports Basic Logs — only DCR-based custom log tables, `ContainerLogV2`, and `AppTraces` currently qualify. Basic Logs queries are also restricted to a subset of KQL: `where`, `extend`, `project` (and its variants), and `parse`/`parse-where` are supported, but `join`, `union`, and aggregate functions like `summarize` are **not**.

## Understanding Sentinel Permissions and Roles

Once your workspace exists, controlling who can do what inside it comes down to **Azure role-based access control (RBAC)**. Roles can be assigned directly on the Sentinel workspace, or inherited from a subscription or resource group it belongs to.

### Sentinel-Specific Built-In Roles
All grant at least read access; each level adds more:

| Role | View data/incidents | Manage incidents | Create/edit workbooks & analytics rules | Run playbooks |
|---|---|---|---|---|
| **Sentinel Reader** | Yes | No | No | No |
| **Sentinel Responder** | Yes | Yes | No | No |
| **Sentinel Contributor** | Yes | Yes | Yes | No |
| **Sentinel Contributor + Logic App Contributor** | Yes | Yes | Yes | Yes |

There's also **Sentinel Automation Contributor**, which lets Sentinel itself attach playbooks to automation rules — this is a service role, not meant to be assigned to a person.

Best practice: assign these roles at the **resource group** level containing the workspace, so they apply consistently to all supporting resources deployed alongside it.

### Roles for Specific Tasks
A few responsibilities need something beyond the core four roles:

- **Running playbooks (SOAR):** Playbooks are built on Azure Logic Apps — a separate resource type — so team members who need to build or trigger them need the **Logic App Contributor** role.
- **Letting automation rules run playbooks:** Sentinel uses a dedicated service account (not your personal identity) to execute playbooks from automation rules — a deliberate security boundary. Granting that service account permission on the resource group containing the playbooks requires **Owner** rights on your part.
- **Adding data connectors:** Requires write permissions on the Sentinel workspace, plus whatever additional permissions the specific connector's documentation calls for.
- **Guest users assigning incidents:** Beyond the Responder role, guest accounts also need the **Directory Reader** Microsoft Entra role — something regular (non-guest) accounts get by default.
- **Creating/deleting workbooks:** Requires either Sentinel Contributor, or a lesser Sentinel role paired with the Azure Monitor **Workbook Contributor** role. This is only needed to create or delete workbooks — not to use existing ones.

### Broader Azure and Log Analytics Roles
Beyond Sentinel-specific roles, standard Azure roles (**Owner**, **Contributor**, **Reader**) and Log Analytics roles (**Log Analytics Contributor**, **Log Analytics Reader**) also grant access — often more broadly than intended. For example, someone with Sentinel Reader *and* Azure Contributor (not Sentinel Contributor) can still edit Sentinel data, because Azure Contributor grants that regardless of the Sentinel-specific role. If your goal is to scope access tightly to Sentinel alone, you'll need to carefully audit and strip prior broader role assignments — while making sure you don't break something else that depends on them.

### Custom Roles
If the built-in roles don't map cleanly to how your organization splits responsibilities, Azure RBAC supports custom roles, assignable at the management-group, subscription, or resource-group scope — same as the built-ins.

## Wrapping Up

Getting the workspace architecture and access model right at the start pays off — region and tenant decisions are largely permanent, table tiering directly drives your cost curve, and RBAC is what keeps "manageable for the team" from turning into "everyone has Contributor on everything." Treat this planning phase as seriously as you'd treat any other piece of security infrastructure, because in a real sense, it is one.

---

*Next up: connecting data sources and configuring your first analytics rules. Questions or corrections? Open an issue on this repo.*