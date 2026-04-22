# TLOA Lab Exercises

Hands-on offensive security exercises executed in the **TLOA (Threat Lab Offensive Architecture)** home lab environment, with ATT&CK mapping, tooling documentation, and detection analysis via Wazuh + Sysmon.

> All exercises are performed in an isolated, controlled lab environment for educational purposes only.

---

## Lab Environment

| Component | Details |
|---|---|
| **Domain** | `tloa.local` |
| **Network** | VMware Host-Only — VMnet1 (`192.168.204.x`) |
| **DC** | Windows Server 2025 — DC01 (Wazuh Agent 002) |
| **Target** | Windows 10 Pro — Sysmon + Wazuh Agent |
| **Attacker** | Kali Linux |
| **SIEM** | Ubuntu Server 24.04 — Wazuh Manager 4.7.5 |

---

## Exercises

| # | Exercise | ATT&CK Technique | Status |
|---|---|---|---|
| 01 | [AD Enumeration — BloodHound](./AD-Enumeration/) | T1087.002, T1069.002 | ✅ Complete |
| 02 | [Kerberoasting — svc-backup](./Kerberoasting/) | T1558.003 | 🔄 In Progress |
| 03 | [Lateral Movement — Pass-the-Hash](./Lateral-Movement/) | T1550.002, T1021.006 | ⬜ Pending |
| 04 | [Privilege Escalation](./Privilege-Escalation/) | T1548.002 | ✅ Complete |
| 05 | [Credential Dumping — LSASS](./Credential-Dumping/) | T1003.001 | ✅ Complete |
| 06 | [OSINT — ControlID Infrastructure](./OSINT-ControlID/) | T1596, T1590 | 🔄 In Progress |

---

## ATT&CK Coverage

```
Initial Access → Execution → Persistence → Privilege Escalation
     ↓
Credential Access → Discovery → Lateral Movement
```

Techniques documented: **T1003.001 · T1046 · T1053.005 · T1087.002 · T1112 · T1548.002 · T1550.002 · T1558.003 · T1590 · T1596**

---

## Tools Used

`BloodHound` `SharpHound` `Impacket` `Mimikatz` `CrackMapExec` `hashcat` `nmap` `Wazuh` `Sysmon` `Atomic Red Team`

---

## Related Repositories

- [Threat-Intel-Reports](https://github.com/H160R2/Threat-Intel-Reports)
- [Bash-Log-Threat-Analyzer](https://github.com/H160R2/Bash-Log-Threat-Analyzer)
