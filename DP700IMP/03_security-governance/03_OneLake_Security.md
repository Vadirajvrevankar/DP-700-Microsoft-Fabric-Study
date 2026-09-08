# DP-700 — OneLake Security Notes

## 1. BIG PICTURE

OneLake Security controls who can access data stored in OneLake.

```text
User
  ↓
OneLake Security
  ↓
Tables / Folders
  ↓
Data
```

### Memory Trick

> **SQL Security → SQL layer**  
> **OneLake Security → OneLake data layer**

---

# 2. ONELAKE SECURITY = RBAC

OneLake Security uses role-based access control.

```text
User
 ↓
Role
 ↓
Permission
 ↓
OneLake Data
```

It is **deny-by-default**.

### Memory Trick

> **OneLake Security = RBAC + DENY BY DEFAULT**

---

# 3. SUPPORTED ITEMS

OneLake security roles support:

- **Lakehouse**
- **Azure Databricks Mirrored Catalog**
- **Mirrored Database**

### NOT SUPPORTED

- **Warehouse**

### Exam Trick

> **Lakehouse → YES**  
> **Warehouse → NO**

---

# 4. DefaultReader — MOST IMPORTANT

`DefaultReader` is a very important exam trap.

It uses **virtual membership**.

Anyone with **ReadAll** can effectively receive DefaultReader membership.

```text
User has ReadAll
       ↓
Virtual membership
       ↓
DefaultReader
       ↓
Access
```

---

# 5. WHY DefaultReader CAN DEFEAT CUSTOM ROLES

Suppose you create:

```text
RestrictedRole
```

and want limited access.

But DefaultReader is still active:

```text
User
├── RestrictedRole → Limited access
└── DefaultReader → Broad access
```

The user can still receive broader access.

### Best Practice

When creating a custom role intended to narrow access:

> **Delete or restrict DefaultReader deliberately.**

### Memory Trick

> **CUSTOM RESTRICTION NOT WORKING → CHECK DefaultReader FIRST**

---

# 6. WORKSPACE ROLES CAN OVERRIDE ONELAKE READ RESTRICTIONS

Workspace roles:

- Admin
- Member
- Contributor
- Viewer

According to the study material:

```text
Admin       → implicit Write
Member      → implicit Write
Contributor → implicit Write
Viewer      → no implicit Write
```

Therefore:

> Admin, Member, and Contributor's implicit **Write** overrides a OneLake Security **Read** restriction.

### Viewer

Viewer is meaningfully restricted by OneLake Security because Viewer does not have the same implicit Write.

### Memory Trick

> **Admin / Member / Contributor → WRITE WINS**

> **Viewer → OneLake restriction matters**

---

# 7. CUSTOM ONELAKE SECURITY ROLES

Example:

```text
Sales Table

Country | Revenue
India   | 10000
USA     | 20000
UK      | 15000
```

Create:

```text
IndiaAnalyst
      ↓
India data
```

Then:

```text
User
 ↓
IndiaAnalyst
 ↓
India data
```

But always check:

1. DefaultReader
2. Workspace role
3. Other OneLake roles

---

# 8. VIRTUAL MEMBERSHIP vs MANUAL MEMBERSHIP

## Manual Membership

You explicitly maintain users:

```text
Role
 ↓
User A
User B
User C
```

You must maintain the list yourself.

## Virtual Membership

Membership is derived from another permission/group relationship.

```text
Fabric permission/group
        ↓
Virtual membership
        ↓
OneLake role
```

### Advantage

Virtual membership can track Fabric item permission changes automatically.

### Best Practice

> Prefer **virtual membership** when the role should automatically follow Fabric item permission changes.

### Memory Trick

> **AUTO-TRACK PERMISSIONS → VIRTUAL MEMBERSHIP**

---

# 9. SCHEMA-ENABLED LAKEHOUSES

When planning to rely on OneLake Security's RLS/CLS preview capabilities:

> Use a **schema-enabled Lakehouse**.

### Exam Clue

