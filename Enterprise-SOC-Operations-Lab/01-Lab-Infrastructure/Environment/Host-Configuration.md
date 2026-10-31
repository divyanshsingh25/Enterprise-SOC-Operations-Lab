# ⚙️ Host & Endpoint Auditing Configuration

Proper telemetry generation depends on OS-level auditing policies. By default, Windows does not log detailed process command lines or deep PowerShell script execution. This guide configures high-visibility endpoint auditing.

---

## 1. Windows Endpoint Advanced Audit Policy Configuration

Execute the following commands in an elevated PowerShell session (Run as Administrator) on `win10-victim`:

### Step 1: Configure Core Audit Categories via `auditpol.exe`

```powershell
# Detailed Tracking - Process Creation (EventID 4688)
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable

# Detailed Tracking - Process Termination (EventID 4689 - Optional)
auditpol /set /subcategory:"Process Termination" /success:disable /failure:disable

# Account Logon - Credential Validation (EventID 4776)
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable

# Logon/Logoff - Logon & Special Logon (EventID 4624, 4672)
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Special Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Logoff" /success:enable /failure:disable

# Account Management - User Account Management (EventID 4720, 4726, 4738)
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable

# Privilege Use - Sensitive Privilege Use (EventID 4673, 4674)
auditpol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable

# Policy Change - Audit Policy Change (EventID 4719)
auditpol /set /subcategory:"Audit Policy Change" /success:enable /failure:enable

# System - Security State Change & System Integrity (EventID 4608, 4616)
auditpol /set /subcategory:"Security State Change" /success:enable /failure:enable
auditpol /set /subcategory:"System Integrity" /success:enable /failure:enable
```

---

## 2. Enable Command-Line Process Auditing (EventID 4688)

By default, EventID 4688 records the executable name but omits command-line arguments. To capture full parameters (e.g. `powershell.exe -enc ...`):

```powershell
# Enable command-line argument logging in EventID 4688
New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" -Force | Out-Null
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
  -Name "ProcessCreationIncludeCmdLine_Enabled" -Value 1 -Type DWord
```

Verify with:
```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
  -Name "ProcessCreationIncludeCmdLine_Enabled"
```

---

## 3. PowerShell Enhanced Telemetry (ScriptBlock & Module Logging)

PowerShell is a primary attack vector for living-off-the-land techniques. Enable deep script block auditing (`EventID 4104`) and module execution (`EventID 4103`):

```powershell
# Create Policy Registry Paths
$psPolicyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell"
$sbLoggingPath = "$psPolicyPath\ScriptBlockLogging"
$modLoggingPath = "$psPolicyPath\ModuleLogging"
$modNamesPath = "$modLoggingPath\ModuleNames"

New-Item -Path $sbLoggingPath -Force | Out-Null
New-Item -Path $modNamesPath -Force | Out-Null

# Enable ScriptBlock Logging (Logs de-obfuscated script content to Microsoft-Windows-PowerShell/Operational)
Set-ItemProperty -Path $sbLoggingPath -Name "EnableScriptBlockLogging" -Value 1 -Type DWord

# Enable Module Logging (Captures all pipeline execution)
Set-ItemProperty -Path $modLoggingPath -Name "EnableModuleLogging" -Value 1 -Type DWord
Set-ItemProperty -Path $modNamesPath -Name "*" -Value "*" -Type String
```

---

## 4. Windows Firewall Configuration

### On Physical Host (Splunk Indexer):
Open TCP port `9997` inbound so the forwarder can stream logs:
```powershell
New-NetFirewallRule -DisplayName "Splunk Ingestion Port 9997" `
  -Direction Inbound -LocalPort 9997 -Protocol TCP -Action Allow -Profile Any
```

### On `win10-victim` (Victim VM):
Enable RDP (Port 3389) for brute-force attack simulation:
```powershell
# Enable Remote Desktop service
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

---

## 5. Kali Linux Adversary Network Configuration

Assign static IP `192.168.10.150` on `kali-adversary`:

```bash
# Using NetworkManager CLI (nmcli)
sudo nmcli connection modify "Wired connection 1" \
  ipv4.addresses 192.168.10.150/24 \
  ipv4.gateway 192.168.10.1 \
  ipv4.dns "1.1.1.1,8.8.8.8" \
  ipv4.method manual

# Reapply connection
sudo nmcli connection up "Wired connection 1"

# Verify connectivity to victim
ping -c 3 192.168.10.100
```

