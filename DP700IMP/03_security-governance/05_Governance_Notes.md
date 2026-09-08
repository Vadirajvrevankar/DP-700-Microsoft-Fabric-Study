# DP-700 — Governance Notes

## 1. BIG PICTURE

Think about governance as four questions:

```text
GOVERNANCE
    |
    +── Sensitivity Labels
    |      → How sensitive is the data?
    |
    +── Endorsements
    |      → Can I trust this content?
    |
    +── Auditing
    |      → What happened?
    |
    +── Retention
           → How long do we keep audit records?
```

### Memory Trick

> **LABEL = SENSITIVITY**  
> **ENDORSEMENT = TRUST**  
> **AUDIT = WHAT HAPPENED**  
> **RETENTION = HOW LONG**

---

# 2. SENSITIVITY LABELS

Sensitivity labels classify data based on how sensitive it is.

Examples:

```text
Public
Internal
Confidential
Highly Confidential
```

### Main Question

> **How sensitive is this content?**

### Memory Trick

> **SENSITIVITY LABEL = CLASSIFICATION**

---

# 3. HOW SENSITIVITY LABELS ARE APPLIED

Sensitivity labels can be applied:

- Manually
- Through a default policy
- Programmatically

Conceptually:

```text
Sensitivity Label
      |
 +----+----+
 |    |    |
Manual Policy Programmatic
```

---

# 4. SENSITIVITY LABEL INHERITANCE

This is a very important exam topic.

```text
Fabric → Fabric       ✅
Fabric → Power BI     ✅
Power BI → Fabric     ❌
```

The most important direction to memorize:

> **Power BI → Fabric does NOT inherit the label.**

### Memory Trick

> **FABRIC → POWER BI = YES**

> **POWER BI → FABRIC = NO**

---

# 5. SENSITIVITY LABEL EXAMPLE

Suppose:

```text
Fabric Lakehouse
      ↓
Sensitivity Label = Confidential
      ↓
Power BI content
```

The sensitivity label can inherit downstream.

But:

```text
Power BI
   ↓
Fabric
```

does not inherit the sensitivity label in that direction.

### Exam Trap

> "Sensitivity labels flow both ways between Fabric and Power BI."

**WRONG**

Remember:

> **Power BI → Fabric = unsupported**

---

# 6. DOMAIN-LEVEL DEFAULT SENSITIVITY LABEL

You can have a tenant-level default sensitivity label.

Sometimes a particular domain needs a stricter default.

Example:

```text
Tenant Default
      ↓
Internal
```

But:

```text
Finance Domain
      ↓
Confidential
```

The domain-level default can override the tenant default **within that domain's scope**.

---

# 7. IMPORTANT REQUIREMENT

A domain admin cannot simply configure a domain-level default label without the required tenant setting.

The flow is:

```text
Tenant Admin
      ↓
Enable tenant-wide feature
      ↓
Domain Admin
      ↓
Set domain default label
```

### Memory Trick

> **TENANT ENABLES → DOMAIN CONFIGURES**

---

# 8. BEST PRACTICE — DOMAIN DEFAULT LABELS

Do not create different domain defaults unnecessarily.

Use a domain-level default only when the domain genuinely needs a stricter policy than the tenant default.

### Memory Trick

> **STRICTER NEED → DOMAIN DEFAULT**

> **NO SPECIAL NEED → KEEP TENANT DEFAULT**

---

# 9. ENDORSEMENTS

Endorsements answer:

> **Can I trust this content?**

Important endorsement levels:

```text
Promoted
    ↓
Certified
    ↓
Master data
```

The key is to remember **who can apply each**.

---

# 10. PROMOTED

Promoted means the content is useful/recommended.

Requirement:

> **Any write-permission holder can promote content.**

Conceptually:

```text
Write permission
      ↓
Promoted
```

### Memory Trick

> **PROMOTED → WRITE**

---

# 11. CERTIFIED

Certified indicates content has been reviewed and is considered trustworthy.

Requirement:

> **Authorized reviewer**

Conceptually:

```text
Write permission
       +
Authorized reviewer
       ↓
Certified
```

Not every person with write permission can necessarily certify content.

### Memory Trick

> **CERTIFIED → REVIEWED**

---

# 12. MASTER DATA

