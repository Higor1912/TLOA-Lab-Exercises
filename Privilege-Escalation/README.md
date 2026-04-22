# Privilege Escalation — UAC Bypass

**ATT&CK Technique:** [T1548.002 — Bypass User Account Control](https://attack.mitre.org/techniques/T1548/002/)  
**Tactic:** Privilege Escalation, Defense Evasion  
**Environment:** TLOA Lab (`tloa.local`)  
**Date:** <!-- YYYY-MM-DD -->  
**Status:** ✅ Complete  

---

## Objective

Bypass UAC on the Windows 10 target to elevate from a standard user context to high-integrity (Administrator) without triggering a UAC prompt — enabling subsequent credential dumping and persistence.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Atomic Red Team | T1548.002 test execution |
| PowerShell | UAC bypass technique |

---

## Execution

### Atomic Red Team — T1548.002

```powershell
Invoke-AtomicTest T1548.002
```

---

## Evidence

> Screenshots in `img/` folder.

---

## Detection — Wazuh / Sysmon

| SIEM | Alert | Event ID | Result |
|---|---|---|---|
| Sysmon | Process Create with elevated token | 1 | <!-- ✅ / ❌ --> |
| Wazuh | UAC bypass behavior | — | <!-- ✅ / ❌ --> |

---

## Lessons Learned

<!-- Fill with actual findings -->

---

## References

- [MITRE ATT&CK — T1548.002](https://attack.mitre.org/techniques/T1548/002/)
- [Atomic Red Team — T1548.002](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1548.002/T1548.002.md)
