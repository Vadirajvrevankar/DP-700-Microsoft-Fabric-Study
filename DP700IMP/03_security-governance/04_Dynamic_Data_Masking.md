# DP-700 — Dynamic Data Masking (DDM) Notes

## 1. WHAT IS DDM?

**Dynamic Data Masking (DDM)** hides sensitive column values when users query the data.

It does **not change the actual stored value**.

```text
Actual value:
9876543210

User sees:
xxxx
```

The database still contains the original value.

### Memory Trick

> **DDM = HIDE THE VALUE**

---

# 2. DDM IS NOT ENCRYPTION

DDM is **not encryption**.

```text
DDM
→ Masks the value
→ Stored value remains unchanged
```

### Exam Trick

> **DDM ≠ Encryption**

DDM is mainly a usability/security safeguard, not cryptographic protection.

---

# 3. DDM IS NOT ACCESS CONTROL

DDM masks values, but it does not prevent access to the column.

```text
User has access
      ↓
Value is masked
```

It does NOT mean:

```text
User has no access
```

For genuinely sensitive data, combine:

```text
RLS + CLS + OLS
        +
       DDM
```

### Memory Trick

> **DDM = HIDE**

> **RLS/CLS/OLS = CONTROL ACCESS**

---

# 4. WHY DDM ALONE IS NOT ENOUGH

Suppose:

```text
Actual Salary = ₹80,000
```

The user sees:

```text
Salary = ******
```

A determined user may still try crafted queries, such as range conditions, to infer information about the masked value.

Therefore:

> **DDM does not prevent inference through crafted WHERE-clause range queries.**

### Most Important DDM Fact

> **MASKED ≠ PROTECTED FROM INFERENCE**

---

# 5. FOUR DDM MASK FUNCTIONS

The four mask functions are:

```text
1. default()
2. email()
3. random(m,n)
4. partial()
```

## 5.1 default()

Provides a standard/general mask.

### Memory

> **default() → STANDARD MASK**

---

## 5.2 email()

Designed for email addresses.

Example:

```text
john.smith@example.com
```

is displayed in a masked email format.

### Memory

> **email() → EMAIL**

---

## 5.3 random(m,n)

Used for numeric values.

```text
random(m,n)
```

generates a random value within the specified range.

### Memory

> **random(m,n) → NUMBERS**

---

## 5.4 partial()

Also known as **Custom String**.

It allows part of a value to remain visible while masking the rest.

Conceptually:

```text
Actual:
1234567890

Masked:
123*****90
```

The exact pattern depends on the mask configuration.

### Memory

> **partial() → SHOW SOME, HIDE SOME**

---

# 6. FOUR FUNCTIONS — EXAM MEMORY

```text
default()     → Standard mask
email()       → Email
random(m,n)   → Numeric random value
partial()     → Partial / Custom String
```

### Super Memory Trick

> **D-E-R-P**

**D**efault  
**E**mail  
**R**andom  
**P**artial

---

# 7. ADDING A MASK TO AN EXISTING COLUMN

Use:

```sql
ALTER TABLE ...
ALTER COLUMN ...
ADD MASKED WITH (FUNCTION = '...')
```

Conceptually:

```text
Existing column
      ↓
ALTER COLUMN
      ↓
ADD MASKED
      ↓
Column is masked
```

---

# 8. REMOVING A MASK

Use:

```sql
DROP MASKED
```

### Memory Trick

```text
ADD MASKED  → Add mask
DROP MASKED → Remove mask
```

---

# 9. UNMASK PERMISSION

A user with:

```sql
GRANT UNMASK
```

can see the **real value** instead of the masked value.

```text
DDM
 ↓
Masked value

UNMASK
 ↓
Real value
```

### Memory Trick

> **UNMASK → REAL VALUE**

---

# 10. GRANT UNMASK TO ROLES

Best practice:

> Grant `UNMASK` to **roles**, not individual users.

Instead of:

```text
User A → UNMASK
User B → UNMASK
User C → UNMASK
```

prefer:

```text
Users
  ↓
SQL Role
  ↓
UNMASK
```

Example:

```text
FinanceManagers
      ↓
   UNMASK
```

### Why?

It is much easier to manage permissions at scale.

### Memory Trick

> **UNMASK → ROLE, NOT USER**

---

# 11. CONTROL ALSO REVEALS MASKED VALUES

According to the study material:

```text
UNMASK
   OR
CONTROL on database
```

can reveal real values.

Therefore:

```text
UNMASK  → Real value
CONTROL → Real value
```

---

# 12. WORKSPACE ROLES AND DDM

This is a major exam trap.

Workspace roles:

- Admin
- Member
- Contributor
- Viewer

According to the study material:

```text
Admin
Member
Contributor
      ↓
CONTROL implicitly
      ↓
Can see real masked values
```

Therefore:

> **Admin, Member, and Contributor bypass DDM masking implicitly.**

---

# 13. WHAT ABOUT VIEWER?

Viewer does not have the same implicit `CONTROL`.

Therefore DDM can meaningfully mask values for a Viewer.

Conceptually:

```text
Admin       → Real value
Member      → Real value
Contributor → Real value
Viewer      → Masked value
```

### Memory Trick

> **A-M-C → CONTROL → REAL VALUE**

> **Viewer → DDM can mask**

---

# 14. IMPORTANT EXAM SCENARIO

Question:

> Admin can see the real value even though DDM is configured. How do you prevent this?

Do not immediately change the DDM configuration.

