# TC-001: Detection Validation Report — Encoded PowerShell

## 1. Validation Metadata

| Field | Detail |
|---|---|
| **Test Case ID** | `TC-001` |
| **Associated Rule** | `DET-ENDP-001` |
| **Test Name** | Encoded PowerShell Execution Detection Validation |
| **Target Host** | `chopper` |
| **Target Index** | `sysmon_logs` |
| **Target Event ID** | Sysmon Event ID 1 (Process Create) |
| **Sourcetype** | `XmlWinEventLog:Sysmon` |
| **Execution User** | `CHOPPER\ruddy` |
| **Execution Timestamp** | 2026-11-04 18:34:53.370 |
| **Validation Status** | **`VALIDATED`** |

---

## 2. Validation Criteria & Success Metrics

| Requirement | Success Condition | Verification Method | Status | Observation / Evidence |
|---|---|---|---|---|
| **Endpoint Telemetry Generation** | Sysmon Event ID 1 generated on `chopper` with full `-EncodedCommand` arguments in `CommandLine`. | Event Record ID 36335 generated with command line `"C:\Windows\System32\...\powershell.exe" -NoProfile -EncodedCommand RwBlAHQALQBEAGEAdABlAA==`. | **`PASS`** | Verified via Sysmon Operational channel. |
| **Log Forwarding** | Event successfully forwarded across TCP `9997` without delay or truncation. | Splunk Universal Forwarder indexed event into `sysmon_logs` in real time. | **`PASS`** | Confirmed present in Splunk Index. |
| **Detection Rule Trigger** | Detection SPL query isolates the event and identifies `CommandLine` encoded flags. | Tested SPL search returned all 4 execution attempts on host `chopper`. | **`PASS`** | 4 events captured across testing interval. |
| **Field Extraction Accuracy** | `Image`, `CommandLine`, `ParentImage`, and `User` fields cleanly extracted. | Extracted via XML regular expression and validated in Splunk statistics table. | **`PASS`** | All fields mapped to CIM standards. |
| **Payload Integrity & Safety** | Decoded payload verified to ensure no hostile action occurred. | Decoded Base64 payload evaluated to benign `Get-Date`. | **`PASS`** | Safe lab testing verified. |

---

## 3. Execution Verification Record

* **Execution Host**: `chopper`
* **Executing Account**: `CHOPPER\ruddy` (Integrity Level: Medium)
* **Parent Process**: `powershell.exe` (PID: 3836)
* **Child Process**: `powershell.exe` (PID: 2084)
* **Process GUID**: `{782327da-2d75-6aeb-6e03-000000001100}`
* **Recorded Hashes**:
  * `MD5`: `2E5A8590CF6848968FC23DE3FA1E25F1`
  * `SHA256`: `9785001B00CF755EDDB8AF294A373C0B87B2498660F724E76C4D53F9C217C7A3`
* **Evidence Backing**:
  * Console execution: [`TC001-Endpoint-Execution.png`](../../14-Evidence/Splunk-Screenshots/TC-001/TC001-Endpoint-Execution.png)
  * Splunk search table: [`TC001-Splunk-Statistics-Table.jpg`](../../14-Evidence/Splunk-Screenshots/TC-001/TC001-Splunk-Statistics-Table.jpg)
  * Splunk raw XML event: [`TC001-Splunk-Raw-Event.jpg`](../../14-Evidence/Splunk-Screenshots/TC-001/TC001-Splunk-Raw-Event.jpg)
* **Validation Outcome**: **`VALIDATED`** (Detection logic confirmed functional and repeatable).
