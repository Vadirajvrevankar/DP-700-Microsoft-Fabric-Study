# DP-700 Microsoft Fabric — Monitoring

## Quick Exam Sheet

### Monitor Hub
- Covers **17 item types**.
- ❌ **Dataflow Gen1 is not included** — common distractor.
- Best for quick operational status.
- Limited history: **30 days / 100 rows**.

### Pipeline Monitoring
- **Rerun from failed activity** = resume the pipeline from the failed activity rather than rerunning from the beginning.
- Do not confuse this pipeline-specific capability with generic retry behavior.

### Dataflow Gen2
- Detailed log download window: **28 days**.
- Refresh history UI: **50 rows**.
- Refresh history in OneLake: **250 rows / 6 months**.

### Spark Monitoring
Recognize:
- **Livy Log**
- **Driver Log**
- **Executor Log**
- Spark open-source metrics APIs

Use Monitor Hub/Recent Runs for status, application details for deep diagnosis, and APIs for programmatic monitoring.

### Eventstream Monitoring
Two tabs:
- **Data insights**
- **Runtime logs**

Runtime log severities:
- Warning
- Error
- Information

### Eventhouse Monitoring
Five monitoring table families:
1. Metrics
2. Command logs
3. Data operation logs
4. Ingestion results logs
5. Query logs

Memory: **M-C-D-I-Q**.

### Workspace Monitoring
- Provides a more durable/queryable monitoring approach than the limited Monitor Hub history.
- Monitoring data can be queried through an Eventhouse/KQL-based environment.

### Capacity Metrics App
- **Admin-installed**.
- Capacity-wide focus.
- CU utilization and throttling.
- ❌ **Does not support alerts**.

---

# 1. Monitor Hub

Monitor Hub is a centralized place in Microsoft Fabric to monitor the status of supported Fabric items.

Think:

> **Monitor Hub = one screen to see what is running, completed, failed, or in progress.**

## Important exam fact

Monitor Hub covers **17 item types**.

But:

> ❌ **Dataflow Gen1 is NOT included.**

### Memory trick

**Monitor Hub = 17 types − Dataflow Gen1**

## History limitation

Monitor Hub is not a permanent historical log. Its stated limits are approximately:
- **30 days**
- **100 rows**

Therefore, use Workspace Monitoring when you need more durable/queryable monitoring history.

| Requirement | Surface |
|---|---|
| Quickly check current/recent status | Monitor Hub |
| Long-term/queryable monitoring | Workspace Monitoring |
| Query monitoring data with KQL | Workspace Monitoring |
| Capacity utilization | Capacity Metrics app |

Memory:

> **Monitor Hub = Quick status**  
> **Workspace Monitoring = Queryable history**

---

# 2. Pipeline Monitoring

If a pipeline fails:

```text
Activity 1 ✅
Activity 2 ✅
Activity 3 ❌
Activity 4 ⏸️
```

You can use **Rerun from failed activity** to resume from the failed activity.

```text
Activity 1 ✅
Activity 2 ✅
Activity 3 ❌
             ↓
       Rerun from here
             ↓
Activity 3 🔄
Activity 4 🔄
```

## Retry vs Rerun from failed activity

### Retry
Try the failed operation again according to retry configuration.

### Rerun from failed activity
Resume the pipeline execution starting at the failed activity.

### Exam signal

> "Resume a pipeline execution from the activity that failed."

👉 **Rerun from failed activity**

---

# 3. Dataflow Gen2 Monitoring

## Detailed logs

Detailed logs have a:

> **28-day download window**

Memory:

**Detailed logs = 28 days**

## Refresh history

| Location | Limit |
|---|---:|
| UI | **50 rows** |
| OneLake | **250 rows / 6 months** |

Do not confuse the 28-day detailed-log window with the refresh-history row limits.

---

# 4. Spark Monitoring

Spark monitoring can be understood in layers:

```text
                Spark Monitoring
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
    High-level status          Deep diagnosis
          │                         │
   Monitor Hub                 Application details
   Recent Runs                       │
                                      ↓
                              Driver / Executor
                                  logs
                                      │
                                      ↓
                              REST/API access
```