The issue is the workspace role.

Admin has implicit `CONTROL`.

Therefore:

> Fix the **workspace role assignment**.

### Memory Trick

> **ADMIN SEES REAL VALUE → CHECK WORKSPACE ROLE**

---

# 15. sys.masked_columns

Use:

```sql
sys.masked_columns
```

to audit which columns currently have masking configured.

Think:

```text
sys.masked_columns
       ↓
Which columns are masked?
```

### Exam Clue

> "You need to verify which columns currently have DDM configured."

### Answer

> **`sys.masked_columns`**

### Memory Trick

> **MASK AUDIT → sys.masked_columns**

---

# 16. COMPLETE DDM FLOW

```text
Sensitive Column
       ↓
Add MASK
       ↓
DDM
       ↓
User queries data
       ↓
Does user have UNMASK / CONTROL?
       |
    +--+--+
    |     |
   YES    NO
    |     |
    ↓     ↓
 Real   Masked
 value   value
```

---

# 17. DDM + RLS + CLS + OLS

For genuinely sensitive information:

```text
                 Sensitive Data
                       |
          +------------+------------+
          |            |            |
         RLS          CLS          OLS
          |            |            |
        Rows        Columns       Objects
                       |
                       v
                      DDM
                       |
                  Mask values
```

### What each one does

**RLS**

Restrict rows.

**CLS**

Restrict columns.

**OLS**

Restrict objects.

**DDM**

Mask values.

---

# 18. COMMON EXAM SCENARIOS

## Scenario 1

> Hide sensitive values from normal users.

### Answer

**DDM**

---

## Scenario 2

> Completely prevent users from accessing a sensitive column.

### Answer

**CLS**

Not DDM.

---

## Scenario 3

> Hide a value but still allow access to the column.

### Answer

**DDM**

---

## Scenario 4

> User has `UNMASK`.

### Result

User sees the **real value**.

---

## Scenario 5

> User has database `CONTROL`.

### Result

User can see the **real masked values**.

---

## Scenario 6

> Admin can see the real value despite DDM.

### Why?

Admin has implicit **CONTROL**.

---

## Scenario 7

> You need to know which columns currently have masks.

### Answer

```sql
sys.masked_columns
```

---

## Scenario 8

> Hundreds of users need permission to see unmasked values.

### Best Practice

Grant:

```text
UNMASK → SQL Role
```

not individual users.

---

## Scenario 9

> User tries to infer a masked numeric value using range conditions.

### Answer

DDM does **not** prevent this type of inference.

---

# 19. COMMON EXAM TRAPS

## Trap 1

> DDM is encryption.

**WRONG**

> DDM ≠ encryption.

---

## Trap 2

> DDM prevents users from accessing the column.

**WRONG**

DDM masks the value.

CLS controls column access.

---

## Trap 3

> DDM prevents inference attacks.

**WRONG**

Range queries can potentially be used to infer information.

---

## Trap 4

> Admin is protected by DDM like a normal Viewer.

**WRONG**

Admin has implicit `CONTROL`.

---

## Trap 5

> Grant UNMASK to every user.

**NOT BEST PRACTICE**

Prefer:

```text
Users → Role → UNMASK
```

---

## Trap 6

> DDM alone provides strong protection for sensitive data.

**WRONG**

Use:

```text
RLS / CLS / OLS + DDM
```

when genuine protection is required.

---

# 20. FINAL MEMORY SHEET

```text
DDM
→ Dynamic Data Masking
→ Masks VALUES
→ Does NOT change stored data
→ NOT encryption
→ NOT access control

FOUR FUNCTIONS
→ default()
→ email()
→ random(m,n)
→ partial()

ADD MASK
→ ALTER TABLE ... ALTER COLUMN ... ADD MASKED

REMOVE MASK
→ DROP MASKED

UNMASK
→ Reveals real value

CONTROL
→ Also reveals real value

WORKSPACE ROLES
→ Admin / Member / Contributor
→ Implicit CONTROL
→ Can bypass masking

VIEWER
→ DDM can meaningfully mask

BEST PRACTICE
→ UNMASK to ROLES
→ Combine DDM with RLS/CLS/OLS

AUDIT
→ sys.masked_columns

IMPORTANT LIMITATION
→ DDM does NOT prevent inference
→ Crafted WHERE/range queries can reveal information
```

---

# 21. ONE-LINE DP-700 EXAM TRICKS

> **Hide values? → DDM**

> **Restrict rows? → RLS**

> **Restrict columns? → CLS**

> **Restrict objects? → OLS**

> **DDM = encryption? → NO**

> **UNMASK? → Real values**

> **CONTROL? → Real values**

> **Admin/Member/Contributor? → Implicit CONTROL**

> **Grant UNMASK to whom? → Roles**

> **Audit masked columns? → `sys.masked_columns`**

> **Add mask? → `ADD MASKED`**

> **Remove mask? → `DROP MASKED`**

> **Four functions? → `default()`, `email()`, `random(m,n)`, `partial()`**

> **DDM prevents inference? → NO**

> **Genuine sensitive-data protection? → DDM + RLS/CLS/OLS**

---

# 22. TOP 5 TO MEMORIZE

```text
1. DDM → MASK VALUE

2. UNMASK / CONTROL → REAL VALUE

3. Admin / Member / Contributor → CONTROL implicitly

4. DDM does NOT prevent inference

5. DDM + RLS/CLS/OLS → Stronger protection
```
