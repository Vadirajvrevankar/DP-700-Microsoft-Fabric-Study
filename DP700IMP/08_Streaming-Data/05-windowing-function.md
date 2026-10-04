# DP-700 — Windowing in Eventstream, KQL & Spark

## 1. What Is Windowing?

**Windowing** means dividing a continuous stream of events into groups based on time, activity, or another boundary so that you can calculate:

- COUNT
- SUM
- AVG
- MIN / MAX
- Other aggregations

Example:

```text
Continuous events
      ↓
Create windows
      ↓
COUNT / SUM / AVG
      ↓
Results
```

Example requirement:

> Calculate the number of orders every 5 minutes.

This is a **windowed aggregation**.

---

# 2. Two Major Ways Windows Are Defined

## 2.1 Clock-Driven Windows

The clock determines the window boundaries.

Examples:

- Tumbling
- Hopping

Example:

```text
10:00
10:05
10:10
10:15
```

The clock determines when windows start/end.

---

## 2.2 Activity-Driven Windows

The activity of the events determines the window.

Example:

- Session window

If a user remains active, events belong to the same session.

After a specified period of inactivity, the session ends.

### Memory Trick

> **Clock → Tumbling / Hopping**

> **Activity → Session**

---

# 3. Five Conceptual Window Types

The five conceptual window types in these notes are:

1. Tumbling
2. Hopping
3. Sliding
4. Session
5. Snapshot

---

# 4. Tumbling Window

A **tumbling window** is:

> **Fixed size + no overlap**

Example:

```text
5-minute tumbling window

10:00 ───── 10:05
10:05 ───── 10:10
10:10 ───── 10:15
10:15 ───── 10:20
```

The windows do not overlap.

### Example

Requirement:

> Count orders every 5 minutes.

Use:

**Tumbling window**

### Memory Trick

> **Tumbling = Fixed + No Overlap**

---

# 5. Hopping Window

A **hopping window** has:

> **Fixed window size + overlap**

It is controlled by:

- Window size
- Hop/slide interval

Example:

```text
Window size = 10 minutes
Hop = 5 minutes
```

Windows:

```text
10:00 ───────── 10:10
       Window 1

10:05 ───────── 10:15
       Window 2

10:10 ───────── 10:20
       Window 3
```

The windows overlap.

### Memory Trick

> **Hopping = Fixed + Overlap**

---

# 6. Tumbling vs Hopping

| Feature | Tumbling | Hopping |
|---|---|---|
| Fixed size | Yes | Yes |
| Overlap | ❌ No | ✅ Yes |
| Hop interval | No | Yes |
| Example | Every 5 min | 10-min window every 5 min |

---

# 7. Sliding Window

A **sliding window** represents a moving range of recent data.

Your study notes describe it as:

> **Emits on content change.**

Conceptually:

```text
Current time
     ↓

───────[ Recent time range ]───────
              ↓
        continuously moves
```

The result can change as new events/content affect the window.

### Exam Signal

If the question specifically describes:

> Sliding window

do not confuse it with a simple tumbling bucket.

### Memory Trick

> **Sliding = Moving**

---

# 8. Session Window

A **session window** is:

> **Activity-gap-driven**

It does not use fixed clock boundaries like a tumbling window.

Instead, events remain in the same session until there has been no activity for a specified period.

Example:

```text
10:00 → click
10:01 → click
10:03 → purchase
10:04 → click

        ↓

Same session
```

Then:

```text
No activity for 30 minutes
        ↓
Session ends
```

If the user returns later:

```text
10:40 → click
```

a new session begins.

### Example

Online shopping:

```text
10:00 → Opens website
10:02 → Searches phone
10:04 → Opens product
10:06 → Adds to cart
10:08 → Checkout
```

These activities belong to one session.

If there is no activity for the configured gap, the session ends.

### Memory Trick

> **Session = Activity + Inactivity Gap**

---

# 9. Snapshot Window

A **snapshot** groups events based on the same timestamp.

Example:

```text
Timestamp = 10:00:00

Event A
Event B
Event C

      ↓

Snapshot group
```

### Exam Signal

If the scenario specifically describes:

> **Same-timestamp grouping**

think:

**Snapshot**

---

# 10. Complete Window Comparison

