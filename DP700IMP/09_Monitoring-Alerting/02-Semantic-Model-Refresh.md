# DP-700 Microsoft Fabric — Semantic Model Refresh & Direct Lake

## 1. The Three Core Concepts

Before studying the details, separate these three concepts:

```text
Import
   ↓
Data is copied into the semantic model

Direct Lake
   ↓
Semantic model reads Delta data directly from OneLake

DirectQuery
   ↓
Queries are sent to the underlying SQL source
```

---

# 2. Import Refresh

With an **Import** semantic model, data is physically copied into the semantic model's in-memory storage.

```text
Source
  ↓
Copy data
  ↓
Semantic Model
  ↓
In-memory data
```

Therefore:

> **Import refresh = data is copied/refreshed into the model.**

For large models, this can take significant time.

---

# 3. Direct Lake Framing

Direct Lake works differently.

Instead of copying all the data into the semantic model, Direct Lake uses data already stored in **Delta tables in OneLake**.

```text
Delta Tables
    ↓
  OneLake
    ↓
Direct Lake
    ↓
Semantic Model
```

A **framing operation** updates the model's metadata/references so that it recognizes the current state of the underlying data.

### Important exam statement

> **Framing = metadata-only refresh**

It does not mean copying the entire dataset into the semantic model like Import.

---

# 4. Why Direct Lake Framing Is Fast

### Import

```text
Source
 ↓
Read data
 ↓
Copy data
 ↓
Process/store
 ↓
Semantic Model
```

### Direct Lake

```text
OneLake Delta data
       ↓
Update metadata/reference
       ↓
Semantic Model
```

Therefore:

> **Import refresh = data movement**

> **Direct Lake framing = metadata/reference update**

### Memory Trick

**Import → Copy data**

**Direct Lake → Frame metadata**

---

# 5. Reframing

A Direct Lake model does not continuously rebuild its metadata every time data changes.

Instead, it can be **reframed**.

```text
OneLake data changes
        ↓
New files / changed data
        ↓
Reframing
        ↓
Semantic model recognizes new state
```

Between framings, Direct Lake queries see a **fixed point-in-time snapshot** of the data.

---

# 6. What Can Trigger Reframing?

Reframing can happen through:

- Automatic updates
- Manual action
- Schedule
- Programmatic calls

```text
Reframing
   │
   ├── Automatic
   ├── Manual
   ├── Scheduled
   └── Programmatic
```

### Exam Signal

If the question asks:

> "What causes a Direct Lake model to recognize the latest underlying data?"

Think:

> **Framing / Reframing**

---

# 7. Direct Lake on OneLake vs SQL Endpoints

This is one of the most important DP-700 distinctions.

There are two Direct Lake scenarios:

```text
Direct Lake
    │
    ├── Direct Lake on OneLake
    │
    └── Direct Lake on SQL endpoints
```

They do not behave identically.

---

# 8. Direct Lake on OneLake

With Direct Lake on OneLake:

> **There is no DirectQuery fallback path.**

Conceptually:

```text
Direct Lake on OneLake
          ↓
       Direct Lake
          ↓
       No fallback
```

If Direct Lake cannot handle something, it does not silently switch to DirectQuery.

This makes behavior more predictable.

### Exam Signal

> **Direct Lake on OneLake = NO DirectQuery fallback**

---

# 9. Direct Lake on SQL Endpoints

Direct Lake on SQL endpoints can have a **DirectQuery fallback**.

```text
Direct Lake on SQL endpoint
          ↓
    Try Direct Lake
          ↓
      Can't support?
          ↓
    DirectQuery fallback
```

This can create unexpected performance problems.

You might think:

> "I'm using Direct Lake."

But a particular query could actually execute through:

> **DirectQuery**

---

# 10. DirectLakeBehavior

For Direct Lake on SQL endpoints, `DirectLakeBehavior` controls how the model behaves regarding fallback.

| Setting | Meaning |
|---|---|
| `Automatic` | Normal automatic behavior, including fallback when applicable |
| `DirectLakeOnly` | Use Direct Lake only; don't fall back to DirectQuery |
| `DirectQueryOnly` | Use DirectQuery only |

---

# 11. Why Use DirectLakeOnly?

During development, you want fallback conditions to appear as errors rather than silently causing slower DirectQuery execution.

### Without DirectLakeOnly

```text
Unsupported condition
       ↓
Silent fallback
       ↓
DirectQuery
       ↓
Slow performance
       ↓
Problem reaches production
```

### With DirectLakeOnly

