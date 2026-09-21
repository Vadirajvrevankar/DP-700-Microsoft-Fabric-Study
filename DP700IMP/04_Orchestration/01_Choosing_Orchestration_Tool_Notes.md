# DP-700 — Choosing the Right Orchestration Tool

## 1. Overview

Fabric provides three important tools for data transformation and orchestration:

| Tool | Main Purpose | Think |
|---|---|---|
| **Dataflow Gen2** | Low-code / Power Query transformation | **TRANSFORM** |
| **Pipeline** | Orchestration and coordination | **CONTROL** |
| **Notebook** | Custom Spark-based logic | **CODE** |

### Core Rule

> **Use the lowest-code tool that can correctly express the required transformation.**

That normally means:

**Dataflow Gen2 → Notebook**

But do **not** force a complex transformation into Dataflow Gen2 simply to avoid writing code.

---

# 2. Dataflow Gen2

## What is Dataflow Gen2?

Dataflow Gen2 is a **low-code Power Query ETL/transform tool**.

It is designed for visually expressing data transformations without requiring extensive programming.

It has **300+ transformations**.

## Best Use Cases

Use Dataflow Gen2 when:

- The scenario asks for **no-code**
- A **citizen developer** is expected to build the solution
- Standard Power Query transformations are sufficient
- The required transformation is supported by its built-in transformation catalog
- Connector-supported transformations are appropriate
- The transformation can be expressed visually

### Examples

- Filter rows
- Remove columns
- Rename columns
- Change data types
- Standard cleansing
- Standard reshaping
- Connector-based ingestion and transformation

## Important Limitation

Do not force a transformation into Dataflow Gen2 when the required logic is beyond what its transformation catalog can reasonably express.

If the scenario requires:

- Custom algorithms
- Machine learning
- Complex custom processing
- Logic beyond built-in transformations

then a **Notebook** is generally the appropriate tool.

### Memory Trick

> **Dataflow Gen2 = Low-Code Transformation**

---

# 3. Pipeline

## What is a Pipeline?

A Pipeline is the **coordination/orchestration layer**.

Use a pipeline when multiple Fabric items need to work together.

### Typical items

- Copy activity
- Dataflow Gen2
- Notebook
- Other pipeline activities

## When Should You Use a Pipeline?

Use a Pipeline when you need:

- Multiple items to run in sequence
- Shared parameters
- Failure-handling paths
- Control flow
- Loops
- Conditions
- Scheduling
- Triggers
- Event-based execution
- Coordination between Dataflow Gen2, Notebook, and Copy activities

### Example

```text
Pipeline
   |
   +---- Copy Data
   |
   +---- Dataflow Gen2
   |
   +---- Notebook
```

The Pipeline controls **when and how** these activities execute.

## Important Exam Rule

> **Only a Pipeline orchestrates other items.**

Dataflow Gen2 and Notebooks can each be scheduled on their own, but neither is the orchestration layer for other Fabric items.

### Memory Trick

> **Pipeline = CONTROL + COORDINATION**

---

# 4. Notebook

## What is a Notebook?

A Notebook provides **Spark-based code execution** for custom and advanced data processing.

It is the appropriate choice when transformation complexity or custom logic exceeds what can reasonably be expressed through Dataflow Gen2.

## Best Use Cases

Use a Notebook for:

- Custom algorithms
- Machine learning
- Complex transformations
- Advanced Spark processing
- Logic beyond built-in Dataflow Gen2 transformations
- Custom, testable logic
- Source-controllable logic

### Memory Trick

> **Notebook = CUSTOM LOGIC + SPARK**

---

# 5. Dataflow Gen2 vs Notebook

| Requirement | Dataflow Gen2 | Notebook |
|---|---:|---:|
| No-code | ✅ | ❌ |
| Citizen developer | ✅ | ❌ |
| Visual transformation | ✅ | ❌ |
| Power Query | ✅ | ❌ |
| 300+ built-in transforms | ✅ | ❌ |
| Custom algorithm | ❌ | ✅ |
| Machine learning | ❌ | ✅ |
| Complex custom logic | ❌ | ✅ |
| Advanced Spark processing | ❌ | ✅ |
| Source-controllable custom code | Limited | ✅ |

## Exam Rule

> **Simple / Low-Code → Dataflow Gen2**

> **Complex / Custom → Notebook**

Do not choose Notebook merely because it can technically perform the transformation.

Choose the **simplest appropriate tool**.

---

# 6. Pipeline vs Dataflow Gen2 vs Notebook

| Capability | Dataflow Gen2 | Pipeline | Notebook |
|---|---:|---:|---:|
| Low-code transformation | ✅ | ❌ | ❌ |
| Power Query transformation | ✅ | ❌ | ❌ |
| Custom Spark logic | ❌ | ❌ | ✅ |
| Schedule itself | ✅ | ✅ | ✅ |
| Trigger other items | ❌ | ✅ | ❌ |
| Sequence multiple items | ❌ | ✅ | ❌ |
| Loops | ❌ | ✅ | ❌ |
| Conditions | ❌ | ✅ | ❌ |
| Failure-branch orchestration | ❌ | ✅ | ❌ |
| Coordinate Copy + Dataflow + Notebook | ❌ | ✅ | ❌ |

