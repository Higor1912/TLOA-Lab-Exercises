# Credential Dumping — LSASS Memory

**ATT&CK Technique:** [T1003.001 — LSASS Memory](https://attack.mitre.org/techniques/T1003/001/)  
**Tactic:** Credential Access  
**Environment:** TLOA Lab (`tloa.local`)  
**Date:** <!-- YYYY-MM-DD -->  
**Status:** ✅ Complete  

---

## Objective

Dump credentials from LSASS memory on the Windows 10 target to extract NTLM hashes and plaintext credentials — simulating post-exploitation credential harvesting that enables lateral movement.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Mimikatz | Latest | LSASS memory dump and credential extraction |
| Task Manager / ProcDump | — | Alternative dump methods |

---

## Execution

### Step 1 — Elevate to SYSTEM / Admin

Prerequisite: local administrator access on target.

### Step 2 — Dump LSASS with Mimikatz

```powershell
privilege::debug
sekurlsa::logonpasswords
```

### Step 3 — Extract hashes

Look for `NTLM:` entries under each logon session.

---

## Evidence

> Screenshots in `img/` folder.

---

## Detection — Wazuh / Sysmon

| SIEM | Alert | Event ID | Result |
|---|---|---|---|
| Sysmon | Process Access — LSASS | 10 | <!-- ✅ / ❌ --> |
| Wazuh | Mimikatz signature | — | <!-- ✅ / ❌ --> |

**Sysmon Event 10** (ProcessAccess targeting `lsass.exe`) is the primary detection signal.

---

## Lessons Learned

<!-- Fill with your actual findings from the lab run -->

---

## References

- [MITRE ATT&CK — T1003.001](https://attack.mitre.org/techniques/T1003/001/)
- [Mimikatz GitHub](https://github.com/gentilkiwi/mimikatz)
