---
layout: post
title: "SIEM vs SOAR"
date: 2025-04-06 00:00:00 +0100
categories: [hacking, theory]
tags: [siem, soar, defence]
---

![siem-soar](pics/siem-soar.png)


### Security Information and Event Management (SIEM)
SIEM is a cybersecurity solution that collects, analyzes, and correlates log and event data from across an organization’s IT infrastructure to detect suspicious activity and support incident response.

#### Main Features

- Log Collection – Gathers data from servers, firewalls, endpoints, applications, etc.

- Normalization – Converts different data formats into a consistent structure.

- Correlation – Identifies patterns and relationships between events to detect threats.

- Alerting – Notifies security teams about anomalies or potential incidents.

- Dashboards & Reporting – Provides real-time visibility and compliance-ready reports.

- Data Retention – Stores logs for historical analysis and forensic investigations.

#### Popular SIEM Tools

- Splunk

- IBM QRadar

- Microsoft Sentinel

- Elastic SIEM

- LogRhythm

### Security Orchestration, Automation, and Response (SOAR)
SOAR is a category of security tools that enables organizations to automate and orchestrate responses to security alerts by integrating various cybersecurity systems, streamlining workflows, and reducing manual effort in incident response.

#### Main Features

- Orchestration – Connects and coordinates tools like SIEMs, firewalls, endpoint protection, and threat intelligence feeds.

- Automation – Runs predefined playbooks to perform tasks like blocking IPs, isolating endpoints, or gathering threat context—automatically.

- Case Management – Centralizes and manages incident investigations, analyst notes, evidence, and audit trails.

- Threat Intelligence Enrichment – Automatically pulls context from threat feeds, WHOIS, GeoIP, sandboxing tools, etc.

- Collaboration & Reporting – Helps teams communicate and track progress on incidents in real time.

#### Popular SOAR Tools

- TheHive (open-source)

- Palo Alto Cortex XSOAR

- Splunk SOAR

- IBM Resilient

- Swimlane

### SIEM vs SOAR – Quick Comparison Table

| Feature | SIEM | SOAR |
|---------|------|------|
| Focus	| Detection & logging | Response & automation |
| Input	| Logs and events | Alerts (often from SIEMs) |
| Output | Alerts, dashboards | Automated actions, tickets, reports |
| Key Benefit | Visibility & threat detection | Faster, consistent response |

### Typical SOC Workflow: SIEM + SOAR in Action
#### Threat Detection (SIEM)

- Collects logs from endpoints, network devices, cloud services.

- Correlates events like failed logins, suspicious PowerShell, file drops.

- Detects a possible brute-force attack.

- Alert is generated.

#### Automated Triage (SOAR)

- SOAR ingests the SIEM alert.

- Enriches the alert:

  - Checks IP reputation.

  - Queries threat intelligence feeds.

  - Pulls recent user activity.

- Assigns severity & urgency.

#### Automated Response (SOAR)

- Depending on the playbook:

  - Blocks suspicious IP at the firewall.

  - Disables the compromised user account in Active Directory.

  - Sends a Slack/email alert to the security team.

  - Opens a Jira ticket and attaches all artifacts.

#### Analyst Review

- Tier 1 SOC analyst reviews the SOAR case.

- Everything’s already enriched and documented.

- They escalate to Tier 2 if needed or mark resolved.

### Conclusion
- While SIEM and SOAR serve different purposes, they complement each other to create a more efficient, responsive, and scalable security operation.
- SIEM provides the visibility and detection capabilities needed to identify threats, while SOAR enhances those capabilities by automating response, orchestration, and investigation.
- When combined, they reduce response time, lower analyst fatigue, and improve overall incident management.
- For any modern SOC, integrating SIEM and SOAR isn’t just an upgrade—it’s a necessity for staying ahead of today's fast-moving threats.