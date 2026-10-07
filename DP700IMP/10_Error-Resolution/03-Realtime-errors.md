# DP-700 Microsoft Fabric — Eventhouse & Eventstream Troubleshooting Notes

## Overview

These notes cover the supplied troubleshooting and exam points around:

- Eventhouse ingestion failures
- `.show ingestion failures`
- `FailureKind`
- Ingestion failure categories
- Retry behavior
- Update policies
- Transactional vs non-transactional update policies
- Eventstream monitoring
- Eventstream schema validation
- Event Hub authorization and `AADSTS65002`
- Eventhouse Always-On / minimum consumption unit
- Queued ingestion and batching latency

---

# 1. First Triage: `.show ingestion failures`

When investigating Eventhouse ingestion problems, the recommended first step is:

```kusto
.show ingestion failures
```

But the important point is:

> **Do not start with the raw unfiltered table. Filter by `FailureKind` first.**

Conceptually:

```text
Ingestion failure
      ↓
.show ingestion failures
      ↓
Filter FailureKind
      ↓
Decide whether retry is appropriate
```

---

# 2. `FailureKind`

`FailureKind` is one of the most important fields for initial ingestion-failure triage.

The two important values are:

```text
Permanent
Transient
```

## Permanent

A permanent failure generally means:

> Retrying the exact same operation is unlikely to solve the problem.

Conceptually:

```text
Failure
  ↓
Permanent
  ↓
Investigate/fix the underlying problem
  ↓
Then retry/reprocess if appropriate
```

---

## Transient

A transient failure indicates a temporary condition.

Conceptually:

```text
Failure
  ↓
Transient
  ↓
Retry may succeed
```

### Exam memory

> **Transient → retry may help**

> **Permanent → fix the cause first**

---

# 3. Why `FailureKind` Should Be the First Filter

Suppose you have many ingestion failures.

Instead of immediately inspecting every record:

```text
Thousands of failures
       ↓
Read everything
       ↓
Slow/manual investigation
```

Start with:

```text
.show ingestion failures
       ↓
FailureKind
       ↓
Permanent vs Transient
```

This quickly tells you whether retrying is even worth considering.

### Key takeaway

> **`FailureKind` is the first triage decision.**

---

# 4. Retention of `.show ingestion failures`

The supplied exam material states:

> `.show ingestion failures` has **14-day retention**.

So don't assume this command provides an unlimited historical failure record.

```text
Ingestion failures
       ↓
.show ingestion failures
       ↓
14-day retention
```

### Exam memory

> **Ingestion failures = 14 days**

---

# 5. Command-Level Coverage

`.show ingestion failures` is described as:

> **Command-level only**

This means it should not be treated as the only diagnostic source for every stage of the ingestion pipeline.

For full-stage coverage, pair it with:

- Metrics
- Diagnostic logs

Conceptually:

```text
.show ingestion failures
        +
Metrics
        +
Diagnostic logs
        ↓
More complete troubleshooting
```

### Exam takeaway

> `.show ingestion failures` = fast ingestion-failure triage, not the only monitoring source.

---

# 6. Ingestion Failure Categories

Know these error-category families:

```text
BadFormat
BadRequest
DataAccessNotAuthorized
DownloadFailed
EntityNotFound
FileTooLarge
InternalServiceError
UpdatePolicyFailure
ThrottledOnEngine
```

These categories help identify what layer or type of problem occurred.

---

# 7. `BadFormat`

Think:

> The incoming data format/content is invalid or unexpected.

Conceptually:

```text
Incoming data
      ↓
Bad format
      ↓
BadFormat
```

### Exam memory

> **BadFormat → data format problem**

---

# 8. `BadRequest`

Think:

> The request itself is invalid.

Conceptually:

```text
Ingestion request
      ↓
Invalid request
      ↓
BadRequest
```

### Memory

> **BadRequest → request problem**

---

# 9. `DataAccessNotAuthorized`

Think:

> The ingestion process is not authorized to access the required data.

Conceptually:

```text
Ingestion
   ↓
Access source/data
   ↓
Authorization failure
   ↓
DataAccessNotAuthorized
```

### Memory

> **DataAccessNotAuthorized → permissions/access**

---

# 10. `DownloadFailed`

Think:

> The ingestion process could not download/read the required source data.

Conceptually:

```text
Source file/data
      ↓
Download
      ↓
Failure
      ↓
DownloadFailed
```

