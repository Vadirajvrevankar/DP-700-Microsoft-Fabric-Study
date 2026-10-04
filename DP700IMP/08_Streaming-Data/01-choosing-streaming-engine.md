# DP-700: Eventstream vs Spark Structured Streaming vs Eventhouse

## 1. Big Picture

Think of a streaming pipeline as:

**Ingest → Transform → Store / Query**

- **Eventstream** → low-code/no-code ingestion, simple transformation, filtering and routing
- **Spark Structured Streaming** → code-first complex streaming transformations
- **Eventhouse + KQL** → very fast interactive querying of event/time-series data

These engines are **not mutually exclusive**. A pipeline can use two or all three.

---

# 2. Eventstream — Default Starting Point

Eventstream is a **low-code/no-code streaming ingestion and transformation tool**.

It can:

- Connect to streaming sources
- Ingest events
- Filter data
- Perform straightforward transformations
- Route data
- Send data to destinations

Think:

> **Eventstream = streaming front door + routing hub**

### Exam signals

If the question says:

- No-code
- Drag-and-drop
- Minimal engineering effort
- Simple filtering
- Simple routing
- Easy streaming ingestion

Think:

> **Eventstream**

### Example

```text
IoT Devices
     ↓
Eventstream
     ↓
Filter invalid events
     ↓
Lakehouse / Eventhouse / other destination
```

### Important

Eventstream is a **transform-and-route hub**, not primarily a query engine.

Memory trick:

> **Eventstream = Move + Transform + Route**

---

# 3. Spark Structured Streaming

Spark Structured Streaming is a **code-first streaming engine**.

Use it when built-in low-code operators are not enough and the transformation requires custom programming.

## Strong Spark scenarios

### Custom joins

```text
Streaming Sales
      +
Customer data
      +
Product data
      +
Business rules
      ↓
Complex transformation
```

### UDFs

UDF = **User Defined Function**.

Example:

```python
def calculate_risk(temperature, pressure):
    ...
```

Custom functions are a strong signal toward Spark.

### ML scoring

```text
Streaming data
      ↓
Spark
      ↓
ML model
      ↓
Prediction
```

### Exam signals

- Custom code → **Spark Structured Streaming**
- UDF → **Spark Structured Streaming**
- Complex transformation → **Spark Structured Streaming**
- Complex multi-source join → **Spark Structured Streaming**
- ML model/scoring → **Spark Structured Streaming**

Memory trick:

> **Spark = CODE**

---

# 4. Eventstream vs Spark

| Requirement | Better choice |
|---|---|
| No-code | **Eventstream** |
| Drag-and-drop | **Eventstream** |
| Simple filtering | **Eventstream** |
| Simple routing | **Eventstream** |
| Custom code | **Spark Structured Streaming** |
| UDF | **Spark Structured Streaming** |
| Complex transformation | **Spark Structured Streaming** |
| Complex multi-source join | **Spark Structured Streaming** |
| ML scoring | **Spark Structured Streaming** |

---

# 5. Eventhouse

Eventhouse becomes important when the main requirement is the **query side**.

It is especially useful for:

- IoT telemetry
- Application logs
- Security events
- Sensor data
- Time-series/event data
- High-cardinality event data
- Very low/sub-second interactive analytics

Think:

> **Eventhouse = fast query/analytics destination**

---

# 6. High Cardinality

Cardinality means the number of distinct values.

### Low cardinality

```text
Gender:
Male
Female
```

### High cardinality

```text
DeviceID:
D000001
D000002
D000003
...
D999999
```

Telemetry systems can contain many unique:

- Device IDs
- User IDs
- Transaction IDs
- IP addresses
- Event IDs

---

# 7. KQL and Eventhouse

Eventhouse uses **Kusto Query Language (KQL)** for querying and analyzing event/time-series data.

Example:

```kusto
Telemetry
| where Timestamp > ago(10m)
| summarize AvgTemperature = avg(Temperature)
    by DeviceID
```

This means:

> For the last 10 minutes, calculate the average temperature for each device.

### Exam signals

- KQL → **Eventhouse**
- Telemetry investigation → **Eventhouse**
- Time-series analytics → **Eventhouse**
- Very low/sub-second query latency → **Eventhouse**

Memory trick:

> **Eventhouse = QUERY**

---

# 8. Eventstream vs Spark vs Eventhouse

| Requirement | Eventstream | Spark Structured Streaming | Eventhouse |
|---|---|---|---|
| No-code | ⭐⭐⭐ | ❌ | ❌ |
| Drag-and-drop | ⭐⭐⭐ | ❌ | ❌ |
| Simple transformation | ⭐⭐⭐ | ⭐⭐ | Not primary purpose |
| Complex code | ❌ | ⭐⭐⭐ | ❌ |
| UDF | ❌ | ⭐⭐⭐ | ❌ |
| ML scoring | ❌ | ⭐⭐⭐ | Not primary purpose |
| Streaming ingestion | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Complex joins | Limited by operators | ⭐⭐⭐ | Not primary purpose |
| Fast interactive queries | Not primary purpose | Not primary purpose | ⭐⭐⭐ |
| KQL | ❌ | ❌ | ⭐⭐⭐ |
| Time-series analytics | Routing/transform | Processing | ⭐⭐⭐ |
| Very low/sub-second query requirement | Not primary choice | Not primary choice | ⭐⭐⭐ |

---

# 9. The Three Engines Can Work Together

They are **not mutually exclusive**.

A common architecture can be:

```text
Streaming Sources
       ↓
   Eventstream
       ↓
      Spark
       ↓
   Eventhouse
       ↓
      KQL
       ↓
Dashboard / Investigation
```

Each engine has a different responsibility.

---