```text
OneLake Security
+
RLS / CLS preview
```

Think:

> **Schema-enabled Lakehouse**

---

# 10. SHORTCUT SECURITY

A shortcut can point to data somewhere else.

```text
Lakehouse A
     |
   Shortcut
     |
     v
Target data
```

The important question is:

> **Whose identity is used to access the target?**

There are two important modes:

1. Passthrough
2. Delegated

---

# 11. PASSTHROUGH IDENTITY

With **passthrough**, the querying user's identity is passed to the target.

```text
User
 ↓
Shortcut
 ↓
Target
```

The target sees:

> **The actual querying user**

Therefore, that user needs appropriate permissions on the target.

### Memory Trick

> **PASSTHROUGH → USER**

---

# 12. DELEGATED IDENTITY

Delegated modes use the **item owner's identity** instead of the querying user's identity.

```text
User
 ↓
Shortcut
 ↓
Item Owner's identity
 ↓
Target
```

Examples mentioned in the study material:

- Direct Lake over SQL
- T-SQL Delegated identity

### Memory Trick

> **DELEGATED → OWNER**

---

# 13. PASSTHROUGH vs DELEGATED

| Mode | Identity used |
|---|---|
| **Passthrough** | Querying user |
| **Delegated** | Item owner |

### Super Memory Trick

> **PASS = USER**  
> **DELEGATE = OWNER**

---

# 14. SQL ANALYTICS ENDPOINT — USER'S IDENTITY ACCESS MODE

If OneLake security consistency across engines is a requirement:

> Enable **User's identity access mode** on the SQL analytics endpoint early.

Conceptually:

```text
SQL Endpoint
      ↓
User's identity
      ↓
OneLake Security
```

### Exam Clue

> "Need consistent OneLake security across engines."

### Answer

**User's identity access mode**

---

# 15. MULTIPLE ONELAKE SECURITY ROLES

A user can belong to multiple roles.

Important rule:

> **Table access across multiple roles is UNIONED.**

Example:

```text
Role A → Table A
Role B → Table B
```

Effective access:

```text
Table A + Table B
```

### Memory Trick

> **ACROSS ROLES → UNION**

---

# 16. RLS / CLS WITHIN A ROLE

Within a role, RLS and CLS restrictions intersect.

```text
Role
 ├── RLS → Department = IT
 └── CLS → Hide Salary
```

The restrictions work together.

### Memory Trick

> **WITHIN ROLE → INTERSECTION**

---

# 17. IMPORTANT CLS EXCEPTION

The study material specifically notes:

> **SQL-endpoint CLS intersects across all of a user's roles.**

So remember:

```text
General OneLake role behavior
→ Table access across roles = UNION
→ RLS/CLS within a role = INTERSECTION

SQL-endpoint CLS
→ Intersects across all user's roles
```

---

# 18. SECURITY DECISION TREE

```text
Question about OneLake data?
             |
            YES
             |
             v
     OneLake Security Role
             |
      +------+------+
      |             |
   Restrict      Shortcut?
   access           |
      |        +----+----+
      |        |         |
      |   Passthrough  Delegated
      |        |         |
      |       USER      OWNER
      |
      v
Check DefaultReader
      |
      v
Check Workspace Role
```

---

# 19. COMMON EXAM SCENARIOS

## Scenario 1

> Custom OneLake role restricts access, but users still see everything.

### Answer

**Check DefaultReader first.**

---

## Scenario 2

> User has ReadAll and therefore receives DefaultReader virtual membership.

### Answer

**DefaultReader uses virtual membership for users with ReadAll.**

---

## Scenario 3

> Admin has a OneLake Read restriction.

### Answer

Admin's implicit **Write** overrides the OneLake Read restriction.

---

## Scenario 4

> Viewer has a OneLake Read restriction.

### Answer

The restriction can meaningfully apply because Viewer does not have the same implicit Write.

---

## Scenario 5

> Which items support OneLake security roles?

### Answer

- Lakehouse
- Azure Databricks Mirrored Catalog
- Mirrored Database

