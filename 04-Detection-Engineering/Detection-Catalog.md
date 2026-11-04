# 📖 Detection Catalog

Master repository index of all detection rules engineered, validated, and maintained within the **Enterprise SOC Operations Lab**.

---

## 🛡️ Detection Rule Inventory

| Rule ID | Detection Name | MITRE ATT&CK | Data Source | Primary Index | Severity | Validation Test | Rule Status |
|---|---|---|---|---|---|---|---|
| `DET-ENDP-001` | [Encoded PowerShell Execution](Endpoint/DET-ENDP-001_T1059.001_encoded-powershell.spl) | [T1059.001](https://attack.mitre.org/techniques/T1059/001/) | Sysmon Event ID 1 | `sysmon_logs` | Medium | [`TC-001`](../08-Attack-Simulation/Windows-Attacks/TC-001-Encoded-PowerShell.md) | **`VALIDATED`** |

---

## 📊 Catalog Metrics

* **Total Cataloged Rules**: 1
* **Production / Validated**: 1 (`DET-ENDP-001`)
* **In Progress / Planned**: 0
* **Retired**: 0
