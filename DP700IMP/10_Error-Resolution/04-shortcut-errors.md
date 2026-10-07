# DP-700 Microsoft Fabric — OneLake Security, SQL Identity Modes & Shortcuts

## Overview

These notes cover:

- SQL analytics endpoint identity modes
- User identity / passthrough
- Delegated identity
- OneLake security
- RLS, CLS, OLS
- Authentication and authorization errors
- Delegated-token caching
- OneLake security synchronization
- Cached query results
- Transitive shortcuts
- Shortcut limits
- Security behavior across Spark and SQL
- Direct Lake over SQL / T-SQL delegated mode
- Mode-switch impact on metadata and SQL roles

---

# 1. SQL Analytics Endpoint Identity Modes

A SQL analytics endpoint can use different identity behaviors when accessing OneLake data.

The two important concepts in these notes are:

```text
User identity / Passthrough
```

and:

```text
Delegated identity
```

The key question is:

> **Whose identity is used when accessing the target data?**

---

# 2. User Identity / Passthrough

**Passthrough** means the calling user's identity is passed through to the target.

Conceptually:

```text
User
 ↓
SQL analytics endpoint
 ↓
User's identity
 ↓
Target OneLake data
```

The target sees the **calling user**.

The supplied study material identifies this as the default for:

> **OneLake-to-OneLake**

---

# 3. Delegated Identity

With **delegated mode**, the connection uses a configured identity instead of the calling user's identity.

Conceptually:

```text
User
 ↓
SQL analytics endpoint
 ↓
Configured connection identity
 ↓
Target
```

The important distinction is:

```text
Passthrough
    ↓
Calling user's identity

Delegated
    ↓
Configured connection identity
```

---

# 4. External Shortcuts and Delegated Mode

The supplied exam material states:

> **Delegated identity is required for all external shortcuts.**

Therefore:

```text
OneLake-to-OneLake
    ↓
Passthrough is the default
```

while:

```text
External shortcut
    ↓
Delegated identity required
```

### Exam memory

> **Passthrough = calling user**

> **Delegated = configured connection identity**

---

# 5. OneLake Security

OneLake security can enforce:

- **RLS** — Row-Level Security
- **CLS** — Column-Level Security
- **OLS** — Object-Level Security

These determine what a user is allowed to see/access.

---

# 6. RLS — Row-Level Security

RLS controls:

> **Which rows a user can see.**

Example:

```text
Sales table

India
USA
UK
```

A user might be allowed to see only:

```text
India
```

Conceptually:

```text
Table
 ↓
RLS
 ↓
Only authorized rows visible
```

---

# 7. CLS — Column-Level Security

CLS controls:

> **Which columns a user can access.**

Example:

```text
CustomerID
Name
Salary
Address
```

A user may be allowed to see:

```text
CustomerID
Name
Address
```

but not:

```text
Salary
```

---

# 8. OLS — Object-Level Security

OLS controls access to objects.

Think:

```text
Object
 ↓
Is the user allowed to access it?
```

The supplied material groups:

```text
RLS
CLS
OLS
```

as OneLake security controls that need to be considered when choosing identity mode.

---

# 9. Why User Identity Mode Is Preferred for OneLake Security

The study recommendation is:

> Default to **user identity mode** for SQL analytics endpoints whenever OneLake security must be enforced consistently across engines.

Why?

Because the calling user's identity is preserved.

Conceptually:

```text
User
 ↓
SQL
 ↓
OneLake
 ↓
Evaluate user's permissions
 ↓
RLS / CLS / OLS
```

This provides consistent user-specific enforcement.

### Memory

> **Need OneLake security consistently → prefer user identity mode.**

---

# 10. Delegated Mode and OneLake Security

The supplied material states:

> Delegated mode blocks shortcuts to sources with OneLake RLS/CLS/OLS.

This is **by design**, not a platform bug.

Conceptually:

```text
Delegated mode
      ↓
Shortcut source has OneLake RLS/CLS/OLS
      ↓
Blocked
```

### Exam trap

Do NOT interpret this as:

> "The shortcut is broken."

Think:

> **Delegated mode is incompatible with those protected shortcut sources according to the supplied rules.**

