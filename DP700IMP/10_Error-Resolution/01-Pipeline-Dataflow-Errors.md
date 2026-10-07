# DP-700 Microsoft Fabric — Pipeline & Dataflow Gen2 Troubleshooting Notes

## Overview

These notes explain the supplied troubleshooting points in an exam-focused way: pipeline activity diagnostics, retries, Dataflow Gen2 destinations, gateway connectivity, diagnostics, refresh limits, Power Query errors, capacity/throttling, and query folding.

---

# 1. Pipeline Activity Output JSON

When a pipeline activity fails, its **Output JSON** is one of the first places to investigate.

Important fields:

```text
errorCode
message
failureType
target
```

Think:

| Field | What it tells you |
|---|---|
| `failureType` | Broad failure category |
| `errorCode` | Specific error |
| `message` | Explanation/details |
| `target` | Affected activity/target |

---

# 2. `failureType`

The two important values are:

```text
UserError
SystemError
```

## UserError

A `UserError` points toward a user/configuration/input/logic-related problem. A simple rerun may not fix the underlying issue.

```text
Configuration/input problem
        ↓
Activity fails
        ↓
failureType = UserError
        ↓
Investigate configuration/input
```

## SystemError

A `SystemError` indicates a system/service/infrastructure-side failure. If the problem is transient, retrying or rerunning may help.

```text
Temporary system/service problem
        ↓
Activity fails
        ↓
failureType = SystemError
        ↓
Consider retry/rerun if appropriate
```

### Exam memory

> **failureType = first triage**  
> **errorCode = specific problem**

---

# 3. Why Read `failureType` Before `errorCode`?

The recommendation is to use `failureType` for the first triage step.

```text
failureType
    ↓
Understand broad category
    ↓
errorCode
    ↓
Find specific failure
    ↓
message + target
    ↓
Choose action
```

If it is a `SystemError`, a transient failure may make a retry worthwhile. If it is a `UserError`, repeatedly rerunning without fixing the underlying configuration is unlikely to help.

> **First classify, then investigate the exact code.**

---

# 4. `LSROBOTokenFailure`

`LSROBOTokenFailure` means the **stored refresh token has been invalidated**.

The supplied material identifies possible causes such as:

- Conditional Access changes
- Password change
- Device removal

Conceptually:

```text
Stored refresh token
        ↓
Token becomes invalid
        ↓
LSROBOTokenFailure
        ↓
Authentication/connection problem
```

### Action

For this scenario, the study point says to **update/save the pipeline** so the stored authentication state can be refreshed.

### Exam signal

```text
LSROBOTokenFailure
        ↓
Stored refresh token invalidated
        ↓
Update/save pipeline
```

---

# 5. Retry vs Rerun From Failed Activity

These are **different mechanisms**.

```text
Retry
```

is automatic activity-level behavior, while:

```text
Rerun from failed activity
```

is a pipeline-run recovery operation.

---

# 6. Activity-Level Retry

Activity-level retry is configured with settings such as:

```text
Retry count
Retry interval
```

It is useful when calling external systems that have **known transient failure modes**.

Example:

```text
External API call
      ↓
Temporary timeout
      ↓
Automatic retry
      ↓
Success
```

Without retry:

```text
External API call
      ↓
Temporary failure
      ↓
Pipeline fails
      ↓
Manual intervention
```

### Best practice

> Configure activity-level retry for routine transient failures instead of relying on manual reruns.

---

# 7. Rerun From Failed Activity

Suppose a pipeline has:

```text
Activity A → SUCCESS
Activity B → SUCCESS
Activity C → FAILED
Activity D → NOT RUN
```

A rerun from the failed activity conceptually becomes:

```text
Activity A → SKIP
Activity B → SKIP
Activity C → RUN AGAIN
Activity D → RUN
```

The purpose is to avoid repeating work that has already succeeded.

### Memory

> **Rerun from failed activity = skip already-successful work and restart from the failure point.**

---

# 8. Retry vs Rerun — Exam Table

| Feature | Retry | Rerun from failed activity |
|---|---|---|
| Type | Automatic | Manual/operational |
| Scope | Activity | Pipeline execution from failed point |
| Main purpose | Handle transient failures | Recover a failed pipeline run |
| Configuration | Retry count/interval | Chosen during rerun |
| Avoids routine manual intervention | Yes | No |
| Skips previously successful work | Not the main concept | Yes |

### Critical exam distinction

> **Built-in retry ≠ rerun from failed activity**

---

# 9. Dataflow Gen2 Downstream Destinations

The supplied recommendation is:

> Prefer a **Lakehouse/Warehouse data destination** over the **Dataflows connector** for downstream items that read a Dataflow Gen2 output when you want to avoid the internal API timeout failure mode described in the material.

Less desirable path for that scenario:

```text
Dataflow Gen2
      ↓
Dataflows connector
      ↓
Downstream item
```

Preferred path:

```text
Dataflow Gen2
      ↓
Lakehouse / Warehouse
      ↓
Downstream item
```

The idea is to have the downstream item read from persisted data rather than depending on the internal Dataflows connector path.

### Exam memory

> **Dataflow output → Lakehouse/Warehouse → downstream item**

when the question is targeting the connector timeout failure mode described here.

---

# 10. Gateway Connectivity — Port Asymmetry

A key point in the supplied material is the difference between writing and staging read-back.

## Writing

Uses:

```text
HTTPS / TCP 443
```

## Gateway + Dataflow Gen2 staging read-back

Requires:

```text
TCP 1433
```

This explains the confusing scenario:

> **First query succeeds, but a referencing query fails.**

The initial write can work through HTTPS/443 while the later staging read-back requires TCP 1433.

---

# 11. Gateway Port Diagram

```text
             Dataflow Gen2
                  │
                  │ Write
                  ▼
             HTTPS 443
                  │
                  ▼
               Gateway
```

For staging read-back:

```text
             Dataflow Gen2
                  │
                  │ Staging read-back
                  ▼
              TCP 1433
                  │
                  ▼
               Gateway
```

### Exam signal

If the question says:

> The first query succeeds, but the referencing query fails through a gateway.

Think:

> **TCP 1433**

---

# 12. Why the First Query Can Work but Referencing Query Fails

Conceptually:

```text
First query
    ↓
Write/stage operation
    ↓
HTTPS 443
    ↓
SUCCESS
```

Then:

```text
Referencing query
    ↓
Read staged data back
    ↓
TCP 1433
    ↓
Blocked/unavailable
    ↓
FAILURE
```

Therefore, a successful first query does not prove that every required gateway network path is available.

---

# 13. Gateway Diagnostics

Gateway diagnostics can provide detailed logs for troubleshooting.

The supplied best practice is to enable **Admin consent for gateway diagnostics proactively**.

Why?

During an incident you may need:

```text
Download detailed logs
```

If the prerequisite consent has not already been enabled, it can delay investigation.

Recommended preparation:

```text
Gateway diagnostics
       ↓
Admin consent enabled
       ↓
Detailed logs ready when needed
```

### Exam memory

> **Prepare gateway diagnostics before the incident, not during it.**

---

# 14. Dataflow Gen2 Refresh Limits

The supplied study material identifies these limits:

| Limit | Value |
|---|---:|
| CI/CD refreshes | **300 / 24h** |
| Non-CI/CD refreshes | **150 / 24h** |
| Maximum time per query evaluation | **8 hours** |
| Maximum total refresh duration | **24 hours** |
| Maximum staged/destination queries | **50** |

### Memory

> **300 / 150 / 8 / 24 / 50**

---

# 15. 300 vs 150 Refreshes

The supplied material distinguishes the refresh limits:

```text
CI/CD
 ↓
300 refreshes / 24h
```

and:

```text
Non-CI/CD
 ↓
150 refreshes / 24h
```

### Exam memory

> **CI/CD = 300**  
> **Non-CI/CD = 150**

---

# 16. 8-Hour Query Evaluation Limit

A single query evaluation has a limit of:

```text
8 hours
```

Do not confuse this with the total refresh limit.

> **8 hours = individual query evaluation**

---

# 17. 24-Hour Total Refresh Limit

The total Dataflow Gen2 refresh has a limit of:

```text
24 hours
```

So remember:

```text
8 hours  → individual query evaluation
24 hours → total refresh
```

### Memory

> **8 = query**  
> **24 = whole refresh**

---

# 18. 50 Staged/Destination Queries

The supplied material identifies:

```text
50
```

as the maximum number of staged/destination queries.

This is a number worth memorizing for Dataflow Gen2 limit questions.

---

# 19. Power Query Error Families

Three important error families are:

```text
DataFormat.Error
DataSource.Error
Expression.Error
```

The easiest memory model is:

```text
DATA → CONNECTION → LOGIC
```

---

# 20. `DataFormat.Error`

Think:

> **The data has an unexpected format or shape.**

Conceptually:

```text
Source data
    ↓
Unexpected format/shape
    ↓
DataFormat.Error
```

Examples conceptually include values that do not match the expected data format.