# 10. Eventstream → Spark → Eventhouse Example

Imagine a manufacturing company with thousands of machines.

Each machine sends:

```text
MachineID
Temperature
Pressure
Vibration
Timestamp
```

## Stage 1 — Eventstream

Eventstream receives the streaming events.

It can perform straightforward:

- Filtering
- Routing
- Basic transformations

```text
Machines
   ↓
Eventstream
```

## Stage 2 — Spark

Suppose the business now needs:

```text
Temperature
+
Pressure
+
Vibration
      ↓
Complex business logic
      ↓
ML model
      ↓
Failure prediction
```

Spark is a better fit.

```text
Eventstream
     ↓
Spark Structured Streaming
```

## Stage 3 — Eventhouse

Users need to investigate the results quickly.

```text
Spark
  ↓
Eventhouse
  ↓
KQL
```

Example:

```kusto
MachineEvents
| where Timestamp > ago(1h)
| summarize avg(Temperature) by MachineID
```

This illustrates:

> **Design pipelines as a chain of engines rather than forcing one engine to do everything.**

---

# 11. Operational Overhead vs Compute Cost

Do not consider only raw compute cost.

Suppose the requirement is simply:

```text
Remove records where Temperature is NULL
```

If Eventstream can do this easily, using a large code-based Spark solution may introduce unnecessary engineering effort.

The real cost is closer to:

> **Compute cost + Engineering/operational effort**

### Eventstream

```text
Source → Filter → Destination
```

Usually requires less engineering effort.

### Spark

```text
Notebook
↓
Spark code
↓
Libraries
↓
Runtime considerations
↓
Testing
↓
Deployment
↓
Monitoring
```

Therefore:

> The engine with the lowest raw compute cost is not necessarily the lowest overall cost.

---

# 12. Revisit the Engine Choice as Requirements Grow

Engine selection is not permanent.

A pipeline may start as:

```text
Eventstream
↓
Filter
↓
Lakehouse
```

Later, requirements may grow:

- Complex joins
- Custom calculations
- ML scoring
- Complicated business rules

Then the architecture may evolve:

```text
Eventstream
       ↓
Spark
       ↓
Lakehouse / Eventhouse
```

Important lesson:

> **A workload that starts as simple Eventstream filtering can outgrow no-code operators as business logic grows.**

---

# 13. DP-700 Exam Tips

| Wording in question | Think |
|---|---|
| "No-code" | **Eventstream** |
| "Drag-and-drop" | **Eventstream** |
| "Minimal engineering effort" | **Eventstream** |
| "Simple filtering" | **Eventstream** |
| "Custom code" | **Spark Structured Streaming** |
| "UDF" | **Spark Structured Streaming** |
| "ML model" | **Spark Structured Streaming** |
| "Complex multi-source join" | **Spark Structured Streaming** |
| "Sub-second query latency" | **Eventhouse** |
| "KQL" | **Eventhouse** |
| "Telemetry investigation" | **Eventhouse** |
| "Time-series analytics" | **Eventhouse** |

---

# 14. Exam Trap — Multiple Engines Can Be Correct

Do not assume every streaming question has only one engine.

### Scenario 1

> Ingest streaming data, perform simple filtering, and route it to multiple destinations without writing code.

**Answer: Eventstream**

### Scenario 2

> Ingest streaming data and apply a custom ML model before storing the results.

**Answer: Spark Structured Streaming**

### Scenario 3

> Analysts need to query millions of telemetry events with very low latency using KQL.

**Answer: Eventhouse**

### Scenario 4

> Ingest events, perform simple filtering, apply complex custom processing, and provide fast telemetry analysis.

Possible architecture:

```text
Eventstream → Spark → Eventhouse
```

---

# 15. Windowing

Windowing means grouping or processing streaming events over time-based windows.

The relevant window types are:

- Tumbling
- Hopping
- Sliding
- Session

These concepts are available across the streaming surfaces, with different syntax and implementation.

## Tumbling Window

Non-overlapping windows:

```text
10:00 ───── 10:05
10:05 ───── 10:10
10:10 ───── 10:15
```

Each event belongs to one window.

## Hopping Window

Overlapping windows that move by a fixed hop:

```text
10:00 ───── 10:10
       10:05 ───── 10:15
              10:10 ───── 10:20
```

## Sliding Window

A continuously moving time range.

Think:

> "Look at the last 5 minutes continuously."

## Session Window

Groups events based on periods of activity separated by inactivity.

```text
User activity
● ● ● ●       ● ● ●
|-----------| |-----|
  Session 1   Session 2
```

Important:

> Windowing is not exclusive to one engine. The syntax and implementation differ between surfaces.

---

# 16. DP-700 Decision Tree

### Question 1

Is the requirement mainly **no-code/simple ingestion and transformation**?

→ **Eventstream**

### Question 2

Does the transformation require **custom programming**?

→ **Spark Structured Streaming**

### Question 3

Is the main requirement **extremely fast interactive querying of telemetry/time-series data**?

→ **Eventhouse + KQL**

### Question 4

Are multiple stages described?

Don't force yourself to select only one.

You may have:

```text
Eventstream
     ↓
Spark
     ↓
Eventhouse
```

---

# 17. Ultimate Memory Trick

## Eventstream = ROUTE

**Ingest → Filter → Transform → Route**

## Spark = CODE

**Complex logic → Joins → UDFs → ML**

## Eventhouse = QUERY

**KQL → Telemetry → Time-series → Very low-latency analytics**

### One-line exam memory

> **Eventstream moves and transforms. Spark performs complex code-based processing. Eventhouse makes streaming/event data extremely fast to query.**

### Biggest lesson

> **Don't force one engine to do everything. Choose the engine according to the stage and requirement.**