```text
Unsupported condition
       ↓
Hard error
       ↓
Developer discovers problem
       ↓
Fix model/query
       ↓
Production
       ↓
Predictable behavior
```

### Memory Trick

> **DirectLakeOnly = Fail loudly, don't fall back.**

### Recommended development approach

> Set `DirectLakeBehavior = DirectLakeOnly` in development to surface fallback conditions before production.

---

# 12. Direct Lake Comparison

| Feature | Direct Lake on OneLake | Direct Lake on SQL Endpoint |
|---|---|---|
| Direct Lake | ✅ | ✅ |
| DirectQuery fallback | ❌ | ✅ Possible |
| Fallback behavior controlled by `DirectLakeBehavior` | Not applicable in the same way | ✅ |
| `DirectLakeOnly` useful for preventing fallback | Not needed for fallback prevention | ✅ |
| Silent performance degradation from fallback | Much lower | Possible |

### Exam Trap

Question:

> "Which Direct Lake architecture has no DirectQuery fallback?"

Answer:

> **Direct Lake on OneLake**

---

# 13. Diagnosing DirectQuery Fallback

Use:

```kusto
EVALUATE TABLETRAITS()
```

Look for:

> **DirectLakeFallbackInfo**

Conceptually:

```text
TABLETRAITS()
      ↓
Inspect tables
      ↓
DirectLakeFallbackInfo
      ↓
Identify fallback information
```

### Exam Signal

If you see:

> `EVALUATE TABLETRAITS()`

Think:

> **Diagnose Direct Lake table/fallback behavior**

---

# 14. Why TABLETRAITS() Is Useful

Suppose your model contains:

```text
Customer
Sales
Product
Inventory
```

You discover unexpected slow performance.

Instead of guessing, inspect table traits and look for fallback information.

```text
Slow query
    ↓
Check Direct Lake behavior
    ↓
TABLETRAITS()
    ↓
DirectLakeFallbackInfo
    ↓
Identify affected table
```

The diagnostic information can be examined table by table.

---

# 15. Enhanced Refresh API

The **enhanced refresh API** allows programmatic control over semantic model refreshes.

It is especially useful for:

- Large semantic models
- Selective refresh
- Table/partition scoping
- Automation
- Monitoring
- Cancellation

---

# 16. Why Use Enhanced Refresh?

Imagine a model containing:

```text
Sales
Customer
Product
Inventory
Transactions
HistoricalSales
```

Suppose only Sales changed.

Instead of:

```text
Refresh entire model
```

you can target the required object:

```text
Enhanced Refresh
       ↓
Refresh only required tables/partitions
```

This is particularly useful for large models.

---

# 17. Table and Partition Scoping

Enhanced refresh supports targeted refreshes.

You can refresh:

- A specific table
- A specific partition

Example:

```text
Semantic Model
      │
      ├── Customer
      ├── Product
      ├── Sales
      │     ├── 2024
      │     ├── 2025
      │     └── 2026
      └── Inventory
```

If only the 2026 Sales partition changed:

```text
Refresh
   ↓
Sales
   ↓
2026 partition only
```

rather than refreshing the entire model.

### Exam Signal

> **Large model + selective table/partition refresh**

👉 **Enhanced refresh API**

---

# 18. Enhanced Refresh API HTTP Verbs

Memorize these:

| HTTP Verb | Purpose |
|---|---|
| `POST` | Start refresh |
| `GET` | Get status/list refresh information |
| `DELETE` | Cancel refresh |

### Memory Trick

> **POST = Start**

> **GET = Check**

> **DELETE = Cancel**

---

# 19. Important DELETE Detail

`DELETE` cancels:

> **Enhanced-triggered refreshes only**

Do not assume DELETE is a generic cancellation mechanism for every semantic model refresh.

### Exam Trap

> "Which HTTP method cancels an enhanced refresh?"

👉 **DELETE**

---

# 20. Refresh Monitoring Flow

Enhanced refresh is asynchronous.

Conceptually:

```text
POST
 ↓
Start refresh
 ↓
Refresh running
 ↓
GET
 ↓
Check status
 ↓
Completed / Failed
```

So the POST starts the operation; GET can be used to inspect its status.

---

# 21. Scheduled Refresh Limits

The limits in these notes are:

### Pro

> **8 refreshes/day**

### PPU / Premium / Fabric capacity

> **48 refreshes/day**

```text
Pro
 ↓
8/day

PPU / Premium / Fabric capacity
 ↓
48/day
```

### Exam Trick