### Memory

> **DataFormat = problem with the DATA FORMAT**

---

# 21. `DataSource.Error`

Think:

> **The problem is accessing or communicating with the data source.**

Conceptually:

```text
Power Query
    ↓
Data source
    ↓
Connectivity/access problem
    ↓
DataSource.Error
```

Examples can include source availability or connectivity problems.

### Memory

> **DataSource = problem reaching/accessing the source**

---

# 22. `Expression.Error`

Think:

> **The Power Query/M expression or transformation logic is invalid.**

Conceptually:

```text
M expression
     ↓
Invalid logic/reference
     ↓
Expression.Error
```

### Memory

> **Expression = problem with M/transformation logic**

---

# 23. Error Family Comparison

| Error | Think | Layer |
|---|---|---|
| `DataFormat.Error` | Bad data format/shape | Data |
| `DataSource.Error` | Connectivity/source access | Connection |
| `Expression.Error` | Invalid M logic/expression | Logic |

### Ultimate memory

> **Format = Data**  
> **Source = Connection**  
> **Expression = Logic**

---

# 24. Capacity / Throttling Errors

Not every pipeline failure is a pipeline-code problem.

The supplied material identifies these capacity-related indicators:

```text
2003
CapacityLimitExceeded
HTTP 430
```

Think:

```text
Workload increases
      ↓
Capacity pressure
      ↓
Throttling / capacity limit
      ↓
Failure
```

---

# 25. How to Troubleshoot Capacity Errors

For these capacity-related scenarios, the supplied recommendation is to use the **Capacity Metrics app** to investigate capacity usage/throttling.

```text
Capacity-related error
        ↓
Capacity Metrics app
        ↓
Check capacity pressure/throttling
```

### Exam signal

If you see:

```text
2003
CapacityLimitExceeded
HTTP 430
```

think:

> **Capacity / throttling**

not automatically:

> **Pipeline logic bug**

---

# 26. Query Folding

Query folding is important because a Dataflow Gen2 refresh can become significantly slower **without producing an error**.

The key idea is that Power Query can push supported transformations back to the source.

Conceptually:

```text
Power Query transformation
          ↓
Can source perform it?
          ↓
Query folding
          ↓
Source performs more of the work
```

This can reduce unnecessary data movement and improve performance.

---

# 27. Query Folding Failure Can Be Silent

Suppose a refresh changes from:

```text
20 minutes
```

to:

```text
2 hours
```

but there is:

```text
No error
```

The supplied guidance is:

> Treat the refresh-duration regression as a **query-folding investigation**, not a "wait and see" problem.

Conceptually:

```text
Performance regression
       ↓
No explicit error
       ↓
Investigate query folding
       ↓
Check whether transformations are still being pushed to the source
```

### Exam memory

> **Slow but successful can still indicate a query-folding problem.**

---

# 28. Query Folding: Good vs Poor Scenario

## Folding works

```text
Source
  ↓
Filter pushed to source
  ↓
Source processes/filter data
  ↓
Smaller result
  ↓
Power Query
  ↓
Faster refresh
```

## Folding is lost

```text
Source
  ↓
Large dataset transferred
  ↓
Power Query performs more transformation work
  ↓
More processing/data movement
  ↓
Slower refresh
```

The important exam point is not that every slow refresh is automatically folding-related, but that a **refresh-duration regression with no error should prompt a query-folding investigation** according to the supplied study guidance.

---

# 29. Troubleshooting Decision Tree

```text
Pipeline/Dataflow problem
          │
          ▼
Read Output JSON
          │
          ▼
Check failureType
     ┌────┴────┐
     ▼         ▼
 UserError  SystemError
     │         │
     ▼         ▼
Check       Consider
config      transient issue
            / retry
     │
     ▼
Check errorCode
     │
     ▼
Check message + target
     │
     ▼
Choose fix
```

---

# 30. Common Scenario → Answer Map

| Scenario | Think / Action |
|---|---|
| Need first triage of activity failure | Read `failureType` |
| Need specific failure details | Read `errorCode` + `message` |
| `LSROBOTokenFailure` | Stored refresh token invalidated; update/save pipeline |
| External system has routine transient failures | Configure activity-level retry |
| Earlier activities succeeded; later activity failed | Rerun from failed activity |
| Downstream item reads Dataflow Gen2 output | Prefer Lakehouse/Warehouse destination for the timeout scenario described |
| First gateway query succeeds, referencing query fails | Check TCP 1433 |
| Need detailed gateway logs during incident | Admin consent for gateway diagnostics should already be enabled |
| CI/CD Dataflow Gen2 refresh limit | 300 / 24h |
| Non-CI/CD refresh limit | 150 / 24h |
| Individual query evaluation limit | 8h |
| Total refresh limit | 24h |
| Staged/destination query limit | 50 |
| `DataFormat.Error` | Data format/shape |
| `DataSource.Error` | Source/connectivity |
| `Expression.Error` | M/transformation logic |
| `2003` / `CapacityLimitExceeded` / `HTTP 430` | Capacity/throttling |
| Refresh becomes much slower with no error | Investigate query folding |

