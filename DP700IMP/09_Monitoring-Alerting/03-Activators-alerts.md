# DP-700 Microsoft Fabric — Activator Notes

## 1. What is Activator?

**Fabric Activator** is a no-code event-detection engine that watches data/events, evaluates conditions, and performs actions when rules match.

### Basic flow

```text
Events
   ↓
Objects
   ↓
Conditions
   ↓
Rules
   ↓
Actions
```

### Four core concepts

| Concept | Meaning |
|---|---|
| **Event** | Something happening or incoming data |
| **Object** | The individual thing being monitored |
| **Condition** | What Activator should detect |
| **Rule** | What should happen when the condition matches |

**Memory trick:**

> **Event → Object → Condition → Rule → Action**

---

# 2. Events

An event is information that Activator receives.

Examples:

- Pipeline failed
- Temperature = 95°C
- Package status = Delayed
- Sales decreased
- Capacity event occurred

## Alert/event sources

Activator alert authoring can be initiated from several Fabric experiences:

- Eventstreams
- KQL querysets
- Real-Time dashboards
- Power BI visuals
- Real-Time hub events
- Warehouse SQL query **(preview)**

### Real-Time hub events

Important event categories include:

- Job events
- Workspace item events
- OneLake events
- Capacity overview events

---

# 3. Objects

An **object** represents the individual thing being monitored.

Objects allow **per-instance evaluation**.

Example:

```text
Device A → Check condition
Device B → Check condition
Device C → Check condition
```

Another example:

```text
Package 1001 → Delayed?
Package 1002 → Delayed?
Package 1003 → Delayed?
```

### Exam signal

If the question says:

> Evaluate a condition independently for each device, package, pipeline, or other instance.

Think:

**OBJECTS**

---

# 4. Conditions

A condition defines what Activator should detect.

There are two important styles:

## Stateless condition

A raw comparison.

Example:

```text
Temperature > 90°C
```

It asks:

> Is the value currently above 90?

## Stateful condition

A stateful condition looks at a **transition/change** rather than only a raw value.

Important vocabulary:

- `BECOMES`
- `DECREASES`
- `INCREASES`
- `EXIT RANGE`
- Absence of data / heartbeat

### Memory trick

> **Stateless = What is true now?**  
> **Stateful = What changed?**

---

# 5. Why Stateful Conditions Matter

Suppose a machine continuously reports:

```text
91
92
93
94
95
```

A simple condition:

```text
Temperature > 90
```

could repeatedly satisfy the condition.

This can create alert spam:

```text
91 → Alert
92 → Alert
93 → Alert
94 → Alert
95 → Alert
```

A transition-based condition is better when the requirement is to alert when the value **becomes** abnormal.

Conceptually:

```text
89
 ↓
90
 ↓
92  ← BECOMES >90
 ↓
94
 ↓
96
```

The important event is the transition into the abnormal state.

### Exam signal

If the scenario mentions:

- sustained condition
- repeated firing
- alert spam
- only alert when the state changes

Think:

**STATEFUL CONDITION**

---

# 6. BECOMES

`BECOMES` focuses on a transition into a condition.

Example:

```text
Temperature <= 90
        ↓
Temperature > 90
        ↓
     ALERT
```

Memory:

> **BECOMES = Did it become true?**

---

# 7. DECREASES

Use `DECREASES` when the important event is a reduction.

Example:

```text
Sales:
₹10 lakh
₹9.5 lakh
₹9 lakh
₹8 lakh
```

The important change is:

```text
Sales DECREASES
```

This is different from simply asking:

```text
Sales < ₹10 lakh
```

---

# 8. INCREASES

Use `INCREASES` when the important event is an increase.

Example:

```text
CPU:
40%
45%
55%
70%
```

The important change is:

```text
CPU INCREASES
```

---

# 9. EXIT RANGE

Suppose an acceptable temperature range is:

```text
20°C — 80°C
```

The important transition is:

```text
Inside range
     ↓
Outside range
     ↓
   ALERT
```

Example:

```text
50 → 60 → 70 → 85
                  ↑
             EXIT RANGE
```

---

# 10. Absence of Data / Heartbeat

Sometimes the important condition is that data **stops arriving**.

Example:

```text
10:00 → Heartbeat
10:01 → Heartbeat
10:02 → Heartbeat
10:03 → Heartbeat
10:04 → No heartbeat
10:05 → No heartbeat
```