> **Pro = 8/day**

> **PPU/Premium/Fabric = 48/day**

---

# 22. Four Consecutive Failures

A scheduled semantic model refresh can be automatically deactivated after:

> **4 consecutive failures**

Think:

```text
Failure 1 ❌
Failure 2 ❌
Failure 3 ❌
Failure 4 ❌
      ↓
Schedule deactivated
```

This can cause a production schedule to stop running.

---

# 23. Monitor Refresh History Proactively

Do not wait until the schedule has been dead for days.

For example:

```text
Refresh 1 ❌
Refresh 2 ❌
Refresh 3 ❌
       ↓
⚠️ Warning sign
       ↓
Investigate before #4
```

The objective is:

> **Detect repeated failures early and fix the underlying problem before schedule deactivation.**

---

# 24. Workspace Lineage

A semantic model refresh failure does not necessarily mean the semantic model itself is broken.

There may be an upstream dependency.

Example:

```text
Source
  ↓
Dataflow
  ↓
Lakehouse
  ↓
Semantic Model
  ↓
Report
```

Suppose the semantic model refresh fails.

You might initially think:

> "Semantic model problem."

But the actual issue could be:

```text
Dataflow ❌
   ↓
Lakehouse not updated
   ↓
Semantic Model refresh ❌
```

Therefore:

> **Check workspace lineage before assuming the semantic model is the root cause.**

---

# 25. Lineage Troubleshooting Pattern

When a semantic model refresh fails:

```text
Semantic Model ❌
       ↓
Check lineage
       ↓
What does it depend on?
       ↓
Check upstream items
       ↓
Find actual failure
```

### Exam Signal

> **Refresh failure + dependencies/upstream**

👉 **Check workspace lineage**

---

# 26. `sempy.fabric.semantic_model`

`sempy` provides Fabric/Python capabilities that can be used from notebooks.

Your notes specifically highlight:

```python
sempy.fabric.semantic_model
```

This provides notebook-native semantic model refresh orchestration.

Conceptually:

```text
Notebook
   ↓
sempy
   ↓
Semantic Model
   ↓
Refresh
   ↓
Monitor status
```

This is useful when refresh needs to be part of a notebook-driven workflow.

---

# 27. RefreshExecutionDetails

`RefreshExecutionDetails` is used for refresh execution status/details.

Conceptually:

```text
Start refresh
     ↓
RefreshExecutionDetails
     ↓
Poll status
     ↓
Running
     ↓
Completed / Failed
```

This makes notebooks useful for automated refresh orchestration and status polling.

---

# 28. Complete Monitoring Architecture

```text
                 Semantic Model
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ↓              ↓              ↓
      Portal         REST API       Notebook
        │              │              │
        ↓              ↓              ↓
 Refresh history   Enhanced API     sempy
        │              │              │
        │          POST / GET /      Refresh
        │          DELETE            execution
        │
        └──────────────┬──────────────┘
                       ↓
                 Monitor / Health
                       │
                       ↓
                Workspace Lineage
                       │
                 Check upstream
```

---

# 29. Direct Lake vs Import — Critical Comparison

| Concept | Import | Direct Lake |
|---|---|---|
| Stores imported copy | ✅ | ❌ |
| Reads Delta data from OneLake | ❌ | ✅ |
| Refresh copies data | ✅ | ❌ |
| Framing | ❌ | ✅ |
| Metadata/reference update | Not the core refresh model | ✅ |
| Large-model refresh advantage | Lower | High potential |
| DirectQuery fallback | Not a Direct Lake behavior | Depends on Direct Lake architecture |

### Memory

> **Import = Bring the data to the model**

> **Direct Lake = Let the model read the data in OneLake**

---

# 30. Direct Lake on OneLake vs SQL Endpoint

This is probably the highest-value comparison in this topic.

```text
                 Direct Lake
                     │
           ┌─────────┴─────────┐
           ↓                   ↓
      On OneLake          SQL Endpoint
           │                   │
           ↓                   ↓
      Direct Lake          Direct Lake
           │                   │
           ↓              Can fallback
      NO fallback              ↓
                         DirectQuery
```

### Remember

> **OneLake = No DirectQuery fallback**

> **SQL endpoint = DirectQuery fallback possible**

---

# 31. Development Best Practice

Recommended development approach:

```text
Development
     ↓
DirectLakeBehavior
     ↓
DirectLakeOnly
```

Why?

Because you want problems to appear as errors during development.

