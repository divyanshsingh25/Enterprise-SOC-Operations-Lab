# 🛡️ Enterprise SOC Operations Lab

An enterprise-grade, modular Security Operations Center (SOC) engineering lab built with Splunk Enterprise, Sysmon, Windows, and Linux. Architected to support end-to-end detection engineering, attack simulation, alert triage, incident response, threat hunting, and security automation workflows.

---

## 🏗️ Lab Architecture

```text
┌──────────────────────────────────────────────────────────────┐
│                  Physical Host — Windows                     │
│            Splunk Enterprise  •  localhost:8000              │
│                   Indexer + Search Head                      │
│                      Port 9997 ◄─────────────────────┐       │
└──────────────────────────────────────────────────────────────┘
                                                        │ TCP 9997
                                                        │ (log forwarding)
┌──────────────────────────┐      ┌─────────────────────────────────┐
│     Kali Linux VM        │      │      Windows 10 VM (Victim)     │
│       Attacker           │─────►│   Splunk Universal Forwarder    │
│                          │      │   Sysmon (SwiftOnSecurity)      │
│  nmap  •  Metasploit     │      │   Windows Event Logs            │
│  Hydra •  CrackMapExec   │      │   Security / System / App       │
│  Atomic Red Team         │      │                                 │
└──────────────────────────┘      └─────────────────────────────────┘
```

---

## 📁 Repository Structure

```text
Enterprise-SOC-Operations-Lab/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── 00-Documentation/
│   ├── SOC-Operations-Framework.md
│   ├── Naming-Conventions.md
│   └── Data-Ingestion-Standards.md
│
├── 01-Lab-Infrastructure/
│   ├── Architecture/
│   │   ├── Network-Topology.md
│   │   └── Data-Flow-Architecture.md
│   ├── Environment/
│   │   ├── Virtual-Environment-Specs.md
│   │   └── Host-Configuration.md
│   └── Installation/
│       ├── README.md
│       ├── splunk-installation.md
│       ├── sysmon-installation.md
│       └── universal-forwarder-setup.md
│
├── 02-Data-Collection/
│   ├── Windows/
│   ├── Linux/
│   ├── Network/
│   └── Splunk-Configuration/
│
├── 03-Data-Analysis/
│   ├── SPL-Fundamentals/
│   ├── Authentication-Analysis/
│   ├── Endpoint-Analysis/
│   ├── Network-Analysis/
│   └── Process-Analysis/
│
├── 04-Detection-Engineering/
│   ├── Authentication/
│   ├── Endpoint/
│   ├── Network/
│   └── Detection-Catalog.md
│
├── 05-Alert-Triage/
│
├── 06-Incident-Response/
│   └── Playbooks/
│
├── 07-Threat-Hunting/
│   └── Hunting-Queries/
│
├── 08-Attack-Simulation/
│   ├── Authentication-Attacks/
│   ├── Windows-Attacks/
│   ├── Network-Attacks/
│   ├── Linux-Attacks/
│   └── Simulation-Results/
│
├── 09-SOC-Dashboards/
│   ├── SOC-Overview/
│   ├── Authentication-Monitoring/
│   ├── Endpoint-Monitoring/
│   ├── Network-Monitoring/
│   └── Screenshots/
│
├── 10-Threat-Intelligence/
│
├── 11-Security-Automation/
│   ├── Python-Scripts/
│   ├── Log-Parsing/
│   ├── IOC-Extraction/
│   └── Alert-Automation/
│
├── 12-Incident-Reports/
│   ├── Case-001/
│   ├── Case-002/
│   └── Case-003/
│
├── 13-Detection-Validation/
│   ├── Test-Cases/
│   ├── Detection-Results/
│   └── False-Positive-Testing/
│
└── 14-Evidence/
    ├── Splunk-Screenshots/
    ├── Event-Logs/
    ├── Detection-Results/
    └── Investigation-Evidence/
```

---

## 🚀 Quick Start & Operational Lifecycle

```text
01-Lab-Infrastructure  ──►  02-Data-Collection  ──►  03-Data-Analysis
                                                            │
                                                            ▼
08-Attack-Simulation   ◄──  04-Detection-Engineering
        │
        ▼
13-Detection-Validation ──►  05-Alert-Triage    ──►  06-Incident-Response
                                                            │
                                                            ▼
14-Evidence             ◄──  12-Incident-Reports ◄──  07-Threat-Hunting
```

### Prerequisites