Master data is for genuinely trustworthy/reviewed data.

Requirement:

> **Authorized reviewer + data item**

The study material specifies that Master data applies to **data items only**.

### Memory Trick

> **MASTER DATA → TRUSTED DATA**

---

# 13. ENDORSEMENT COMPARISON

| Endorsement | Who can apply? | Purpose |
|---|---|---|
| **Promoted** | Any write-permission holder | Recommend useful content |
| **Certified** | Authorized reviewers | Indicate reviewed/trusted content |
| **Master data** | Authorized reviewers | Trusted master data; data items only |

### Super Memory Trick

```text
PROMOTED
→ WRITE

CERTIFIED
→ AUTHORIZED REVIEWER

MASTER
→ AUTHORIZED REVIEWER + DATA ITEM
```

---

# 14. WHY OVER-CERTIFICATION IS BAD

If everything is marked Certified:

```text
CERTIFIED
CERTIFIED
CERTIFIED
CERTIFIED
```

users cannot easily distinguish genuinely reviewed content from ordinary content.

Therefore:

> **Only certify/master data when it has genuinely been reviewed.**

### Memory Trick

> **TOO MUCH CERTIFICATION → CERTIFICATION LOSES VALUE**

---

# 15. AUDITING

Auditing answers:

> **What happened?**

Fabric activity auditing uses the:

> **Microsoft Purview unified audit log**

Conceptually:

```text
Fabric Activity
      ↓
Unified Audit Log
      ↓
Microsoft Purview
```

---

# 16. SEARCHING AUDIT DATA

Audit information can be searched through the UI or programmatically.

The study material specifically mentions:

```text
Search-UnifiedAuditLog
```

Conceptually:

```text
Fabric Activity
      ↓
Unified Audit Log
      ↓
Search-UnifiedAuditLog
```

### Memory Trick

> **AUDIT SEARCH → Search-UnifiedAuditLog**

---

# 17. POWER BI ADMIN PORTAL AUDITING

This is an important exam trap.

Having tenant-wide unified audit logging does **not automatically mean** the required Fabric/Power BI events are being captured.

You also need:

> **Power BI admin portal auditing enabled**

Conceptually:

```text
Unified Audit Logging
        +
Power BI Admin Portal Auditing
        ↓
Fabric / Power BI activity captured
```

### Memory Trick

> **AUDIT LOG ≠ AUTOMATIC CAPTURE**

> **ENABLE POWER BI AUDITING**

---

# 18. AUDIT RETENTION

Audit records are not kept forever.

According to the study material:

| License / Setup | Default retention |
|---|---:|
| **Non-E5** | 180 days |
| **E5** | 1 year |
| **Add-on** | Up to 10 years |

### Important

The older:

> **90-day figure**

is not the current figure in this study material.

### Memory Trick

> **180 DAYS → NON-E5**

> **1 YEAR → E5**

> **10 YEARS → ADD-ON**

---

# 19. CUSTOM AUDIT RETENTION

If your organization has compliance requirements, configure custom retention **before** records expire.

Conceptually:

```text
Audit Records
      ↓
Retention Policy
      ↓
Keep for required period
```

### Important

If a record has already expired:

> You cannot retroactively recover it by extending the retention policy later.

### Memory Trick

> **EXPIRED = GONE**

> **RETENTION MUST BE SET BEFORE EXPIRY**

---

# 20. COMPLETE GOVERNANCE FLOW

Think about governance as four questions:

```text
1. HOW SENSITIVE?
       ↓
Sensitivity Label

2. CAN I TRUST IT?
       ↓
Endorsement

3. WHAT HAPPENED?
       ↓
Audit

4. HOW LONG KEEP RECORD?
       ↓
Retention
```

---

# 21. COMMON EXAM SCENARIOS

## Scenario 1

> Fabric content has a sensitivity label and is consumed in Power BI.

### Answer

**Fabric → Power BI = YES**

---

## Scenario 2

> Power BI content has a sensitivity label and is moved to Fabric.

### Answer

**Power BI → Fabric = NO**

---

## Scenario 3

> Domain needs a stricter default sensitivity label than the tenant.

### Answer

Use:

**Domain-level default sensitivity label**

But the required tenant-wide feature must first be enabled.

---

## Scenario 4