```text
Unsupported condition
       ↓
Hard error
       ↓
Developer fixes it
       ↓
Production
       ↓
Predictable Direct Lake behavior
```

### Memory Trick

> **Development → DirectLakeOnly → Fail fast**

---

# 32. Exam Scenario Examples

### Question 1

A developer wants to ensure a Direct Lake model never silently falls back to DirectQuery.

**Answer:**

> `DirectLakeBehavior = DirectLakeOnly`

---

### Question 2

Which Direct Lake architecture has no DirectQuery fallback?

**Answer:**

> **Direct Lake on OneLake**

---

### Question 3

You suspect a Direct Lake table is falling back to DirectQuery. What can you inspect?

**Answer:**

> `EVALUATE TABLETRAITS()` and `DirectLakeFallbackInfo`

---

### Question 4

A very large semantic model needs only two partitions refreshed.

**Answer:**

> **Enhanced refresh API with table/partition scoping**

---

### Question 5

Which HTTP verb starts an enhanced refresh?

**Answer:**

> **POST**

---

### Question 6

Which HTTP verb checks refresh status?

**Answer:**

> **GET**

---

### Question 7

Which HTTP verb cancels an enhanced-triggered refresh?

**Answer:**

> **DELETE**

---

### Question 8

A scheduled refresh has failed four times consecutively.

What can happen?

**Answer:**

> The scheduled refresh can be **automatically deactivated**.

---

### Question 9

A semantic model refresh fails. What should you check before blaming the semantic model?

**Answer:**

> **Workspace lineage and upstream dependencies**

---

### Question 10

You want to orchestrate semantic model refreshes from a Fabric notebook.

What can you use?

**Answer:**

> `sempy.fabric.semantic_model`

---

# 33. ⭐ Final DP-700 Memory Sheet

```text
IMPORT
↓
Copies data
↓
Full data refresh


DIRECT LAKE
↓
Reads Delta data from OneLake
↓
Framing = metadata/reference update
↓
Fast refresh/reframing


DIRECT LAKE ON ONELAKE
↓
NO DirectQuery fallback


DIRECT LAKE ON SQL ENDPOINT
↓
DirectQuery fallback possible
↓
Controlled by DirectLakeBehavior


DirectLakeBehavior
├── Automatic
├── DirectLakeOnly
└── DirectQueryOnly


DEVELOPMENT
↓
DirectLakeOnly
↓
Expose fallback conditions as errors


DIAGNOSE FALLBACK
↓
EVALUATE TABLETRAITS()
↓
DirectLakeFallbackInfo


ENHANCED REFRESH API
├── POST → Start
├── GET → Status/List
└── DELETE → Cancel
     (enhanced-triggered refreshes)


LARGE MODEL
↓
Table/partition scoping
↓
Don't refresh everything unnecessarily


SCHEDULED REFRESH
├── Pro → 8/day
└── PPU/Premium/Fabric → 48/day


4 CONSECUTIVE FAILURES
↓
Schedule can deactivate


REFRESH FAILURE
↓
Check Workspace Lineage
↓
Look for upstream failure


NOTEBOOK ORCHESTRATION
↓
sempy.fabric.semantic_model
↓
RefreshExecutionDetails
↓
Poll refresh status
```

---

# 34. 🧠 One-Line Memory Tricks

| Topic | Remember |
|---|---|
| Import | **Copies data** |
| Direct Lake framing | **Updates metadata/references** |
| OneLake Direct Lake | **No fallback** |
| SQL endpoint Direct Lake | **Can fallback** |
| `DirectLakeOnly` | **Fail instead of fallback** |
| `TABLETRAITS()` | **Diagnose fallback** |
| `DirectLakeFallbackInfo` | **Fallback details** |
| `POST` | **Start** |
| `GET` | **Check** |
| `DELETE` | **Cancel** |
| Enhanced refresh | **Selective table/partition refresh** |
| Pro | **8/day** |
| PPU/Premium/Fabric | **48/day** |
| 4 failures | **Schedule deactivation** |
| Lineage | **Check upstream** |
| `sempy` | **Notebook refresh orchestration** |

---

# ⭐ Biggest Exam Distinction

> **Import refresh = copy data**

> **Direct Lake framing = update metadata/references**

> **Direct Lake on OneLake = no DirectQuery fallback**

> **Direct Lake on SQL endpoint = fallback can occur**

> **`DirectLakeOnly` = expose fallback problems as errors**

> **Enhanced refresh = selectively refresh tables/partitions**

> **4 consecutive scheduled failures = schedule can be deactivated**