---

# 11. Identity Modes — Comparison

| Feature | User identity / Passthrough | Delegated |
|---|---|---|
| Identity used | Calling user | Configured connection identity |
| OneLake-to-OneLake default | Yes | No |
| External shortcuts | Not the required mode | Required |
| User-specific security | Preserved | Uses configured identity |
| OneLake RLS/CLS/OLS on protected shortcut source | Can be enforced with user identity | Blocks the shortcut according to these notes |

### Memory

> **Passthrough = USER**

> **Delegated = CONFIGURED IDENTITY**

---

# 12. Authentication vs Authorization

This is essential for understanding:

```text
401
403
404
```

They are NOT the same.

---

# 13. HTTP 401 — Authentication Problem

```text
401
```

means:

> **No valid authentication/credentials.**

Think:

```text
Who are you?
   ↓
Authentication failed
   ↓
401
```

Examples conceptually:

- Missing credential
- Invalid/expired authentication

### Memory

> **401 = Not authenticated**

---

# 14. HTTP 403 — Authorization Problem

```text
403
```

means:

> The identity is authenticated, but it is **not authorized** to perform the operation.

Think:

```text
Who are you?
   ↓
Known/authenticated
   ↓
Are you allowed?
   ↓
NO
   ↓
403
```

### Memory

> **403 = Authenticated, but not authorized**

---

# 15. HTTP 404 — Target Problem

```text
404
```

means the target cannot be found.

The supplied material specifically highlights:

- Target moved
- Target renamed
- Target deleted

Conceptually:

```text
Shortcut
 ↓
Target
 ↓
Target moved/renamed/deleted
 ↓
404
```

### Memory

> **404 = Target not found**

---

# 16. 401 vs 403 vs 404

| Code | Meaning | Think |
|---:|---|---|
| **401** | No valid authentication | Who are you? |
| **403** | Authenticated but not authorized | Are you allowed? |
| **404** | Target unavailable/not found | Where is the target? |

### Ultimate memory

> **401 = Authentication**

> **403 = Authorization**

> **404 = Target**

---

# 17. Delegated Credential Cache

Delegated authentication can involve cached storage tokens.

The supplied material identifies a cache window of:

> **30–60 minutes**

This can explain situations where a permission change doesn't appear immediately.

Example:

```text
Permission revoked
      ↓
Expected: access immediately disappears
      ↓
Cached delegated token still exists
      ↓
Access may remain temporarily visible
```

---

# 18. Why a Revoked User May Still See Data

Suppose:

```text
10:00
User has access
```

Then:

```text
10:05
Access revoked
```

But a cached delegated token may remain valid temporarily.

The supplied guidance says:

> Storage-token cache can last approximately **30–60 minutes**.

Therefore:

```text
Revocation
    ↓
Cache still active
    ↓
Temporary continued visibility/access
```

This should not automatically be treated as a security breach.

### Incident runbook recommendation

Document the:

> **30–60 minute delegated-token cache window**

so support teams know to account for this behavior.

---

# 19. OneLake Security Synchronization

The supplied material states:

> OneLake security synchronization can take **up to 5 minutes**.

So there are two different timing concepts:

```text
Delegated storage-token cache
        ↓
30–60 minutes
```

and:

```text
OneLake security synchronization
        ↓
Up to 5 minutes
```

Don't confuse them.

---

# 20. Cached Query Results

The supplied notes also identify:

> Cached query results across an identity-mode switch can persist for **up to 1 hour**.

So you can have:

```text
Mode switch
   ↓
Cached query result
   ↓
Potentially visible for up to 1 hour
```

### Important timing numbers

Memorize:

```text
30–60 min → delegated storage-token cache
Up to 5 min → OneLake security sync
Up to 1 hour → cached query results across mode switch
```

---

# 21. Why Timing Matters in Incident Troubleshooting

Suppose someone says:

> "I revoked access, but the user can still see the data."

Don't immediately conclude:

```text
Security breach
```

First consider:

```text
Token cache?
Security sync delay?
Cached query result?
```

Use the relevant timing window.

### Memory

> **Not every delayed permission change is a security failure.**

