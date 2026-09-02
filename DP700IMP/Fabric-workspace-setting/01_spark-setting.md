# Microsoft Fabric — Workspace & Spark Settings
## DP-700 + Data Engineer Interview Notes

---

# 1. Workspace

### Concept
**Workspace** = Collaborative container for Microsoft Fabric items.

### What it does
Provides a place to:
- Create and organize Fabric items
- Collaborate
- Manage access/security
- Configure workspace-level settings

### Keywords
`Container` · `Collaboration` · `Security` · `Fabric Items` · `Workspace Settings`

### Typical Scenario
Your team needs one location to organize notebooks, Lakehouses,
pipelines, warehouses, reports, and other Fabric items.

### Answer
**Workspace**

### Memory Trick
> **Workspace = WHERE Fabric items live**

---

# 2. Starter Pool

### Concept
**Starter Pool** = Microsoft-managed, pre-warmed Spark compute.

### What it does
Provides fast Spark startup without requiring you to manage the compute.

### Keywords
`Microsoft-managed` · `Pre-warmed` · `Fast startup` · `Development` · `Ad-hoc`

### Typical Scenario
A developer is experimenting with notebooks and wants the fastest
startup with minimal administration.

### Answer
**Starter Pool**

### Memory Trick
> **Starter Pool = Fast + Simple**

---

# 3. Custom Pool

### Concept
**Custom Pool** = Configurable Spark compute.

### What it does
Allows control over:
- Node family
- Node size
- Autoscale
- Dynamic Executor Allocation

### Keywords
`Custom compute` · `Production` · `Node configuration` · `Scaling`

### Typical Scenario
A production workload requires specific compute resources and
custom scaling behavior.

### Answer
**Custom Pool**

### Memory Trick
> **Custom Pool = Compute Control**

---

# 4. Node Family

### Concept
**Node Family** = Determines the type of CPU/memory characteristics.

### What it does

**General Purpose**
- Balanced CPU + memory
- Standard ETL workloads

**Memory Optimized**
- More memory
- Large joins
- Large aggregations
- Memory-intensive workloads

### Keywords
`CPU` · `Memory` · `General Purpose` · `Memory Optimized`

### Typical Scenario
A workload performs large joins and frequently requires more memory.

### Answer
**Memory Optimized Node Family**

### Memory Trick
> **Node Family = Type of resources**

---

# 5. Node Size

### Concept
**Node Size** = Amount of compute resources available per node.

### What it does
Larger nodes generally provide:
- More CPU
- More memory
- More compute capacity

### Keywords
`CPU` · `Memory` · `Small` · `Medium` · `Large` · `XLarge`

### Typical Scenario
A workload is resource-intensive and requires more CPU and memory
per node.

### Answer
**Increase Node Size**

### Memory Trick
> **Node Size = Resources per node**

---

# 6. Autoscale

### Concept
**Autoscale** = Automatically changes the number of nodes.

### What it does
Scales the Spark pool between configured minimum and maximum nodes.

Example:
- Minimum = 2 nodes
- Maximum = 10 nodes

### Keywords
`Node count` · `Scale up/down` · `Elastic` · `Min/Max nodes`

### Typical Scenario
Workload demand changes throughout the day and the number of nodes
should automatically increase or decrease.

### Answer
**Autoscale**

### Memory Trick
> **Autoscale = NODES**

---

# 7. Dynamic Executor Allocation

### Concept
**Dynamic Executor Allocation** = Dynamically changes the number of
Spark executors based on workload demand.

### What it does
Adds or removes executors depending on workload requirements.

### Keywords
`Executors` · `Dynamic` · `Resource utilization`

### Typical Scenario
A Spark workload should automatically increase or decrease the
number of executors as demand changes.

### Answer
**Dynamic Executor Allocation**

### Memory Trick
> **DAE = EXECUTORS**

---

# 8. Autoscale vs Dynamic Executor Allocation

### Concept
Both provide dynamic resource adjustment, but at different levels.

### What it does

```text
Autoscale
    ↓
Changes NUMBER OF NODES

Dynamic Executor Allocation
    ↓
Changes NUMBER OF EXECUTORS
```

### Keywords
`Nodes` · `Executors` · `Scaling`

### Typical Scenario
The question says:
> "Automatically change the number of nodes."

### Answer
**Autoscale**

If it says:
> "Automatically change the number of executors."

### Answer
**Dynamic Executor Allocation**

### Memory Trick
> **Nodes = Autoscale | Executors = DAE**

---

# 9. Driver

### Concept
**Driver** = Coordinator of the Spark application.

