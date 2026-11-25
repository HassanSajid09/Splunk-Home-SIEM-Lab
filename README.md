# Splunk Home SIEM Lab — Windows & Linux Log Monitoring

## Overview

This project is a hands-on home SIEM lab built to practice centralized log collection, monitoring, log parsing, and security investigation using Splunk Enterprise.

The lab integrates Windows and Linux log sources with Splunk to explore how system activity is collected, indexed, searched, and analyzed from a defender's perspective.

The focus is not only on ingesting data, but also on understanding log pipelines, troubleshooting forwarding issues, and correcting event parsing problems.

## Objectives

- Create and organize custom Splunk indexes.
- Collect Windows Security, System, and Application event logs.
- Collect Linux authentication and system logs.
- Configure Splunk Universal Forwarder on Windows and Linux.
- Enable PowerShell Script Block Logging.
- Ingest PowerShell Event ID 4104.
- Test HTTP Event Collector (HEC) using `curl`.
- Forward Linux syslog messages using rsyslog.
- Troubleshoot sourcetype and event line-breaking configurations.
- Validate data ingestion using SPL.

## Lab Architecture

```text
              ┌──────────────────────────┐
              │    Splunk Enterprise    │
              │      Ubuntu Server      │
              │                        │
              │  Indexes:              │
              │  win_sec              │
              │  win_sysmon           │
              │  linux               │
              │  network             │
              │  cloud               │
              │  threat_intel        │
              └────────────┬───────────┘
                           │
               ┌───────────┴───────────┐
               │                       │
        TCP Forwarding           HTTP Event
           Port 9997             Collector
                                Port 8088
               │                       │
       ┌───────┴───────┐         ┌─────┴──────┐
       │               │         │            │
   Windows 11        Kali      curl        rsyslog
   Universal        Universal   HEC test    TCP syslog
   Forwarder        Forwarder
       │               │
       │               │
 Security/System/    auth.log/
 Application        syslog
 PowerShell
 Operational
```

> Replace the example addresses and ports with your actual lab configuration before publishing.

## Lab Environment

| Component | Role |
|---|---|
| Ubuntu Server | Splunk Enterprise / Indexer |
| Windows 11 | Windows event log source |
| Kali Linux | Linux log source and testing |
| Splunk Universal Forwarder | Log collection and forwarding |
| rsyslog | Syslog forwarding |
| HTTP Event Collector | HTTP-based event ingestion |

## Index Design

Custom indexes were created to separate different types of telemetry.

| Index | Purpose |
|---|---|
| `win_sec` | Windows Security, System, and Application logs |
| `win_sysmon` | Windows PowerShell and additional Windows telemetry |
| `linux` | Linux authentication and system logs |
| `network` | Network-related data and test events |
| `cloud` | Reserved for future cloud log ingestion |
| `threat_intel` | Reserved for future threat intelligence data |

The lab uses a 90-day retention setting for the indexes as an initial retention policy.

## Windows Log Collection

The Windows Universal Forwarder is configured to collect:

- Security
- System
- Application
- PowerShell Operational logs

### Windows log inputs

Example configuration:

```ini
[WinEventLog://Security]
disabled = 0
index = win_sec

[WinEventLog://System]
disabled = 0
index = win_sec

[WinEventLog://Application]
disabled = 0
index = win_sec

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = win_sysmon
```

The forwarder sends collected events to the Splunk Enterprise receiving port.

### PowerShell Script Block Logging

PowerShell Script Block Logging was enabled to capture executed script content in the PowerShell Operational log.

Event ID **4104** is used to validate the collection of script block events.

Example search:

```spl
index=win_sysmon EventCode=4104
```

## Linux Log Collection

The Linux Universal Forwarder is configured to monitor authentication and system log files.

### Linux inputs

```ini
[monitor:///var/log/auth.log]
disabled = 0
index = linux
sourcetype = linux_secure

[monitor:///var/log/syslog]
disabled = 0
index = linux
sourcetype = syslog
```

This provides visibility into authentication activity, system messages, and other operating system events.

