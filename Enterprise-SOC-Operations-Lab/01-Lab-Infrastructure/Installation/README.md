# 🛠️ Lab Installation & Deployment Guide

This directory contains the step-by-step deployment and installation procedures for all core software and telemetry agents in the **Enterprise SOC Operations Lab**.

---

## 📋 Installation Sequence

Follow the installation guides strictly in this numerical order:

```text
[ Step 1: Splunk Enterprise ] ──► [ Step 2: Sysmon & SwiftOnSecurity ] ──► [ Step 3: Universal Forwarder ]
   (Host Indexer / Port 9997)            (Victim VM Telemetry)                 (Log Forwarding Agent)
```

| Order | Module | Target System | Guide Link | Key Outcome |
|---|---|---|---|---|
| **01** | Splunk Enterprise 9.x | Physical Host | [`splunk-installation.md`](splunk-installation.md) | Web UI active at `localhost:8000`, receiving port `9997` configured. |
| **02** | Sysmon v15.x | `win10-victim` VM | [`sysmon-installation.md`](sysmon-installation.md) | SwiftOnSecurity ruleset active, logging to `Microsoft-Windows-Sysmon/Operational`. |
| **03** | Splunk Universal Forwarder | `win10-victim` VM | [`universal-forwarder-setup.md`](universal-forwarder-setup.md) | `inputs.conf` & `outputs.conf` configured, shipping telemetry to host `9997`. |

---

## 🧪 Post-Installation Validation Checks

Once all three components are installed, run these verification steps:

### 1. Check Network Connectivity from Victim to Splunk
On `win10-victim` PowerShell:
```powershell
Test-NetConnection -ComputerName 192.168.10.1 -Port 9997
```
*Expected Result*: `TcpTestSucceeded : True`

### 2. Verify Sysmon Service
On `win10-victim` PowerShell:
```powershell
Get-Service Sysmon64
```
*Expected Result*: `Status : Running`

### 3. Verify Telemetry Ingestion in Splunk Web
Log into Splunk Web (`https://localhost:8000`) and run in Search & Reporting:
```spl
index=* (sourcetype="WinEventLog:*" OR sourcetype="*sysmon*") | stats count by host, sourcetype, index
```
*Expected Result*: Events appearing from host `win10-victim` under indexes `wineventlog` and `sysmon`.