---

# 22. Most-Permissive-Wins Behavior

This is an important OneLake security trap.

Suppose a principal receives two roles:

```text
Role A → Restrictive
Role B → Permissive
```

The supplied material says:

> **Most-permissive-wins**

This means the restrictive role can be defeated by the more permissive role.

Conceptually:

```text
Principal
  ├── Restrictive role
  └── Permissive role
          ↓
   Most permissive wins
          ↓
Restrictive RLS may be defeated
```

---

# 23. Why Roles Should Be Mutually Exclusive

To avoid accidentally defeating RLS:

> Keep restrictive and permissive security roles **mutually exclusive per principal**.

Instead of:

```text
User
 ├── Restrictive role
 └── Permissive role
```

make the assignments clear so a principal doesn't receive conflicting roles.

### Exam memory

> **Most-permissive-wins → conflicting role assignments can silently defeat RLS.**

---

# 24. Transitive Shortcuts

A transitive shortcut is a shortcut that points through another shortcut.

Conceptually:

```text
Source
  ↓
Shortcut A
  ↓
Shortcut B
  ↓
Target
```

This creates a shortcut chain.

---

# 25. Shortcut Chain Limits

The supplied exam material states:

> Maximum **5** direct shortcut-to-shortcut links.

So:

```text
Shortcut
 ↓
Shortcut
 ↓
Shortcut
 ↓
Shortcut
 ↓
Shortcut
```

can reach the documented limit.

### Exam memory

> **Maximum transitive shortcut links = 5**

---

# 26. Shortcuts Per OneLake Path

Another limit:

> Maximum **10 shortcuts per single OneLake path**.

So memorize both:

```text
5 → shortcut-to-shortcut chain limit
10 → shortcuts per single OneLake path
```

These are different limits.

---

# 27. Why Avoid Long Shortcut Chains?

Even though the platform allows up to 5 links, the study recommendation is:

> Avoid building chains beyond **2–3 hops** when possible.

Why?

Because shorter chains are easier to:

- Understand
- Troubleshoot
- Maintain
- Diagnose when a target moves

Example:

### Simple

```text
A → B → C
```

Easy to understand.

### Complex

```text
A → B → C → D → E → F
```

If F moves:

```text
Which shortcut is broken?
```

Troubleshooting becomes harder.

### Memory

> **5 = hard platform limit**

> **2–3 = practical design recommendation**

---

# 28. Target Movement and Shortcuts

Suppose:

```text
Shortcut A
    ↓
Shortcut B
    ↓
Target
```

and the target is:

```text
Renamed
Moved
Deleted
```

The shortcut chain can break.

This can result in:

```text
404
```

### Exam connection

```text
404
 ↓
Target moved/renamed/deleted
```

---

# 29. Mode Switching

Switching a SQL analytics endpoint between:

```text
User identity
```

and:

```text
Delegated identity
```

has important consequences.

The supplied material says that switching modes drops:

- Inline metadata objects
  - TVFs
  - Scalar functions
- SQL roles

Therefore, these should be scripted beforehand.

---

# 30. Why Script TVFs and Scalar Functions?

Suppose your SQL analytics endpoint has:

```text
TVF
Scalar function
```

You switch identity mode.

According to the supplied material:

```text
Mode switch
   ↓
Inline metadata objects dropped
```

Therefore:

```text
Before mode switch
      ↓
Script objects
      ↓
Switch mode
      ↓
Recreate if needed
```

---

# 31. Why Script SQL Roles?

The same applies to SQL roles.

Before switching modes:

```text
SQL roles
   ↓
Script them
   ↓
Switch identity mode
   ↓
Recreate/reapply as needed
```

### Exam memory

> **Mode switch → TVFs + scalar functions + SQL roles can be dropped.**

---

# 32. Direct Lake Over SQL / T-SQL Delegated Mode

This is one of the most important engine-identity differences.

The supplied material states:

> Direct Lake over SQL / T-SQL delegated mode uses the **item owner's identity**, not the calling user's identity.

This can produce confusing row-count differences.

---

# 33. Why Spark and SQL Can Show Different Row Counts

Imagine:

```text
Spark
 ↓
Calling user's identity
 ↓
RLS applied
 ↓
100 rows
```

But:

```text
SQL / T-SQL delegated mode
 ↓
Item owner's identity
 ↓
Different permissions/security context
 ↓
500 rows
```

So the same underlying data can produce:

```text
Spark → 100 rows
SQL   → 500 rows
```

This does not necessarily mean the data is corrupted.

The identities used by the engines can differ.

---

# 34. Classic Spark-vs-SQL Trap

If an exam scenario says:

> Spark returns fewer rows than SQL.

Check:

> **Which identity is being used?**

Especially when:

```text
Direct Lake over SQL
+
T-SQL delegated mode
```

the supplied material says the SQL path uses:

> **Item owner's identity**

not the calling user's identity.

### Memory

> **Delegated SQL → Item owner**

---

# 35. Identity and Security by Engine

The supplied material describes an important difference:

```text
Spark
 ↓
Always enforces OneLake security
```

while:

```text
SQL user-identity mode
 ↓
Enforces OneLake security
```

and:

```text
SQL delegated mode
 ↓
Does not enforce the protected OneLake RLS/CLS/OLS behavior
described in these shortcut scenarios
```

### Exam takeaway

OneLake security behavior depends on both:

```text
Engine
+
Identity mode
```

---

# 36. OneLake Security Comparison

| Scenario | Identity/security behavior |
|---|---|
| Spark | OneLake security enforced |
| SQL user identity mode | OneLake security enforced |
| SQL delegated mode | OneLake RLS/CLS/OLS behavior differs; protected shortcut sources are blocked |
| Direct Lake over SQL / T-SQL delegated | Uses item owner's identity |

---

# 37. Full Troubleshooting Flow — Shortcuts

```text
Shortcut fails
     ↓
Check HTTP code
     │
     ├── 401
     │    ↓
     │  Authentication problem
     │
     ├── 403
     │    ↓
     │  Authorization problem
     │
     └── 404
          ↓
       Target moved/renamed/deleted
```

Then check:

```text
Identity mode
      ↓
Passthrough or Delegated?
      ↓
Who is actually accessing the target?
```

---

# 38. Full Troubleshooting Flow — Delayed Security Changes

```text
Permission revoked
       ↓
User still sees data?
       ↓
Check timing
       │
       ├── Delegated token cache
       │       ↓
       │    30–60 min
       │
       ├── OneLake security sync
       │       ↓
       │    Up to 5 min
       │
       └── Cached query results
               ↓
            Up to 1 hour
```

Don't immediately assume a breach.

---

# 39. Full Troubleshooting Flow — Mode Switch

```text
Need to change SQL identity mode
          ↓
Before switching:
          ↓
Script:
  - TVFs
  - Scalar functions
  - SQL roles
          ↓
Switch mode
          ↓
Recreate/reapply required metadata/security
```

---

# 40. Full Shortcut Design Guidance

```text
Shortcut chain
      ↓
Can technically go up to 5 links
      ↓
But prefer 2–3 hops
      ↓
Easier maintenance
      ↓
Easier diagnosis
```

Also remember:

```text
Maximum shortcut-to-shortcut links = 5
Maximum shortcuts per OneLake path = 10
```

---

# 41. DP-700 Exam Tips

## Identity Modes

### Passthrough

> Calling user's identity.

Default for:

> OneLake-to-OneLake.

### Delegated

> Configured connection identity.

Required for:

> External shortcuts.

---

## HTTP Status Codes

```text
401 → Authentication
403 → Authorization
404 → Target not found
```

### Memory

> **401 = Who are you?**

> **403 = Are you allowed?**

> **404 = Where is it?**

---

## Timing Windows

```text
30–60 min → Delegated storage-token cache
Up to 5 min → OneLake security sync
Up to 1 hour → Cached query results across mode switch
```

---

## OneLake Security

Know:

```text
RLS → Rows
CLS → Columns
OLS → Objects
```

And:

> **Most-permissive-wins**

Therefore:

> Keep restrictive and permissive roles mutually exclusive per principal.

---

## Shortcut Limits

