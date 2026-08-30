# Lesson 3 – Real-Time Intelligence

## 3.1 – Get Started with Real-Time Intelligence

### 1. What is Real-Time Intelligence?
- **Real-Time Intelligence** in Microsoft Fabric is used to ingest, process, analyze, visualize, and act on data as it arrives.
- It is designed for scenarios where decisions need to be made quickly.

**Remember:**  
`Real-Time Intelligence = Detect → Analyze → Visualize → Act`

### 2. Why Real-Time Analytics Matters
Real-time processing is useful when waiting for a batch process is too slow.

Common scenarios:
- Fraud detection
- IoT monitoring
- Live sales/transaction tracking
- Application monitoring
- Anomaly detection

**Remember:**  
`Real-time = Act on data quickly after it arrives`

### 3. Real-Time Intelligence Toolset

#### Eventstream
- Ingests and processes streaming data.
- Routes events to destinations.

> **Eventstream → Ingest + Process + Route**

#### Eventhouse
- Stores and analyzes real-time/event data.
- Uses KQL for querying.

> **Eventhouse → Store + Analyze**

#### KQL Database
- Contains tables for event data.
- Data can be queried using KQL.

#### Real-Time Dashboard
- Visualizes changing/real-time data.
- Useful for operational monitoring.

> **Dashboard → Visualize**

#### Activator
- Monitors data for specified conditions.
- Triggers actions when conditions are met.

> **Activator → Detect + Act**

### 4. End-to-End Real-Time Flow

```text
Streaming Source
       ↓
   Eventstream
       ↓
   Eventhouse
       ↓
      KQL
     ↙   ↘
Dashboard  Activator
              ↓
            Action
```

### 5. No-Code / Low-Code Experience
- Many Real-Time Intelligence components provide visual configuration.
- Common streaming scenarios can be built without writing a complete custom streaming application.
- KQL is still used when querying and analyzing data.

### 6. KQL
- **KQL = Kusto Query Language**.
- Used extensively for querying and analyzing event data.
- Especially important with Eventhouse/KQL databases.
- Real-Time Dashboards can use KQL query results.

**Remember:**  
`KQL → Query and analyze real-time data`

### 7. Real-Time vs Batch

| Batch | Real-Time |
|---|---|
| Processes data periodically | Processes data as it arrives |
| Higher latency is acceptable | Low latency is important |
| Historical analysis | Live monitoring |
| Example: Daily report | Example: Fraud alert |

---

## 3.2 – Use Eventstream in Microsoft Fabric

### 1. What is Eventstream?
- **Eventstream** is used to ingest, transform, and route real-time streaming data.
- It provides a visual experience for building streaming data flows.

**Remember:**  
`Eventstream = Ingest + Transform + Route`

### 2. Streaming Sources
Supported sources can include:
- Azure Event Hubs
- Azure IoT Hub
- Kafka / Confluent
- Sample data
- Custom endpoints
- Other supported streaming/event sources

**Remember:**  
`Source → Eventstream`

### 3. In-Stream Transformations
Eventstream can transform streaming data while it is flowing.

Examples:
- Filter
- Manage/expand fields
- Group/aggregate
- Union
- Other supported transformations

**Remember:**  
`Eventstream can transform data while it is flowing.`

### 4. Destinations
Eventstream can route data to supported destinations, including:
- **Eventhouse / KQL Database**
- **Lakehouse**
- **Activator**
- Other supported endpoints

### 5. Visual Canvas
- Eventstream provides a visual canvas.
- Components can be connected to create a streaming flow.

```text
Source
  ↓
Eventstream
  ↓
Transformation
  ↓
Destination
```

### 6. Data Preview
- Data preview capabilities help inspect incoming streaming data.
- Useful for understanding the event structure and validating transformations.

### 7. Continuous Streaming
- Eventstream is designed for continuously arriving data.
- It processes events as they arrive instead of waiting for a traditional batch cycle.

**Example:**

```text
IoT Device
    ↓
Eventstream
    ↓
Filter
    ↓
Eventhouse
```

### 8. Eventstream vs Dataflow Gen2

| Eventstream | Dataflow Gen2 |
|---|---|
| Streaming/event processing | Data integration/transformation |
| Continuous data | Run-based ETL |
| Processes data as it flows | Processes data during a run |
| Streaming scenarios | General ETL scenarios |
| Visual streaming flow | Power Query experience |

**Remember:**  
`Eventstream → Streaming`  
`Dataflow Gen2 → ETL / Data Transformation`

### 9. Eventstream vs Eventhouse

> **Eventstream → Move + Process**

> **Eventhouse → Store + Analyze**

---

## 3.3 – Work with Real-Time Data in an Eventhouse

### 1. Eventhouse Recap
- Eventhouse stores and analyzes real-time/event data.
- Data is organized in KQL databases and tables.
- KQL is used to query the data.
- Eventstream is a common ingestion path.