This can indicate:

- Device failure
- Connection problem
- Source stopped sending data

### Exam clue

> "Alert when a device stops sending events."

Think:

**Absence of data / heartbeat**

---

# 11. Rules

A rule connects a condition to one or more actions.

Example:

```text
Pipeline fails
      ↓
Condition
      ↓
Rule
      ↓
Send Teams message
```

A rule can have multiple related actions.

Example:

```text
Pipeline fails
      ↓
     Rule
    ↙    ↘
 Teams   Remediation
         Notebook
```

---

# 12. Multiple Actions on One Rule

If the same condition requires multiple responses, prefer combining the actions on one rule.

Example:

> When a pipeline fails:
>
> 1. Notify the team.
> 2. Run remediation.

Preferred concept:

```text
Condition
    ↓
 One Rule
   ↙   ↘
Teams  Remediation
       Pipeline/Notebook
```

Instead of creating separate rules for the same condition.

### Memory

> **One condition → one rule → multiple related actions**

---

# 13. Activator Actions

Common actions include:

### Notifications

- Email
- Microsoft Teams
- Power Automate

### Fabric actions

Depending on feature availability/preview status:

- Run a pipeline
- Run a notebook
- Run a Spark job definition
- Run a dataflow
- Run a user-defined function (UDF)
- Run a copy job

### Other

- Publish a business event **(preview)**

---

# 14. Pipeline Failure Alerting

There are two important patterns.

## Pattern 1 — Per-pipeline job events

Use Fabric job events for a specific pipeline.

Conceptually:

```text
Real-Time hub
     ↓
Job events
     ↓
Specific pipeline
     ↓
Activator
     ↓
Notification
```

Useful when you care about a particular pipeline.

---

# 15. Workspace-Wide Pipeline Failure Alerting

When the number of pipelines becomes large or keeps growing, creating one rule per pipeline can become difficult to maintain.

Instead:

```text
Workspace
   ↓
Workspace Monitoring
   ↓
ItemJobEventLogs
   ↓
KQL Queryset
   ↓
Activator
   ↓
One Rule
```

The key log table from the study material is:

```text
ItemJobEventLogs
```

Conceptually, the query looks for failed jobs:

```kusto
ItemJobEventLogs
| where Status == "Failed"
```

### When to prefer this pattern

Use workspace-wide monitoring when:

- There are many pipelines/items.
- The number of items is growing.
- You want one workspace-wide failure rule.
- Maintaining individual rules would become cumbersome.

### Memory

> **Small/specific scope → per-item alert**  
> **Large/growing scope → workspace monitoring + KQL**

---

# 16. Alert Authoring Is Decentralized

You can create alerts from the Fabric experience where the data already exists.

Examples:

```text
Eventstream
    ↓
Set Alert
```

```text
KQL queryset
    ↓
Set Alert
```

```text
Real-Time dashboard
    ↓
Set Alert
```

```text
Power BI visual
    ↓
Set Alert
```

```text
Real-Time hub
    ↓
Set Alert
```

These experiences ultimately use an **Activator item** for alerting.

### Exam takeaway

> **Alert creation is decentralized, but Activator is the underlying alert/action engine.**

---

# 17. Supported Alert Sources

Know these for the exam:

| Source | Notes |
|---|---|
| **Eventstream** | Streaming/event data |
| **KQL queryset** | Query-driven detection |
| **Real-Time dashboard** | Real-time metrics |
| **Power BI visual** | BI-based alerting |
| **Real-Time hub events** | Fabric events |
| **Warehouse SQL query** | Preview |

---

# 18. Unsupported Alert Sources

These are important exam distractors.

According to the study material:

### Capacity Metrics app ❌

Do not assume that the Capacity Metrics app itself is a direct Activator alert source.

For capacity-related events, think about:

```text
Real-Time hub
      ↓
Capacity overview events
      ↓
Activator
```

### SQL analytics endpoint ❌

The SQL analytics endpoint directly is also identified as an unsupported alert source.

### Exam trap

```text
Capacity Metrics app       ❌
SQL analytics endpoint     ❌
```

---

# 19. Estimated Firing Rate

Before activating a rule, check its estimated firing rate against historical data.

Why?

A rule can accidentally fire much more frequently than expected.

Example:

```text
Historical data
      ↓
Condition tested
      ↓
Estimated firing rate
      ↓
Is it reasonable?
      ↓
Activate rule
```

Suppose the rule would have fired:

```text
10,000 times/hour
```

That is a strong warning that the rule may create alert spam.

### Exam takeaway

> **Preview the estimated firing rate before activating a rule.**

---

# 20. Alert Spam

Alert spam happens when a condition fires repeatedly for a sustained condition.

Example:

```text
CPU > 80%

81 → Alert
82 → Alert
83 → Alert
84 → Alert
85 → Alert
```

Possible improvement:

```text
CPU BECOMES >80%
```

Now the focus is the transition into the state.

### Memory

> **Sustained condition + repeated firing → consider a stateful transition.**

---

# 21. Alert Throttling / Rate Limits

Activator has documented rate limits for areas such as:

- Email
- Teams
- Power Automate
- Fabric item activations

Therefore, an extremely large event spike can cause alerting to be throttled.

Example:

```text
Normal traffic
100 events/minute
        ↓
Huge spike
100,000 events/minute
        ↓
Many rule activations
        ↓
Rate limit / throttling
```

### Exam scenario

If the question says:

> Alerts stopped during a huge event spike.

Think:

**THROTTLING / RATE LIMIT**

Don't immediately assume Activator itself has failed.

### Important

Know that rate limits exist. Exact limits should be checked against current Microsoft documentation close to the exam if the exact numbers are tested.

---

# 22. Complete Pipeline Failure Example

### Requirement

> Alert when any pipeline in a workspace fails. Notify the team and start remediation.

Architecture:

```text
                Fabric Workspace
                       │
                       ▼
              Workspace Monitoring
                       │
                       ▼
                ItemJobEventLogs
                       │
                       ▼
                  KQL Queryset
                       │
                       ▼
                    Activator
                       │
                  ┌────┴────┐
                  ▼         ▼
              Teams     Remediation
                         Notebook
```

### Concepts

**Event:**

```text
Pipeline job failure
```

**Object:**

```text
Pipeline/job instance
```

**Condition:**

```text
Job becomes Failed
```

**Rule:**

```text
If failure detected
```

**Actions:**

```text
Teams notification
+
Remediation notebook/pipeline
```

---

# 23. Complete IoT Example

### Requirement

> Alert when a machine's temperature becomes greater than 90°C.

### Event

```text
Temperature reading
```

### Object

```text
Machine
```

### Condition

```text
Temperature BECOMES >90°C
```

### Rule

```text
If condition matches
```

### Action

```text
Email + Teams
```

Flow:

```text
Machine
   ↓
Temperature event
   ↓
Object = Machine
   ↓
BECOMES >90°C
   ↓
Activator Rule
   ↓
Email + Teams
```

---

# 24. Stateful vs Stateless — Exam Table

| Requirement | Think |
|---|---|
| Temperature > 90 | Stateless |
| Temperature **BECOMES** >90 | Stateful |
| Sales decreases | Stateful |
| Sales increases | Stateful |
| Value exits acceptable range | Stateful |
| Device stops sending data | Stateful / absence |
| Avoid repeated alerts | Stateful |
| Sustained condition | Stateful |
| Raw comparison | Stateless |

### Memory

> **Stateless = current value**  
> **Stateful = transition/change**

---

# 25. Pipeline Alerting — Exam Table

| Scenario | Best pattern |
|---|---|
| One specific pipeline | Per-pipeline job event |
| Few pipelines | Per-pipeline alert can be practical |
| Hundreds of pipelines | Workspace monitoring |
| Growing number of pipelines | Workspace-wide KQL rule |
| Workspace-wide failure detection | `ItemJobEventLogs` + KQL |
| Notification + remediation | One rule with multiple actions |

---

# 26. Best Practices

## 1. Prefer stateful conditions when repeated firing is a risk

Use:

```text
BECOMES
DECREASES
INCREASES
EXIT RANGE
```

when the requirement is transition-based.

---

## 2. Use workspace monitoring for large/growing environments

Instead of:

```text
Pipeline 1 → Rule
Pipeline 2 → Rule
Pipeline 3 → Rule
...
Pipeline 500 → Rule
```

prefer:

```text
Workspace Monitoring
       ↓
ItemJobEventLogs
       ↓
One KQL queryset
       ↓
One Activator rule
```

---

## 3. Combine related actions