### Most Important Distinction

> **Being schedulable does not make a tool an orchestration tool.**

A Dataflow Gen2 or Notebook may run on a schedule, but **Pipeline** is the tool used to coordinate multiple items.

---

# 7. Pipeline as the Coordination Layer

When more than one item needs to:

- Run in sequence
- Share parameters
- Share failure-handling logic
- Execute conditionally
- Participate in loops
- Respond to events

use a Pipeline.

### Example

```text
Start
  |
  v
Copy Activity
  |
  v
Dataflow Gen2
  |
  v
Notebook
  |
  v
Success
```

If the Dataflow fails:

```text
Dataflow Gen2
     |
     +---- Success → Notebook
     |
     +---- Failure → Failure Handling
```

This type of cross-item control belongs in a **Pipeline**.

---

# 8. Keep Business Logic Out of Deep Pipeline Expressions

Pipeline expressions are useful for:

- Passing parameters
- Connecting values between activities
- Simple conditions
- Configuration
- Wiring activities together

They should not become the main place for complex business logic.

## Better Pattern

```text
Pipeline
   |
   +---- Parameters / Conditions / Wiring
   |
   +---- Notebook
             |
             +---- Complex Business Logic
```

### Why?

Notebook logic can be:

- Custom
- Testable
- Source-controllable
- Easier to maintain for complex transformations

### Memory Trick

> **Pipeline expressions = WIRING**

> **Notebook = BUSINESS LOGIC**

---

# 9. Composed Solutions

A major exam concept is that you do **not** always choose one tool.

Real solutions frequently combine:

> **Pipeline + Dataflow Gen2 + Notebook**

## Example

A company needs to:

1. Copy source data
2. Perform standard cleansing
3. Apply a complex custom algorithm
4. Handle failures

### Solution

```text
Pipeline
   |
   +---- Copy Activity
   |
   +---- Dataflow Gen2
   |        |
   |        +---- Standard cleansing
   |
   +---- Notebook
            |
            +---- Custom algorithm
```

### Responsibility

**Pipeline**
→ Coordinates execution

**Dataflow Gen2**
→ Performs standard low-code transformations

**Notebook**
→ Performs complex/custom logic

## Exam Tip

If a scenario contains both:

- Orchestration requirements
- Transformation-complexity requirements

a **composed solution** is often the correct answer rather than choosing one tool for everything.

---

# 10. Dataflow Gen2 Public Parameters

## Important Exam Topic

Dataflow Gen2 **public parameters are a Preview feature** according to the study material.

A documented restriction applies when **required public parameters** are involved with scheduling/manual triggering.

## Exam Trap

A scenario may say:

> "Create a Dataflow Gen2 with a required public parameter and run it unattended on a schedule."

This combination should immediately make you check the documented restriction.

### Memory Trick

> **Public Parameters = Preview**

> **Required Parameter + Unattended Scheduling = Exam Trap**

Do not assume that because a feature exists, every scheduling/triggering scenario is supported.

---

# 11. Notebook Execution Identity

Notebook execution identity depends on how the Notebook is triggered.

| Execution method | Identity |
|---|---|
| **Interactive** | Current user |
| **Pipeline-invoked** | Pipeline's last-modified user |
| **Scheduled** | Schedule's creator / last-updater |

## Memory Trick

> **Interactive → Me**

> **Pipeline → Pipeline last modifier**

> **Schedule → Schedule creator / last updater**

This can appear in questions involving permissions, access, or execution context.

---

# 12. Exam Decision Tree

Use this decision process.

## Step 1 — Is orchestration required?

Look for:

- Multiple items
- Sequence
- Failure branch
- Loop
- Condition
- Event trigger
- Shared parameters
- Cross-item coordination

### YES

> **Use Pipeline**

Then place the required Dataflow Gen2 / Notebook / Copy activities inside the pipeline.

---

## Step 2 — Is the transformation low-code?

Look for:

- No-code
- Citizen developer
- Power Query
- Standard transformations
- Connector-supported transformation
- Visual transformation

### YES

> **Use Dataflow Gen2**

---

## Step 3 — Is custom or complex logic required?

Look for:

- Custom algorithm
- Machine learning
- Complex Spark processing
- Logic beyond built-in transforms
- Advanced custom processing

### YES

> **Use Notebook**

---

# 13. Scenario-Based Questions

## Scenario 1

A citizen developer needs to clean and reshape data using standard transformations.

### Answer

**Dataflow Gen2**

### Why?

Because the scenario emphasizes:

- Citizen developer
- Low-code
- Standard transformations

---

## Scenario 2

A data engineer needs a custom algorithm that is not available in Power Query.

### Answer

**Notebook**

### Why?

The required logic exceeds the built-in transformation catalog.

---

## Scenario 3

