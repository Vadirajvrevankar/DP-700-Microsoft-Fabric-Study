# DP-700 — Fabric Deployment Pipelines Notes

## Core Idea

**Deployment Pipeline = Move Fabric content between environments/stages.**

Typical flow:

```text
DEV → TEST → PROD
```

Deployment Pipelines are mainly for **stage promotion**. They are different from Git:

> **Git = Version control + review**  
> **Deployment Pipeline = Promote content between stages**

---

# 1. What Is a Deployment Pipeline?

## CONCEPT

**Deployment Pipeline**

## WHAT IT DOES

Moves supported Fabric items from one stage to another.

Example:

```text
Development
     ↓
Testing
     ↓
Production
```

Think of it as a conveyor belt that moves your solution from DEV toward PROD.

### MEMORY TRICK

> Deployment Pipeline = DEV → TEST → PROD

---

# 2. Pipeline Stages

A deployment pipeline can have:

> **2 to 10 stages**

Example:

```text
DEV → TEST
```

or:

```text
DEV → TEST → UAT → PROD
```

### Important Exam Point

After the pipeline is created:

> **The number and names of stages are permanent.**

The public status of a stage can be changed when needed, but the stage count/names aren't freely changed after creation.

### MEMORY TRICK

> Pipeline = 2–10 permanent stages

---

# 3. What Is a Stage?

A stage represents an environment in the deployment pipeline and is associated with a workspace.

Example:

```text
DEV Stage  → DEV Workspace
TEST Stage → TEST Workspace
PROD Stage → PROD Workspace
```

Think:

```text
Pipeline
│
├── DEV
├── TEST
└── PROD
```

---

# 4. The Most Important Fact: Data Is NOT Deployed

## CONCEPT

**Deployment Scope**

## WHAT IT DOES

Deployment Pipelines move supported **content/metadata**, not the actual data.

Example:

```text
Sales Table
-----------
1,000,000 rows
```

Those rows are **not copied** simply because you deploy from DEV to TEST.

Think:

```text
ITEM / METADATA → ✅
ACTUAL DATA     → ❌
```

### Other things that are not simply copied by deployment

- Data
- Permissions
- URLs/IDs
- Workspace settings
- App content

### MEMORY TRICK

> **Deployment ≠ Clone Workspace**

> **DATA IS NEVER DEPLOYED**

This is one of the most important DP-700 facts.

---

# 5. Full Deployment

## CONCEPT

**Full Deployment**

## WHAT IT DOES

Deploys all eligible content from the source stage to the target stage.

Example:

```text
DEV
│
├── Lakehouse
├── Warehouse
├── Pipeline
├── Semantic Model
└── Report
```

Full deployment promotes all eligible content.

### MEMORY TRICK

> Full = Everything eligible

---

# 6. Selective Deployment

## CONCEPT

**Selective Deployment**

## WHAT IT DOES

Deploys only the items you select.

Example:

```text
DEV
│
├── Pipeline A
├── Pipeline B
├── Report A
├── Report B
└── Semantic Model
```

You only want:

```text
Report A
```

Use:

> **Selective Deployment**

### MEMORY TRICK

> Selective = Only what I choose

---

# 7. Select Related

## CONCEPT

**Select Related**

## WHAT IT DOES

Helps include items related to/dependent on the item you're selectively deploying.

Example:

```text
Semantic Model
      ↓
    Report
```

If you select only the report, the report may depend on the semantic model.

Use:

> **Select Related**

This is especially useful for:

- Reports
- Dashboards
- Semantic models

### TYPICAL SCENARIO

> You want to selectively deploy a report and include its dependencies.

**ANSWER:**

> Use **Select Related**.

### MEMORY TRICK

> Selective + Dependencies → Select Related

---

# 8. Backward Deployment

## CONCEPT

**Backward Deployment**

## WHAT IT DOES

Moves content from a later stage back to an earlier stage.

Normal:

```text
DEV → TEST → PROD
```

Backward:

```text
PROD → TEST
```

### ⚠️ Two Important Restrictions

Backward deployment requires:

1. **The target stage must be empty**
2. **Only Full deployment is allowed**

So:

```text
PROD
 ↓
TEST
```

works only when TEST is empty and the deployment is Full.

### Not allowed:

```text
PROD → TEST
Selective
```

### MEMORY TRICK

> **BACKWARD = EMPTY TARGET + FULL ONLY**

This is a very important exam phrase.

---

# 9. Item Pairing

## CONCEPT

**Item Pairing**

## WHAT IT DOES

Fabric needs to determine whether an item in the source stage corresponds to an item in the target stage.

Example:

```text
DEV                     TEST

Sales Report     ↔      Sales Report
```

Fabric uses:

> **Name + Type (+ Folder as a tiebreaker)**

### MEMORY TRICK

> Pairing = Name + Type + Folder

---

# 10. Why Is Pairing Important?

Pairing determines whether deployment should:

> **Update/overwrite the existing paired item**

or

> **Create another item because it isn't paired**

Example:

```text
DEV:
Sales Report

TEST:
Sales Report
```

If they are correctly paired:

```text
DEV Sales Report
       ↓
TEST Sales Report
```

The target item is treated as the corresponding item.

If they aren't paired, deployment can result in a separate/duplicate item rather than updating the intended target.

### MEMORY TRICK

> **PAIRING → OVERWRITE OR DUPLICATE**

---

# 11. Why Check Pairing Status Before Full Deployment?

This is a key best practice.

Suppose someone manually created an item directly in the TEST workspace.

```text
DEV
└── Sales Report

TEST
└── Sales Report  ← added manually
```

You might assume they are the same item.

But Fabric's pairing may not be what you expect.

So before a full deployment, especially after ad hoc items were added directly to a stage workspace:

> **Check the pairing status.**

### MEMORY TRICK

> Before Full Deploy → Check Pairing

---

# 12. Deployment Rules

## CONCEPT

**Deployment Rules**

## WHAT IT DOES

Allows certain stage-specific configuration differences for supported item types.

### ⚠️ Important

Deployment Rules do **not** work for every Fabric item.

The supported types in this study context are:

- Dataflow Gen1
- Semantic model
- Paginated report
- Mirrored database
- Notebook (default lakehouse)

### Not covered by Deployment Rules here

- Lakehouse
- Warehouse
- Pipeline
- Eventstream

For other configurable items, use the appropriate **item-level parameters**.

### MEMORY TRICK

> Deployment Rules = Limited Item Types

---

# 13. Deployment Rules vs Parameters

This is an important exam distinction.

If the question asks about stage-specific configuration for a supported Deployment Rules item:

> Think **Deployment Rules**.

For items such as:

```text
Lakehouse
Warehouse
Pipeline
Eventstream
```

don't automatically choose Deployment Rules.

Think:

> **Item-level parameters**

### MEMORY TRICK

```text
Deployment Rules
      ↓
Specific supported items

Parameters
      ↓
Other configuration differences
```

---

# 14. Deployment Notes

## CONCEPT

**Deployment Note**

## WHAT IT DOES

Records useful information about why a deployment happened.

Example:

```text
"Deploy updated sales semantic model and report."
```

Why is this useful?

Later, deployment history can show:

```text
What happened?
When?
Why?
```

Without notes:

```text
Deployment 1
Deployment 2
Deployment 3
```

It becomes harder to understand the history.

### BEST PRACTICE

> Add a deployment note every time.

### MEMORY TRICK

> Deployment History + Notes = Traceability

---

# 15. Permissions — Two Separate Layers

This is another major exam concept.

There are **two independent permission layers**.

## Layer 1 — Pipeline Permission

**Pipeline Admin**

Controls pipeline-level management/sharing.

```text
Pipeline Admin
      ↓
Pipeline management
```

## Layer 2 — Workspace Role

Controls access to the workspace/content.

```text
Workspace Role
      ↓
Workspace/content access
```

For most deployment actions, you need the appropriate combination of both.

### MEMORY TRICK

> Deployment = Check Pipeline + Workspace

---

# 16. Troubleshooting "Why Can't This Person Deploy?"

Suppose someone says:

> "I'm a Pipeline Admin. Why can't I deploy?"

Don't stop there.

Check:

```text
Pipeline Admin
      +
Workspace Role / Access
```

Pipeline Admin and workspace permissions are separate.

A person can have pipeline management permission but still lack the required workspace/content access.

### EXAM ANSWER

> Check both **Pipeline Admin permissions and workspace role/access**.

### MEMORY TRICK

> Pipeline Admin ≠ Workspace Access

---

# 17. Complete Example

Imagine:

```text
DEV
│
├── Lakehouse
├── Warehouse
├── Semantic Model
├── Sales Report
└── Sales Dashboard

TEST
│
├── Lakehouse
├── Warehouse
├── Semantic Model
└── Sales Report
```

You want to deploy DEV → TEST.

### Step 1 — Check Pairing

Check whether the source and target items are paired correctly.

Especially important if someone manually added items to TEST.

```text
DEV Sales Report
        ↕
TEST Sales Report
```

---

### Step 2 — Choose Deployment Type

Want everything?

> **Full**

Want only certain items?

> **Selective**

---

### Step 3 — If Selective

Suppose you select:

```text
Sales Report
```

The report depends on:

```text
Sales Semantic Model
```

Use:

> **Select Related**

---

### Step 4 — Configuration Differences

