# DP-700 — Schedules, Triggers & Retry

## 1. Pipeline Run Types

There are three ways to start a pipeline run:

1. **On-demand** — manual execution
2. **Scheduled** — time-based execution
3. **Event-based** — execution when an event occurs

### Memory
> ON-DEMAND = Manual  
> SCHEDULED = Time  
> EVENT-BASED = Event

---

# 2. Fixed Schedule

A fixed schedule requires:

- **Start date**
- **End date**

### Exam Rule
> **Fixed schedule = mandatory START + END date**

---

# 3. Interval-Based Schedule

Important exam points:

- **Preview**
- Cannot be edited in place
- Exposes **window start/end time trigger parameters**

### Memory
> **INTERVAL = Preview + Window parameters + Cannot edit in place**

---

# 4. Event-Based Triggers

An event trigger starts a pipeline when a specified event occurs.

For file-arrival scenarios, prefer an event trigger over tight polling.

### Why?

- Lower latency
- Fewer unnecessary pipeline runs
- Avoids repeatedly checking for a file

### Polling

```text
Every 1 minute
     ↓
Check storage
     ↓
File exists?
     ↓
No → Check again
```

### Event-based

```text
File arrives
     ↓
Event detected
     ↓
Pipeline starts
```

### Memory
> **FILE ARRIVAL → EVENT TRIGGER**

---

# 5. Storage Event Triggers

Storage event triggers run through **Data Activator**.

The study material describes:

```text
Storage Event
      ↓
Data Activator
      ↓
Reflex Item
      ↓
Auto-created Eventstream
      ↓
Pipeline Trigger
```

### Exam Point

The **Reflex item** is the actual trigger artifact.

For external sources, an **eventstream can be automatically created**.

### Memory
> **Event Trigger → Data Activator → Reflex**

---

# 6. TriggerEvent and Null-Safe `?`

A trigger-event expression may access event information such as the file name.

### Memorize

```text
@pipeline()?.TriggerEvent?.FileName
```

### Why `?`?

A manual test run may not have a real trigger event.

Therefore:

```text
TriggerEvent = NULL
```

can occur.

Without null-safe access, the expression can fail.

### Memory
> **Manual Test → TriggerEvent may be NULL → use `?`**

---

# 7. Notebook Scheduling

Notebooks and Dataflow Gen2 can be scheduled independently of pipelines.

Notebook execution identity depends on how the run was triggered.

| Trigger | Execution identity |
|---|---|
| Interactive | Current user |
| Pipeline-invoked | Pipeline's last-modified user |
| Scheduled | Schedule's creator / last-updater |

### Memory
> Interactive → YOU  
> Pipeline → Pipeline last modifier  
> Scheduled → Schedule creator / last updater

---

# 8. Production Notebook Schedule Identity

For native notebook schedules, use a **dedicated, correctly-permissioned identity** instead of an individual's personal account.

### Why?

An individual's account might later:

- Lose access
- Change role
- Leave the organization
- Have permissions changed

### Exam Memory

> **Production schedule → Dedicated identity**

---

# 9. Failure Notifications

Set failure notifications on production schedules.

They provide the built-in mechanism to notify someone when a scheduled workload fails.

### Pattern

```text
Production Schedule
      ↓
Workload fails
      ↓
Failure Notification
      ↓
Operator is informed
```

### Memory
> **Production schedule → Failure notification**

---

# 10. Retry

Retry is configured **per activity**.

According to the study material:

- Up to **1000 attempts**
- Configurable retry interval

Retry should be used carefully, especially for expensive activities such as:

- Notebook
- Dataflow

---

# 11. Retry Conditions

Conditional retry is **Preview**.

It applies to:

- **Copy Data**
- **Notebook**
- **Dataflow**
- **Stored Procedure**

### Memory
> **Conditional Retry = Preview + Copy + Notebook + Dataflow + Stored Procedure**

---

# 12. Scope Retry Conditions Narrowly

Do not leave retries blanket-enabled for every possible failure.

Prefer specific retry conditions, such as known transient error codes.

### Good pattern

```text
Transient error
      ↓
Retry
      ↓
Success
```

### Bad pattern

```text
Permanent configuration error
      ↓
Retry
      ↓
Same error
      ↓
Retry again
      ↓
Wasted compute/time
```

### Best Practice

> **Transient failure → Retry**

> **Permanent failure → Fix the cause**

This is especially important for expensive Notebook and Dataflow activities.

---

# 13. Retry Interval Ordering — EXAM TRAP

This is a very important detail:

> **The retry interval always elapses BEFORE the retry condition is checked.**

### Example

```text
Activity fails
      ↓
Wait retry interval
      ↓
Check retry condition
      ↓
Condition matches?
    /         YES        NO
   ↓          ↓
 Retry     No retry
```

Even if the condition does not match, the configured retry interval is still incurred **before** the condition is evaluated.

### Memory

> **WAIT FIRST → CHECK CONDITION SECOND**

---

# 14. Conditional Retry Support

According to the study material, Preview conditional retry covers only:

| Activity | Conditional Retry |
|---|---|
| Copy Data | ✅ |
| Notebook | ✅ |
| Dataflow | ✅ |
| Stored Procedure | ✅ |
| Other activities | ❌ |

### Exam Trap

Do not assume conditional retry applies to every activity.

---

# 15. Choosing Trigger Type