| Window | Main idea | Overlap? | Driven by |
|---|---|---:|---|
| **Tumbling** | Fixed, non-overlapping | ❌ | Clock |
| **Hopping** | Fixed, overlapping | ✅ | Clock |
| **Sliding** | Moving window / emits on content change | Can overlap | Moving/content change |
| **Session** | Activity separated by inactivity gap | Activity-based | Activity |
| **Snapshot** | Same-timestamp grouping | N/A | Timestamp |

### High-Value Memory

> **Tumbling/Hopping = Clock-driven**

> **Session = Activity-driven**

> **Sliding = Moving/content-change behavior**

> **Snapshot = Same timestamp**

---

# 11. How to Choose the Window

Before choosing syntax, identify the window type.

## Question 1: Fixed clock period?

Example:

> Every 5 minutes

Think:

**Tumbling**

---

## Question 2: Fixed window with overlap?

Example:

> Calculate a 10-minute result every 5 minutes.

Think:

**Hopping**

---

## Question 3: Activity with inactivity gap?

Example:

> Group user events until the user is inactive for 20 minutes.

Think:

**Session**

---

## Question 4: Moving/content-change behavior?

Think:

**Sliding**

---

## Question 5: Same timestamp?

Think:

**Snapshot**

---

# 12. Eventstream Windows

Eventstream's **Group by** operator supports:

- Tumbling
- Hopping
- Sliding
- Session

Conceptually:

```text
Eventstream
    ↓
Group by
    ↓
Choose window type
    ↓
Aggregate
```

---

# 13. Eventstream Example

Requirement:

> Count orders every 5 minutes.

Conceptually:

```text
Eventstream
     ↓
Group by
     ↓
Tumbling window
     ↓
5 minutes
     ↓
COUNT
```

### Exam Signal

> **Eventstream + windowed grouping → Group by**

---

# 14. KQL Windowing

The important KQL constructs in these notes are:

- `bin()`
- `summarize`
- `row_window_session()`

---

# 15. KQL `bin()`

`bin()` performs **query-time time bucketing**.

Example:

```kusto
Orders
| summarize Count=count()
    by bin(Timestamp, 5m)
```

This creates 5-minute buckets.

Conceptually:

```text
10:00–10:05
10:05–10:10
10:10–10:15
```

This corresponds to **tumbling-style bucketing**.

---

# 16. How `bin()` Works

Example:

```kusto
bin(Timestamp, 5m)
```

means:

> Put timestamps into 5-minute buckets.

Conceptually:

```text
10:01 → 10:00
10:02 → 10:00
10:04 → 10:00

10:06 → 10:05
10:08 → 10:05
```

Then `summarize` can aggregate each bucket.

---

# 17. Important: KQL `bin()` vs Spark Streaming Windows

This is a key conceptual distinction.

## KQL `bin()`

`bin()` is **query-time bucketing**.

It works against the current contents of the table whenever the query runs.

Conceptually:

```text
Stored KQL table
       ↓
Run query
       ↓
bin()
       ↓
Buckets
       ↓
summarize
```

It is not a continuously maintained streaming window state in the same sense as Spark Structured Streaming windows.

---

# 18. KQL Session Windows

The notes identify:

```kusto
row_window_session()
```

as the KQL construct associated with session-style grouping.

Conceptually:

```text
Activity
  ↓
Events close together
  ↓
Same session

Long inactivity gap
  ↓
New session
```

---

# 19. KQL Hopping and Sliding

According to the comparison in these study notes:

> KQL does not have dedicated hopping/sliding window functions.

The dedicated constructs identified here are:

```text
bin()
↓
Tumbling-style bucketing
```

and:

```text
row_window_session()
↓
Session
```

### Exam Memory

> **KQL → `bin()` for tumbling-style bucketing**

> **KQL → `row_window_session()` for session**

---

# 20. Spark Windowing

Spark Structured Streaming provides explicit window functions.

## Spark Tumbling Window

Syntax:

```python
window("timestamp", "5 minutes")
```

Example:

```python
df.groupBy(
    window("timestamp", "5 minutes")
).count()
```

This creates 5-minute tumbling windows.

### Memory

> `window(col, size)` → **Tumbling**

---

# 21. Spark Hopping Window

Spark supports hopping windows using:

```python
window(
    "timestamp",
    "10 minutes",
    "5 minutes"
)
```

Meaning:

```text
Window size = 10 minutes
Slide = 5 minutes
```

Windows:

```text
10:00–10:10
10:05–10:15
10:10–10:20
```

These overlap.

### Memory

