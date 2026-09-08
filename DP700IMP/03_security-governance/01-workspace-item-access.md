# DP-700 — Fabric Workspace & Item Access Notes

## 1. BIG PICTURE

Fabric security has two main layers:

```text
USER
├── WORKSPACE ROLE
│   ├── Admin
│   ├── Member
│   ├── Contributor
│   └── Viewer
│
└── ITEM PERMISSION
    ├── Read
    ├── ReadAll
    ├── Write
    ├── Reshare
    ├── Execute
    ├── ViewOutput
    └── ViewLogs
```

### Memory Trick

> **WORKSPACE → ROLE**  
> **ONE ITEM → PERMISSION**

---

# 2. WORKSPACE ROLES

| Role | Main purpose |
|---|---|
| **Admin** | Full workspace administration |
| **Member** | Collaborate/manage workspace content |
| **Contributor** | Build and modify content without managing workspace membership |
| **Viewer** | Consume content/read-only access |

### Role hierarchy

**Admin > Member > Contributor > Viewer**

---

# 3. CONTRIBUTOR

Contributor is useful for a developer who needs to:

- Create items
- Modify items
- Build solutions
- Work with workspace content

But should **not** manage workspace membership.

### Exam clue

> User needs to build content but should not add/remove workspace members.

### Answer

**Contributor**

### Memory Trick

> **CONTRIBUTOR = BUILD, DON'T MANAGE MEMBERS**

---

# 4. VIEWER

Viewer is mainly for users who need to **consume** content.

Typical examples:

- Business users
- Report consumers
- Dashboard users
- Users who should not modify workspace content

### Important RLS point

When Row-Level Security (RLS) is important, keeping report consumers as **Viewer** can be deliberate.

### Memory Trick

> **VIEWER = CONSUME**

---

# 5. ITEM PERMISSIONS

Item permissions control access to a **specific Fabric item**.

Examples:

- Lakehouse
- Warehouse
- Notebook
- Semantic model
- Pipeline

### Memory Trick

> **WORKSPACE ROLE = BROAD ACCESS**  
> **ITEM PERMISSION = SPECIFIC ACCESS**

---

# 6. IMPORTANT ITEM PERMISSIONS

| Permission | Meaning |
|---|---|
| **Read** | Can read/access the item |
| **ReadAll** | Can read all data through OneLake APIs/Spark |
| **Write** | Can modify the item |
| **Reshare** | Can share access with others |
| **Execute** | Can execute/run the item where applicable |
| **ViewOutput** | Can view execution output |
| **ViewLogs** | Can view execution logs |

### Important

Permissions such as **Reshare, Execute, ViewOutput, and ViewLogs** are not standalone permissions. They are layered on top of basic access such as **Read or Write**.

---

# 7. ReadData vs ReadAll

This is a common DP-700 exam area.

## ReadData

All four workspace roles have **ReadData** access for SQL/TDS scenarios.

```text
Admin
Member
Contributor
Viewer
     ↓
  ReadData
     ↓
 SQL / TDS
```

## ReadAll

By default:

```text
Admin
Member
Contributor
     ↓
  ReadAll
     ↓
 OneLake APIs / Spark
```

Viewer does **not** automatically get ReadAll.

### Memory Trick

> **SQL/TDS → ReadData**

> **Spark/OneLake → ReadAll**

---

# 8. WORKSPACE ROLE vs ITEM PERMISSION

## Workspace Role

Applies broadly to the workspace.

```text
User
 ↓
Workspace
 ↓
Role
```

Example:

> User is a Contributor in Workspace A.

They have contributor-level capabilities across that workspace.

## Item Permission

Applies to one specific item.

```text
User
 ↓
Specific Item
 ↓
Permission
```

Example:

> User gets Read permission on one Warehouse.

They don't automatically become a Contributor of the entire workspace.

### Memory Trick

> **WORKSPACE = BROAD**  
> **ITEM = SPECIFIC**

---

# 9. INDEPENDENT ACCESS GRANTS

Workspace roles and item permissions can act as separate grants.

Example:

```text
User
├── Workspace role = Viewer
└── Item permission = Write
```

Removing one grant does not necessarily remove access granted through the other.

### Exam clue

> User still has access after their workspace role was removed.

### Check

**Item permissions.**

---

# 10. FULLY REVOKING ACCESS

If you need to completely remove a user's access:

```text
CHECK
├── Workspace role
└── Item permissions
```

