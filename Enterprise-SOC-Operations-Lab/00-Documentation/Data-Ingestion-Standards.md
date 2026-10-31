# 📥 Data Ingestion Standards & Pipeline Hygiene

To prevent Splunk indexing license exhaustion (500MB/day free tier) and reduce background signal-to-noise ratio, all log forwarding pipelines must follow the ingestion standards defined herein.

---

## 1. Ingestion Pipeline Principles

```text
[ Endpoint Event Source ]
        │
        ▼ (Local Filtering: inputs.conf / Sysmon XML)
[ Universal Forwarder Queue ]
        │
        ▼ (TCP / Port 9997 with SSL/TLS capability)
[ Splunk Enterprise Indexer Pipeline ]
        │
        ▼ (Parsing & Index Routing: props.conf / transforms.conf)
[ Target Hot/Warm Storage Index: wineventlog / sysmon ]
```

1. **Filter at the Source**: Suppress routine OS telemetry before it reaches the network stack. Sysmon rules must explicitly exclude high-volume benign executables (e.g., search indexers, standard Windows update tasks).
2. **Deterministic Index Routing**: Every input stanza must explicitly declare an `index` directive. Telemetry must never fall back to the `default` or `main` index.
3. **Structured Timestamps**: All logs must extract sub-second ISO 8601 or standard Windows epoch timestamps to maintain accurate forensic ordering during attack timeline reconstruction.

---

## 2. Ingestion Thresholds & Splunk Free Tier Budgeting

| Log Source | Expected EPS (Idle) | Expected EPS (Active Lab) | Estimated Daily Volume | Throttling Mechanism |
|---|---|---|---|---|
| Windows Security Event Log | 1 - 3 EPS | 25 - 50 EPS | 80 MB - 120 MB | Filter noisy EventIDs (5156, 5158 filtering) |
| Sysmon Telemetry | 2 - 5 EPS | 30 - 75 EPS | 120 MB - 200 MB | SwiftOnSecurity XML exclusion rules |
| Windows System & Application | < 1 EPS | 5 - 10 EPS | 20 MB - 40 MB | Only capture Warnings & Errors |
| Network Telemetry (Suricata/Zeek) | 5 - 10 EPS | 40 - 80 EPS | 100 MB - 150 MB | Ignore broadcast/multicast (LLMNR/mDNS) |
| **Total Lab Baseline** | **~8 - 15 EPS** | **~100 - 215 EPS** | **~320 MB - 510 MB** | Within 500 MB Free Tier Envelope |

---

## 3. High-Noise Event Suppression Rules

The following high-frequency benign events must be suppressed at the Universal Forwarder level unless a specific detection validation test explicitly requires them:

### Windows Filtering Directives (`inputs.conf`)
* **EventID 5156 / 5158**: Windows Filtering Platform connection allowed/blocked (floods index with millions of routine internal loopback socket checks).
* **EventID 4689**: Process exit event (process creation `EventID 4688` / `Sysmon EventID 1` provides sufficient execution tracking without doubling volume).
* **EventID 4690**: An attempt was made to duplicate a handle to an object.

### Sample Suppressive `inputs.conf` Stanza
```ini
[WinEventLog://Security]
disabled = false
index = wineventlog
sourcetype = WinEventLog:Security
# Exclude noisy benign WFP connection events
blacklist1 = EventCode="5156|5158|4690"
# Filter out standard machine account background queries
blacklist2 = Account_Name="ANONYMOUS LOGON|SYSTEM" EventCode="4624"
```

---

## 4. Sysmon XML Tuning Rules
Sysmon configuration rules (`sysmonconfig.xml`) must prioritize:
1. **EventID 1 (Process Creation)**: Full command lines, parent processes, hashes (SHA256, MD5).
2. **EventID 3 (Network Connection)**: Outbound sockets initiated by non-browser utilities (powershell.exe, cmd.exe, certutil.exe, rundll32.exe).
3. **EventID 11 (File Create)**: Executable creation in `C:\Users\*\AppData\*`, `C:\Windows\Temp\*`, and `C:\ProgramData\*`.
4. **EventID 12/13/14 (Registry Events)**: Changes to `Run`, `RunOnce`, Services, and Winlogon keys.