### What it does
- Coordinates the Spark application
- Creates execution plans
- Coordinates jobs, stages, and tasks

### Keywords
`Coordinator` · `Application` · `Execution Plan`

### Typical Scenario
Which Spark component coordinates the application and manages
execution?

### Answer
**Driver**

### Memory Trick
> **Driver = Coordinator**

---

# 10. Executor

### Concept
**Executor** = Worker that executes Spark tasks.

### What it does
- Runs tasks
- Processes data
- Performs computations
- Can store cached/intermediate data

### Keywords
`Worker` · `Tasks` · `Data Processing` · `CPU/Memory`

### Typical Scenario
Which Spark component actually executes tasks on the data?

### Answer
**Executor**

### Memory Trick
> **Executor = Worker**

---

# 11. Environment

### Concept
**Environment** = Reusable Spark software/configuration setup.

### What it does
Can contain:
- Runtime
- Libraries
- Spark configurations
- Dependencies

### Keywords
`Runtime` · `Libraries` · `Dependencies` · `Spark Configuration` · `Reusable`

### Typical Scenario
Twenty notebooks need the same runtime, Python libraries,
dependencies, and Spark configuration.

### Answer
**Environment**

### Memory Trick
> **Environment = Spark Software + Configuration**

---

# 12. Default Environment

### Concept
**Default Environment** = Workspace-level default Spark environment.

### What it does
Provides a standard environment for applicable Spark workloads.

### Keywords
`Workspace default` · `Standardization` · `Consistency`

### Typical Scenario
An organization wants all applicable Spark workloads in a workspace
to use the same default environment unless another environment is
explicitly selected.

### Answer
**Default Environment**

### Memory Trick
> **Default Environment = Workspace's default Spark setup**

---

# 13. Pool vs Environment

### Concept
Pool and Environment solve different problems.

### What it does

```text
POOL
 ↓
Compute
 ↓
Nodes / CPU / Memory / Scaling

ENVIRONMENT
 ↓
Software + Configuration
 ↓
Runtime / Libraries / Dependencies / Spark Config
```

### Keywords
`Pool = Compute`
`Environment = Software`

### Typical Scenario
The question asks for control over node size and scaling.

### Answer
**Pool**

If it asks for runtime and libraries:

### Answer
**Environment**

### Memory Trick
> **Pool = Compute | Environment = Software**

---

# 14. Save

### Concept
**Save** = Save environment changes as a draft.

### What it does
Keeps changes in the editable/draft state.

### Keywords
`Draft` · `Unpublished` · `Changes`

### Typical Scenario
An engineer modifies an environment but is not ready to make
the changes available to Spark sessions.

### Answer
**Save**

### Memory Trick
> **Save = Draft**

---

# 15. Publish

### Concept
**Publish** = Makes environment changes available for Spark sessions.

### What it does
Publishes the environment configuration after changes are ready.

### Keywords
`Available` · `Spark sessions` · `Deploy configuration`

### Typical Scenario
An engineer has finished testing the environment and wants the
updated libraries/configuration to be used by Spark sessions.

### Answer
**Publish**

### Memory Trick
> **Publish = Make Available**

---

# 16. Quick Mode

### Concept
**Quick Mode** = Faster environment publishing.

### What it does
Prioritizes fast publishing, with libraries handled during
Spark session startup.

### Keywords
`Fast publish` · `Startup installation` · `Speed`

### Typical Scenario
A developer frequently changes an environment and wants changes
published as quickly as possible.

### Answer
**Quick Mode**

### Memory Trick
> **Quick = Fast Publish**

---

# 17. Full Mode

### Concept
**Full Mode** = More complete environment preparation.

### What it does
Libraries are pre-deployed, resulting in longer publishing time.

### Keywords
`Pre-deployed libraries` · `Longer publish` · `Production-oriented`

### Typical Scenario
A production workload needs libraries prepared ahead of
Spark session startup.

### Answer
**Full Mode**

### Memory Trick
> **Full = Pre-deployed**

---

# 18. Runtime Version

### Concept
**Runtime Version** = Defines the Spark runtime/software version.

### What it does
Determines important components such as:
- Spark version
- Python version
- Delta/runtime capabilities

### Keywords
`Spark Version` · `Python Version` · `Delta` · `Compatibility`

### Typical Scenario
A workload requires a particular Spark/Python runtime and
compatible libraries.

### Answer
**Runtime Version**

### Memory Trick
> **Runtime = Spark + Python + Runtime capabilities**

---

# 19. GA vs Preview

### Concept
Fabric features can be Generally Available or Preview.

### What it does

**GA**
- Production ready
- Generally supported