## High-level status

Use Monitor Hub / Recent Runs to determine whether a Spark job ran successfully and to inspect execution status.

## Deep diagnosis

Recognize these monitoring/log names:

### Livy Log
Associated with Spark session/job interaction.

### Driver Log
The Spark driver coordinates the Spark application.

```text
Spark Application
       ↓
     Driver
       ↓
Coordinates work
```

### Executor Log
Executors perform distributed work.

```text
Driver
  │
  ├── Executor 1
  ├── Executor 2
  └── Executor 3
```

### Spark open-source metrics APIs
Useful for programmatic/technical monitoring.

### Exam recognition

**Livy + Driver + Executor + metrics APIs → Spark monitoring/diagnostics**

---

# 5. Eventstream Monitoring

Eventstream has two important monitoring tabs:

## Data insights

Used to understand information about the event/data flow.

Think:

> **"What is happening to my data?"**

## Runtime logs

Used for operational/runtime information.

Three severity levels:
- **Warning**
- **Error**
- **Information**

```text
Runtime logs
     │
     ├── Information
     ├── Warning
     └── Error
```

### Memory trick

**Eventstream = Data + Runtime**

---

# 6. Eventhouse Monitoring

Eventhouse monitoring has five monitoring table families.

## 1. Metrics
Performance/resource-related measurements.

> "How is the Eventhouse performing?"

## 2. Command logs
Commands executed against Eventhouse.

> "What commands were executed?"

## 3. Data operation logs
Information about data-related operations.

> "What data operations happened?"

## 4. Ingestion results logs
Useful for troubleshooting ingestion.

> "Did my data successfully get ingested?"

```text
Source
  ↓
Ingestion
  ↓
Success / Failure
```

## 5. Query logs
Information about queries executed against Eventhouse.

> "What queries were executed?"

### Memory trick

**M-C-D-I-Q**

> Metrics → Commands → Data → Ingestion → Queries

---

# 7. Workspace Monitoring

Workspace Monitoring provides a centralized monitoring environment for supported Fabric monitoring data and is useful when you need more than the limited Monitor Hub history.

Typical use cases:
- Query monitoring history.
- Investigate recurring failures.
- Analyze monitoring information over longer periods.
- Use KQL against monitoring data.

Conceptually:

```text
Fabric Items
   │
   ├── Pipelines
   ├── Spark
   ├── Eventstream
   ├── Eventhouse
   └── Other supported items
          │
          ↓
 Workspace Monitoring
          │
          ↓
      Eventhouse
          │
          ↓
         KQL
```

### Key distinction

**Monitor Hub** → operational overview / quick status.

**Workspace Monitoring** → queryable monitoring data/history.

---

# 8. Capacity Metrics App

The Capacity Metrics app is focused on the Fabric **capacity**, not individual job execution.

```text
Fabric Capacity
      ↓
Capacity Metrics
      ↓
CU utilization
Throttling
```

## Installation

> **Admin-installed**

## Focus

### CU utilization
CU means **Capacity Units** and represents capacity consumption.

### Throttling
Shows capacity pressure/regulation of workload execution.

Therefore:

> **Capacity Metrics = capacity health**

not:

> **Pipeline execution monitoring**

## Alerts

> ❌ **The Capacity Metrics app does not support alerts.**

### Exam trap

Remember all four:
- Admin-installed
- Capacity-wide
- CU/throttling focused
- No alerts

---

# 9. Putting Everything Together

```text
                    Microsoft Fabric
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ↓                 ↓                 ↓
   Pipelines           Spark           Eventstream
        │                 │                 │
        ↓                 ↓                 ↓
 Pipeline Monitor    Spark Monitor    Eventstream Monitor
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                    Monitor Hub
                          │
                    Quick status
                          │
                          ↓
                 Workspace Monitoring
                          │
                          ↓
                      Eventhouse
                          │
                          ↓
                         KQL
```

Separately:

```text
             Fabric Capacity
                   │
                   ↓
          Capacity Metrics App
                   │
             ┌─────┴─────┐
             ↓           ↓
        CU utilization  Throttling
```

---

# 10. What to Use for What?