---

# 31. DP-700 Exam Tips

## Pipeline Output JSON

Recognize:

```text
errorCode
message
failureType
target
```

Remember:

> **failureType first → errorCode next → message/target for detail**

---

## `failureType`

Know:

```text
UserError
SystemError
```

Use it for initial triage before digging into the specific error code.

---

## `LSROBOTokenFailure`

Remember:

```text
Stored refresh token invalidated
        ↓
Conditional Access / password change / device removal
        ↓
Update/save pipeline
```

---

## Gateway

Memorize the asymmetry:

```text
Writing             → HTTPS 443
Staging read-back   → TCP 1433
```

Exam clue:

> First query succeeds, referencing query fails.

Think:

> **TCP 1433**

---

## Dataflow Gen2 limits

Memorize:

```text
300 / 150 / 8 / 24 / 50
```

Meaning:

```text
300 → CI/CD refreshes per 24h
150 → non-CI/CD refreshes per 24h
8   → hours per query evaluation
24  → hours total refresh duration
50  → staged/destination queries
```

---

## Power Query errors

```text
DataFormat.Error
       ↓
DATA

DataSource.Error
       ↓
CONNECTION / SOURCE

Expression.Error
       ↓
LOGIC / M
```

---

## Capacity

```text
2003
CapacityLimitExceeded
HTTP 430
```

Think:

> **Capacity / throttling**

Use the **Capacity Metrics app** for the capacity investigation described in the supplied material.

---

## Query folding

```text
Refresh much slower
+
No error
      ↓
Investigate query folding
```

Do not automatically assume the system is simply slow.

---

# 32. Key Takeaways

- A pipeline activity's **Output JSON** is the first and fastest diagnostic surface.
- Read **`failureType` before `errorCode`** for initial triage.
- `UserError` and `SystemError` are the key `failureType` values in these notes.
- `LSROBOTokenFailure` means the stored refresh token was invalidated.
- For that token scenario, update/save the pipeline.
- Configure **activity-level retry** for known transient external-system failures.
- **Retry** and **rerun from failed activity** are separate mechanisms.
- Rerun from failed activity skips work that already succeeded.
- Prefer **Lakehouse/Warehouse destinations** for downstream Dataflow Gen2 consumers when avoiding the Dataflows connector timeout failure mode described here.
- Gateway writing uses **HTTPS 443**; the described staging read-back requires **TCP 1433**.
- Enable **Admin consent for gateway diagnostics** proactively.
- Remember Dataflow Gen2 limits: **300 / 150 / 8 / 24 / 50**.
- `DataFormat.Error` = data/format problem.
- `DataSource.Error` = source/connectivity problem.
- `Expression.Error` = M/transformation logic problem.
- `2003`, `CapacityLimitExceeded`, and HTTP `430` point toward capacity/throttling.
- A refresh becoming much slower **without an error** should trigger a **query-folding investigation**.

---

# 33. Ultimate Memory Sheet

```text
PIPELINE FAILURE
      ↓
failureType first
      ↓
errorCode
      ↓
message / target
      ↓
Choose the right fix
```

```text
TRANSIENT EXTERNAL FAILURE
      ↓
Activity-level RETRY
```

```text
PIPELINE FAILED AFTER EARLIER SUCCESS
      ↓
RERUN FROM FAILED ACTIVITY
```

```text
LSROBOTokenFailure
      ↓
Stored refresh token invalidated
      ↓
Update/save pipeline
```

```text
GATEWAY REFERENCING QUERY FAILURE
      ↓
TCP 1433
```

```text
CAPACITY ERROR
      ↓
2003 / CapacityLimitExceeded / HTTP 430
      ↓
Capacity investigation
```

```text
SLOW REFRESH + NO ERROR
      ↓
QUERY FOLDING INVESTIGATION
```

```text
DATAFORMAT → DATA
DATASOURCE → CONNECTION
EXPRESSION  → LOGIC
```

> **Exam mindset: Identify the failure layer first, then choose the fix.**