### Memory Trick

> **REVOKE → CHECK WORKSPACE + ITEM**

---

# 11. APPS

Fabric apps are useful when you have:

- Multiple audiences
- Different consumers
- Curated content
- A need to distribute content without giving everyone workspace access

```text
Workspace
    ↓
   APP
    ↓
Audience
```

### Important

An app does **not** automatically grant workspace/item access outside the app.

### Exam clue

> Many different audiences need a curated set of reports/content.

### Answer

**Use an App.**

### Memory Trick

> **MANY AUDIENCES → APP**

---

# 12. SERVICE PRINCIPAL

Service principals are useful for:

- CI/CD
- Automation
- Unattended operations
- Programmatic access

```text
CI/CD Pipeline
      ↓
Service Principal
      ↓
Fabric APIs
      ↓
Deploy / Manage
```

### Memory Trick

> **AUTOMATION → SERVICE PRINCIPAL**

---

# 13. GATEWAY ACCESS

Gateway permissions are a separate consideration.

A user being a Contributor, Member, or Admin does not automatically mean they have every required gateway permission.

### Exam clue

> Contributor cannot perform a gateway-related operation.

### Answer

**Check gateway permissions separately.**

### Memory Trick

> **GATEWAY ≠ WORKSPACE ROLE**

---

# 14. COMMON EXAM SCENARIOS

### Scenario 1

Developer needs to create and modify notebooks but should not manage workspace membership.

**Answer: Contributor**

### Scenario 2

Business user only needs to consume reports.

**Answer: Viewer**

### Scenario 3

User needs SQL/TDS data access.

**Answer: ReadData**

### Scenario 4

Spark user needs OneLake data access.

**Answer: ReadAll**

### Scenario 5

User needs access to only one item instead of the whole workspace.

**Answer: Item permission**

### Scenario 6

Many audiences need curated content.

**Answer: App**

### Scenario 7

CI/CD needs unattended access.

**Answer: Service Principal**

### Scenario 8

Workspace role was removed but user still has access.

**Answer: Check item-level permissions**

### Scenario 9

Contributor cannot perform a gateway-related operation.

**Answer: Check gateway permissions separately**

---

# 15. DATABASE PROJECT vs WORKSPACE ACCESS

These concepts are different.

## Database Project

```text
Database
   ↓
Schema
   ↓
Database Project
   ↓
.sqlproj
   ↓
Build
   ↓
.dacpac
   ↓
Deploy
```

### Memory

> **Database Project = Schema Blueprint**

## Workspace Access

```text
User
├── Workspace Role
└── Item Permission
```

### Memory

> **Workspace Access = Who can do what**

---

# 16. DEPLOYMENT PIPELINE vs WORKSPACE ACCESS

## Deployment Pipeline

Moves supported content between environments.

```text
DEV
 ↓
TEST
 ↓
PROD
```

### Memory

> **PIPELINE = MOVE CONTENT**

## Workspace Access

Controls who can work with or consume content.

```text
USER
 ↓
ROLE / PERMISSION
 ↓
WORKSPACE / ITEM
```

### Memory

> **ACCESS = CONTROL USERS**

---

# 17. FINAL DP-700 MEMORY SHEET

```text
WORKSPACE → ROLE
ONE ITEM → PERMISSION

ADMIN → Full workspace administration
MEMBER → Collaboration/management
CONTRIBUTOR → Build, don't manage members
VIEWER → Consume/read

SQL/TDS → ReadData
SPARK/ONELAKE → ReadAll

MANY AUDIENCES → APP
AUTOMATION → SERVICE PRINCIPAL

GATEWAY → Separate permission

REVOKE → CHECK WORKSPACE + ITEM

DATABASE PROJECT → SCHEMA BLUEPRINT
DEPLOYMENT PIPELINE → MOVE CONTENT
WORKSPACE ACCESS → CONTROL USERS
```

---

# 18. ONE-LINE EXAM TRICKS

- **Build but don't manage members? → Contributor**
- **Consume reports? → Viewer**
- **SQL/TDS? → ReadData**
- **Spark/OneLake? → ReadAll**
- **One specific item? → Item permission**
- **Many audiences? → App**
- **CI/CD / unattended automation? → Service Principal**
- **Gateway problem? → Check gateway permissions**
- **Still has access after workspace role removal? → Check item permission**
- **Schema blueprint? → Database Project**
- **DEV → TEST → PROD? → Deployment Pipeline**