| Scenario | Answer |
|---|---|
| Quickly check status of Fabric items | **Monitor Hub** |
| Tenant-wide monitoring overview | **Monitor Hub** |
| Monitor Hub supported item count | **17** |
| Monitor Hub exclusion | **Dataflow Gen1** |
| Resume pipeline from failure | **Rerun from failed activity** |
| Dataflow Gen2 detailed log download window | **28 days** |
| Dataflow Gen2 refresh history in UI | **50 rows** |
| Dataflow Gen2 history in OneLake | **250 rows / 6 months** |
| Deep Spark diagnosis | **Application detail/logs** |
| Spark log names | **Livy, Driver, Executor** |
| Eventstream monitoring tabs | **Data insights, Runtime logs** |
| Eventstream runtime severity | **Warning, Error, Information** |
| Eventhouse monitoring | **Five table families** |
| Long-term/queryable monitoring | **Workspace Monitoring** |
| Query monitoring information | **KQL / Eventhouse** |
| Capacity utilization | **Capacity Metrics app** |
| Capacity throttling | **Capacity Metrics app** |
| Capacity Metrics installation | **Admin** |
| Capacity Metrics alerts | ❌ **Not supported** |

---

# 11. DP-700 Exam Memory Sheet

## Monitor Hub

> **17 item types**

> ❌ **Dataflow Gen1**

> **30 days / 100 rows**

> Quick operational monitoring.

## Pipeline

> **Rerun from failed activity**

Resume from the failed activity rather than rerunning the whole pipeline.

## Dataflow Gen2

> **Detailed logs = 28 days**

> **Refresh UI = 50 rows**

> **OneLake = 250 rows / 6 months**

## Spark

> **Livy → Driver → Executor**

Plus:

> **Spark open-source metrics APIs**

## Eventstream

Two tabs:

> **Data insights**

> **Runtime logs**

Runtime logs:

> **Warning / Error / Information**

## Eventhouse

Five monitoring families:

> **Metrics**

> **Command logs**

> **Data operation logs**

> **Ingestion results logs**

> **Query logs**

Memory:

**M-C-D-I-Q**

## Capacity Metrics

> **Admin-installed**

> **Capacity-wide**

> **CU utilization**

> **Throttling**

> ❌ **No alerts**

---

# 12. The Big Picture Memory Trick

```text
MONITOR HUB
     ↓
"WHAT IS RUNNING / FAILED?"
     ↓
Quick status
     ↓
17 types
     ↓
❌ Dataflow Gen1


PIPELINE
     ↓
"WHERE DID IT FAIL?"
     ↓
Rerun from failed activity


SPARK
     ↓
"WHY DID IT FAIL?"
     ↓
Livy / Driver / Executor logs


EVENTSTREAM
     ↓
"WHAT IS HAPPENING TO STREAM?"
     ↓
Data insights / Runtime logs


EVENTHOUSE
     ↓
"WHAT IS HAPPENING INSIDE EVENTHOUSE?"
     ↓
Metrics / Commands / Data / Ingestion / Queries


WORKSPACE MONITORING
     ↓
"I WANT QUERYABLE MONITORING HISTORY"
     ↓
Eventhouse + KQL


CAPACITY METRICS
     ↓
"IS MY CAPACITY HEALTHY?"
     ↓
CU + Throttling
     ↓
❌ Alerts
```

---

# ⭐ Final DP-700 Exam Signals

- **17 item types** → Monitor Hub
- **Dataflow Gen1 excluded** → Monitor Hub
- **Resume at failed activity** → Pipeline
- **28 days** → Dataflow Gen2 detailed-log download
- **50 rows** → Dataflow Gen2 refresh history UI
- **250 / 6 months** → Dataflow Gen2 OneLake history
- **Livy / Driver / Executor** → Spark monitoring
- **Data insights / Runtime logs** → Eventstream
- **Warning / Error / Information** → Eventstream runtime severity
- **M-C-D-I-Q** → Eventhouse monitoring
- **Queryable historical monitoring + KQL** → Workspace Monitoring
- **CU + throttling** → Capacity Metrics
- **Admin-installed + no alerts** → Capacity Metrics app