**Remember:**  
`Eventstream → Ingest/Process`  
`Eventhouse → Store/Analyze`

### 2. OneLake Integration
- Eventhouse has integration with OneLake and the wider Fabric ecosystem.
- Do not assume all Eventhouse data is simply stored as Delta Parquet like a Lakehouse table.
- Eventhouse has its own real-time storage/query architecture.

**Remember:**  
`Eventhouse integrates with Fabric/OneLake, but it is not the same storage architecture as a Lakehouse.`

### 3. Update Policies
- Update policies automatically transform incoming data from one table into another.
- Useful for creating derived or cleaned tables.

```text
Raw Event Table
      ↓
Update Policy
      ↓
Derived/Cleaned Table
```

**Remember:**  
`Update Policy → Automatically transform incoming data`

### 4. KQL Operators

#### `take` / `limit`
- Limits the number of returned rows.

```kusto
SensorData
| take 10
```

> `take → Limit rows`

#### `extend`
- Creates a calculated column.

```kusto
SensorData
| extend TempFahrenheit = TempCelsius * 9/5 + 32
```

> `extend → Create calculated column`

#### `join`
- Combines related data from two tables based on matching values.

> `join → Combine tables`

#### `render`
- Specifies how query results should be visualized.

```kusto
| render timechart
```

> `render → Visualize`

### 5. KQL Example

```kusto
SensorData
| where Timestamp > ago(30m)
| extend TempFahrenheit = TempCelsius * 9/5 + 32
| summarize AvgTemp = avg(TempFahrenheit) by bin(Timestamp, 5m), SensorId
| render timechart
```

What it does:
1. `where` → Last 30 minutes
2. `extend` → Convert Celsius to Fahrenheit
3. `summarize` → Calculate average
4. `bin()` → Create 5-minute intervals
5. `render` → Display time chart

### 6. Materialized Views
- Materialized views maintain **precomputed results** for supported scenarios.
- Useful for frequently repeated aggregation queries.
- They can improve query performance by avoiding repeated computation.

**Remember:**  
`Materialized View → Precomputed maintained results`

### 7. Retention Policy
- Determines **how long data is retained**.

Example:
`Keep event data for 2 years.`

### 8. Hot Cache
- Determines how long data is kept in fast-access cache for query performance.

Example:
`Keep recent data hot for 7 days.`

**Remember:**  
`Retention → How long data is kept`  
`Hot cache → How long data stays in fast cache`

---

## 3.4 – Create a Real-Time Dashboard

### 1. What is a Real-Time Dashboard?
- Used to visualize and monitor **real-time/near-real-time data**.
- Commonly works with KQL/Eventhouse data.
- Useful for operational monitoring.

**Remember:**  
`Real-Time Dashboard → See what is happening`

### 2. Real-Time Dashboard vs Power BI

#### Real-Time Dashboard
- Designed for real-time/near-real-time monitoring.
- Uses KQL queries.
- Useful for operational scenarios.

#### Power BI
- General-purpose business intelligence.
- Supports rich data modeling and visualization.
- Supports multiple connectivity and refresh approaches.

**Remember:**  
`Real-Time Dashboard → Operational monitoring`  
`Power BI → General BI and reporting`

### 3. Dashboard Tiles
- A dashboard consists of **tiles**.
- Tiles display visualizations based on query results.
- A tile can be backed by a KQL query.

```text
KQL Query
    ↓
Visualization
    ↓
Dashboard Tile
```

### 4. Queryset
- A **queryset** is used to create and work with KQL queries.
- Query results can be used to build dashboard visualizations.

**Remember:**  
`Queryset → Work with KQL queries`  
`Dashboard → Present results`

### 5. Visualization Types
Common visualization types include:
- Time charts
- Bar charts
- Tables
- Area charts
- Maps
- Single-value/KPI-style visuals

### 6. Parameters
- Parameters make dashboards interactive.
- They can dynamically influence/filter data shown by tiles.

Examples:
- Time range
- Device
- Region
- Category

**Remember:**  
`Parameters → Interactive filtering`

### 7. Auto-Refresh
- Real-Time Dashboards can automatically refresh data.
- This helps keep dashboard information current.
- Exact refresh behavior/frequency depends on platform capabilities and configuration.

**Remember:**  
`Auto-refresh → Keep dashboard current`

### 8. Sharing and Permissions
- Dashboard access is controlled through Fabric access and permission mechanisms.
- Workspace permissions can be used to manage access.

**Important:**  
`Sharing/permissions ≠ Row-level security`

RLS is a separate data/semantic-model security concept where supported.

### 9. Operational Monitoring
Useful for monitoring:
- IoT devices
- System health
- Transactions
- Application events
- Sales activity
- Operational metrics

---

## 3.5 – Use Activator in Microsoft Fabric

