# Splunk SOC & Detection Engineering Home Lab

**A hands-on cybersecurity lab focused on SIEM administration, centralized log collection, threat detection, threat hunting, and incident investigation.**

![Splunk](https://img.shields.io/badge/SIEM-Splunk-black?logo=splunk)
![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Windows-blue)
![Focus](https://img.shields.io/badge/Focus-SOC%20%7C%20Blue%20Team-red)
![Status](https://img.shields.io/badge/Status-In%20Progress-orange)

---

## About This Project

This repository documents my journey of building a practical Security Operations Center (SOC) home lab using Splunk Enterprise.

The project is designed around a 60-day, task-by-task roadmap covering the skills required for SOC Level 1 and Level 2 operations, detection engineering, threat hunting, and Splunk administration.

Rather than focusing only on installing tools, the objective is to understand how security telemetry is collected, processed, searched, correlated, and investigated.

The lab includes Windows and Linux endpoints, centralized log collection, network monitoring, threat intelligence, cloud security, detection development, and incident response exercises.

Each phase includes technical configurations, SPL searches, screenshots, troubleshooting notes, and repeatable tests wherever applicable.

## Project Objectives

- Build and maintain a functional SIEM environment using Splunk Enterprise.
- Centralize Windows and Linux security logs.
- Understand Splunk Universal Forwarder and HTTP Event Collector.
- Develop strong Search Processing Language (SPL) skills.
- Monitor endpoint activity using Sysmon and Windows Event Logs.
- Integrate network telemetry from Suricata and Zeek.
- Develop detections based on MITRE ATT&CK techniques.
- Perform threat intelligence enrichment and automation.
- Integrate AWS CloudTrail for cloud security monitoring.
- Build security dashboards, alerts, and detection rules.
- Investigate simulated security incidents.
- Develop practical Splunk administration and troubleshooting skills.

## Lab Architecture

The environment is built around a virtualization-based home lab.

### Infrastructure

| Component | Operating System / Tool | Purpose |
|---|---|---|
| Host machine | EndeavourOS | Main virtualization host |
| SIEM server | Ubuntu Server | Splunk Enterprise |
| Windows endpoint | Windows 11 | Windows event collection and endpoint monitoring |
| Linux endpoint | Kali Linux | Linux log collection and security testing |
| Network monitoring | Suricata | Network threat detection |
| Firewall | pfSense | Network traffic control and monitoring |
| Additional monitoring | Wazuh | Endpoint security monitoring |
| Log collection | Splunk Universal Forwarder | Endpoint log forwarding |
| Event ingestion | Splunk HEC | HTTP-based event ingestion |

Some components are already part of my existing home lab, while their integration into the Splunk environment is being developed progressively.

### Architecture Diagram

```text
                  ENDEAVOUROS HOST
                         |
                  VirtualBox / VMs
                         |
        +----------------+----------------+
        |                |                |
   Ubuntu Server      Windows 11       Kali Linux
   Splunk Enterprise  UF + Sysmon      UF + rsyslog
        |                |                |
        +----------------+----------------+
                         |
                  Splunk Receiving
                    TCP 9997
                         |
                SPLUNK ENTERPRISE
                         |
        +----------------+----------------+
        |                |                |
     win_sec         win_sysmon         linux
        |                |                |
        +----------------+----------------+
                         |
                  SPL Searches
                         |
             Dashboards / Alerts / Hunting

       Additional planned integrations:
       Suricata | Zeek | pfSense | AWS
       Threat Intelligence | SOAR
```

The diagram represents the intended overall architecture. Individual integrations will be marked complete only after they are configured and verified.

## Technology Stack

| Category | Technologies |
|---|---|
| SIEM | Splunk Enterprise |
| Operating Systems | EndeavourOS, Ubuntu Server, Windows 11, Kali Linux |
| Log Collection | Universal Forwarder, rsyslog |
| Event Ingestion | HTTP Event Collector |
| Windows Monitoring | Windows Event Logs, PowerShell Logging, Sysmon |
| Network Security | Suricata, Zeek, pfSense |
| Endpoint Monitoring | Wazuh |
| Threat Intelligence | VirusTotal, AbuseIPDB, AlienVault OTX, URLhaus |
| Cloud Security | AWS CloudTrail, VPC Flow Logs, GuardDuty |
| Detection Engineering | SPL, Sigma, MITRE ATT&CK |
| Automation | Python, SOAR |
| Virtualization | VirtualBox |

## Splunk Index Design

The lab uses separate indexes to organize telemetry by source and purpose.

| Index | Description |
|---|---|
| `win_sec` | Windows Security, System, and Application events |
| `win_sysmon` | Windows Sysmon and designated PowerShell telemetry |
| `linux` | Linux authentication and system logs |
| `network` | Network monitoring and related events |
| `cloud` | Cloud security telemetry |
| `threat_intel` | Threat intelligence data |

Initial retention target: 90 days, subject to storage limits and index configuration.

## Project Roadmap

The project is divided into nine phases.

### Phase 0 — Lab Preparation

**Goal:** Prepare the infrastructure and documentation before full SIEM implementation.

Tasks:
- Prepare virtualization environment.
- Configure Ubuntu Server, Windows, and Kali VMs.
- Establish VM networking and static addressing.
- Document Windows Event IDs and Sysmon Event IDs.
- Study MITRE ATT&CK.
- Prepare lab architecture documentation.

**Deliverables:** Architecture diagram, VM specifications, network documentation, and security cheat sheets.

### Phase 1 — Splunk Installation & Log Ingestion

**Goal:** Build the centralized log collection infrastructure.

Tasks:
- Install Splunk Enterprise on Ubuntu Server.
- Configure indexes and data retention.
- Install Universal Forwarder on Windows.
- Collect Windows Security, System, and Application logs.
- Configure Linux Universal Forwarder.
- Collect Linux authentication and system logs.
- Enable PowerShell Script Block Logging (Event ID 4104).
- Configure HEC using curl.
- Configure rsyslog forwarding.
- Troubleshoot sourcetype and event line-breaking issues.

**Deliverables:** Splunk configurations, ingestion health checks, troubleshooting notes, and log collection evidence.

### Phase 2 — SPL Mastery

**Goal:** Develop the ability to search, transform, analyze, and correlate security events.

Topics:
- search, where, table, fields, sort, dedup, rename
- stats, chart, timechart, top, rare
- eval, rex, case, if, coalesce
- Lookups and subsearches
- Join, append, transaction
- Eventstats and streamstats
- tstats, data models, and CIM
- Macros, saved searches, tags, and event types
- Boss of the SOC (BOTS) exercises

**Deliverables:** SPL cheat sheet, practical searches, BOTS write-ups, and performance comparisons.

### Phase 3 — Sysmon & Endpoint Detection

**Goal:** Develop endpoint monitoring and detection capabilities.

Tasks:
- Install Sysmon on Windows.
- Configure Sysmon logging.
- Integrate Sysmon with Splunk.
- Study relevant Windows Event IDs.
- Perform controlled Atomic Red Team tests.
- Investigate credential dumping, PowerShell activity, persistence, LOLBins, and lateral movement.
- Develop SPL detections.
- Analyze false positives.
- Map detections to MITRE ATT&CK.

**Deliverables:** Sysmon configuration, detection rules, test evidence, and ATT&CK Navigator layer.

### Phase 4 — Network Detection

**Goal:** Monitor network activity and identify suspicious behavior.

Tasks:
- Configure Suricata and ET Open rules.
- Configure Zeek.
- Forward network telemetry into Splunk.
- Simulate network scans using Nmap.
- Investigate SSH brute-force activity.
- Analyze DNS anomalies and suspicious network traffic.
- Monitor web attacks against authorized vulnerable lab applications.
- Develop network detection searches.

**Deliverables:** Network detection rules, configuration documentation, attack simulations, and investigation evidence.

### Phase 5 — Threat Intelligence & Automation

**Goal:** Enrich security events with external threat intelligence.

Tasks:
- Integrate AbuseIPDB, AlienVault OTX, and URLhaus.
- Create threat intelligence lookups.
- Automate feed updates.
- Integrate VirusTotal enrichment using Python.
- Analyze file hashes and reputation results.
- Configure SOAR Community Edition.
- Build an enrichment playbook.

**Deliverables:** Enrichment scripts, lookup configurations, automation documentation, and playbook exports.

### Phase 6 — Cloud Security

**Goal:** Extend monitoring to cloud infrastructure.

Tasks:
- Prepare a secured AWS environment.
- Enable CloudTrail.
- Configure S3 log storage.
- Integrate AWS logs with Splunk.
- Explore VPC Flow Logs and GuardDuty.
- Develop detections for suspicious IAM activity, root usage, unusual regions, and cloud configuration changes.

**Deliverables:** Sanitized IAM policy examples, cloud detections, and configuration documentation.

### Phase 7 — Detection Engineering & Dashboards

**Goal:** Build a structured detection and monitoring environment.

Tasks:
- Create scheduled and real-time alerts.
- Configure alert thresholds and throttling.
- Develop Sigma detection rules.
- Convert Sigma rules into SPL.
- Explore Risk-Based Alerting.
- Build dashboards for authentication, endpoint, network, cloud, and threat intelligence.
- Investigate simulated incidents.
- Prepare incident reports.

**Deliverables:** Detection catalog, Sigma rules, dashboard exports, alerts, and three incident reports.

### Phase 8 — Splunk Administration & Enterprise Features

**Goal:** Develop practical Splunk administration and maintenance skills.

Tasks:
- Create users and roles.
- Configure role-based access control.
- Explore authentication settings.
- Use Monitoring Console.
- Configure Deployment Server.
- Practice configuration management with btool.
- Investigate splunkd.log and _internal.
- Practice backups and recovery.
- Explore data model acceleration.
- Review the Enterprise trial and Free license transition.

**Deliverables:** Administration documentation, deployment configuration, troubleshooting notes, and final lab review.

---

## Repository Structure

```text
splunk-soc-lab/
│
├── README.md
├── .gitignore
├── LICENSE
│
├── architecture/
├── phase-0-lab-preparation/
├── phase-1-installation-ingestion/
├── phase-2-spl-mastery/
├── phase-3-sysmon-endpoint-detection/
├── phase-4-network-detection/
├── phase-5-threat-intel-automation/
├── phase-6-cloud-security/
├── phase-7-detection-engineering/
├── phase-8-splunk-administration/
│
├── detection-catalog/
├── incident-reports/
├── documentation/
└── linkedin-posts/
```

Each phase contains its own technical configurations, searches, screenshots, and implementation notes as applicable.

## Project Progress

The following table tracks implementation status.

| Phase | Focus | Status |
|---|---|---|
| 0 | Lab Preparation | Existing infrastructure |
| 1 | Installation & Ingestion | In Progress |
| 2 | SPL Mastery | Planned |
| 3 | Sysmon & Endpoint Detection | Planned |
| 4 | Network Detection | Planned |
| 5 | Threat Intelligence & Automation | Planned |
| 6 | Cloud Security | Planned |
| 7 | Detection Engineering | Planned |
| 8 | Splunk Administration | Planned |

Status will be updated as each task is implemented and verified.

## Evidence & Documentation Standards

Every completed phase should contain four types of evidence:

1. **Configuration:** Actual sanitized configuration files, SPL searches, or scripts.
2. **Screenshots:** Evidence from Splunk searches, dashboards, terminal outputs, or configuration pages.
3. **Technical Notes:** What was configured, what went wrong, and how it was resolved.
4. **Repeatable Tests:** A controlled event or simulation demonstrating that the implementation works.

This approach makes the repository a record of practical work rather than just a collection of copied commands.

## Security & Repository Hygiene

Before publishing any files:

- Never commit passwords, API keys, HEC tokens, or credentials.
- Never publish private keys or authentication files.
- Sanitize real public IP addresses and sensitive infrastructure details.
- Remove AWS account IDs and sensitive IAM information.
- Review screenshots before uploading.
- Use environment variables or secure credential storage for API integrations.
- Keep `.gitignore` updated.
- Check Git history for accidentally committed secrets.

## Learning Outcomes

Through this project, I aim to develop practical experience in:

- SIEM deployment and administration.
- Windows and Linux security monitoring.
- Log collection, forwarding, and parsing.
- SPL and security data analysis.
- Endpoint and network threat detection.
- Threat intelligence enrichment.
- Detection engineering.
- MITRE ATT&CK mapping.
- Security dashboards and alerting.
- Incident investigation and reporting.
- Troubleshooting real-world SIEM problems.

## Future Improvements

The lab will continue to evolve with additional log sources, detections, automation workflows, and investigation scenarios.

The long-term objective is to build a documented, repeatable SOC environment that demonstrates the complete process from raw security telemetry to detection, investigation, and response.

---

**Disclaimer:** This is a personal cybersecurity learning environment intended for defensive security monitoring and authorized lab testing. Attack simulations are performed only against systems I own or have permission to test.

**Learning by building. Learning by breaking. Learning by investigating.**