A solution must run:

```text
Copy → Dataflow → Notebook
```

and use failure handling.

### Answer

**Pipeline**

### Why?

Multiple items + sequencing + failure handling = orchestration.

---

## Scenario 4

A solution needs standard cleansing followed by a complex Spark transformation.

### Answer

**Pipeline + Dataflow Gen2 + Notebook**

### Why?

- Dataflow Gen2 → standard cleansing
- Notebook → complex transformation
- Pipeline → coordinates both

---

## Scenario 5

A Dataflow Gen2 has a required public parameter and must run unattended on a schedule.

### Answer

**Check the documented Preview restriction.**

### Why?

Required public parameters have a documented scheduling/manual-triggering restriction in the study material.

---

# 14. Common Exam Traps

## Trap 1 — "Notebook can do everything"

❌ Do not automatically choose Notebook.

Use the lowest-code appropriate tool.

---

## Trap 2 — "Dataflow Gen2 can be scheduled, therefore it is an orchestration tool"

❌ Incorrect.

Scheduling itself does not make it an orchestration layer.

> **Pipeline = orchestration**

---

## Trap 3 — "Put all business logic into pipeline expressions"

❌ Avoid complex business logic in deeply nested expressions.

Use:

> Expressions → wiring

> Notebook → complex business logic

---

## Trap 4 — "Avoid code at all costs"

❌ Incorrect.

If Dataflow Gen2 cannot express the transformation appropriately, use Notebook.

---

## Trap 5 — "Everything must be done in one tool"

❌ Incorrect.

Use a composed architecture when requirements span multiple capabilities.

> **Pipeline + Dataflow Gen2 + Notebook**

---

## Trap 6 — "Public parameters are fully production-ready"

❌ Incorrect according to the study material.

They are currently **Preview**, with a documented restriction around required public parameters and scheduling/manual triggering.

---

# 15. Quick Comparison

| Requirement | Choose |
|---|---|
| No-code | **Dataflow Gen2** |
| Citizen developer | **Dataflow Gen2** |
| Power Query | **Dataflow Gen2** |
| Standard transformations | **Dataflow Gen2** |
| 300+ transforms | **Dataflow Gen2** |
| Custom algorithm | **Notebook** |
| Machine learning | **Notebook** |
| Complex Spark logic | **Notebook** |
| Business logic that is complex/testable | **Notebook** |
| Multiple items | **Pipeline** |
| Sequence | **Pipeline** |
| Loop | **Pipeline** |
| Condition | **Pipeline** |
| Failure branch | **Pipeline** |
| Event trigger | **Pipeline** |
| Cross-item coordination | **Pipeline** |
| Dataflow + Notebook in one workflow | **Pipeline** |

---

# 16. Key Takeaways

- **Dataflow Gen2 = low-code Power Query ETL**
- **Pipeline = orchestration and coordination**
- **Notebook = unrestricted/custom Spark code**
- Dataflow Gen2 has **300+ transformations**
- Start with the **lowest-code appropriate tool**
- Do not force complex transformations into Dataflow Gen2
- Pipeline is the **orchestration layer for other items**
- Pipeline provides scheduling, triggers, control flow, loops, conditions, and failure-branch orchestration
- Dataflow Gen2 and Notebook can each be scheduled independently
- Dataflow Gen2 and Notebook do **not** orchestrate other items
- Pipeline expressions are primarily for **wiring/configuration**
- Keep complex, custom, testable, source-controllable business logic in Notebooks
- Custom algorithms, ML, and logic beyond Power Query → **Notebook**
- Dataflow Gen2 public parameters are **Preview**
- Required public parameters + unattended scheduling/manual triggering is a documented restriction/exam trap
- Real solutions often combine **Pipeline + Dataflow Gen2 + Notebook**
- Notebook execution identity depends on whether execution is interactive, pipeline-invoked, or scheduled

---

# 17. One-Minute Revision

```text
DATAFLOW GEN2
↓
LOW-CODE
POWER QUERY
300+ TRANSFORMS
CITIZEN DEVELOPER
STANDARD TRANSFORMATION

PIPELINE
↓
ORCHESTRATION
SCHEDULE
TRIGGER
LOOP
CONDITION
FAILURE HANDLING
COORDINATE ITEMS

NOTEBOOK
↓
SPARK
CUSTOM CODE
COMPLEX LOGIC
ML
ADVANCED TRANSFORMATION
```

## Final Memory Formula

> **DATAFLOW = TRANSFORM**

> **PIPELINE = ORCHESTRATE**

> **NOTEBOOK = CUSTOM CODE**

### Shortcut

> **LOW-CODE → DATAFLOW**

> **CONTROL → PIPELINE**

> **COMPLEX LOGIC → NOTEBOOK**

### Most Important Exam Rule

> **If multiple items must work together → PIPELINE**

> **If standard low-code transformation → DATAFLOW GEN2**

> **If custom/complex transformation → NOTEBOOK**

> **If both orchestration and complex transformation are required → COMBINE THEM**