### 1. What is Activator?
- **Activator** is a Fabric capability for **real-time, event-driven automation**.
- It monitors data for specified conditions.
- When a condition is met, it can trigger an action.
- It reduces the need for manual dashboard monitoring.

**Remember:**  
`Activator = Detect → Decide → Act`

### 2. Why Use Activator?
- Automatically responds to changing data.
- Detects important events quickly.
- Triggers actions without manual intervention.
- Useful for operational monitoring and automated responses.

### 3. Core Idea

```text
Data / Event
     ↓
Activator monitors
     ↓
Condition / Rule
     ↓
Condition TRUE
     ↓
Action
```

**Remember:**  
`If condition is met → Take action`

### 4. Data Sources
Activator can work with supported Fabric data sources/integrations, including:
- Eventstream
- Eventhouse / KQL data
- Power BI semantic models

### 5. Activator Building Blocks

#### Events
- Represents something that happens or changes in data.
- Examples:
  - Temperature reading
  - New transaction
  - New order
  - Machine status change

> **Event = Something happened/changed**

#### Objects
- Represents the thing an event is associated with.
- Examples:
  - Device
  - Customer
  - Machine
  - Order

> **Object = Thing being monitored**

#### Properties
- Attributes or fields of an object.
- Used when evaluating conditions.

Examples:
- Temperature
- Sales amount
- Customer balance
- Machine status

> **Property = Attribute of an object**

#### Rules
- Define the condition Activator monitors.
- When the condition becomes true, the configured action can be triggered.

Example:
`Temperature > 90°C`

> **Rule = Condition to monitor**

### 6. Condition Types

#### Threshold
- Triggers when a value crosses a limit.

Example:
`Temperature > 90°C`

#### Comparison Over Time
- Evaluates how a value changes over a period.

Example:
`Temperature increased significantly during the last 10 minutes.`

#### Absence of Data
- Detects when expected data/event does not arrive.

Example:
`No event received from Device A for 10 minutes.`

Useful for:
- Device failure
- Communication failure
- Missing data

**Remember:**  
`Activator can detect both "something happened" and "something did not happen."`

### 7. Actions
When a rule is triggered, Activator can perform supported actions such as:
- Teams notification
- Email
- Power Automate flow
- Other supported integrations/actions

**Remember:**  
`Condition TRUE → Action`

### 8. IoT Example

```text
Temperature Sensor
       ↓
   Eventstream
       ↓
    Eventhouse
       ↓
     Activator
       ↓
Rule: Temperature > 90°C
       ↓
 Teams Notification
       ↓
Maintenance Team
```

### 9. Activator vs Real-Time Dashboard

#### Real-Time Dashboard
> **Shows you what is happening.**

#### Activator
> **Acts when something happens.**

**Remember:**  
`Dashboard → See`  
`Activator → Act`

### 10. Activator vs Eventhouse

#### Eventhouse
- Stores and analyzes real-time/event data.

#### Activator
- Monitors conditions and triggers actions.

**Remember:**  
`Eventhouse = Store + Analyze`  
`Activator = Detect + Act`

---

# Lesson 3 – Final Revision

| Topic | Main Purpose |
|---|---|
| **3.1 Real-Time Intelligence** | Complete real-time analytics experience |
| **Eventstream** | Ingest/process/route streaming data |
| **Eventhouse** | Store/analyze event data |
| **KQL** | Query real-time/event data |
| **3.2 Eventstream** | Continuous streaming ingestion and processing |
| **3.3 Eventhouse** | KQL, update policies, materialized views, retention/cache |
| **3.4 Real-Time Dashboard** | Visualize and monitor live data |
| **Queryset** | Create/work with KQL queries |
| **Dashboard Tile** | Display query results |
| **3.5 Activator** | Detect conditions and trigger actions |
| **Event** | Something happened/changed |
| **Object** | Thing being monitored |
| **Property** | Attribute of an object |
| **Rule** | Condition to monitor |
| **Action** | Response to a triggered rule |

## Lesson 3 in One Picture

```text
                  REAL-TIME INTELLIGENCE
                           ↓
                    Streaming Sources
                           ↓
                       Eventstream
                           ↓
                       Eventhouse
                           ↓
                          KQL
                       ↙       ↘
             Real-Time Dashboard  Activator
                    ↓                 ↓
                  SEE               ACT
                    ↓                 ↓
               Monitoring          Action
```

## 10 Things to Remember

1. **Eventstream → Ingest/process/route**
2. **Eventhouse → Store/analyze**
3. **KQL → Query**
4. **Real-Time Dashboard → Visualize**
5. **Activator → Detect and act**
6. **`where` → Filter**
7. **`extend` → Calculated column**
8. **`summarize` → Aggregate**
9. **`bin()` → Time buckets**
10. **Materialized View → Precomputed results**

## Final Memory

> **Source → Eventstream → Eventhouse → KQL → Dashboard / Activator**


