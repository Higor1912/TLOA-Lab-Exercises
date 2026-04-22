# [Exercise Name]

**ATT&CK Technique:** [T1XXX.XXX — Technique Name](https://attack.mitre.org/techniques/T1XXX/XXX/)  
**Tactic:** [Tactic Name]  
**Environment:** TLOA Lab (`tloa.local`)  
**Date:** YYYY-MM-DD  
**Status:** ✅ Complete / 🔄 In Progress  

---

## Objective

> One or two sentences describing what this exercise aimed to simulate and why it matters.

---

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| Tool A | vX.X | Description |
| Tool B | vX.X | Description |

---

## Execution

### Step 1 — [Name]

Brief explanation of what was done and why.

```bash
# command used
example-command --flag target
```

**Output:**
```
paste relevant output here
```

---

### Step 2 — [Name]

```bash
example-command
```

---

## Evidence

> Add screenshots to an `img/` subfolder and reference them here.

![Screenshot description](./img/screenshot-01.png)

---

## Detection — Wazuh / Sysmon

### Was it detected?

| SIEM | Alert | Rule ID | Result |
|---|---|---|---|
| Wazuh | [Alert name] | XXXXX | ✅ Detected / ❌ Not detected |
| Sysmon | Event ID [X] | — | ✅ Generated / ❌ Not generated |

### Alert Details

```json
{
  "rule": {
    "id": "XXXXX",
    "description": "Alert description here"
  },
  "agent": {
    "name": "DC01"
  }
}
```

### Analysis

Explain what the alert shows (or why it wasn't triggered). What would a SOC analyst see?

---

## Lessons Learned

- What worked as expected
- What surprised you
- What would improve detection

---

## References

- [MITRE ATT&CK — T1XXX](https://attack.mitre.org/techniques/T1XXX/)
- [Reference 2](https://link)