> `window(col, size, slide)` → **Hopping**

---

# 22. Spark Session Window

Spark provides:

```python
session_window("timestamp", "30 minutes")
```

Conceptually:

```text
Activity
Activity
Activity
    ↓
Same session

30 minutes inactivity
    ↓
Session ends
```

### Memory

> `session_window(col, gap)` → **Session**

---

# 23. Spark Syntax Cheat Sheet

```text
window(col, size)
        ↓
Tumbling
```

```text
window(col, size, slide)
        ↓
Hopping
```

```text
session_window(col, gap)
        ↓
Session
```

---

# 24. Engine Comparison

| Concept | Eventstream | KQL | Spark |
|---|---|---|---|
| Tumbling | Group by → Tumbling | `bin()` + `summarize` | `window(col, size)` |
| Hopping | Group by → Hopping | No dedicated function in these notes | `window(col, size, slide)` |
| Sliding | Group by → Sliding | No dedicated function in these notes | Depends on implementation |
| Session | Group by → Session | `row_window_session()` | `session_window(col, gap)` |
| Snapshot | Conceptual type | Not covered as a dedicated construct here | Not covered as a dedicated construct here |

### High-Value Exam Rule

> **Identify the engine first, then translate the requirement into that engine's syntax.**

---

# 25. Same Requirement — Three Engines

Requirement:

> **Calculate the number of events every 5 minutes using a tumbling window.**

## Eventstream

```text
Group by
   ↓
Tumbling
   ↓
5 minutes
   ↓
Count
```

## KQL

```kusto
Events
| summarize Count=count()
    by bin(Timestamp, 5m)
```

## Spark

```python
df.groupBy(
    window("Timestamp", "5 minutes")
).count()
```

The **business requirement is identical**.

The **syntax changes according to the engine**.

### Memory Trick

> **Requirement stays the same. Engine syntax changes.**

---

# 26. Why Identify the Engine First?

Suppose the exam says:

> "Using Spark Structured Streaming, calculate a 5-minute tumbling aggregate."

Think:

```text
Spark
 ↓
Tumbling
 ↓
window("timestamp", "5 minutes")
```

If it says:

> "Using KQL, group events into 5-minute buckets."

Think:

```text
KQL
 ↓
bin(timestamp, 5m)
 ↓
summarize
```

If it says:

> "Using Eventstream, calculate a 5-minute aggregate."

Think:

```text
Eventstream
 ↓
Group by
 ↓
Tumbling
```

---

# 27. Watermarks

A **watermark** is especially important for Spark Structured Streaming.

It helps Spark control how long it retains state for operations such as:

- Windowed aggregations
- Deduplication

The key idea:

> **Watermark = how much late data the system is willing to tolerate while retaining state.**

---

# 28. Why Does Spark Need a Watermark?

Imagine events:

```text
10:01
10:02
10:03
```

But an event belonging to:

```text
10:00
```

arrives later because of network or processing delay.

That is a **late event**.

Spark needs to know:

> How long should I keep the old window state in case more late events arrive?

The watermark helps answer this.

---

# 29. `withWatermark()`

Spark explicitly configures the watermark.

Example:

```python
df.withWatermark(
    "timestamp",
    "10 minutes"
)
```

A windowed aggregation can then be written as:

```python
df \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(
        window("timestamp", "5 minutes")
    ) \
    .count()
```

The pieces have different jobs:

```text
withWatermark()
      ↓
Bounds state / late-data tolerance

window()
      ↓
Defines the aggregation window

count()
      ↓
Calculates the result
```

---

# 30. Watermark Should Match Business Tolerance

Suppose the business requirement is:

> Events can arrive up to 30 minutes late.

A 2-minute watermark may be too aggressive.

The configuration should be chosen based on the workload's real late-data tolerance.

Example:

```python
withWatermark("timestamp", "30 minutes")
```

### Principle

> **Size the watermark according to the business's actual tolerance for late data.**

---

# 31. What If the Watermark Is Too Small?

Suppose:

```text
Business tolerance = 30 minutes
Watermark = 5 minutes
```

Very late events may arrive after Spark has already finalized and cleaned up the relevant state.

Those events can therefore be excluded from the already-finalized window.

So:

```text
Smaller watermark
       ↓
Less state
       ↓
Less late-data tolerance
       ↓
Greater risk of excluding very late data
```

---

# 32. What If the Watermark Is Too Large?

Suppose:

```text
Watermark = 2 hours
```

Spark can retain state for longer.

Therefore:

```text
Larger watermark
       ↓
More late-data tolerance
       ↓
More state retained
       ↓
Higher state/memory cost
```

### Fundamental Trade-Off

> **More tolerance → more state**

> **Less state → less tolerance**

---

# 33. Watermark Does NOT Reject Late Events From the Stream

This is a very important exam trap.

A watermark does **not** mean:

> "Reject every event that arrives late."

Instead, it controls how long streaming state is retained.

If an event arrives after the relevant window has already been finalized:

```text
Late event
    ↓
Window already finalized
    ↓
Event may be excluded from that finalized result
```

Therefore:

> **Watermark = state-retention boundary**

not:

> **Watermark = source-level rejection mechanism**

---

# 34. Watermark + Windowed Aggregation

Recommended Spark pattern:

```python
df \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(
        window("timestamp", "5 minutes")
    ) \
    .count()
```

Think:

```text
withWatermark()
      ↓
How long to retain state / tolerate late data

window()
      ↓
How to divide events

count()
      ↓
What to calculate
```

---

# 35. Watermark + Deduplication

The same state-management principle applies to streaming deduplication.

Without a watermark, Spark may need to remember an increasingly large amount of state.

Conceptually:

```text
Events
  ↓
Watermark
  ↓
Bound state
  ↓
Deduplication
```

Therefore, your study notes emphasize pairing:

- `dropDuplicates()`
- Windowed aggregation

with:

```python
.withWatermark(...)
```

to bound state growth.

---

# 36. Eventstream Late-Arrival Tolerance

Eventstream also has the concept of **late-arrival tolerance** for window processing.

The principle is similar:

```text
More tolerance
     ↓
Keep state longer
     ↓
Greater chance of including late events
```

versus:

```text
Less tolerance
     ↓
Finalize sooner
     ↓
Less state
     ↓
Greater chance of excluding very late events
```

### Memory

> **Spark → Watermark**

> **Eventstream → Late-arrival tolerance**

---

# 37. Clock-Driven vs Activity-Driven — Final Understanding

## Clock-Driven

The clock determines boundaries:

```text
10:00
10:05
10:10
10:15
```

Examples:

- Tumbling
- Hopping

---

## Activity-Driven

Activity determines boundaries:

```text
User active
   ↓
Event
Event
Event
   ↓
Inactivity gap
   ↓
Session ends
```

Example:

- Session

### Memory Trick

> **Clock → Tumbling/Hopping**

> **Activity → Session**

---

# 38. Window Selection Decision Tree

Use this during the exam:

```text
What does the requirement describe?
              │
              ▼
       Fixed time period?
          /          \
        Yes           No
        │             │
        ▼             ▼
   Overlapping?    Activity gap?
     /    \          /    \
   No      Yes      Yes    No
   │        │        │
   ▼        ▼        ▼
Tumbling  Hopping  Session
```

For the other conceptual types:

```text
Moving/content-change behavior
        ↓
Sliding

Same-timestamp grouping
        ↓
Snapshot
```

---

# 39. Exam Trap: Don't Pick Syntax First

Requirement:

> "Calculate sales every 5 minutes."

Do not immediately write code.

First identify:

```text
Every 5 minutes
      ↓
Fixed clock period
      ↓
Tumbling
```

Then identify the engine:

```text
Eventstream?
KQL?
Spark?
```

Then select the syntax.

### Correct order

```text
1. Identify requirement
        ↓
2. Identify window type
        ↓
3. Identify engine
        ↓
4. Select syntax
```

Or, when the engine is explicitly given:

```text
1. Identify engine
        ↓
2. Identify window type
        ↓
3. Select syntax
```

---

# 40. Exam Scenario Examples

## Scenario 1

> Count orders every 5 minutes with no overlap.

**Answer:**

Tumbling window.

---

## Scenario 2

> Calculate a 10-minute result every 5 minutes.

**Answer:**

Hopping window.

Reason:

```text
Window = 10 minutes
Hop = 5 minutes
```

Therefore windows overlap.

---

## Scenario 3

> Group user activity until the user has been inactive for 30 minutes.

**Answer:**

Session window.

---

## Scenario 4

> Use Eventstream to perform a windowed aggregation.

**Answer:**

Use:

```text
Group by
```

and select the appropriate window type.

---

## Scenario 5

