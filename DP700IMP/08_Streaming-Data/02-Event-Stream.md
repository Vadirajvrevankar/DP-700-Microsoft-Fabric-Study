# DP-700 / Microsoft Fabric Eventstream — CDC, Routing, Processing & Real-Time Hub

## 1. Why This Topic Matters

These concepts are useful both for the DP-700 exam and for real-world Data Engineering.

They help you understand:

- CDC and change capture
- Streaming ingestion
- Content-based routing
- No-code event transformations
- Eventhouse ingestion patterns
- At-least-once delivery
- Deduplication and idempotency
- Real-Time hub and reusable real-time assets

---

# 2. CDC Connector + DeltaFlow

## What is CDC?

**CDC = Change Data Capture.**

CDC captures changes made to a database rather than repeatedly reading the entire table.

Typical changes are:

- INSERT
- UPDATE
- DELETE

Example:

```text
Database
   ↓
CDC
   ↓
Only changed records
```

## DeltaFlow

When the source database is supported by a CDC connector, prefer the CDC connector with **DeltaFlow (preview)** rather than manually parsing CDC JSON.

Conceptually:

```text
Database
   ↓
CDC Connector
   ↓
DeltaFlow
   ↓
Eventstream
   ↓
Destination
```

### Exam signal

If the question says:

> The source database is supported by a CDC connector.

Think:

**CDC connector + DeltaFlow**

### Memory trick

> **Supported database → CDC connector → DeltaFlow**

---

# 3. Derived Streams

A **derived stream** lets you create additional streams from an existing Eventstream.

It is especially useful for **content-based routing**.

Example:

```text
                 ┌→ Bengaluru stream → Destination A
Main Eventstream ├→ Mysore stream    → Destination B
                 └→ Hubballi stream  → Destination C
```

The routing decision can be based on event content.

For example:

```text
IF City = 'Bengaluru'
        ↓
Bengaluru Derived Stream
```

## Why use derived streams?

Instead of creating multiple independent Eventstreams:

```text
Eventstream 1 → Bengaluru
Eventstream 2 → Mysore
Eventstream 3 → Hubballi
```

you can use one Eventstream and create derived streams.

### Exam signal

> Route events to different destinations based on event content.

**Answer: Derived streams**

### Memory trick

> **Derived stream = one stream → multiple routes**

---

# 4. Event Processing Before Ingestion

This is especially important when the destination is **Eventhouse**.

Use **event processing before ingestion** when events need to be filtered, aggregated, or otherwise processed before they enter Eventhouse.

Example:

```text
Source
  ↓
Eventstream
  ↓
Filter / Aggregate / Group by
  ↓
Eventhouse
```

Example requirement:

> Filter abnormal sensor values before storing them in Eventhouse.

Use:

**Event processing before ingestion**

---

# 5. Direct Ingestion

Use **Direct ingestion** when you want a raw or archival copy.

Architecture:

```text
Source
   ↓
Eventstream
   ↓
Eventhouse
```

No pre-ingestion transformation is required.

### Exam signal

> Store raw events for archival.

Think:

**Direct ingestion**

### Compare

| Requirement | Choice |
|---|---|
| Filter/aggregate before Eventhouse | Event processing before ingestion |
| Store raw/archival copy | Direct ingestion |

---

# 6. At-Least-Once Delivery

Eventstream delivery is **at-least-once**.

This means an event can be delivered one or more times.

It does **not** mean exactly-once delivery.

Example:

```text
Event #100
     ↓
Destination
```

Normally:

```text
Event #100
```

But retry behavior can result in:

```text
Event #100
Event #100
```

Therefore, downstream systems should be designed to handle duplicates.

## Important exam point

Do not assume:

> Eventstream automatically guarantees exactly-once delivery.

Remember:

> **Eventstream = At-least-once**

---

# 7. Deduplication and Idempotency

Because delivery is at-least-once, duplicates can occur.

Example:

```text
EventID | Customer | Amount
1001    | Ravi     | 500
1001    | Ravi     | 500
```

The destination layer should be designed to handle duplicates.

A common design is to use a unique event identifier:

```text
EventID
```

and make the destination processing idempotent.

## Idempotent

**Idempotent = safe to repeat.**

