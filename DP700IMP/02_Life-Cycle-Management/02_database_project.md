# DP-700 — Fabric Database Projects Notes

## Core Idea

**Database Project = Database Schema Blueprint**

A SQL Database Project is a declarative, version-controlled representation of a database's **schema**.

```text
Real Database
     ↓
Database Project
     ↓
Schema Blueprint
```

It focuses on structure such as tables, views, procedures, and other schema objects.

**Important:**

```text
SCHEMA → Database Project ✅
TABLE DATA → Not included ❌
```

**Memory Trick:**  
> Database Project = Database Blueprint

---

# 1. Auto-Generated `.sqlproj`

## CONCEPT

**Auto-generated SQL Database Project**

## WHAT IT DOES

When a supported Fabric **Warehouse or SQL database** is committed to Git, Fabric automatically generates the SQL database project representation.

You do not normally create the project from scratch.

```text
Warehouse / SQL Database
          ↓
       Commit
          ↓
SQL Database Project
```

### `.sqlproj`

A `.sqlproj` file is the SQL database project file.

### Important Best Practice

**Never hand-edit Fabric's auto-generated `.sqlproj`.**

Treat it as generated project/build metadata rather than a source file you manually maintain.

**MEMORY TRICK:**

> Commit Database → Project Auto-Generated

> `.sqlproj` = Project File → Don't Hand-Edit

---

# 2. Schema vs Data

## CONCEPT

**Database Project Scope**

## WHAT IT DOES

Database Projects represent the **schema/structure** of the database.

Example:

```sql
CREATE TABLE Customers
(
    CustomerID INT,
    Name VARCHAR(100),
    Email VARCHAR(200)
)
```

This is schema.

Actual rows such as:

```text
CustomerID | Name
-----------|------
1          | Ravi
2          | John
3          | Sarah
```

are table data and are not part of the database project's schema representation.

### Memory Trick

> Database Project = Structure, not business data.

---

# 3. Declarative Schema

## CONCEPT

**Declarative Database Project**

## WHAT IT DOES

Declarative means you describe **what the database should look like**.

Example:

```text
Customers table
├── CustomerID
├── Name
└── Email
```

The deployment system determines the changes needed to make the target database match the desired schema.

**MEMORY TRICK:**

> Declarative = Describe the Desired State

---

# 4. Build

## CONCEPT

**Build**

## WHAT IT DOES

Build validates/compiles the database project and produces a deployable package.

```text
Database Project
       ↓
      BUILD
       ↓
    .dacpac
```

### MEMORY TRICK

> Build → DACPAC

---

# 5. `.dacpac`

## CONCEPT

**DACPAC**

## WHAT IT DOES

A `.dacpac` is a deployable package representing the database schema.

```text
Database Project
       ↓
      Build
       ↓
    .dacpac
```

For the exam, remember:

> **Build → `.dacpac`**

---

# 6. SqlPackage Publish

## CONCEPT

**SqlPackage**

## WHAT IT DOES

SqlPackage can publish a `.dacpac` to a target database.

It compares the desired schema with the existing target and applies the required schema changes.

```text
Database Project
       ↓
      Build
       ↓
    .dacpac
       ↓
SqlPackage Publish
       ↓
Target Database
```

### MEMORY TRICK

> DACPAC → SqlPackage → Database

### Complete Chain

> **Project → Build → DACPAC → SqlPackage → Database**

---

# 7. Schema Compare

## CONCEPT

**Schema Compare**

## WHAT IT DOES

Schema Compare shows the differences between a database project and a live database.

```text
Database Project
        VS
Live Database
```

Example:

```text
PROJECT                 LIVE DATABASE

Customers               Customers
Products                Products
Orders                  Orders
Age column              No Age column
```

It is useful for reviewing changes before committing or deploying.

### TYPICAL SCENARIO

> A developer wants to see what schema changes will occur before deployment.

**ANSWER:**

> Use Schema Compare.

### MEMORY TRICK

> Schema Compare = Find Differences

---

# 8. Validate Before Commit or Deployment

Before committing or deploying significant schema changes:

```text
Make Schema Change
       ↓
Build / Schema Compare
       ↓
Review
       ↓
Commit / Deploy
```

This is especially important for `ALTER TABLE` changes.

### MEMORY TRICK

> Change → Check → Commit

---

# 9. ALTER TABLE Drop/Recreate Gotcha

## CONCEPT

**ALTER TABLE Data-Loss Risk**

## WHAT IT DOES

For some schema changes, the deployment mechanism may determine that a table needs to be dropped and recreated.

You might expect:

```text
ALTER TABLE
     ↓
Small modification
```

But the resulting deployment can potentially be:

```text
DROP TABLE
     ↓
CREATE TABLE
     ↓
Potential DATA LOSS ⚠️
```

Therefore, validate the generated schema changes before deployment.

### TYPICAL SCENARIO

> A developer changes a Warehouse table and wants to avoid an unexpected drop/recreate operation.

**ANSWER:**

> Build/validate the database project and review the schema differences before deployment.

### MEMORY TRICK

> ALTER TABLE → Check for Drop/Recreate ⚠️

---

# 10. Reference / Lookup Data

## CONCEPT

**Reference Data**

## WHAT IT DOES

Reference or lookup data is small, relatively static data that should remain consistent across environments.

Examples:

```text
Country
-------
India
USA
UK
Canada
```

or:

```text
CustomerStatus
--------------
Active
Inactive
Pending
```

You may need the same values in:

```text
DEV
TEST
PROD
```

Do not rely on manually inserting these values separately in each environment.

Use deployment scripts.

### MEMORY TRICK

> Reference Data → Deployment Script

---

# 11. Post-Deployment Scripts

## CONCEPT

**Post-Deployment Script**

## WHAT IT DOES

A post-deployment script runs after the main schema deployment and can be used for reference/lookup data that needs to stay consistent across environments.

```text
Deploy Schema
      ↓
Post-Deployment Script
      ↓
Reference Data
```

### Important

This does NOT mean all business table data is managed by the database project.

It is especially useful for static/reference data.

### MEMORY TRICK

> Reference Data → Post-Deployment Script

---

# 12. Pre-Deployment vs Post-Deployment

You may encounter both types of deployment scripts.

### Pre-Deployment

Runs before the main deployment actions.

### Post-Deployment

Runs after the main deployment actions.

For DP-700, remember:

> Use deployment scripts for static/reference data that needs to stay consistent across environments.

---

# 13. SQL Security

## CONCEPT

**SQL Security Definitions**

## WHAT IT DOES

Database Projects primarily manage the database schema.

SQL security definitions are outside the database project's tracked scope described here.

Examples:

```text
Users
Permissions
Security Definitions
```

Keep appropriate security definitions in **separate version-controlled SQL scripts**.

```text
Database Project
      ↓
Schema

Separate SQL Scripts
      ↓
Security
```

### MEMORY TRICK

> Database Project = Schema

> Security = Separate Scripts

---

# 14. Actual Table Data

Do not confuse reference data with normal business data.

Example:

```text
Sales
-----
500 million transaction rows
```

This is business data and is not part of the database project's schema representation.

```text
Database Project
      ↓
Schema ✅

Business Data
      ↓
Not Included ❌
```

### Three Things to Separate

```text
SCHEMA
   ↓
Database Project

REFERENCE DATA
   ↓
Pre/Post-Deployment Scripts

BUSINESS TABLE DATA
   ↓
Not Part of Database Project
```

---

# 15. VS Code + SQL Database Projects Extension

## CONCEPT

**Local Database Project Development**

## WHAT IT DOES

For larger schema changes, you can clone the source-controlled project locally and use:

> **VS Code + SQL Database Projects extension**

Flow:

```text
Git
 ↓
Clone Project
 ↓
VS Code
 ↓
SQL Database Projects Extension
 ↓
Make Schema Changes
 ↓
Build / Validate
 ↓
Commit
```

### When is this useful?

Especially when you have many database objects and large schema changes.

### MEMORY TRICK

> Big Schema Changes → VS Code + SQL Database Projects

---

# 16. Database Projects + Git

These work together.

```text
Fabric Database
       ↓
Database Project
       ↓
Git
```

### Database Project

Answers:

> **What should the database schema look like?**

### Git

Answers:

> **How do we track and review changes?**

### MEMORY TRICK

> Database Project = Schema

> Git = Version + Review

---

# 17. Database Projects vs Deployment Pipelines

They are complementary, not competing.

## Database Projects + Git

Focus on:

- Version control
- Incremental changes
- Collaboration
- Review
- Pull requests

```text
Developer
   ↓
Schema Change
   ↓
Database Project
   ↓
Git
   ↓
Review / PR
```

## Deployment Pipelines

Focus on:

> Promoting a solution between environments.

```text
DEV
 ↓
TEST
 ↓
PROD
```

### MEMORY TRICK

> Git = Track + Review

> Deployment Pipeline = Promote

---

# 18. SqlPackage and Fabric Update-from-Source-Control

A key exam point is that **SqlPackage publish is the deployment engine underneath manual `.dacpac` publishing and Fabric's Update-from-source-control workflow**.

The important mental model is:

```text
Source-Controlled Database Project
          ↓
      Schema Package
          ↓
     Deployment Engine
          ↓
     Target Database
```

So do not think of Git Update as merely copying files. The database schema is deployed to the target database using database deployment mechanisms.

---

# ⭐ Complete Real-Life Example

Imagine your company has:

```text
Sales Warehouse
│
├── Customers
├── Products
├── Orders
└── Sales
```

You add:

```text
Customers.LoyaltyLevel
```

### Step 1 — Make the schema change

```text
Customers
---------
CustomerID
Name
Email
LoyaltyLevel ← NEW
```

### Step 2 — Database Project represents the schema

### Step 3 — Build

```text
Build
 ↓
.dacpac
```

### Step 4 — Use Schema Compare

```text
Project VS Live Database
```

Review the differences.

### Step 5 — Check for ALTER TABLE issues

Make sure the deployment won't unexpectedly require:

```text
DROP
 +
CREATE
```

which could cause data loss.

### Step 6 — Commit to Git

The schema change is now version-controlled.

### Step 7 — Deploy

```text
.dacpac
   ↓
SqlPackage Publish
```

### Step 8 — Reference Data

If you need standard lookup values:

```text
Post-Deployment Script
        ↓
Reference Data
```

### Step 9 — Security

If security definitions are required:

```text
Separate Version-Controlled SQL Scripts
```

---

# 🔥 Complete Database Project Flow

```text
                 FABRIC DATABASE
                       ↓
                DATABASE PROJECT
                       ↓
                  SCHEMA CHANGE
                       ↓
            BUILD / SCHEMA COMPARE
                       ↓
                    .dacpac
                       ↓
              SqlPackage Publish
                       ↓
                TARGET DATABASE
```

Git surrounds the development process:

```text
Developer
    ↓
Database Project
    ↓
Git
    ↓
Review / PR
```

Deployment environments:

```text
DEV
 ↓
TEST
 ↓
PROD
```

Reference data:

```text
Schema Deployment
       ↓
Post-Deployment Script
       ↓
Reference Data
```

Security:

```text
SQL Security
       ↓
Separate Version-Controlled SQL Scripts
```

---

# 🧠 DP-700 Exam Cheat Sheet

| Concept | Simple Meaning |
|---|---|
| Database Project | Database schema blueprint |
| `.sqlproj` | Database Project file |
| Auto-generated | Fabric generates it when supported DB is committed to Git |
| Build | Validates/compiles project and produces DACPAC |
| `.dacpac` | Deployable database schema package |
| SqlPackage | Publishes DACPAC to target database |
| Schema Compare | Shows schema differences |
| Schema | Database structure |
| Table data | Not included in Database Project |
| Reference data | Use pre/post-deployment scripts |
| SQL security | Separate version-controlled SQL scripts |
| ALTER TABLE | Watch for drop/recreate/data-loss issue |
| VS Code extension | Useful for larger schema changes |
| Git | Version control and review |
| Deployment Pipeline | Stage promotion, e.g. DEV → TEST → PROD |

---

# ⭐ 10 Things to Memorize

1. **Database Project = Schema Blueprint.**
2. **Commit Warehouse/SQL Database to Git → Project is auto-generated.**
3. **Don't hand-edit Fabric's generated `.sqlproj`.**
4. **Build → `.dacpac`.**
5. **`.dacpac` → SqlPackage Publish → Database.**
6. **Schema Compare → Find Differences.**
7. **ALTER TABLE → Watch for Drop/Recreate → Possible Data Loss.**
8. **Reference/Lookup Data → Post-Deployment Scripts.**
9. **SQL Security → Separate Version-Controlled Scripts.**
10. **Git = Incremental Changes/Review | Deployment Pipeline = Stage Promotion.**

---

# 🔥 Ultimate Memory Trick

```text
DATABASE PROJECT
      ↓
    SCHEMA

BUILD
      ↓
   DACPAC

DACPAC
      ↓
SQLPACKAGE
      ↓
 DATABASE

SCHEMA COMPARE
      ↓
FIND DIFFERENCES

ALTER TABLE
      ↓
CHECK DROP/RECREATE ⚠️

REFERENCE DATA
      ↓
POST-DEPLOYMENT SCRIPT

SECURITY
      ↓
SEPARATE SQL SCRIPTS

GIT
      ↓
VERSION + REVIEW

DEPLOYMENT PIPELINE
      ↓
DEV → TEST → PROD
```

## One Sentence to Remember

> **Database Project manages the schema, Build creates the DACPAC, SqlPackage deploys it, Schema Compare finds differences, scripts handle reference data, security stays separate, Git tracks the changes, and Deployment Pipelines promote them between environments.**