| Requirement | Choose |
|---|---|
| Manual execution | On-demand |
| Specific time | Scheduled |
| File arrival | Event-based |
| Avoid tight polling | Event-based |
| Fixed schedule | Fixed schedule |
| Window-oriented interval | Interval-based |
| Event information may be NULL | Use `?` |
| Production native notebook schedule | Dedicated identity |
| Production failure awareness | Failure notification |
| Temporary/transient failure | Retry |
| Permanent failure | Do not blindly retry |

---

# 16. Common Exam Traps

## Trap 1 — Tight polling for file arrival

❌ Prefer event-based triggering when available.

> **File arrival → Event trigger**

## Trap 2 — Forgetting `?`

Risky pattern:

```text
@pipeline().TriggerEvent.FileName
```

Memorize:

```text
@pipeline()?.TriggerEvent?.FileName
```

## Trap 3 — Assuming TriggerEvent always exists

❌ Manual tests may not contain a real trigger event.

## Trap 4 — Forgetting Data Activator / Reflex

Remember:

> Storage event → Data Activator → Reflex

## Trap 5 — Personal identity for production schedules

❌ Prefer a dedicated, correctly-permissioned identity.

## Trap 6 — Retry every failure

❌ Scope retries to appropriate transient failures.

## Trap 7 — Checking retry condition before waiting

❌ Incorrect.

Correct order:

> **Retry interval → condition evaluation**

## Trap 8 — Conditional retry applies everywhere

❌ Incorrect.

Preview conditional retry applies to:

> **Copy + Notebook + Dataflow + Stored Procedure**

## Trap 9 — Forgetting failure notifications

Production schedules should have failure notifications configured.

---

# 17. Best Practices

### 1. File Arrival
> Prefer event-based triggers over tight polling.

### 2. TriggerEvent
Use:

```text
@pipeline()?.TriggerEvent?.FileName
```

### 3. Retry
Scope retry conditions narrowly to specific transient error codes.

### 4. Production Notebook Schedule
Use a dedicated, correctly-permissioned identity.

### 5. Production Schedules
Configure failure notifications.

---

# 18. Complete Architecture Example

```text
File Arrives
     ↓
Storage Event
     ↓
Data Activator
     ↓
Reflex
     ↓
Pipeline
     |
     +---- Copy Data
     |
     +---- Dataflow Gen2
     |
     +---- Notebook
     |
     +---- Failure Handling
     |
     +---- Failure Notification
```

---

# 19. Key Takeaways

- Three ways to start a pipeline run:
  - On-demand
  - Scheduled
  - Event-based via Data Activator
- Fixed schedules require a **mandatory start and end date**.
- Interval-based schedules are **Preview**.
- Interval-based schedules **cannot be edited in place**.
- Interval-based schedules expose **window start/end time trigger parameters**.
- Prefer event-based triggers over tight polling for file-arrival scenarios.
- Event triggers can reduce unnecessary runs and latency.
- Storage event triggers run through **Data Activator**.
- The **Reflex item** is the actual trigger artifact.
- External event sources can have an **auto-created eventstream**.
- Memorize:
  ```text
  @pipeline()?.TriggerEvent?.FileName
  ```
- The `?` operator protects against NULL trigger references during manual testing.
- Notebooks and Dataflow Gen2 can schedule independently of pipelines.
- Notebook execution identity depends on trigger type.
- Interactive → current user.
- Pipeline-invoked → pipeline's last-modified user.
- Scheduled → schedule's creator / last-updater.
- Use a dedicated identity for production native notebook schedules.
- Configure failure notifications on production schedules.
- Retry is configured **per activity**.
- Retry supports up to **1000 attempts** and a configurable interval.
- Conditional retry is **Preview**.
- Conditional retry applies to **Copy Data, Notebook, Dataflow, and Stored Procedure**.
- Scope retry conditions narrowly.
- The **retry interval always elapses before condition evaluation**.

---

# 20. One-Minute Revision

```text
PIPELINE START
│
├── ON-DEMAND
│     └── Manual
│
├── SCHEDULED
│     ├── Fixed
│     │     └── START + END mandatory
│     │
│     └── Interval
│           ├── Preview
│           ├── Cannot edit in place
│           └── Window parameters
│
└── EVENT-BASED
      └── Data Activator / Reflex
```

```text
FILE ARRIVAL
    ↓
EVENT TRIGGER
    ↓
LOWER LATENCY + LESS POLLING
```

```text
TRIGGEREVENT
    ↓
May be NULL during manual test
    ↓
Use ?
    ↓
@pipeline()?.TriggerEvent?.FileName
```

```text
RETRY
    ↓
Activity fails
    ↓
WAIT RETRY INTERVAL
    ↓
CHECK CONDITION
    ↓
Match → Retry
No match → No retry
```

## Final Memory Formula

> **FILE ARRIVAL → EVENT**

> **FIXED → START + END**

> **INTERVAL → PREVIEW + NO IN-PLACE EDIT**

> **TRIGGEREVENT → `?`**

> **PRODUCTION SCHEDULE → DEDICATED IDENTITY + FAILURE NOTIFICATION**

> **TRANSIENT ERROR → RETRY**

> **RETRY → WAIT FIRST, CHECK CONDITION SECOND**

> **CONDITIONAL RETRY → COPY + NOTEBOOK + DATAFLOW + STORED PROCEDURE**