### Memory

> **DownloadFailed → download problem**

---

# 11. `EntityNotFound`

Think:

> The referenced entity cannot be found.

For example, conceptually:

```text
Requested entity
      ↓
Not found
      ↓
EntityNotFound
```

### Memory

> **EntityNotFound → referenced object doesn't exist**

---

# 12. `FileTooLarge`

Think:

> The input file exceeds the applicable size limit.

```text
Large file
    ↓
Size limit exceeded
    ↓
FileTooLarge
```

### Memory

> **FileTooLarge → file size problem**

---

# 13. `InternalServiceError`

Think:

> An internal service-side problem occurred.

```text
Ingestion
   ↓
Fabric/service issue
   ↓
InternalServiceError
```

This may be a situation where retry can be relevant depending on the failure classification.

---

# 14. `UpdatePolicyFailure`

This points toward a problem involving an **update policy**.

Conceptually:

```text
Source table
     ↓
Update policy
     ↓
Derived table
     ↓
Failure
     ↓
UpdatePolicyFailure
```

The next important question is whether the update policy is:

```text
Transactional
```

or:

```text
Non-transactional
```

---

# 15. `ThrottledOnEngine`

Think:

> The ingestion operation is being throttled by the underlying engine/capacity.

Conceptually:

```text
High workload
     ↓
Engine throttling
     ↓
ThrottledOnEngine
```

This is a signal to investigate capacity/throughput pressure rather than automatically treating it as a data-format problem.

---

# 16. `General_RetryAttemptsExceeded`

This is an important exam clue.

If you see:

```text
General_RetryAttemptsExceeded
```

it means:

> **The platform has already retried and eventually gave up.**

Therefore, don't blindly add endless application-level retries.

Conceptually:

```text
Failure
  ↓
Platform retry
  ↓
Retry
  ↓
Retry
  ↓
Attempts exhausted
  ↓
General_RetryAttemptsExceeded
```

### Exam memory

> **RetryAttemptsExceeded = the platform already retried.**

---

# 17. Update Policies

An update policy can automatically transform data from a source table into a target/derived table.

Conceptually:

```text
Source table
     ↓
Update policy
     ↓
Transformation
     ↓
Target / derived table
```

Example idea:

```text
RawEvents
   ↓
Update Policy
   ↓
SummaryEvents
```

The important troubleshooting question is:

> What happens to the source if the target update fails?

That depends on transactionality.

---

# 18. Transactional Update Policy

A **transactional update policy** means the source and target are treated together for the operation.

The supplied material states:

> If the derived target fails, the source and target roll back together.

Conceptually:

```text
Source
  +
Target
  ↓
Transactional update
  ↓
Target succeeds
  → Commit both

Target fails
  → Roll back source + target
```

### Why is this important?

It provides stronger correctness.

The source isn't considered successfully landed if the required derived target could not be updated.

---

# 19. Non-Transactional Update Policy

A non-transactional update policy behaves differently.

The supplied material states:

> The source commits, while only the target write is skipped when the target operation fails.

Conceptually:

```text
Source
  +
Target
  ↓
Non-transactional update
  ↓
Source succeeds
  → Source commits

Target fails
  → Target write skipped
```

The raw/source data remains available even though the derived table wasn't updated.

---

# 20. Transactional vs Non-Transactional

| Feature | Transactional | Non-transactional |
|---|---|---|
| Source commits if target fails? | ❌ No | ✅ Yes |
| Target failure rolls back source? | ✅ Yes | ❌ No |
| Correctness coupling | Strong | Looser |
| Blast radius | Larger | Smaller |
| Default recommendation | Only when required | Safer default for most derived tables |

### Key memory

> **Transactional = source + target succeed/fail together**

> **Non-transactional = source can land even if derived target fails**

---

# 21. When Should You Use Transactional?

Use a transactional update policy when:

> The derived table's correctness is a **hard requirement** for considering the source data successfully landed.

Example:

```text
Raw source
   ↓
Mandatory derived table
   ↓
Business process depends on derived table
```

If the derived table fails, you don't want the source to be considered successfully landed.

Then:

```text
Transactional
```

makes sense.

---

# 22. Why Non-Transactional Is Usually Safer

For many derived/summary tables, you don't necessarily want a target transformation failure to block raw ingestion.

Example:

```text
RawEvents
    ↓
DerivedSummary
```

If the summary transformation fails:

```text
RawEvents → Still lands
DerivedSummary → Fails
```

You can fix/reprocess the derived table later.

This isolates the failure.

### Memory

> **Non-transactional = smaller blast radius**

---

# 23. What Does "Blast Radius" Mean?

Blast radius means:

> How much otherwise-good work is affected by a failure?

### Transactional

```text
Target fails
    ↓
Source rolls back too
    ↓
Larger blast radius
```

### Non-transactional

```text
Target fails
    ↓
Source still commits
    ↓
Smaller blast radius
```

---

# 24. Eventstream Monitoring

Eventstream monitoring has two important areas:

```text
Data insights
Runtime logs
```

They answer different questions.

---

# 25. Data Insights

**Data insights** are about the flow of data and throughput/status.

Think:

```text
How much data is flowing?
Is the stream healthy?
What is the throughput/status?
```

### Memory

> **Data insights = FLOW / THROUGHPUT**

---

# 26. Runtime Logs

**Runtime logs** provide engine-level details.

The supplied material identifies severities:

```text
Warning
Error
Information
```

Think:

```text
Something went wrong
       ↓
Runtime logs
       ↓
Engine-level details
```

### Memory

> **Runtime logs = WHY / ENGINE DETAIL**

---

# 27. Data Insights vs Runtime Logs

| Monitoring area | Main question |
|---|---|
| **Data insights** | How is data flowing? |
| **Runtime logs** | What happened inside the engine? |

### Exam memory

> **Data insights = throughput/status**

> **Runtime logs = engine-level cause**

---

# 28. Eventstream Schema Validation

One important operational recommendation is:

> Validate the schema using the **live sample view before saving** an Eventstream processor.

Why?

Suppose your processor expects:

```text
CustomerID
Temperature
Timestamp
```

But the actual incoming event has:

```text
customer_id
temp
event_time
```

Your processor may not match the incoming schema correctly.

If you don't validate before saving:

```text
Processor saved
      ↓
Stream starts
      ↓
Unexpected schema mismatch
      ↓
Under-matching / incorrect processing
```

---

# 29. Live Sample View

The safer workflow is:

```text
Incoming event
      ↓
Live sample view
      ↓
Inspect actual schema
      ↓
Configure processor
      ↓
Validate fields
      ↓
Save
```

### Exam memory

> **Validate schema BEFORE saving the processor.**

Not:

```text
Save
 ↓
Wait for problems
 ↓
Investigate schema
```

---

# 30. What Does "Under-Matching" Mean?

Under-matching means your processor's matching logic doesn't match as many incoming events/fields as expected.

For example:

```text
Expected:
DeviceId

Incoming:
deviceId
```

If the matching/configuration doesn't align, events may not be processed as intended.

### Key idea

> **Live sample = verify the actual incoming schema before committing the processor configuration.**

---

# 31. Eventstream Connection Error — `AADSTS65002`

This is a very important exam scenario.

Error:

```text
AADSTS65002
```

The supplied material says this means:

> The Eventstream is not preauthorized to access an Event Hub at the tenant level.

This is an **authorization/preauthorization issue**, not simply an end-user connection setting.

---

# 32. Why Reconfiguring the Connection Won't Solve `AADSTS65002`

Imagine:

```text
Eventstream
     ↓
Event Hub
     ↓
Tenant-level authorization
```

If the tenant has not preauthorized the required access:

```text
User changes connection settings
       ↓
Still not authorized
       ↓
AADSTS65002
```

The problem exists above the individual connection configuration.

---

# 33. What Should You Do for `AADSTS65002`?

The supplied guidance says:

> Escalate it to a **tenant administrator immediately**.

Do not repeatedly:

```text
Edit connection
Save
Retry
Edit connection
Save
Retry
```

Instead:

```text
AADSTS65002
      ↓
Recognize tenant-level preauthorization problem
      ↓
Tenant admin
      ↓
Fix authorization
```

### Exam memory

> **AADSTS65002 = tenant-level authorization → tenant admin**

---

# 34. Eventhouse Always-On

Suppose an Eventhouse is feeding a time-sensitive Eventstream destination.

You want the Eventhouse to be ready to respond without an unexpected startup delay.

The supplied recommendation is:

> Enable **Eventhouse Always-On** or set a **minimum consumption unit**.

Conceptually:

```text
Eventstream destination
        ↓
Time-sensitive workload
        ↓
Eventhouse
        ↓
Always-On / minimum CU
```

---

# 35. Why Always-On Matters

If the workload is time-sensitive, startup/availability delays can affect downstream processing.

Therefore:

```text
Time-sensitive destination
       ↓
Avoid avoidable startup delay
       ↓
Always-On
```

or:

```text
Set minimum consumption unit
```

### Exam signal

If the scenario emphasizes:

- time-sensitive destination
- Eventhouse
- immediate/consistent availability

Think:

> **Always-On or minimum consumption unit**

---

# 36. Queued Ingestion and Batching Latency

Queued ingestion may batch data before it becomes visible.

This can create latency.

Important:

> **Batching latency is not automatically a failure.**

Conceptually:

```text
Events arrive
    ↓
Queued ingestion
    ↓
Batching
    ↓
Ingestion
    ↓
Data becomes available
```

There may be a delay because the system is intentionally batching data.

---

# 37. How to Troubleshoot Batching Latency

Don't immediately conclude:

```text
Eventhouse ingestion is broken
```

Instead:

```text
Observed delay
      ↓
Check ingestion policy/threshold
      ↓
Determine expected batching behavior
      ↓
Only troubleshoot further if behavior exceeds expectations
```

### Exam memory

> **Queued ingestion delay can be expected behavior.**

---

# 38. Complete Ingestion Troubleshooting Flow

Use this in the exam:

```text
Eventhouse ingestion problem
           ↓
.show ingestion failures
           ↓
Filter by FailureKind
           │
      ┌────┴────┐
      ▼         ▼
 Permanent   Transient
      │         │
      ▼         ▼
Fix cause    Retry may help
      │
      ▼
Check specific category
```

Then consider:

```text
BadFormat
BadRequest
DataAccessNotAuthorized
DownloadFailed
EntityNotFound
FileTooLarge
InternalServiceError
UpdatePolicyFailure
ThrottledOnEngine
```

---

# 39. Complete Update Policy Decision

```text
Update policy
     ↓
Does target correctness determine
whether source is considered landed?
     │
     ├── YES
     │     ↓
     │  Transactional
     │     ↓
     │  Source + target rollback together
     │
     └── NO
           ↓
       Non-transactional
           ↓
       Source commits
           ↓
       Target can fail independently
```

### Memory

> **Hard correctness requirement → Transactional**

> **Most derived/summary tables → Non-transactional**

---

# 40. Complete Eventstream Monitoring Decision

```text
Eventstream problem
       ↓
What are you asking?
       │
       ├── "How is data flowing?"
       │       ↓
       │   Data insights
       │
       └── "Why did the engine fail?"
               ↓
           Runtime logs
               ↓
       Warning/Error/Information
```

---

# 41. Complete Eventstream Schema Decision

```text
Create processor
      ↓
Live sample view
      ↓
Check actual schema
      ↓
Configure fields/matching
      ↓
Validate
      ↓
Save processor
```

### Avoid:

```text
Save first
 ↓
Wait
 ↓
Discover schema mismatch
```

---

# 42. Complete `AADSTS65002` Decision

```text
Eventstream → Event Hub connection
          ↓
AADSTS65002
          ↓
Tenant-level preauthorization missing
          ↓
Tenant administrator
```

### Don't:

```text
Repeatedly edit user connection settings
```

---

# 43. DP-700 Exam Tips

## `.show ingestion failures`

Remember:

```text
.show ingestion failures
```

Important:

- Filter by **`FailureKind`** first.
- `FailureKind` = `Permanent` or `Transient`.
- Retention = **14 days**.
- Command-level diagnostic surface.
- Pair with metrics/diagnostic logs for broader coverage.

---

## Ingestion categories

Memorize:

```text
BadFormat
BadRequest
DataAccessNotAuthorized
DownloadFailed
EntityNotFound
FileTooLarge
InternalServiceError
UpdatePolicyFailure
ThrottledOnEngine
```

---

## Retry

```text
Transient
    ↓
Retry may help
```

```text
General_RetryAttemptsExceeded
    ↓
Platform already retried
    ↓
Don't blindly add endless application retries
```

---

## Update policies

```text
Transactional
    ↓
Source + target rollback together
```

```text
Non-transactional
    ↓
Source commits
    ↓
Target write can fail independently
```

