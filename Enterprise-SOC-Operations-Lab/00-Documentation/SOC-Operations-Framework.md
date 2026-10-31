# 🏢 SOC Operations Framework

## 1. Executive Summary & Mission
The **Enterprise SOC Operations Lab** serves as a modular, high-fidelity security operations proving ground. The primary mission of this SOC is to achieve continuous detection readiness, high-fidelity alerting, systematic adversary containment, and actionable threat intelligence integration through rigorous blue-team operational workflows.

This framework defines the operational tier models, triage matrices, service level agreements (SLAs), and standardized escalation pathways governing all lab activities.

---

## 2. SOC Operational Tier Model

```text
┌─────────────────────────────────────────────────────────────────┐
│                     SOC Management & Metrics                    │
│     Operational Readiness • Detection Health • Threat Modeling  │
└────────────────────────────────┬────────────────────────────────┘
                                 │
         ┌───────────────────────┴───────────────────────┐
         ▼                                               ▼
┌───────────────────────────────┐     ┌───────────────────────────────┐
│     Tier 1: Alert Triage      │     │  Tier 3: Detection & Hunting  │
│ • Initial Alert Validation    │     │ • Hypothesis-Driven Hunting   │
│ • True/False Positive Filter  │     │ • Detection Rule Engineering  │
│ • IOC Correlation & Escalation│     │ • Attack Simulation & Testing │
└───────────────┬───────────────┘     └───────────────▲───────────────┘
                │                                     │
                ▼                                     │
┌───────────────────────────────┐                     │
│ Tier 2: Incident Response     │                     │
│ • Deep-Dive Log Investigation │                     │
│ • Host/Network Containment    │─────────────────────┘
│ • Timeline Reconstruction     │ (Feeds Detection Improvements)
│ • Formal Incident Reporting   │
└───────────────────────────────┘
```

### Tier 1: Alert Triage & Validation
* **Primary Responsibility**: 24/7 alert monitoring, initial validation, triage qualification, and enrichment.
* **Key Tasks**:
  * Assess inbound Splunk alerts and correlate host/network context.
  * Filter benign anomalies and classify false positives.
  * Extract observable indicators (hashes, IP addresses, domains, command-line arguments).
  * Escalate confirmed true-positive incidents to Tier 2 within defined SLAs.

### Tier 2: Incident Handling & Deep-Dive Analysis
* **Primary Responsibility**: Root-cause analysis, timeline construction, containment, and remediation.
* **Key Tasks**:
  * Execute scenario-specific incident response playbooks.
  * Correlate endpoint telemetry (Sysmon, Windows Event Logs) with perimeter network traffic.
  * Execute host-level containment measures (network isolation, account disablement, process termination).
  * Compile formal post-incident reports (`12-Incident-Reports/`).

### Tier 3: Detection Engineering & Threat Hunting
* **Primary Responsibility**: Proactive hunting, adversary emulation, detection engineering, and analytics validation.
* **Key Tasks**:
  * Formulate threat hunting hypotheses based on MITRE ATT&CK techniques.
  * Develop production-grade SPL detection rules (`04-Detection-Engineering/`).
  * Validate detections against offensive playbooks executed in `08-Attack-Simulation/`.
  * Tune out baseline noise in `13-Detection-Validation/False-Positive-Testing/`.

---

## 3. Incident Severity Classification & SLAs

All security events are prioritized based on adversary potential impact, asset criticality, and scope of compromise.

| Severity Level | Definition | Example Indicators | Target MTTA (Ack) | Target MTTR (Remediate) |
|---|---|---|---|---|
| **P1 - Critical** | Active widespread breach, domain compromise, or ransomware deployment. | Domain Controller compromise, Mass file encryption, Active C2 channel on mission-critical system. | **≤ 15 minutes** | **≤ 2 hours** |
| **P2 - High** | Unauthorized administrative access, lateral movement, or privileged credential dumping. | LSASS memory dump (`Sysmon EventID 10`), Pass-the-Hash / SMB lateral movement, Mimikatz usage. | **≤ 30 minutes** | **≤ 4 hours** |
| **P3 - Medium** | Suspicious local execution, persistence mechanism installation, or reconnaissance. | Scheduled task creation (`EventID 4698`), LOLBIN reconnaissance (`certutil`, `whoami`), internal port scan. | **≤ 2 hours** | **≤ 24 hours** |
| **P4 - Low** | Low-fidelity anomalies, single failed authentications, or informational policy breaches. | Isolated brute-force attempts, policy scans, anomalous port probing. | **≤ 8 hours** | **≤ 72 hours** |

* **MTTA**: Mean Time to Acknowledge (detection trigger to active analyst investigation).
* **MTTR**: Mean Time to Remediate (containment and threat neutralization).

---

## 4. Incident Response Lifecycle (NIST SP 800-61r2)

```text
[ Preparation ] ────────► [ Detection & Analysis ] ────────► [ Containment & Eradication ]
       ▲                                                                   │
       │                                                                   ▼
[ Post-Incident Review ] ◄───────────────────────────────────── [ Recovery ]
```

1. **Preparation**: Maintaining hardened VM environments, active Universal Forwarders, baseline Sysmon configurations, and operational playbooks.
2. **Detection & Analysis**: Querying Splunk, correlating event IDs, validating alerts, and determining the attacker's foothold.
3. **Containment**: Short-term containment (isolating victim VM, revoking credentials) and long-term containment (blocking malicious IPs/ports).
4. **Eradication**: Terminating malicious processes, removing persistence artifacts (registry keys, scheduled tasks), and cleaning malware binaries.
5. **Recovery**: Restoring endpoints from clean snapshots, re-enabling monitoring, and conducting heightened surveillance.
6. **Post-Incident Activity**: Documenting lessons learned, updating detection rules in `04-Detection-Engineering/`, and preserving digital evidence in `14-Evidence/`.

