# SIEM vs SOAR

## SIEM
- **SIEM (Security Information and Event Management)** tells us what suspicious activity happened.
- It focuses on **monitoring, detection, and alerting**.

## SOAR
- **SOAR = Security Orchestration, Automation, and Response**
- Helps us **respond quickly and consistently** to incidents.

### What SOAR Does
- Automates repetitive SOC 
- Orchestrates actions across tools
- Executes response playbooks
- Reduces response time (MTTR)

### Typical SOAR Actions
- Disable compromised user
- Block IP on firewall
- Isolate endpoint
- Quarantine phishing email
- Create ServiceNow/Jira ticket
- Notify SOC teams

### How SOAR Works
Incident → Playbook → Automated Actions → Closure

### Examples of SOAR Platforms
- Azure Logic Apps (used by Sentinel)
- Cortex XSOAR
- Splunk SOAR

### Key Question SOAR Answers
- “How fast can we respond and contain?”