**Preview**
- Evaluation/testing
- May change or have limitations

### Keywords
`GA` · `Production` · `Preview` · `Testing`

### Typical Scenario
A production workload requires a stable, generally available feature.

### Answer
**GA**

### Memory Trick
> **GA = Production | Preview = Evaluation**

---

# 20. High Concurrency

### Concept
**High Concurrency** = Allows compatible Spark workloads to share
a Spark session.

### What it does
Can reduce:
- Session startup overhead
- Resource consumption
- Duplicate Spark sessions

### Keywords
`Shared Session` · `Compatible Workloads` · `Same User` · `Session Reuse`

### Typical Scenario
Multiple compatible notebooks need to run for the same user and
the organization wants to reduce separate Spark session overhead.

### Answer
**High Concurrency**

### Memory Trick
> **High Concurrency = Shared Spark Session**

---

# 21. Session Timeout

### Concept
**Session Timeout** = Controls how long an idle Spark session can remain active.

### What it does
Allows idle sessions to be terminated and resources released.

### Keywords
`Idle Session` · `Terminate` · `Release Resources`

### Typical Scenario
The organization wants idle Spark sessions to stop consuming
resources.

### Answer
**Session Timeout**

### Memory Trick
> **Idle Session → Timeout → Release**

---

# 22. Spark Configuration

### Concept
**Spark Configuration** = Settings used to tune Spark behavior.

### What it does
Controls areas such as:
- Shuffle
- Executor memory
- Executor CPU
- Parallelism

### Keywords
`Performance` · `Shuffle` · `Memory` · `Cores` · `Parallelism`

### Typical Scenario
A Spark workload needs tuning of shuffle partitions and executor
resources.

### Answer
**Spark Configuration**

### Memory Trick
> **Spark Config = Performance Tuning**

---

# 23. spark.sql.shuffle.partitions

### Concept
Controls the number of partitions used for shuffle operations.

### What it does
Affects shuffle operations such as:
- Joins
- Aggregations
- Group By
- Sorts

### Keywords
`Shuffle` · `Partitions` · `Parallelism`

### Typical Scenario
A large aggregation creates a shuffle bottleneck and partitioning
needs to be tuned.

### Answer
**spark.sql.shuffle.partitions**

### Memory Trick
> **Shuffle → Partitions**

---

# 24. spark.executor.memory

### Concept
Controls executor memory.

### What it does
Provides memory resources for Spark executors.

### Keywords
`Executor` · `Memory` · `RAM`

### Typical Scenario
Executors are experiencing memory pressure and the workload requires
more executor memory.

### Answer
**spark.executor.memory**

### Memory Trick
> **Executor Memory = RAM**

---

# 25. spark.executor.cores

### Concept
Controls CPU cores available to an executor.

### What it does
Determines CPU capacity available to an executor for running tasks,
subject to available resources and configuration.

### Keywords
`Executor` · `CPU` · `Cores` · `Parallelism`

### Typical Scenario
You need to configure the CPU capacity available to each executor.

### Answer
**spark.executor.cores**

### Memory Trick
> **Executor Cores = CPU**

---

# 26. Native Execution Engine (NEE)

### Concept
**Native Execution Engine** = Fabric-optimized execution engine for
supported Spark operations.

### What it does
Can provide:
- Faster execution
- Better resource efficiency
- Improved Delta workload performance

### Keywords
`Native` · `Optimization` · `Performance` · `Supported Operations`

### Typical Scenario
A company wants to use Fabric's optimized execution path for
supported Spark operations.

### Answer
**Native Execution Engine**

### Memory Trick
> **NEE = Optimized Spark Execution**

---

# 27. Automatic JVM Fallback

### Concept
Unsupported NEE operations can fall back to JVM-based Spark execution.

### What it does

```text
Spark Operation
      ↓
     NEE
      ↓
Unsupported
      ↓
JVM Spark
```

### Keywords
`Unsupported NEE` · `Automatic Fallback` · `JVM`

### Typical Scenario
A Spark operation isn't supported by NEE and the workload should
continue without manually rewriting the operation.

### Answer
**Automatic JVM Fallback**

### Memory Trick
> **NEE can't handle it → JVM fallback**

---

# 28. Spark UI

### Concept
**Spark UI** = Tool for monitoring and troubleshooting Spark workloads.

### What it does
Helps investigate:
- Jobs
- Stages
- Tasks
- Executors
- Failures
- Shuffle
- Resource usage
- Performance bottlenecks

### Keywords
`Monitoring` · `Troubleshooting` · `Performance` · `Jobs` · `Stages`

