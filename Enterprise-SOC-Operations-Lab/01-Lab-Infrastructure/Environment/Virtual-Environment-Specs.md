# 🖥️ Virtual Environment Specifications & Resource Allocations

## 1. System Requirements & Hardware Sizing

To operate the **Enterprise SOC Operations Lab** smoothly with concurrent adversary simulation, endpoint telemetry collection, and real-time SIEM indexing, the physical host machine must meet the following baseline hardware specifications.

### Physical Host Machine Requirements
* **Processor**: Modern x86_64 Multi-Core CPU (Intel Core i5/i7/i9 8th Gen+ or AMD Ryzen 5/7/9 3000 Series+ with VT-x/AMD-V virtualization enabled in BIOS).
* **Memory (RAM)**: Minimum 16 GB Physical RAM (32 GB recommended for optimal multi-VM performance).
* **Storage**: SSD (NVMe or SATA III) with at least 120 GB free space.
* **Hypervisor Compatibility**: VMware Workstation Pro / Player (v17+) or Oracle VirtualBox (v7.0+).

---

## 2. Virtual Machine Specification Matrix

| VM Profile | Operating System | vCPUs | Allocated RAM | Disk Capacity | Network Mode | Primary Services & Software |
|---|---|---|---|---|---|---|
| **SIEM Central Host** | Windows 10/11 Pro (Physical Host) | Host (4+ cores) | 8 GB Dedicated | 50 GB Dedicated | Host Interface (`192.168.10.1`) | Splunk Enterprise 9.x (Indexer + Search Head, Web UI) |
| **win10-victim** | Windows 10 Enterprise (64-bit) | 2 vCPUs | 4 GB | 40 GB (Dynamically Allocated) | Host-Only (`VMnet1`): `192.168.10.100` | Splunk Universal Forwarder, Sysmon v15.x, RDP (3389), SMB (445) |
| **kali-adversary** | Kali Linux 2024.x (64-bit) | 2 vCPUs | 4 GB | 30 GB (Thin Provisioned) | Host-Only (`VMnet1`): `192.168.10.150` | Nmap, Hydra, Metasploit Framework, NetExec, Python3 |

---

## 3. Virtual Switch Configuration (`VMnet1`)

To guarantee strict isolation and predictable IP addressing, the lab employs a custom virtual switch:

### VMware Virtual Network Editor Configuration:
1. Open **Virtual Network Editor** (`vmnetcfg.exe`) as Administrator.
2. Select **VMnet1**:
   * **Type**: `Host-Only` (Connect VMs internally in a private network).
   * **Connect a host virtual adapter to this network**: Checked.
   * **Use local DHCP service to distribute IP addresses**: **Disabled** (Static assignment is strictly enforced for predictable log ingestion).
   * **Subnet IP**: `192.168.10.0`
   * **Subnet Mask**: `255.255.255.0`
3. Click **Apply** and **OK**.

---

## 4. Virtual Machine Snapshot Management

Strict snapshot discipline ensures repeatable attack-defense cycles without re-installing operating systems from scratch.

### Snapshot Milestones:
* **`win10-victim` Milestone 1 - Fresh OS**: Taken immediately following initial OS install and Windows updates.
* **`win10-victim` Milestone 2 - Baseline Instrumented**: Taken after installing Sysmon, configuring Advanced Audit Policies, deploying Splunk Universal Forwarder, and verifying active log ingestion in Splunk Web. **(Primary operational revert point)**.
* **`kali-adversary` Milestone 1 - Ready**: Taken after complete package update (`sudo apt update && sudo apt upgrade -y`) and test execution of attack toolsets.

