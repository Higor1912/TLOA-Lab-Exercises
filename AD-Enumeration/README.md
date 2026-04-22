# AD Enumeration — BloodHound + SharpHound

**ATT&CK Technique:** [T1087.002 — Domain Account](https://attack.mitre.org/techniques/T1087/002/) · [T1069.002 — Domain Groups](https://attack.mitre.org/techniques/T1069/002/)  
**Tactic:** Discovery  
**Environment:** TLOA Lab (`tloa.local`)  
**Date:** <!-- YYYY-MM-DD -->  
**Status:** 🔄 In Progress  

---

## Objective

Enumerate Active Directory objects in `tloa.local` using SharpHound to collect data and BloodHound to identify attack paths, privilege escalation vectors, and Kerberoastable accounts — specifically the pre-configured `svc-backup` service account.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| SharpHound | Latest | AD data collection |
| BloodHound | Latest | Graph-based attack path analysis |
| PowerShell | — | Execution on Windows 10 target |

---

## Execution

### Step 1 — Run SharpHound on the target (Windows 10)

SharpHound was executed on the domain-joined Windows 10 machine to collect AD data from `tloa.local`.

```powershell
# Download or transfer SharpHound.exe to target
.\SharpHound.exe -c All --outputdirectory C:\Temp\
```

**Collected data includes:** users, groups, GPOs, sessions, ACLs, trusts.

---

### Step 2 — Transfer the ZIP to Kali

```bash
# From Kali, retrieve the BloodHound zip output
scp user@192.168.204.X:C:/Temp/*.zip ./
```

---

### Step 3 — Import into BloodHound

1. Start Neo4j: `sudo neo4j start`
2. Open BloodHound and log in
3. Upload the ZIP file via the upload button
4. Run query: **"Find all Kerberoastable Users"**

---

### Step 4 — Identify attack paths

Key findings from the BloodHound graph:

- `svc-backup` is Kerberoastable (SPN configured)
- <!-- document additional paths here -->

---

## Evidence

> Add screenshots to `img/` folder.

| Screenshot | Description |
|---|---|
| `img/01-sharpound-run.png` | SharpHound execution output |
| `img/02-bloodhound-graph.png` | Full AD graph in BloodHound |
| `img/03-kerberoastable.png` | Kerberoastable users query result |

---

## Detection — Wazuh / Sysmon

### Was it detected?

| SIEM | Alert | Result |
|---|---|---|
| Wazuh | LDAP enumeration | <!-- ✅ / ❌ --> |
| Sysmon | Event ID 1 (Process Create — SharpHound) | <!-- ✅ / ❌ --> |

### Analysis

<!-- Fill after running the exercise and checking Wazuh dashboard -->

SharpHound generates significant LDAP traffic. Sysmon Event ID 1 should capture the process creation. Wazuh rule 60106 or similar may trigger on suspicious LDAP queries.

---

## Lessons Learned

<!-- Fill after completing the exercise -->

---

## References

- [MITRE ATT&CK — T1087.002](https://attack.mitre.org/techniques/T1087/002/)
- [MITRE ATT&CK — T1069.002](https://attack.mitre.org/techniques/T1069/002/)
- [BloodHound Documentation](https://bloodhound.readthedocs.io/)
- [SharpHound GitHub](https://github.com/BloodHoundAD/SharpHound)