If the same event is processed again, it should not create an unintended duplicate or incorrect result.

### Memory trick

> **At-least-once → design for duplicates → use deduplication/idempotent writes**

---

# 8. Seven No-Code Event Processor Operators

The seven no-code operators are:

1. Filter
2. Manage fields
3. Aggregate
4. Group by
5. Union
6. Expand
7. Join

---

## 8.1 Filter

Keeps only events that satisfy a condition.

Example:

```text
Temperature > 30
```

```text
All Events
    ↓
  Filter
    ↓
High-temperature Events
```

### Memory

> **Filter = keep what I want**

---

## 8.2 Manage Fields

Used to work with fields/columns.

Example:

```text
FirstName
LastName
Age
City
```

You may need to select, rename, or modify fields.

### Memory

> **Manage fields = work with columns**

---

## 8.3 Aggregate

Performs calculations such as:

- COUNT
- SUM
- AVG

Example:

```text
Orders
   ↓
Aggregate
   ↓
Total Orders
```

### Memory

> **Aggregate = calculate**

---

## 8.4 Group By

Groups events based on a field.

Example:

```text
Device | Temperature
A      | 25
A      | 27
B      | 30
B      | 31
```

Group by:

```text
Device
```

produces groups for A and B.

### Important exam point

Your notes specifically identify **Group by as the windowed operator**.

### Memory

> **Group by = group events, especially over windows**

---

## 8.5 Union

Combines streams.

```text
Stream A
    ↓
   Union
    ↑
Stream B
```

Result:

```text
Stream A + Stream B
```

### Memory

> **Union = combine streams**

---

## 8.6 Expand

Used to expand complex or nested data into more usable data.

Conceptually:

```text
Complex / nested data
        ↓
      Expand
        ↓
Expanded data
```

### Memory

> **Expand = open up complex data**

---

## 8.7 Join

Combines related streams based on matching information.

Example:

```text
Stream A
CustomerID | Order
101        | 500

Stream B
CustomerID | City
101        | Bengaluru
```

Join on:

```text
CustomerID
```

Result:

```text
CustomerID | Order | City
101        | 500   | Bengaluru
```

### Memory

> **Join = combine related data**

---

# 9. SQL Operator

A **SQL operator (preview)** is available for code-oriented logic within Eventstream.

Think:

```text
Standard no-code transformations
        ↓
7 event processor operators
```

For SQL-based logic:

```text
SQL operator
```

### Exam signal

> Use SQL logic inside Eventstream.

Think:

**SQL operator**

---

# 10. Pre-Ingestion Operator Support

Only the following support a pre-ingestion operator directly:

1. **Lakehouse**
2. **Eventhouse — event processing before ingestion**
3. **Derived stream**
4. **Activator**

Other destinations require a **derived stream as an intermediate hop**.

Conceptually:

```text
Eventstream
     ↓
Derived Stream
     ↓
Transformation
     ↓
Destination
```

### Exam memory

> **Lakehouse + Eventhouse + Derived Stream + Activator = direct pre-ingestion support**

For other destinations:

> **Use Derived Stream as the bridge.**

---

# 11. Six Destinations

Eventstream has **six destinations**.

The important architecture concept is:

```text
SOURCE
   ↓
EVENTSTREAM
   ↓
PROCESS / ROUTE
   ↓
DESTINATION
```

The delivery guarantee remains:

> **At-least-once**

Therefore, destinations should be designed to handle possible duplicate events.

---

# 12. Real-Time Hub

**Real-Time hub** is a central place for discovering real-time data assets.

Before creating a new Eventstream, check whether the required source or a similar transformed stream already exists.

Conceptually:

```text
Existing real-time assets
          ↓
     Real-Time hub
```

According to the notes:

- Every Eventstream output automatically appears in Real-Time hub.
- Accessible KQL tables automatically appear in Real-Time hub.
- No manual registration is required.

### Exam signal

> Check whether an existing real-time stream can be reused.

Think:

**Real-Time hub**

---

# 13. Complete Architecture Example

Imagine an IoT system.

```text
IoT Devices
     │
     ▼
 Eventstream
     │
     ├── Filter
     │
     ├── Manage fields
     │
     ├── Aggregate
     │
     ├── Group by
     │
     ├── Join
     │
     └── Derived streams
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
   Lakehouse Eventhouse Activator
```