### Linux validation searches

Authentication logs:

```spl
index=linux sourcetype=linux_secure
```

Syslog:

```spl
index=linux sourcetype=syslog
```

## HTTP Event Collector (HEC)

HEC was configured to allow HTTP-based event ingestion into Splunk.

A test event was sent from Kali Linux using `curl`.

Example:

```bash
curl -k https://SPLUNK_SERVER:8088/services/collector/event \
-H "Authorization: Splunk YOUR_HEC_TOKEN" \
-H "Content-Type: application/json" \
-d '{"event":"HEC test event from Kali","sourcetype":"hec_test","index":"network"}'
```

The token is intentionally excluded from this repository.

Successful ingestion was validated using:

```spl
index=network sourcetype=hec_test
```

## rsyslog Forwarding

Linux logs were also forwarded using rsyslog over TCP to a Splunk TCP input.

Example rsyslog configuration:

```conf
*.* @@SPLUNK_SERVER:514
```

- `@` indicates UDP forwarding.
- `@@` indicates TCP forwarding.

A test message can be generated with:

```bash
logger "RSYSLOG TEST: Kali forwarding to Splunk"
```

Validation search:

```spl
index=linux "RSYSLOG TEST"
```

## Sourcetype & Event-Breaking Troubleshooting

One of the troubleshooting exercises involved intentionally misconfiguring event line-breaking behavior.

### The problem

Multiple log lines were being merged into a single event rather than being indexed as separate events.

This can cause inaccurate event counts, incorrect timestamps, and unreliable searches or detections.

### Incorrect configuration

A deliberately incorrect parsing configuration was used to demonstrate the effect of event merging.

### Corrected configuration

The parsing configuration was corrected using `props.conf`:

```ini
[lab_fixed]
SHOULD_LINEMERGE = false
LINE_BREAKER = ([\r\n]+)
```

This configuration tells Splunk to treat individual newline-delimited records as separate events.

> Parsing configuration changes generally affect newly ingested data. Previously indexed events are not automatically reparsed, so a fresh test file or sourcetype was used for validation.

## Health Check

A basic health-check search was used to verify data distribution across indexes, sourcetypes, and hosts.

```spl
index=*
| stats count by index, sourcetype, host
| sort index
```

This helps identify which data sources are reporting, which sourcetypes are present, and whether expected hosts are sending data.

## Troubleshooting Lessons

During the lab, I learned to verify each part of the ingestion pipeline instead of assuming that a configured forwarder means data is successfully arriving.

Key troubleshooting checks included:

- Verifying that Splunk is running.
- Confirming that the receiving port is enabled.
- Checking connectivity between VMs.
- Validating Universal Forwarder service status.
- Reviewing `inputs.conf` and `outputs.conf`.
- Confirming index names exist.
- Checking sourcetype assignment.
- Investigating event line-breaking behavior.
- Validating ingestion through SPL searches.

## Screenshots

The screenshots folder documents the lab configuration and verification process.

1. Lab architecture
2. Splunk indexes and retention configuration
3. Windows log ingestion
4. Linux log ingestion
5. HEC test event
6. Sourcetype/event-breaking issue
7. Corrected event parsing
8. Final health-check results

## Skills Practiced

- Splunk Enterprise administration
- Log ingestion and forwarding
- Windows event log monitoring
- Linux log monitoring
- Universal Forwarder configuration
- HTTP Event Collector
- rsyslog
- SPL searches
- Event parsing and line breaking
- SIEM troubleshooting

## Next Steps

- Create Windows authentication detections.
- Monitor failed logons (Event ID 4625).
- Monitor successful logons (Event ID 4624).
- Analyze PowerShell activity (Event ID 4104).
- Build Linux SSH authentication detections.
- Integrate Sysmon telemetry.
- Build dashboards for Windows, Linux, and network activity.
- Create alerting and incident investigation workflows.

---

**Disclaimer:** This is a personal cybersecurity learning lab built for defensive monitoring, log analysis, and authorized testing.