> Anyone with write permission should be able to recommend useful content.

### Answer

**Promoted**

---

## Scenario 5

> Content needs formal review before being marked trustworthy.

### Answer

**Certified**

---

## Scenario 6

> Trusted master data needs endorsement.

### Answer

**Master data**

Remember:

> Authorized reviewer + data item.

---

## Scenario 7

> Need to determine what Fabric activity occurred.

### Answer

**Unified audit log**

---

## Scenario 8

> Unified audit logging is enabled, but Fabric/Power BI events are not being captured as expected.

### Check

**Power BI admin portal auditing**

---

## Scenario 9

> Need to search audit activity programmatically.

### Answer

```text
Search-UnifiedAuditLog
```

---

## Scenario 10

> Compliance requires audit records to be retained longer.

### Answer

Configure a **custom audit retention policy before records expire**.

---

# 22. COMMON EXAM TRAPS

## Trap 1

> Sensitivity labels inherit both directions between Fabric and Power BI.

**WRONG**

```text
Fabric → Power BI = YES
Power BI → Fabric = NO
```

---

## Trap 2

> Any write-permission user can certify content.

**WRONG**

Write permission is enough for:

> **Promoted**

Certification requires:

> **Authorized reviewer**

---

## Trap 3

> Master data can be applied to anything.

**WRONG**

The study material specifies:

> **Master data → data items only**

---

## Trap 4

> Tenant audit logging automatically captures all Fabric activity.

**WRONG**

Power BI admin portal auditing also needs to be enabled.

---

## Trap 5

> Audit retention is still 90 days.

**WRONG**

Current study material:

```text
Non-E5 → 180 days
E5     → 1 year
Add-on → up to 10 years
```

---

## Trap 6

> You can extend retention after records have already expired.

**WRONG**

Expired records cannot be recovered through a later retention-policy change.

---

# 23. FINAL MEMORY SHEET

```text
SENSITIVITY LABEL
→ Classification
→ Manual / Policy / Programmatic

INHERITANCE
→ Fabric → Fabric = YES
→ Fabric → Power BI = YES
→ Power BI → Fabric = NO

DOMAIN DEFAULT LABEL
→ Can override tenant default
→ Tenant-wide feature must be enabled first
→ Use only when genuinely needed

ENDORSEMENT
→ Promoted = Write permission
→ Certified = Authorized reviewer
→ Master data = Authorized reviewer + Data item

AUDITING
→ Unified Audit Log
→ Microsoft Purview
→ Search-UnifiedAuditLog

POWER BI AUDITING
→ Must be enabled
→ Tenant unified audit logging alone isn't enough

RETENTION
→ Non-E5 = 180 days
→ E5 = 1 year
→ Add-on = up to 10 years
→ Set custom retention BEFORE expiry
```

---

# 24. ONE-LINE DP-700 EXAM TRICKS

> **How sensitive is the data? → Sensitivity Label**

> **Can I trust the content? → Endorsement**

> **Useful/recommended content? → Promoted**

> **Formally reviewed content? → Certified**

> **Trusted master data? → Master data**

> **Fabric → Power BI label? → YES**

> **Power BI → Fabric label? → NO**

> **Domain-specific stricter label? → Domain default**

> **Who enables domain label delegation? → Tenant Admin**

> **What happened? → Unified Audit Log**

> **Programmatic audit search? → Search-UnifiedAuditLog**

> **Audit capture not working? → Check Power BI admin portal auditing**

> **Non-E5 retention? → 180 days**

> **E5 retention? → 1 year**

> **Extended retention? → Up to 10 years with add-on**

> **Already expired records? → Cannot retroactively recover**

---

# 25. TOP 7 TO MEMORIZE

```text
1. Fabric → Power BI = label inheritance YES

2. Power BI → Fabric = label inheritance NO

3. Promoted = WRITE permission

4. Certified = AUTHORIZED REVIEWER

5. Master data = AUTHORIZED REVIEWER + DATA ITEM

6. Audit = UNIFIED AUDIT LOG
   + Power BI admin portal auditing

7. Retention = 180 days / 1 year / up to 10 years
```

### Final Memory

```text
LABEL       → SENSITIVITY
ENDORSEMENT → TRUST
AUDIT       → ACTIVITY
RETENTION   → TIME
```