Delivery:

```text
Eventstream
     ↓
AT-LEAST-ONCE
```

Therefore:

```text
Destination
     ↓
Deduplication / Idempotent handling
```

Discoverability:

```text
Eventstream outputs
KQL tables
Derived streams
      ↓
Real-Time hub
```

---

# 14. DP-700 Decision Tree

## Step 1 — Supported database CDC source?

```text
YES
 ↓
CDC connector + DeltaFlow
```

Avoid unnecessary hand-written JSON parsing.

---

## Step 2 — Content-based routing?

```text
YES
 ↓
Derived stream
```

---

## Step 3 — Processing before Eventhouse ingestion?

```text
YES
 ↓
Event processing before ingestion
```

---

## Step 4 — Need a raw/archival Eventhouse copy?

```text
YES
 ↓
Direct ingestion
```

---

## Step 5 — Concern about duplicates?

```text
Eventstream
    ↓
At-least-once
    ↓
Destination handles duplicates
```

---

## Step 6 — Does an existing real-time asset possibly exist?

```text
YES
 ↓
Check Real-Time hub
```

---

# 15. Exam Memory Table

| Requirement | Think |
|---|---|
| Database change capture | **CDC** |
| Supported CDC database | **CDC connector + DeltaFlow** |
| Route based on event content | **Derived stream** |
| Filter before Eventhouse ingestion | **Event processing before ingestion** |
| Raw Eventhouse copy | **Direct ingestion** |
| No-code transformation | **Eventstream operators** |
| Calculate count/sum/average | **Aggregate** |
| Time-window grouping | **Group by** |
| Combine streams | **Union** |
| Combine related data | **Join** |
| Complex nested data | **Expand** |
| Work with columns | **Manage fields** |
| Keep matching events | **Filter** |
| Duplicate delivery | **At-least-once** |
| Existing real-time asset | **Real-Time hub** |
| SQL logic inside Eventstream | **SQL operator** |

---

# 16. Ultimate Memory Tricks

### CDC

> **Supported database → CDC connector → DeltaFlow**

### Routing

> **Content-based routing → Derived stream**

### Eventhouse

> **Process first → Event processing before ingestion**

> **Raw copy → Direct ingestion**

### Delivery

> **Eventstream → At-least-once**

### Reliability

> **At-least-once → Handle duplicates**

### Discovery

> **Existing stream → Check Real-Time hub**

### Operators

> **Filter → Fields → Aggregate → Group → Union → Expand → Join**

---

# 17. Real-World Knowledge vs Exam Knowledge

These concepts are useful beyond DP-700.

### Exam knowledge

You need to recognize exact Fabric terminology:

```text
CDC + DeltaFlow
Derived stream
Event processing before ingestion
Direct ingestion
At-least-once
Real-Time hub
```

### Real-world Data Engineering knowledge

You should understand why these patterns exist:

```text
CDC
 ↓
Streaming ingestion
 ↓
Transformation
 ↓
Routing
 ↓
Storage / Analytics
 ↓
Deduplication
```

These ideas transfer to technologies such as:

- Azure Data Engineering
- Apache Kafka
- Spark Structured Streaming
- Event Hubs
- Databricks
- Eventhouse / KQL

---

# 18. One-Page Revision

```text
                 DATABASE
                    │
                    ▼
             CDC CONNECTOR
                    │
                    ▼
                DeltaFlow
                    │
                    ▼
               EVENTSTREAM
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
       Filter    Aggregate   Join
          │
          ▼
    Derived Stream
          │
      ┌───┼────┐
      ▼   ▼    ▼
   Route Route Route
      │   │    │
      └───┼────┘
          ▼
      Destinations

Eventstream delivery:
        ↓
 AT-LEAST-ONCE
        ↓
Destination handles duplicates

Existing real-time assets:
        ↓
   REAL-TIME HUB
```

## Final Memory

> **CDC → DeltaFlow**
>
> **Route → Derived Stream**
>
> **Process before Eventhouse → Event Processing**
>
> **Raw copy → Direct Ingestion**
>
> **Duplicates → At-least-once**
>
> **Find existing assets → Real-Time hub**
>
> **Transform → 7 operators**