### Exam clue

> Derived table correctness is mandatory for considering source data "landed" → **Transactional**

Otherwise:

> **Non-transactional is generally safer for derived/summary tables.**

---

## Eventstream monitoring

```text
Data insights
    ↓
Throughput / status / flow
```

```text
Runtime logs
    ↓
Engine-level detail
    ↓
Warning / Error / Information
```

---

## Eventstream schema

> **Live sample view → validate schema → save processor**

---

## Event Hub authorization

```text
AADSTS65002
    ↓
Tenant-level preauthorization problem
    ↓
Tenant admin
```

---

## Eventhouse availability

For a time-sensitive Eventstream destination:

```text
Eventhouse
   ↓
Always-On
```

or:

```text
Minimum consumption unit
```

---

## Queued ingestion

> Batching latency can be expected behavior.

Check the relevant ingestion policy/threshold before declaring a failure.

---

# 44. Key Takeaways

- Start ingestion-failure triage with `.show ingestion failures` filtered by **`FailureKind`**.
- `FailureKind` is **Permanent** or **Transient**.
- **Transient** means retry may be useful; **Permanent** means investigate/fix first.
- `.show ingestion failures` has **14-day retention**.
- It is command-level diagnostic information, so pair it with metrics and diagnostic logs for broader coverage.
- Know the major ingestion error categories: `BadFormat`, `BadRequest`, `DataAccessNotAuthorized`, `DownloadFailed`, `EntityNotFound`, `FileTooLarge`, `InternalServiceError`, `UpdatePolicyFailure`, `ThrottledOnEngine`.
- `General_RetryAttemptsExceeded` means the platform already retried and gave up.
- **Transactional update policy** = source and target roll back together on failure.
- **Non-transactional update policy** = source commits even if target processing fails.
- Use transactional behavior when derived-table correctness is a hard requirement for considering source data landed.
- Non-transactional is generally the safer default for many derived/summary tables because it reduces blast radius.
- Eventstream **Data insights** answers flow/throughput questions.
- Eventstream **Runtime logs** provide engine-level details at warning/error/information severities.
- Validate Eventstream processor schema using the **live sample view before saving**.
- `AADSTS65002` indicates a tenant-level preauthorization issue for the Event Hub connection and should be escalated to a tenant admin.
- For time-sensitive Eventstream destinations backed by Eventhouse, use **Always-On** or a **minimum consumption unit**.
- Queued-ingestion batching latency can be expected behavior; check the relevant policy/threshold before treating it as a failure.

---

# 45. Ultimate Memory Sheet

```text
INGESTION FAILURE
        ↓
.show ingestion failures
        ↓
Filter FailureKind FIRST
        │
        ├── Permanent
        │      ↓
        │   Fix cause
        │
        └── Transient
               ↓
          Retry may help
```

```text
FAILURE CATEGORIES
 ↓
BadFormat
BadRequest
DataAccessNotAuthorized
DownloadFailed
EntityNotFound
FileTooLarge
InternalServiceError
UpdatePolicyFailure
ThrottledOnEngine
```

```text
UPDATE POLICY
 ↓
Hard correctness requirement?
      │
      ├── YES → Transactional
      │          ↓
      │     Source + Target rollback
      │
      └── NO  → Non-transactional
                 ↓
            Source commits
            Target can fail
```

```text
EVENTSTREAM MONITORING
 ↓
Data insights  → FLOW / THROUGHPUT
Runtime logs   → ENGINE / WHY
```

```text
EVENTSTREAM PROCESSOR
 ↓
Live sample
 ↓
Validate schema
 ↓
Save
```

```text
AADSTS65002
 ↓
Tenant-level preauthorization
 ↓
Tenant Admin
```

```text
TIME-SENSITIVE EVENTSTREAM DESTINATION
 ↓
Eventhouse
 ↓
Always-On OR minimum consumption unit
```

```text
QUEUED INGESTION
 ↓
Batching latency
 ↓
Can be expected
 ↓
Check policy/threshold before troubleshooting
```

## Final Exam Memory

> **FailureKind decides the first move.**

> **Transactional decides the blast radius.**

> **Data insights tells you how the stream is flowing.**

> **Runtime logs tell you what the engine is doing.**

> **AADSTS65002 = tenant admin.**

> **Live sample before saving.**

> **Queued ingestion delay ≠ automatically a failure.**
