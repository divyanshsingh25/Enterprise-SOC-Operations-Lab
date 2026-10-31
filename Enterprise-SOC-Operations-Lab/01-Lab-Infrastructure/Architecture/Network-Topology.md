# 🌐 Lab Network Topology & Infrastructure Architecture

## 1. Network Topology Overview

The **Enterprise SOC Operations Lab** is deployed across an isolated virtual networking layout designed to balance realistic adversary simulation with containment and stability.

```text
                               ┌────────────────────────────────────────────────────────┐
                               │               PHYSICAL WORKSTATION / HOST              │
                               │                                                        │
                               │  OS: Windows 10/11 Pro 64-bit                          │
                               │  Role: SIEM Indexer & Search Head                      │
                               │  IP: 192.168.10.1 (VMnet Host Adapter)                 │
                               │  Services:                                             │
                               │    • Splunk Web UI: https://localhost:8000             │
                               │    • Splunk Ingestion Port: TCP 9997                   │
                               │    • Splunk Management Port: TCP 8089                  │
                               └───────────────────────────┬────────────────────────────┘
                                                           │
                                                           │ Isolated Virtual Switch
                                                           │ (VMnet1 / Host-Only Network)
                                                           │ Subnet: 192.168.10.0/24
                                                           │
                      ┌────────────────────────────────────┴────────────────────────────────────┐
                      │                                                                         │
                      ▼                                                                         ▼
┌───────────────────────────────────────────┐             ┌───────────────────────────────────────────┐
│           KALI LINUX ADVERSARY VM         │             │         WINDOWS 10 VICTIM ENDPOINT        │
├───────────────────────────────────────────┤             ├───────────────────────────────────────────┤
│ Hostname: kali-adversary                  │             │ Hostname: win10-victim                    │
│ IP: 192.168.10.150                        │             │ IP: 192.168.10.100                        │
│ Gateway: 192.168.10.1                     │             │ Gateway: 192.168.10.1                     │
│ DNS: 1.1.1.1, 8.8.8.8                     │             │ Telemetry Agents:                         │
│ Offensive Toolkit:                        │             │   • Splunk Universal Forwarder (v9.x)     │
│   • Nmap (Reconnaissance)                 │             │   • Sysmon v15.x (SwiftOnSecurity Rules)  │
│   • Hydra (Authentication Attacks)        │             │ Ingestion Target: 192.168.10.1:9997       │
│   • Metasploit Framework (Exploitation)   │             │ Open Exposed Ports for Emulation:         │
│   • CrackMapExec / NetExec (Lateral Move) │             │   • TCP 3389 (RDP)                        │
│   • Atomic Red Team (TTP Emulation)       │             │   • TCP 445 (SMB)                         │
└───────────────────────────────────────────┘             └───────────────────────────────────────────┘
```

---

## 2. IP Addressing & Subnet Schema

| Host Identifier | Device Role | Virtual Interface | Static / Assigned IP | Subnet Mask | Default Gateway |
|---|---|---|---|---|---|
| `splunk-indexer` | SIEM Central Core | Physical / Host-Only VMnet | `192.168.10.1` | `255.255.255.0` | N/A (Host) |
| `win10-victim` | Victim Target Endpoint | VMware Host-Only (VMnet1) | `192.168.10.100` | `255.255.255.0` | `192.168.10.1` |
| `kali-adversary` | Adversary Emulation Node | VMware Host-Only (VMnet1) | `192.168.10.150` | `255.255.255.0` | `192.168.10.1` |

---

## 3. Communication Matrix & Firewall Rules

| Traffic Flow | Source | Destination | Protocol / Port | Purpose |
|---|---|---|---|---|
| **SIEM Log Forwarding** | `win10-victim` (`192.168.10.100`) | `splunk-indexer` (`192.168.10.1`) | TCP `9997` | Splunk Universal Forwarder log shipping |
| **SIEM Management** | Local Browser | `splunk-indexer` (`localhost`) | TCP `8000` | Analyst Web UI access |
| **RDP Attack Emulation** | `kali-adversary` (`192.168.10.150`) | `win10-victim` (`192.168.10.100`) | TCP `3389` | Hydra brute-force simulation (EventID 4625) |
| **SMB / Lateral Move** | `kali-adversary` (`192.168.10.150`) | `win10-victim` (`192.168.10.100`) | TCP `445` | CrackMapExec credential validation (EventID 4624) |
| **Port Scanning** | `kali-adversary` (`192.168.10.150`) | `win10-victim` (`192.168.10.100`) | TCP/UDP 1-1024 | Nmap SYN/Connect scans (Sysmon EventID 3) |
| **Reverse Shell Payload** | `win10-victim` (`192.168.10.100`) | `kali-adversary` (`192.168.10.150`) | TCP `4444` / `8443` | Reverse TCP handler connection (EventID 3, 1) |

---

## 4. Isolation & Safety Controls
1. **Host-Only / Custom VMnet**: All adversary and victim VMs reside on an isolated virtual switch (`VMnet1` or Host-Only) with NAT enabled only temporarily for initial package installations.
2. **Snapshot Baselines**: A clean baseline snapshot is maintained on `win10-victim` (`Snapshot: Baseline-Clean-Installed`) allowing immediate rollbacks following attack simulation testing.