```text
5 → maximum shortcut-to-shortcut links
10 → maximum shortcuts per single OneLake path
```

Practical recommendation:

```text
Prefer 2–3 hops
```

---

## Delegated Mode + OneLake Security

Remember:

> Delegated mode blocks shortcuts to sources protected by OneLake RLS/CLS/OLS.

This is:

> **By design, not a bug.**

---

## Direct Lake / SQL Identity Trap

```text
Direct Lake over SQL
+
T-SQL delegated mode
        ↓
Item owner's identity
```

Not:

```text
Calling user's identity
```

This can explain:

> **Spark vs SQL row-count mismatch**

---

## Mode Switching

Before changing SQL endpoint identity mode:

```text
Script TVFs
Script scalar functions
Script SQL roles
```

because the supplied material states these are dropped on a mode switch.

---

# 42. Key Takeaways

- Prefer **user identity mode** when consistent OneLake RLS/CLS/OLS enforcement across engines is required.
- **Passthrough** means the calling user's identity reaches the target.
- **Delegated** means the configured connection identity reaches the target.
- OneLake-to-OneLake uses passthrough by default.
- External shortcuts require delegated identity.
- `401` = authentication problem.
- `403` = authorization problem.
- `404` = target moved/renamed/deleted or otherwise not found.
- Delegated storage tokens can remain cached for **30–60 minutes**.
- OneLake security synchronization can take **up to 5 minutes**.
- Cached query results across a mode switch can persist for **up to 1 hour**.
- Most-permissive-wins can silently defeat restrictive RLS assignments.
- Keep restrictive and permissive security roles mutually exclusive per principal.
- Transitive shortcuts allow up to **5 shortcut-to-shortcut links**, but **2–3 hops** are preferable for maintainability.
- A single OneLake path can have up to **10 shortcuts** according to the supplied material.
- Delegated mode blocks shortcuts to sources protected by OneLake RLS/CLS/OLS.
- `AADSTS`-style identity issues should be distinguished from simple target/path failures.
- Switching SQL identity modes can drop **TVFs, scalar functions, and SQL roles**, so script them first.
- Direct Lake over SQL / T-SQL delegated mode uses the **item owner's identity**, which can explain Spark-vs-SQL row-count differences.

---

# 43. Ultimate Memory Sheet

```text
IDENTITY
 ↓
Passthrough = CALLING USER
Delegated   = CONFIGURED IDENTITY
```

```text
ONE LAKE → ONE LAKE
 ↓
Passthrough is default
```

```text
EXTERNAL SHORTCUT
 ↓
Delegated required
```

```text
HTTP
 ↓
401 = Authentication
403 = Authorization
404 = Target
```

```text
SECURITY TIMING
 ↓
30–60 min = Delegated token cache
Up to 5 min = OneLake security sync
Up to 1 hour = Cached query results
```

```text
ONELAKE SECURITY
 ↓
RLS = ROWS
CLS = COLUMNS
OLS = OBJECTS
```

```text
ROLE ASSIGNMENTS
 ↓
Restrictive + Permissive
        ↓
Most-permissive wins
        ↓
Can defeat RLS
```

```text
SHORTCUTS
 ↓
5 = max transitive shortcut links
10 = max shortcuts per OneLake path
2–3 = preferred practical chain length
```

```text
MODE SWITCH
 ↓
Script first:
TVFs
Scalar functions
SQL roles
 ↓
Switch mode
```

```text
DIRECT LAKE OVER SQL
+
T-SQL DELEGATED
 ↓
ITEM OWNER'S IDENTITY
 ↓
Possible Spark vs SQL row-count mismatch
```

## Final Exam Memory

> **Passthrough = USER**

> **Delegated = CONFIGURED IDENTITY**

> **401 = AUTHENTICATION**

> **403 = AUTHORIZATION**

> **404 = TARGET**

> **Most-permissive-wins = security role trap**

> **5 = shortcut chain limit**

> **10 = shortcuts per OneLake path**

> **2–3 = recommended practical chain length**

> **Mode switch = script TVFs, scalar functions, and SQL roles first**

> **Delegated SQL = item owner's identity**

> **Need consistent OneLake security = prefer user identity mode**