Do not automatically use Deployment Rules for:

```text
Lakehouse
Warehouse
Pipeline
Eventstream
```

Use appropriate item-level parameters instead.

---

### Step 5 — Add Deployment Note

Example:

```text
"Deploy updated sales report and semantic model."
```

---

### Step 6 — Check Permissions

If deployment fails:

```text
Pipeline Admin?
      +
Workspace Role?
```

Check both.

---

# 18. Full vs Selective vs Backward

| Method | Meaning | Important Rule |
|---|---|---|
| **Full** | Deploy all eligible content | Normal stage promotion |
| **Selective** | Deploy selected items | Use Select Related for dependencies |
| **Backward** | Later stage → earlier stage | Empty target + Full only |

---

# 19. What Does NOT Get Deployed?

Remember these:

```text
Data              ❌
Permissions       ❌
URLs / IDs        ❌
Workspace settings ❌
App content       ❌
```

Deployment moves eligible content/metadata, not a complete copy of the workspace.

---

# 20. Complete Deployment Pipeline Picture

```text
                    DEPLOYMENT PIPELINE
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
            DEV           TEST          PROD
             │             │             │
             └─────────────┴─────────────┘
                           ↓
                    Deploy Content
                           ↓
              Metadata / Supported Items
                           ↓
                    NOT Actual Data
```

Deployment options:

```text
FULL
 ↓
All eligible content

SELECTIVE
 ↓
Selected content
 ↓
Select Related = Dependencies

BACKWARD
 ↓
Later → Earlier
 ↓
EMPTY TARGET
 ↓
FULL ONLY
```

---

# 🔥 DP-700 Exam Cheat Sheet

| Topic | Remember |
|---|---|
| Deployment Pipeline | Promotes content between stages |
| Stages | 2–10 |
| Stage names/count | Permanent after creation |
| Full deployment | All eligible content |
| Selective deployment | Selected items |
| Select Related | Helps include dependencies |
| Backward deployment | Later stage → earlier stage |
| Backward requirement | Empty target + Full only |
| Item pairing | Name + Type + Folder tiebreaker |
| Pairing determines | Overwrite vs duplicate |
| Check pairing | Especially before full deploy after ad hoc items |
| Deployment Rules | Only specific supported item types |
| Deployment Rule examples | Dataflow Gen1, Semantic model, Paginated report, Mirrored database, Notebook |
| Lakehouse/Warehouse/Pipeline/Eventstream | Use item-level parameters |
| Data | Never deployed |
| Permissions | Not copied |
| URLs/IDs | Not copied |
| Workspace settings | Not copied |
| App content | Not copied |
| Deployment note | Add every time |
| Pipeline Admin | Pipeline management/sharing |
| Workspace Role | Content/workspace access |
| Troubleshooting deployment | Check both permission layers |

---

# ⭐ 10 Things to Memorize

1. **Deployment Pipeline = Move content between stages.**
2. **2–10 stages; stage names/count are permanent after creation.**
3. **Full = Everything eligible.**
4. **Selective = Only selected items.**
5. **Select Related = Include dependencies.**
6. **Backward = Later → Earlier, empty target, Full only.**
7. **Pairing = Name + Type (+ Folder tiebreaker).**
8. **Data is NEVER deployed.**
9. **Deployment Rules work only for specific supported item types; use parameters for other configurable items.**
10. **Deployment permissions = Pipeline Admin + Workspace Role.**

---

# 🔥 Ultimate Memory Trick

```text
PIPELINE
   ↓
2–10 STAGES

FULL
   ↓
EVERYTHING ELIGIBLE

SELECTIVE
   ↓
SELECT WHAT YOU WANT
   ↓
SELECT RELATED = DEPENDENCIES

BACKWARD
   ↓
LATER → EARLIER
   ↓
EMPTY TARGET
   ↓
FULL ONLY

PAIRING
   ↓
NAME + TYPE (+ FOLDER)
   ↓
OVERWRITE OR DUPLICATE

DEPLOYMENT RULES
   ↓
LIMITED ITEM TYPES

PARAMETERS
   ↓
OTHER CONFIGURATION DIFFERENCES

DATA
   ↓
NEVER DEPLOYED

PERMISSIONS
   ↓
PIPELINE ADMIN + WORKSPACE ROLE

DEPLOYMENT NOTE
   ↓
HISTORY / TRACEABILITY
```

## 🧠 One Sentence to Remember Everything

> **"Deployment Pipelines move content from DEV to TEST to PROD; Full deploys everything eligible, Selective deploys chosen items, Select Related handles dependencies, Backward requires an empty target and Full deployment, pairing decides overwrite vs duplicate, rules apply only to specific item types, data is never deployed, and both Pipeline Admin and workspace access matter."**