> Use KQL to bucket events into 5-minute periods.

**Answer:**

Use:

```kusto
bin(Timestamp, 5m)
```

with `summarize`.

---

## Scenario 6

> Use Spark Structured Streaming for a 5-minute tumbling aggregation.

**Answer:**

Use:

```python
window("timestamp", "5 minutes")
```

and pair the streaming aggregation with an appropriate:

```python
.withWatermark(...)
```

---

## Scenario 7

> Use Spark for a 10-minute window that advances every 5 minutes.

**Answer:**

Hopping window:

```python
window("timestamp", "10 minutes", "5 minutes")
```

---

## Scenario 8

> Use Spark to group events into user sessions with a 30-minute inactivity gap.

**Answer:**

```python
session_window("timestamp", "30 minutes")
```

---

# 41. High-Value Exam Table

| Requirement | Window |
|---|---|
| Fixed + no overlap | **Tumbling** |
| Fixed + overlap | **Hopping** |
| Moving/content-change | **Sliding** |
| Activity + inactivity gap | **Session** |
| Same timestamp | **Snapshot** |

---

# 42. High-Value Engine Table

| Engine | Key construct |
|---|---|
| Eventstream | **Group by** |
| KQL tumbling-style bucketing | **`bin()` + `summarize`** |
| KQL session | **`row_window_session()`** |
| Spark tumbling | **`window(col, size)`** |
| Spark hopping | **`window(col, size, slide)`** |
| Spark session | **`session_window(col, gap)`** |

---

# 43. Spark Watermark Cheat Sheet

```python
.withWatermark("timestamp", "10 minutes")
```

Remember:

> **Watermark = bound state + tolerate late data**

It does **not** simply mean:

> Reject late events.

### Trade-Off

```text
More tolerance
     ↓
More state
```

```text
Less tolerance
     ↓
Less state
     ↓
More risk of excluding very late data
```

---

# 44. Complete Mental Model

```text
                WINDOWING
                    │
       ┌────────────┴────────────┐
       │                         │
   Clock-driven             Activity-driven
       │                         │
   ┌───┴────┐                    │
   │        │                    │
Tumbling  Hopping             Session
   │        │                    │
   │        │                    │
No overlap  Overlap        Inactivity gap
```

Additional conceptual types:

```text
Sliding
↓
Moving/content-change behavior

Snapshot
↓
Same-timestamp grouping
```

Then:

```text
             ENGINE
               │
      ┌────────┼────────┐
      │        │        │
 Eventstream  KQL     Spark
      │        │        │
 Group by    bin()   window()
             /        /      \
     row_window_   Tumbling Hopping
      session()       |
                    session_window()
```

---

# 45. Final DP-700 Cheat Sheet

## Window Types

```text
TUMBLING
↓
Fixed
↓
No overlap
↓
Clock-driven
```

```text
HOPPING
↓
Fixed
↓
Overlap
↓
Clock-driven
↓
Window size + hop
```

```text
SLIDING
↓
Moving/content-change behavior
```

```text
SESSION
↓
Activity-driven
↓
Inactivity gap
```

```text
SNAPSHOT
↓
Same-timestamp grouping
```

---

## Eventstream

```text
Group by
↓
Tumbling
Hopping
Sliding
Session
```

---

## KQL

```text
bin()
↓
Tumbling-style time bucketing
```

```text
row_window_session()
↓
Session
```

KQL does not have dedicated hopping/sliding functions in this comparison.

---

## Spark

```text
window(col, size)
↓
Tumbling
```

```text
window(col, size, slide)
↓
Hopping
```

```text
session_window(col, gap)
↓
Session
```

---

## Spark Watermark

```python
.withWatermark("timestamp", "10 minutes")
```

Think:

> **Watermark = How long should streaming state be retained to tolerate late data?**

---

# 46. Ultimate Memory Trick

> **Tumbling = Fixed, No Overlap**

> **Hopping = Fixed, Overlap**

> **Sliding = Moving**

> **Session = Activity**

> **Snapshot = Same Timestamp**

Then:

> **Eventstream = Group by**

> **KQL = `bin()` / `row_window_session()`**

> **Spark = `window()` / `session_window()`**

And:

> **Spark Window Aggregation → `withWatermark()`**

Finally:

> **Engine First → Window Type → Syntax**

For Spark:

> **Watermark = State Retention + Late-Data Tolerance**

Not:

> **Watermark = Reject Late Events**