**Not Warehouse.**

---

## Scenario 6

> Shortcut should evaluate the permissions of the person querying the data.

### Answer

**Passthrough**

---

## Scenario 7

> Shortcut should use the owner's identity.

### Answer

**Delegated identity**

---

## Scenario 8

> Need automatic role membership that follows Fabric item permission changes.

### Answer

**Virtual membership**

---

## Scenario 9

> Need OneLake RLS/CLS preview capabilities.

### Answer

**Schema-enabled Lakehouse**

---

## Scenario 10

> Need consistent OneLake security across engines through the SQL analytics endpoint.

### Answer

Enable:

**User's identity access mode**

---

# 20. COMMON EXAM TRAPS

## Trap 1

> "I created a custom role, so DefaultReader no longer matters."

**WRONG**

Always check:

> **DefaultReader**

---

## Trap 2

> OneLake Read restriction will restrict an Admin to Read.

**WRONG**

Admin has implicit Write.

Same principle applies to:

- Admin
- Member
- Contributor

---

## Trap 3

> Warehouse supports OneLake security roles.

**WRONG**

Supported:

- Lakehouse
- Azure Databricks Mirrored Catalog
- Mirrored Database

---

## Trap 4

> Passthrough uses the item owner's identity.

**WRONG**

> **Passthrough → querying user's identity**

---

## Trap 5

> Delegated mode uses the querying user's identity.

**WRONG**

> **Delegated → item owner's identity**

---

## Trap 6

> Multiple OneLake roles intersect their table access.

**WRONG**

Table access across roles is:

> **UNION**

---

# 21. FINAL MEMORY SHEET

```text
ONELAKE SECURITY
→ RBAC
→ DENY BY DEFAULT

SUPPORTED
→ Lakehouse
→ Azure Databricks Mirrored Catalog
→ Mirrored Database

NOT SUPPORTED
→ Warehouse

DefaultReader
→ Virtual membership
→ Anyone with ReadAll
→ Can defeat custom restrictions
→ Delete/restrict deliberately

WORKSPACE ROLE
→ Admin / Member / Contributor
→ Implicit Write
→ Overrides OneLake Read restriction

VIEWER
→ OneLake restriction meaningfully applies

MEMBERSHIP
→ Virtual = automatically tracks permissions
→ Manual = explicitly maintained

SCHEMA-ENABLED LAKEHOUSE
→ OneLake RLS/CLS preview

SHORTCUT
→ Passthrough = USER identity
→ Delegated = OWNER identity

SQL ENDPOINT
→ User's identity access mode
→ Useful for cross-engine consistency

MULTIPLE ROLES
→ Table access = UNION
→ RLS/CLS within role = INTERSECTION

SQL-ENDPOINT CLS
→ Intersects across user's roles
```

---

# 22. ONE-LINE DP-700 EXAM TRICKS

> **Custom role not restricting? → Check DefaultReader**

> **DefaultReader membership? → Anyone with ReadAll**

> **Admin/Member/Contributor + OneLake Read restriction? → Implicit Write wins**

> **Viewer + OneLake restriction? → Restriction matters**

> **OneLake security supported item? → Lakehouse / Mirrored Catalog / Mirrored Database**

> **Warehouse OneLake security role? → NO**

> **Shortcut uses querying user's identity? → Passthrough**

> **Shortcut uses owner's identity? → Delegated**

> **Automatic role membership? → Virtual membership**

> **OneLake RLS/CLS preview? → Schema-enabled Lakehouse**

> **Cross-engine identity consistency? → User's identity access mode**

> **Multiple roles table access? → UNION**

> **Restrictions within a role? → INTERSECTION**

> **SQL-endpoint CLS across roles? → INTERSECTION**

---

# 23. TOP 5 TO MEMORIZE

```text
1. DefaultReader → CHECK FIRST

2. Admin / Member / Contributor
   → implicit WRITE

3. Passthrough
   → USER identity

4. Delegated
   → OWNER identity

5. Multiple roles
   → UNION
```