Instead of:

```text
Rule 1 → Teams
Rule 2 → Remediation
```

prefer:

```text
One Rule
  ├── Teams
  └── Remediation
```

when both respond to the same condition.

---

## 4. Preview firing rate

Before activating:

```text
Rule
 ↓
Historical firing estimate
 ↓
Check for alert spam
 ↓
Activate
```

---

## 5. Re-check Activator features close to the exam

Activator is a fast-moving Fabric area.

Preview features, naming, supported sources, and availability can change.

Therefore:

> **Verify current Microsoft documentation close to exam day.**

---

# 27. DP-700 Exam Tips

### 🔥 Know these four concepts cold

> **Events → Objects → Conditions → Rules**

Objects are important because they allow **per-instance evaluation**.

---

### 🔥 Know stateful vocabulary

```text
BECOMES
DECREASES
INCREASES
EXIT RANGE
ABSENCE OF DATA / HEARTBEAT
```

---

### 🔥 Know alert sources

```text
Eventstream
KQL queryset
Real-Time dashboard
Power BI visual
Real-Time hub events
Warehouse SQL query (preview)
```

---

### 🔥 Know unsupported distractors

```text
Capacity Metrics app          ❌
SQL analytics endpoint       ❌
```

---

### 🔥 Know actions

```text
Email
Teams
Power Automate
Pipeline
Notebook
Spark job definition
Dataflow
UDF
Copy job
Business event (preview)
```

---

### 🔥 Pipeline failure alerting

Two valid patterns:

```text
Per-pipeline:
Real-Time hub → Job events → Activator
```

or:

```text
Workspace-wide:
Workspace monitoring
       ↓
ItemJobEventLogs
       ↓
KQL queryset
       ↓
Activator
```

---

### 🔥 Huge spike + alerts stopped

Think:

> **Rate limiting / throttling**

---

# 28. One-Page Memory Sheet

```text
                    ACTIVATOR
                        │
          ┌─────────────┴─────────────┐
          │                           │
        EVENTS                     OBJECTS
          │                           │
          └─────────────┬─────────────┘
                        ↓
                    CONDITION
                        │
             ┌──────────┴──────────┐
             │                     │
        STATELESS               STATEFUL
        > < =                   BECOMES
                                DECREASES
                                INCREASES
                                EXIT RANGE
                                ABSENCE
                        │
                        ↓
                      RULE
                        │
              ┌─────────┼─────────┐
              ↓         ↓         ↓
            EMAIL     TEAMS    FABRIC ACTION
                                  │
                         Pipeline / Notebook /
                         Dataflow / Spark /
                         Copy job / UDF
```

## Most important memory lines

> **Activator = detect → evaluate → act**

> **Event = something happened**

> **Object = what individual thing am I watching?**

> **Condition = what should I detect?**

> **Rule = what should happen?**

> **Stateless = current value**

> **Stateful = transition/change**

> **BECOMES = avoid repeated firing on a sustained state**

> **Large workspace = Workspace monitoring + KQL**

> **Pipeline failures = Job events or `ItemJobEventLogs`**

> **Same condition + multiple responses = one rule + multiple actions**

> **Huge spike + alerts stop = throttling**

> **Check firing rate before activating**

---

# 29. Final Exam Mental Model

When you see an Activator question, ask these questions in order:

```text
1. WHAT IS HAPPENING?
       ↓
     EVENT

2. WHAT INDIVIDUAL THING AM I WATCHING?
       ↓
     OBJECT

3. WHAT EXACTLY SHOULD I DETECT?
       ↓
   CONDITION

4. IS IT A RAW VALUE OR A TRANSITION?
       ↓
Stateless / Stateful

5. WHAT SHOULD HAPPEN?
       ↓
     RULE

6. WHAT ACTION(S)?
       ↓
Email / Teams / Pipeline / Notebook / etc.

7. COULD THIS FIRE MANY TIMES?
       ↓
Stateful condition / firing-rate check

8. IS THE ENVIRONMENT LARGE?
       ↓
Workspace monitoring + KQL

9. DID ALERTS STOP DURING A HUGE SPIKE?
       ↓
Throttling / rate limits
```

## Ultimate memory trick

> **WATCH → DETECT → DECIDE → ACT**
>
> **Activator watches EVENTS for OBJECTS, detects CONDITIONS through RULES, and performs ACTIONS.**
