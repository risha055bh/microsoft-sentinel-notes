# Connect Common Event Format Logs to Microsoft Sentinel

Many on-premises network appliances — firewalls, proxies, IDS/IPS devices — send log data in **Common Event Format (CEF)**, a structured Syslog-based format. Microsoft Sentinel can ingest this data using the **Common Event Format (CEF) via AMA** connector, which relies on a dedicated Linux log forwarder running the Azure Monitor Agent (AMA).

This post covers the deployment options for the CEF connector and how to run through the setup, based on a hands-on Microsoft Learn module.

---

## Scenario

You're a Security Operations Analyst at a company that has implemented Microsoft Sentinel. You need to collect log data from on-premises network appliances. Since the appliances provide their data in a structured format, the **Common Event Format connector** is the right fit.

The overall flow:
1. Install an on-premises **Linux host** to act as a log forwarder
2. Follow the CEF connector page's instructions to configure the Linux host
3. Configure the network appliances to forward their logs to the Linux host
4. The Linux host then forwards those logs on to the Microsoft Sentinel workspace

By the end of this module, CEF log data can be sent to a Microsoft Sentinel workspace using the provided data connector.

**Learning objectives:**
- Explain the Common Event Format connector deployment options in Microsoft Sentinel
- Configure the CEF via AMA connector on a Linux log forwarder

**Prerequisite:** Basic knowledge of operational concepts such as monitoring, logging, and alerting.

---

## Planning for the CEF Connector

<img width="1233" height="621" alt="image" src="https://github.com/user-attachments/assets/cf02df39-cae2-4aea-82be-d0fa4a2cc9fd" />


The **CEF via AMA** connector uses the **Azure Monitor Agent (AMA)** on a dedicated Linux log forwarder to bridge communication between network appliances and Microsoft Sentinel. Many vendor-specific Sentinel data connectors use CEF via AMA as their underlying collection mechanism.

<img width="1232" height="618" alt="image" src="https://github.com/user-attachments/assets/2ac94ea0-6a4d-41b2-a028-50a56e0033c3" />


### Deployment Topology

- **Linux VM in Azure as the forwarder:** On-premises Syslog sources securely send events to the Azure Linux VM. AMA on that VM then forwards the logs to the Sentinel workspace.
- **VM in another cloud, or an on-premises machine, as the forwarder:** Syslog sources securely send events to the Linux VM with AMA installed, which then securely forwards the logs to the Sentinel workspace.

Either way, the Linux forwarder sits between the network appliances and Sentinel, acting as the collection and forwarding point.

### Security Considerations

- Configure the forwarder machine's security according to your organization's security policy (e.g., align network configuration with corporate network security policy, adjust the daemon's ports and protocols as needed).
- To use **TLS** communication between the Syslog source and the Syslog forwarder, configure the Syslog daemon (`rsyslog` or `syslog-ng`) to communicate over TLS.

### Prerequisites

**Supported 64-bit operating systems** (AMA supports 64-bit only):

| OS | Supported Versions |
|---|---|
| Amazon Linux | 2, 2023 |
| Oracle Linux | 8, 9 |
| Red Hat Enterprise Linux (RHEL) Server | 8, 9 |
| Debian GNU/Linux | 10, 11, 12 |
| Ubuntu Linux | 20.04 LTS, 22.04 LTS, 24.04 LTS |
| SUSE Linux Enterprise Server | 15 |

**Supported Syslog daemon versions:**

| Daemon | Version |
|---|---|
| syslog-ng | 2.1 – 3.22.1 |
| rsyslog | v8 |

**Supported Syslog RFCs:** RFC 3164, RFC 5424

**Permissions:** Elevated (`sudo`) access is required on the forwarder machine.

---

## Connecting Your External Solution Using the CEF Connector

A Linux machine needs to be designated and configured to forward logs from the security solution to the Microsoft Sentinel workspace. This machine can be physical or virtual, hosted on-premises, in Azure, or in another cloud.

The **CEF via AMA** connector uses a **Data Collection Rule (DCR)** to configure AMA on the Linux forwarder. AMA handles both agent installation and log forwarding — **no manual script installation is required**.

Once configured, AMA on the Linux forwarder:
- Listens for CEF messages from the built-in Linux Syslog daemon on **TCP port 514**
- Forwards CEF messages to the Microsoft Sentinel workspace, where they're parsed and enriched

### Setup Steps

1. In the **Microsoft Sentinel** portal, select **Data connectors**.
2. Search for and select **Common Event Format (CEF) via AMA**.
3. Select **Open connector page** on the details pane.

<img width="1683" height="849" alt="image" src="https://github.com/user-attachments/assets/daf46cd3-1396-4b5d-96b8-39578af94330" />

5. Under **Configuration**, select **+ Create data collection rule**.
7. On the **Basic** tab, enter a name for the data collection rule, then select the subscription and resource group.
8. On the **Resources** tab, select the Linux machine designated as the log forwarder.
9. On the **Collect** tab, confirm the CEF facility and log level settings.
10. Select **Review + create**, then **Create**. AMA is automatically installed on the Linux forwarder if it isn't already present.
11. Configure each network appliance to forward its syslog events to the Linux forwarder on **UDP or TCP port 514**.

<img width="1302" height="679" alt="image" src="https://github.com/user-attachments/assets/053202cf-a552-4c7f-8c4a-9f0995c376d8" />

### Forwarding Both Plain Syslog and CEF from the Same Machine

If the same log forwarder machine will be used to forward both plain Syslog messages **and** CEF messages, edit the Syslog configuration file on each source machine that sends logs in CEF format — remove the facilities used to send CEF messages. This avoids **duplicate events** landing in both the `Syslog` and `CommonSecurityLog` tables.

---

## Summary

- The **CEF via AMA** connector lets Microsoft Sentinel ingest structured log data (Common Event Format) from on-premises network appliances via a dedicated Linux forwarder
- AMA (Azure Monitor Agent) handles both installation and forwarding — the older manual deployment script is no longer required
- The forwarder can be an Azure VM, a VM in another cloud, or an on-premises machine
- A **Data Collection Rule (DCR)** is what ties the Linux forwarder's AMA configuration to the Sentinel workspace
- AMA listens on **TCP 514**; network appliances forward to the Linux host on **UDP or TCP 514**
- If forwarding both plain Syslog and CEF from the same machine, remove CEF-sending facilities from the plain Syslog config to avoid duplicate events in `Syslog` and `CommonSecurityLog`

---

## References

- [Connect Common Event Format Logs to Microsoft Sentinel (Microsoft Learn)](https://learn.microsoft.com/en-us/training/modules/connect-common-event-format-logs-to-azure-sentinel/)
- [Ingest Syslog and CEF Messages to Microsoft Sentinel with the Azure Monitor Agent](https://learn.microsoft.com/en-us/azure/sentinel/connect-cef-syslog-ama)
- [Azure Monitor Agent Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/agents-overview)
- [Data Collection Rules in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-collection-rule-overview)