### Typical Scenario
A Spark job is taking too long and you need to identify the
slow stage or failed tasks.

### Answer
**Spark UI**

### Memory Trick
> **Slow Spark Job → Spark UI**

---

# 29. Spark Execution Hierarchy

### Concept
Spark breaks work into multiple execution levels.

### What it does

```text
Application
     ↓
    Job
     ↓
   Stage
     ↓
   Task
     ↓
Executor executes Task
```

### Keywords
`Job` · `Stage` · `Task` · `Executor`

### Typical Scenario
You need to identify the smallest unit of work executed by an executor.

### Answer
**Task**

### Memory Trick
> **Job → Stage → Task**

---

# 30. Spark Troubleshooting

### Concept
Use Spark UI to identify the source of performance problems.

### What it does
Investigate:
- Slow stages
- Failed tasks
- Executor problems
- Memory pressure
- Shuffle bottlenecks
- Partition issues
- Resource utilization

### Keywords
`Performance` · `Bottleneck` · `Shuffle` · `Memory` · `Failed Tasks`

### Typical Scenario
A Spark pipeline suddenly becomes slow. You need to determine
whether the problem is caused by stages, tasks, executors,
shuffle, or resources.

### Answer
**Use Spark UI and investigate the execution details.**

### Memory Trick
> **Slow Job → Job → Stage → Task → Executor**

---

# ⭐ FINAL SCENARIO CHEAT SHEET

| Scenario Keyword | Answer |
|---|---|
| Fast startup | **Starter Pool** |
| Microsoft-managed | **Starter Pool** |
| Custom compute | **Custom Pool** |
| Production compute configuration | **Custom Pool** |
| CPU/memory type | **Node Family** |
| More CPU/memory per node | **Node Size** |
| Change number of nodes | **Autoscale** |
| Change number of executors | **Dynamic Executor Allocation** |
| Coordinate Spark application | **Driver** |
| Execute Spark tasks | **Executor** |
| Runtime + libraries + dependencies | **Environment** |
| Workspace-wide Spark default | **Default Environment** |
| Compute resources | **Pool** |
| Spark software/configuration | **Environment** |
| Keep changes as draft | **Save** |
| Make environment changes available | **Publish** |
| Fast publishing | **Quick Mode** |
| Pre-deployed libraries | **Full Mode** |
| Spark/Python runtime | **Runtime Version** |
| Production-ready feature | **GA** |
| Testing/evaluation feature | **Preview** |
| Share Spark session | **High Concurrency** |
| Idle Spark session | **Session Timeout** |
| Shuffle tuning | **spark.sql.shuffle.partitions** |
| Executor memory | **spark.executor.memory** |
| Executor CPU | **spark.executor.cores** |
| Optimized Spark execution | **Native Execution Engine** |
| NEE unsupported operation | **JVM Fallback** |
| Slow Spark workload | **Spark UI** |
| Smallest Spark execution unit | **Task** |

---

# 🔥 TOP 15 TO MEMORIZE

1. **Workspace = Where Fabric items live**
2. **Starter Pool = Fast + Microsoft managed**
3. **Custom Pool = Configurable compute**
4. **Node Family = Resource type**
5. **Node Size = Resources per node**
6. **Autoscale = Nodes**
7. **DAE = Executors**
8. **Driver = Coordinator**
9. **Executor = Worker**
10. **Environment = Runtime + Libraries + Configuration**
11. **Save = Draft**
12. **Publish = Available**
13. **High Concurrency = Shared session**
14. **NEE = Optimized execution**
15. **Spark UI = Troubleshooting**

---

# 🧠 GOLDEN MEMORY MAP

```text
WORKSPACE
   ↓
Fabric Items
   ↓
SPARK WORKLOAD
   ↓
POOL
   ↓
Compute / Nodes
   ↓
Autoscale → Nodes
   ↓
DAE → Executors
   ↓
ENVIRONMENT
   ↓
Runtime + Libraries + Configuration
   ↓
SPARK SESSION
   ↓
Driver → Executors
   ↓
Jobs → Stages → Tasks
   ↓
Spark UI
```

## One Final Rule

> **DP-700 Scenario Question = Find the keyword first, then choose the setting.**

```text
NODES       → Autoscale
EXECUTORS   → Dynamic Executor Allocation
COMPUTE     → Pool
RUNTIME     → Environment
LIBRARIES   → Environment
DRAFT       → Save
AVAILABLE   → Publish
SHARED      → High Concurrency
OPTIMIZED   → NEE
UNSUPPORTED → JVM Fallback
SLOW JOB    → Spark UI
```