| Component | Role | Resource |
|---|---|---|
| VMware / VirtualBox | Hypervisor platform | [vmware.com](https://www.vmware.com) |
| Splunk Enterprise | SIEM core (Indexer & Search Head) | [splunk.com](https://www.splunk.com/en_us/download/splunk-enterprise.html) |
| Splunk Universal Forwarder | Endpoint log ingestion agent | [splunk.com](https://www.splunk.com/en_us/download/universal-forwarder.html) |
| Sysmon | Host telemetry & process instrumentation | [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) |
| Kali Linux | Adversary emulation & offensive toolsets | [kali.org](https://www.kali.org/get-kali/) |

---

## 📂 Modular Module Breakdown

### `00-Documentation`
High-level lab standards, operational workflows, SOC procedures, and change records.

### `01-Lab-Infrastructure`
Network topology diagrams, host environment configurations, and deployment guides:
* **Architecture/**: Network segmentation and hardware/VM specifications.
* **Environment/**: Hypervisor profiles, IP schemas, and operating system configs.
* **Installation/**: Step-by-step installation guides for Splunk Enterprise, Sysmon, and Universal Forwarder.

### `02-Data-Collection`
Telemetry pipelines, forwarder manifests, and source references:
* **Windows/**: Windows Security/System event IDs and Sysmon telemetry references.
* **Linux/**: Syslog, auth.log, and auditd configurations.
* **Network/**: Firewall, DNS, and proxy logging telemetry.
* **Splunk-Configuration/**: Production configuration stanzas (`inputs.conf`, `outputs.conf`, `indexes.conf`, `props.conf`, `transforms.conf`).

### `03-Data-Analysis`
Interactive log investigation and data exploration:
* **SPL-Fundamentals/**: Search Processing Language baseline commands and data wrangling patterns.
* **Authentication-Analysis/**: Kerberos, NTLM, and logon event analysis.
* **Endpoint-Analysis/**: Host process trees, command-line arguments, and DLL injections.
* **Network-Analysis/**: Port scans, protocol anomalies, and abnormal traffic patterns.
* **Process-Analysis/**: Parent-child relationship auditing and LOLBIN tracking.

### `04-Detection-Engineering`
Production detection rule repository and lifecycle tracking:
* **Authentication/**: Credential stuffing, brute-force, and golden/silver ticket rules.
* **Endpoint/**: Ransomware indicators, persistence triggers, and process masquerading.
* **Network/**: Lateral movement, C2 beaconing, and egress anomalies.
* **Detection-Catalog.md**: Master index of active detections mapped to MITRE ATT&CK techniques.

### `05-Alert-Triage`
Standardized alert verification frameworks, false-positive triage matrix, and analyst escalation trees.

### `06-Incident-Response`
Operational workflows for containment, eradication, and post-incident handling:
* **Playbooks/**: Systematic runbooks for specific threat scenarios (Ransomware, Compromised Credentials, C2 Beaconing).

### `07-Threat-Hunting`
Hypothesis-driven threat hunting methodologies:
* **Hunting-Queries/**: Structured SPL queries for identifying advanced persistent threats that bypass automated alerting.

### `08-Attack-Simulation`
Controlled adversary emulation scenarios and telemetry generation:
* **Authentication-Attacks/**: Password spraying, Kerberoasting, and brute-force simulations.
* **Windows-Attacks/**: Persistence via registry run keys, scheduled tasks, and token manipulation.
* **Network-Attacks/**: Port reconnaissance, beacon simulations, and SMB exploitation.
* **Linux-Attacks/**: Sudo privilege escalation and bash history tampering.
* **Simulation-Results/**: Detailed execution logs and verification reports.

### `09-SOC-Dashboards`
Splunk Enterprise dashboard XML sources and analyst visualizations:
* **SOC-Overview/**: High-level metric dashboards and alert burndown charts.
* **Authentication-Monitoring/**: Real-time authentication failure spikes and geo-velocity panels.
* **Endpoint-Monitoring/**: Process launch volumes, unsigned driver alerts, and service modifications.
* **Network-Monitoring/**: Connection timelines and external destination tracking.
* **Screenshots/**: Exported views of active dashboards in operation.

### `10-Threat-Intelligence`
Adversary profiling, indicator of compromise (IOC) feeds, and threat actor TTP mappings.

### `11-Security-Automation`
Custom scripts and pipelines designed to streamline repetitive security operations:
* **Python-Scripts/**: Operational automation and API orchestration.
* **Log-Parsing/**: Telemetry cleanup and structured JSON/CSV converters.
* **IOC-Extraction/**: Automated regex and parser utilities for raw log feeds.
* **Alert-Automation/**: Webhook push triggers and alerting scripts.

### `12-Incident-Reports`
Formal case writeups and post-mortem investigations:
* **Case-001/** to **Case-003/**: Evidence summaries, root cause analyses, timeline reconstructions, and remediation recommendations.

### `13-Detection-Validation`
Quality assurance and continuous detection testing:
* **Test-Cases/**: Expected inputs and test payloads.
* **Detection-Results/**: True-positive trigger validations.
* **False-Positive-Testing/**: Baseline noise stress tests.

### `14-Evidence`
Raw forensic artifacts and proof of detection:
* **Splunk-Screenshots/**: Visual captures of triggered alerts and search outputs.
* **Event-Logs/**: Exported `.evtx` samples and raw log dumps.
* **Detection-Results/**: Raw SIEM output records.
* **Investigation-Evidence/**: Packet captures, memory artifacts, and disk timeline extracts.

---

## 📚 References & Resources

- [Splunk Enterprise Documentation](https://docs.splunk.com)
- [SwiftOnSecurity Sysmon Configuration](https://github.com/SwiftOnSecurity/sysmon-config)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [Splunk Boss of the SOC (BOTS)](https://github.com/splunk/botsv3)
- [Splunk Security Essentials](https://splunkbase.splunk.com/app/3435)
- [TryHackMe SOC Analyst Path](https://tryhackme.com/path/outline/soclevel1)

---

## ⚠️ Disclaimer

This lab is built for **educational, testing, and defensive research purposes only**. All attacks are performed inside an **isolated virtual network environment**. Never execute these techniques against production systems or networks without explicit authorized written consent.

---

## 👤 Author

**Divyansh Singh**
- 🐙 GitHub: [divyanshsingh25](https://github.com/divyanshsingh25)
- 💼 LinkedIn: [Divyansh Singh](https://www.linkedin.com/in/divyansh-singh-8b8955381)

---

*Learn. Attack. Detect. Repeat.* 🔁
