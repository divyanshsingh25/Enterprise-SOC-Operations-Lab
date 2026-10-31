# 🏷️ Enterprise SOC Taxonomy & Naming Conventions

Standardization across queries, indices, rules, and documentation is vital for scalable enterprise security operations. This standard establishes strict naming rules for all artifacts developed within the lab.

---

## 1. Detection Rule Naming Scheme

Detection rules maintained in [`04-Detection-Engineering/`](../04-Detection-Engineering/) must adhere to the following convention:

```text
DET-<CATEGORY>-<ID>_<MITRE-ID>_<BRIEF-DESCRIPTION>.spl
```

### Components:
* **`DET`**: Standard prefix identifying a detection rule.
* **`<CATEGORY>`**: The technical operational category:
  * `AUTH` - Authentication and identity (Kerberos, NTLM, RDP, Brute Force).
  * `ENDP` - Endpoint telemetry (Process creation, injection, DLL hijacking).
  * `NETW` - Network anomalies (Beaconing, scanning, lateral movement).
  * `PERS` - Persistence (Registry run keys, Scheduled tasks, Services).
  * `PRIV` - Privilege escalation (UAC bypass, token impersonation).
* **`<ID>`**: 3-digit zero-padded incremental sequence number (e.g., `001`, `002`).
* **`<MITRE-ID>`**: Primary MITRE ATT&CK technique code (e.g., `T1110`, `T1059.001`).
* **`<BRIEF-DESCRIPTION>`**: Hyphen-separated descriptive title (lower-case or kebab-case).

### Examples:
* `DET-AUTH-001_T1110.001_rdp-brute-force-spike.spl`
* `DET-ENDP-002_T1059.001_powershell-encoded-command.spl`
* `DET-NETW-003_T1046_internal-horizontal-port-scan.spl`
* `DET-PERS-004_T1547.001_registry-run-key-modification.spl`

---

## 2. Splunk Index & Sourcetype Taxonomy

All ingested data must be directed to designated, isolated indices to optimize search performance and RBAC:

| Index Name | Intended Log Source | Primary Sourcetypes | Retention Target |
|---|---|---|---|
| `wineventlog` | Native Windows Event Channels | `WinEventLog:Security`<br>`WinEventLog:System`<br>`WinEventLog:Application` | 90 Days |
| `sysmon` | Microsoft Sysinternals Sysmon Telemetry | `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational`<br>`sysmon` | 90 Days |
| `os_linux` | Linux OS and Authentication Logs | `syslog`, `auth.log`, `auditd` | 90 Days |
| `network` | Firewalls, Zeek, Suricata, DNS, Proxy | `stream:tcp`, `pan:traffic`, `bro_dns` | 30 Days |
| `soc_alerts` | Internal SIEM Notable Events / Triggered Alerts | `alert:notable`, `splunk:alert` | 180 Days |

---

## 3. Common Information Model (CIM) Field Standards

All SPL queries and detection logic must adopt Splunk CIM field names to remain vendor-agnostic and modular:

```text
┌───────────────────────┬───────────────────────┬────────────────────────────────────────┐
│ Field Name            │ Field Type            │ Description / Canonical Usage          │
├───────────────────────┼───────────────────────┼────────────────────────────────────────┤
│ dest                  │ String / IP / Host    │ Destination host or IP address         │
│ src                   │ String / IP / Host    │ Source host or IP address              │
│ user                  │ String                │ Subject or target user identity        │
│ process_name          │ String                │ Target executable name (e.g. cmd.exe)  │
│ process_path          │ String                │ Full binary execution directory path   │
│ process_id            │ Integer               │ Process Identifier (PID)               │
│ parent_process_name   │ String                │ Calling / spawning parent executable   │
│ parent_process_id     │ Integer               │ Parent Process Identifier (PPID)       │
│ command               │ String                │ Command-line execution arguments       │
│ action                │ String                │ Outcome: allowed, blocked, success,    │
│                       │                       │ failure                                │
└───────────────────────┴───────────────────────┴────────────────────────────────────────┘
```

---

## 4. Threat Hunting Hypothesis Taxonomy

Hunting hypotheses documented in [`07-Threat-Hunting/`](../07-Threat-Hunting/) follow a structured identifier:

```text
TH-<DOMAIN>-<TECHNIQUE-ID>_<HYPOTHESIS-TITLE>
```

* **Example**: `TH-HOST-T1055_process-injection-svchost-anomalies.md`
* **Example**: `TH-NETW-T1071.001_dns-tunneling-high-entropy-queries.md`

---

## 5. Incident Reporting & Investigation Naming

Cases cataloged under [`12-Incident-Reports/`](../12-Incident-Reports/) follow this structure:

```text
INC-<YEAR>-<CASE-NUMBER>_<INCIDENT-NAME>
```

* **Directory format**: `Case-001/`, `Case-002/`, `Case-003/`
* **Report Master File**: `INC-2026-001_RDP-Brute-Force-Compromise.md`
* **Evidence Directory**: `14-Evidence/Investigation-Evidence/INC-2026-001/`

