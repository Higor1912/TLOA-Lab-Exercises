# OSINT — ControlID Infrastructure Recon

**ATT&CK Technique:** [T1596 — Search Open Technical Databases](https://attack.mitre.org/techniques/T1596/) · [T1590 — Gather Victim Network Information](https://attack.mitre.org/techniques/T1590/)  
**Tactic:** Reconnaissance  
**Target:** `controlid.com.br` (ASSA ABLOY VDP — Bugcrowd)  
**Scope:** Passive and semi-passive reconnaissance only  
**Date:** <!-- YYYY-MM-DD -->  
**Status:** 🔄 In Progress  

---

## Objective

Map the external attack surface of `controlid.com.br` using passive and semi-passive OSINT techniques — subdomain enumeration, ASN lookup, certificate transparency, and Shodan — within the scope of the ASSA ABLOY VDP on Bugcrowd.

> All recon was conducted within the authorized scope of the Bugcrowd VDP program.

---

## Tools Used

| Tool | Purpose |
|---|---|
| `subfinder` / `amass` | Subdomain enumeration |
| `crt.sh` | Certificate transparency logs |
| Shodan | Internet-exposed assets |
| `whois` / `bgp.he.net` | ASN and IP range mapping |
| `httpx` | HTTP probing on discovered subdomains |
| `waybackurls` | Historical URL discovery |

---

## Execution

### Step 1 — Certificate Transparency

```bash
curl -s "https://crt.sh/?q=%25.controlid.com.br&output=json" | jq '.[].name_value' | sort -u
```

---

### Step 2 — Subdomain Enumeration

```bash
subfinder -d controlid.com.br -silent -o subdomains.txt
```

---

### Step 3 — ASN / IP range

```bash
whois -h whois.radb.net -- '-i origin AS<NUMBER>'
```

---

### Step 4 — Shodan

```bash
shodan search 'org:"ControlID"' --fields ip_str,port,hostnames
```

---

### Step 5 — HTTP probing

```bash
cat subdomains.txt | httpx -silent -status-code -title -tech-detect
```

---

## Findings Summary

> Do not include sensitive vulnerability details here — those go directly in Bugcrowd reports.

| Asset | Type | Notes |
|---|---|---|
| <!-- subdomain --> | <!-- SaaS / API / Admin --> | <!-- public / interesting --> |

---

## Evidence

| Screenshot | Description |
|---|---|
| `img/01-crtsh-results.png` | Certificate transparency output |
| `img/02-subfinder.png` | Subdomain enumeration results |
| `img/03-shodan.png` | Shodan results |

---

## Lessons Learned

<!-- Fill after completing recon phase -->

---

## References

- [MITRE ATT&CK — T1596](https://attack.mitre.org/techniques/T1596/)
- [MITRE ATT&CK — T1590](https://attack.mitre.org/techniques/T1590/)
- [ASSA ABLOY VDP — Bugcrowd](https://bugcrowd.com/assaabloy)
- [crt.sh — Certificate Transparency](https://crt.sh/)
