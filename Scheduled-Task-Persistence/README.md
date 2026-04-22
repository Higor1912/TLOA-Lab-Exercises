# Persistence — Scheduled Task / Job

**ATT&CK Technique:** [T1053.005 — Scheduled Task](https://attack.mitre.org/techniques/T1053/005/)  
**Tactic:** Persistence, Privilege Escalation  
**Environment:** TLOA Lab (`tloa.local`)  
**Target:** Windows 10 Pro (domain-joined, Sysmon + Wazuh Agent)  
**Date:** 2026-04-22  
**Status:** ✅ Complete  

---

## Objective

Execute scheduled task persistence on the Windows 10 target using Atomic Red Team (T1053.005), verify that the tasks survive reboots, confirm active execution, and perform cleanup — simulating the full lifecycle of a persistence mechanism: implant → survive → detect → remediate.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Atomic Red Team | T1053.005 test execution |
| PowerShell | Task enumeration and cleanup |
| Wazuh | Alert monitoring |
| Sysmon | Event logging (Event ID 1, 11) |

---

## Execution

### Step 1 — Run Atomic Red Team T1053.005

Executed on the Windows 10 target via PowerShell (Administrator):

```powershell
Invoke-AtomicTest T1053.005
```

Atomic Red Team created multiple scheduled tasks simulating different persistence trigger conditions.

---

### Step 2 — Persistence confirmed across reboots

After rebooting the Windows 10 VM, the scheduled tasks continued to fire — opening `calc.exe` and other processes automatically. This confirmed that the persistence mechanism survived the reboot cycle, which is the primary goal of T1053.005.

**This was observed organically during a later lab session**, when the calculator and other programs started opening unexpectedly — a realistic simulation of how defenders discover persistence in production environments.

---

### Step 3 — Task enumeration (discovery)

Enumerated all suspicious scheduled tasks on the target:

```powershell
Get-ScheduledTask | Where-Object {
    $_.TaskName -like "*atomic*" -or
    $_.TaskName -like "*red*" -or
    $_.Description -like "*atomic*"
}
```

**Tasks discovered:**

| Task Name | Trigger | Notes |
|---|---|---|
| `atomic red team` | Unknown | Generic ART task |
| `ATOMIC-T1053.005` | Scheduled | Primary persistence task |
| `T1053_005_OnLogon` | On Logon | Fires on every user logon |
| `T1053_005_OnStartup` | On Startup | Fires on every system boot |
| `T1053_005_WMI` | WMI Event | WMI-based trigger |
| `spawn` | Unknown | Additional suspicious task |
| `CompMgmtBypass` | Unknown | UAC bypass related |
| `EventViewerBypass` | Unknown | UAC bypass related |

> **Finding:** 8 malicious tasks were created by a single `Invoke-AtomicTest` call, covering multiple trigger types (logon, startup, scheduled, WMI). This demonstrates how a single persistence technique can fan out into multiple footholds.

---

### Step 4 — Cleanup / Remediation

```powershell
$tasks = @(
    "atomic red team",
    "ATOMIC-T1053.005",
    "T1053_005_OnLogon",
    "T1053_005_OnStartup",
    "T1053_005_WMI",
    "spawn",
    "CompMgmtBypass",
    "EventViewerBypass"
)

foreach ($task in $tasks) {
    Unregister-ScheduledTask -TaskName $task -Confirm:$false -ErrorAction SilentlyContinue
    Write-Host "Removed: $task"
}
```

Post-cleanup verification returned empty — all tasks successfully removed.

---

## Evidence

| Screenshot | Description |
|---|---|
| `img/01-atomic-execution.png` | Atomic Red Team T1053.005 execution |
| `img/02-calc-popup.png` | Calculator opening automatically (persistence firing) |
| `img/03-task-enumeration.png` | PowerShell showing 8 malicious scheduled tasks |
| `img/04-cleanup-confirmed.png` | All tasks removed, verification query returns empty |

---

## Detection — Wazuh / Sysmon

### Was it detected?

| SIEM | Alert | Event ID | Result |
|---|---|---|---|
| Sysmon | Process Create — schtasks.exe | 1 | ✅ Generated |
| Sysmon | File Created — task XML | 11 | ✅ Generated |
| Windows Security | Scheduled Task Created | 4698 | ✅ Generated |
| Wazuh | Suspicious scheduled task | — | <!-- ✅ / ❌ --> |

### Key Windows Events

**Event ID 4698** — A scheduled task was created:
```
Task Name: ATOMIC-T1053.005
Task Content: <Actions><Exec><Command>calc.exe</Command></Exec></Actions>
Subject: TLOA\higor
```

**Sysmon Event ID 1** — Process Create:
```
Image: C:\Windows\System32\schtasks.exe
CommandLine: schtasks /create /tn "ATOMIC-T1053.005" ...
User: TLOA\higor
```

### Detection Analysis

The persistence was not immediately discovered through SIEM alerts — it was discovered **behaviorally**, when unexpected processes (calculator, other programs) started executing automatically after reboots. This mirrors real-world discovery patterns where defenders notice anomalous behavior before correlating it to a specific alert.

This highlights an important detection gap: **scheduled task creation may generate alerts, but defenders must also monitor for unexpected process executions tied to system events (logon, startup).**

---

## Lessons Learned

- A single `Invoke-AtomicTest T1053.005` call creates **8 distinct persistence tasks** covering multiple trigger types — demonstrating the breadth of the technique
- Persistence survived multiple reboots before being discovered, simulating realistic dwell time
- Discovery happened through **behavioral observation** (unexpected process execution), not through a SIEM alert — a common real-world pattern
- Cleanup requires explicitly enumerating and removing each task; rebooting alone does not remove scheduled tasks
- `CompMgmtBypass` and `EventViewerBypass` tasks suggest T1053.005 was paired with T1548.002 (UAC Bypass) in the Atomic test — both techniques compound each other

---

## MITRE ATT&CK Context

Scheduled tasks are one of the most commonly abused persistence mechanisms in real-world intrusions. Threat actors including **Kimsuky** and **BlackBasta** use scheduled tasks for persistence after initial access.

**Detection recommendations:**
- Monitor Event ID 4698 (task created) and 4702 (task updated)
- Alert on `schtasks.exe` spawned by non-system processes
- Baseline expected scheduled tasks and alert on deviations
- Monitor for processes spawned by `taskeng.exe` or `svchost.exe -k netsvcs` with unusual parent-child relationships

---

## References

- [MITRE ATT&CK — T1053.005](https://attack.mitre.org/techniques/T1053/005/)
- [Atomic Red Team — T1053.005](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1053.005/T1053.005.md)
- [NSA/CISA — Detecting Abuse of Scheduled Tasks](https://media.defense.gov/2020/Jun/09/2002313081/-1/-1/0/CSI-DETECT-AND-PREVENT-WEB-SHELL-MALWARE.PDF)
