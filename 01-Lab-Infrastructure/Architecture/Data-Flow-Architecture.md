# 🔄 Data Flow Architecture & Telemetry Pipelines

## 1. End-to-End Telemetry Pipeline

This document delineates how telemetry travels from kernel drivers on the Windows endpoint into Splunk's indexing buckets and search engine.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                             WINDOWS 10 ENDPOINT                             │
│                                                                             │
│  [ Process / Socket / File Actions ]                                        │
│                 │                                                           │
│                 ├──► [ Windows Kernel (NTOSKRNL.EXE) ]                      │
│                 │               │                                           │
│                 │               ├──► [ Windows Event Log Service (eventlog) │
│                 │               │       • Security Channel (4624, 4625...)  │
│                 │               │       • System Channel                    │
│                 │               │       • Application Channel               │
│                 │                                                           │
│                 └──► [ Sysmon Device Driver (SysmonDrv.sys) ]               │
│                                 │ (Applies SwiftOnSecurity XML filters)     │
│                                 ▼                                           │
│                      [ Sysmon Service (Sysmon64.exe) ]                      │
│                                 │                                           │
│                                 ▼                                           │
│                      [ Microsoft-Windows-Sysmon/Operational ]               │
│                                                                             │
│                                 │                                           │
│                                 ▼                                           │
│             ┌───────────────────────────────────────────────┐               │
│             │          SPLUNK UNIVERSAL FORWARDER           │               │
│             │  • inputs.conf (Monitors channels & paths)    │               │
│             │  • Local Event Buffer & Throttling Queue      │               │
│             │  • outputs.conf (Targets Splunk TCP 9997)     │               │
│             └───────────────────────┬───────────────────────┘               │
└─────────────────────────────────────┼───────────────────────────────────────┘
                                      │
                                      │ Encrypted / Raw TCP Stream (Port 9997)
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       SPLUNK ENTERPRISE SIEM (HOST)                         │
│                                                                             │
│  [ splunkd Ingestion Pipeline ]                                             │
│                 │                                                           │
│                 ▼                                                           │
│  [ Parsing Pipeline ]                                                       │
│     • Line breaking & timestamp extraction                                  │
│     • Header & host assignment (win10-victim)                               │
│                 │                                                           │
│                 ▼                                                           │
│  [ Indexing Pipeline ]                                                      │
│     • Evaluates props.conf & transforms.conf                                │
│     • Routes to designated target indices:                                  │
│         ├── index=wineventlog  (Security / System / App)                    │
│         ├── index=sysmon        (Sysmon Operational Telemetry)              │
│         └── index=network       (Perimeter & Socket Telemetry)              │
│                 │                                                           │
│                 ▼                                                           │
│  [ Storage Buckets (Hot ──► Warm ──► Cold) ]                                │
│                 ▲                                                           │
│                 │                                                           │
│  [ Search Pipeline & Web GUI (Port 8000) ]                                  │
│     • Search Head processes SPL queries from analyst                        │
│     • Executes scheduled detection searches & alerts                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Ingestion Stages & Processing Mechanics

### Stage 1: Endpoint Event Instrumentation
* **Native Windows Auditing**: Activated via Local Security Policy (`auditpol.exe`). Generates security events into `%SystemRoot%\System32\Winevt\Logs\Security.evtx`.
* **Sysmon Kernel Driver**: Injected at ring 0 (`SysmonDrv.sys`), intercepting kernel callbacks (`PsSetCreateProcessNotifyRoutine`, `ObRegisterCallbacks`) to record process creations, network socket connections, and remote thread injections.

### Stage 2: Log Shipping (Universal Forwarder)
* The Splunk Universal Forwarder runs as a native Windows service (`SplunkForwarder`).
* Ingestion stanzas configured in `inputs.conf` monitor Windows Event Log channels through native Windows API subscriptions.
* Telemetry is queued in memory to prevent event drops during network micro-outages, then packaged and streamed over TCP port `9997`.

### Stage 3: Indexing & Storage
* Splunk's indexing engine writes raw events and bloom filters into time-partitioned bucket storage directories:
  * **Hot Buckets**: Actively receiving writes in memory and disk (`C:\Program Files\Splunk\var\lib\splunk\<index>\db\hot_*`).
  * **Warm Buckets**: Read-only, indexed, and fully searchable.
  * **Cold Buckets**: Aged data transitioned for long-term retention.

### Stage 4: Search & Detection Correlation
* Analysts execute real-time and historical Search Processing Language (SPL) queries via the Web UI (`localhost:8000`).
* Scheduled searches execute against indexed data to trigger automated notable events in [`05-Alert-Triage/`](../../05-Alert-Triage/).

