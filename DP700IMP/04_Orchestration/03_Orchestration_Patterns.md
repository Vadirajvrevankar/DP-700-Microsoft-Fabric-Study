# DP-700 — Orchestration Patterns

## 1. Overview

This topic focuses on practical orchestration patterns in Microsoft Fabric:

- Parameters vs variables
- Dynamic expressions
- Pipeline activity outputs
- Invoke Pipeline
- Notebook execution patterns
- `notebookutils.notebook.run`
- `notebookutils.notebook.runMultiple`
- `%run`
- DAGs and concurrency
- `exit()`
- Activity outcome paths
- Fail activity
- Failure notifications
- Teams / Outlook notification patterns

### Core Memory

> **PARAMETER = FIXED INPUT**

> **VARIABLE = CHANGING VALUE**

> **PIPELINE = COMPOSE PIPELINES**

> **runMultiple = COMPOSE NOTEBOOKS + DAG**

> **%run = SHARED NAMESPACE**

> **FAIL = DELIBERATE FAILURE**

---

# 2. Parameters vs Variables

## Parameters

Parameters are values supplied when a pipeline run starts.

They are:

> **Immutable during the run**

Use parameters when the value represents an input that should remain fixed.

### Examples

- Source folder
- Environment name
- File path
- Date passed into the pipeline
- Processing configuration

### Memory

> **Parameter = INPUT**

---

## Variables

Variables can change during a pipeline run.

They can be modified using activities such as:

- Set Variable
- Append Variable

Use a variable when the value genuinely needs to change during execution.

### Example

```text
Start
  ↓
Variable = 0
  ↓
Activity
  ↓
Set Variable = 1
```

### Memory

> **Variable = STATE**

---

# 3. Parameter vs Variable — Exam Rule

| Requirement | Use |
|---|---|
| Fixed value for the run | Parameter |
| Input supplied at trigger time | Parameter |
| Value must change during execution | Variable |
| Set during the pipeline | Variable |
| Append during execution | Variable |

### Important Exam Tip

> **Parameters = immutable per run**

> **Variables = mutable during the run**

This distinction resolves many orchestration-pattern questions.

---

# 4. When to Use Variables

Do not automatically create variables for every value.

### Best Practice

> **Use a variable only when a value genuinely needs to change mid-run.**

Otherwise, use a parameter.

### Why?

Parameters keep pipeline inputs:

- Auditable
- Predictable
- Clear
- Easy to understand

### Memory

> **Does the value change?**

> YES → Variable

> NO → Parameter

---

# 5. Dynamic Expressions

Fabric pipelines use the `@` expression language for dynamic values.

Common examples include:

- System variables
- Activity outputs
- Pipeline parameters
- Functions

### Common Functions

```text
concat()
utcNow()
formatDateTime()
```

---

# 6. Important Expression Roots

Know these two expression roots very well:

## Activity Output

```text
@activity('name').output
```

Used to access the output of another activity.

---

## Pipeline Parameter

```text
@pipeline().parameters.x
```

Used to access a pipeline parameter.

### Memory

> **Activity result → `@activity('name').output`**

> **Pipeline input → `@pipeline().parameters.x`**

---

# 7. Bracket `[]` Syntax

Bracket syntax can be used for parameterized subfield access.

The key exam idea is:

> `[]` allows dynamic/parameterized subfield access.

Know that bracket notation may appear in dynamic expressions when a field/property needs to be accessed dynamically.

---

# 8. Keep Dynamic Expressions Readable

Avoid deeply nested expressions such as:

```text
@concat(
    if(...),
    if(...),
    concat(...),
    if(...)
)
```

when the expression becomes difficult to understand.

### Better Pattern

Use a **Set Variable** activity.

```text
Activity
   ↓
Set Variable
   ↓
Variable = clear meaningful name
```

### Best Practice

> Extract deeply nested `concat` / `if` chains into a Set Variable step with a clear name.

### Why?

It improves:

- Readability
- Debugging
- Maintenance
- Run-history troubleshooting

### Memory

> **Complex expression → Set Variable**

---

# 9. Invoke Pipeline

`Invoke Pipeline` is used to compose pipelines.

### Pattern

```text
Parent Pipeline
      ↓
Invoke Pipeline
      ↓
Child Pipeline
```

Use it when you want to break a larger workflow into reusable pipeline components.

### Memory

> **Invoke Pipeline = PIPELINE COMPOSITION**

---

# 10. Notebook Composition Options

There are two important notebook execution patterns:

1. `%run`
2. `notebookutils.notebook.run` / `runMultiple`

They are not the same.

---

# 11. `%run`

`%run` executes another notebook and brings its namespace into the current notebook.

### Key characteristics

- Shared namespace
- Same workspace only
- Maximum nesting depth = **5**
- No recursion

### Memory

> **%run = INLINE / SHARED NAMESPACE**

---

## Example

```python
%run SharedUtilities
```

The referenced notebook's definitions become available in the current notebook context.

---

# 12. `notebookutils.notebook.run`

This executes another notebook as a child execution.

It provides an **isolated child execution** model.

### Key characteristics

- Isolated child execution
- Cross-workspace capable
- Returns the child notebook's `exit()` value

### Memory

> **run() = ISOLATED CHILD NOTEBOOK**

---

# 13. `%run` vs `notebookutils.notebook.run`

| Feature | `%run` | `notebookutils.notebook.run` |
|---|---|---|
| Namespace | Shared | Isolated |
| Same workspace | Yes | Cross-workspace capable |
| Nesting depth | Max 5 | Not the `%run` depth model |
| Recursion | No | Different child-execution model |
| Return value | Not the main pattern | Child `exit()` value |
| Best use | Shared helper code | Child notebook execution |

### Exam Rule

> **%run = shared namespace / small helper code**

> **run() = isolated child execution**

---

# 14. When to Use `%run`

Reserve `%run` for genuinely shared, small helper code.

Example:

```text
Main Notebook
     |
     +---- %run CommonFunctions
```

Use it when the intent is to make helper definitions available in the current namespace.

Do not use `%run` as the main solution for complex dependency orchestration.

---

# 15. `notebookutils.notebook.runMultiple`

`runMultiple` is designed for executing multiple notebooks with dependency relationships.

It supports a **DAG**.

## DAG

DAG = Directed Acyclic Graph.

It represents:

- Dependencies
- Execution order
- Parallel opportunities

### Example

```text
        Notebook A
        /         \
       ↓           ↓
Notebook B      Notebook C
       \           /
        ↓         ↓
          Notebook D
```

Notebook B and C can run in parallel when their dependencies are satisfied.

---

# 16. When to Use `runMultiple`

Use `notebookutils.notebook.runMultiple` when there is:

- Real parallelism
- Dependency ordering
- Multiple notebooks
- Complex notebook execution relationships

### Best Practice

> **Use `runMultiple`'s DAG for real parallelism or dependency ordering.**

---

# 17. `runMultiple` Concurrency

`runMultiple` has a `concurrency` field.

### Important Exam Point

> **Concurrency is a ceiling, not a guarantee.**

Actual parallelism is also constrained by the available **Spark driver core count**.

### Example

If:

```text
concurrency = 10
```

that does not guarantee 10 notebooks will actually execute in parallel.

Actual parallelism can be lower depending on available driver resources.

### Memory

> **Concurrency = MAX, not GUARANTEE**

> **Driver cores can limit actual parallelism**

---

# 18. `%run` vs `runMultiple`

| Requirement | `%run` | `runMultiple` |
|---|---:|---:|
| Shared helper namespace | ✅ | ❌ |
| Small helper code | ✅ | ❌ |
| Dependency DAG | ❌ | ✅ |
| Real notebook parallelism | ❌ | ✅ |
| Dependency ordering | ❌ | ✅ |
| Same-workspace requirement | ✅ | Cross-workspace capable |
| Maximum nesting depth 5 | ✅ | Not the `%run` model |

### Exam Memory

> **%run → shared helpers**

> **runMultiple → DAG / dependencies / parallelism**

---

# 19. `exit()` in Notebooks

`exit()` is used to return an exit value from a Notebook.

Its behavior depends on how the Notebook is executed.

## Interactive Execution

Interactive execution keeps the session alive.

## Pipeline-Invoked Execution

When pipeline-invoked:

> The activity completes with the exit value.

### Important

The study material notes:

> **Do not wrap `exit()` in try/except.**

---

# 20. Referenced Notebook and `exit()`

When a notebook is referenced through `run()`:

> `exit()` stops the **referenced child notebook**.

The caller can access the returned exit value.

### Memory

> **run() → child executes → exit() → value returned to caller**

---

# 21. Activity Outcome Paths

Fabric pipeline activities can connect through four outcome paths:

1. **Upon Success**
2. **Upon Failure**
3. **Upon Completion**
4. **Upon Skip**

### Memory

> **S F C S**

- Success
- Failure
- Completion
- Skip

---

# 22. Upon Success

Runs the downstream activity when the previous activity succeeds.

```text
Activity A
    |
 Success
    ↓
Activity B
```

Use when B should run only after A succeeds.

---

# 23. Upon Failure

Runs the downstream activity when the previous activity fails.

```text
Activity A
    |
 Failure
    ↓
Notification / Recovery
```

Use for:

- Alerts
- Cleanup
- Recovery
- Error handling

---

# 24. Upon Completion

Runs after the upstream activity completes regardless of whether it succeeded or failed.

Conceptually:

```text
Activity A
    |
Completion
    ↓
Activity B
```

Use when the next action should happen after execution finishes regardless of outcome.

---

# 25. Upon Skip

Runs based on the upstream activity being skipped.

This is useful when the workflow needs to respond specifically to a skipped activity.

---

# 26. Multiple Activities on One Outcome Path

This is an important exam detail.

If multiple activities are connected to the same downstream condition:

> **ALL upstream activities must satisfy that condition before the downstream activity fires.**

### Example

```text
Activity A ── Success ──┐
                        ├── Activity D
Activity B ── Success ──┘
```

Activity D runs only when:

> A = Success **AND** B = Success

It is not an OR condition.

### Memory

> **Multiple paths = ALL must satisfy the selected condition**

---

# 27. Fail Activity

The **Fail** activity deliberately fails the pipeline.

It is useful when the pipeline detects a business or validation condition that should be treated as a failure.

### Example

```text
Validation
    ↓
Condition
    ↓
Invalid
    ↓
Fail Activity
```

---

# 28. Always Give Fail an `errorCode`

Best practice:

> Give every Fail activity a specific, greppable `errorCode`.

### Why?

A specific error code makes run-history troubleshooting much faster.

Instead of searching through large logs, operators can search for the known code.

### Example

```text
errorCode = CUSTOMER_DATA_INVALID
```

or:

```text
errorCode = SOURCE_FILE_MISSING
```

### Memory

> **Fail Activity → Specific errorCode**

---

# 29. Failure Notifications

Failure notifications can use activities such as:

- Teams
- Outlook

Route notifications from dedicated **Upon Failure** paths for critical activities.

### Recommended Pattern

```text
Critical Activity
      |
   Failure
      ↓
Teams / Outlook
```

---

# 30. Dedicated Failure Paths

Avoid one shared failure path across many activities unless the intended semantics are:

> **All must fail**

Why?

A dedicated failure path makes it clear which critical activity caused the notification.

### Example

```text
Copy Activity
     |
  Failure
     ↓
Teams Alert

Notebook
     |
  Failure
     ↓
Outlook Alert
```

This provides more precise failure handling.

---

# 31. Four Outcome Paths — Quick Reference

| Path | Fires when |
|---|---|
| Upon Success | Upstream succeeds |
| Upon Failure | Upstream fails |
| Upon Completion | Upstream completes |
| Upon Skip | Upstream is skipped |

### Important

When multiple upstream activities feed a downstream activity through the same outcome condition:

> **All must satisfy that condition.**

---

# 32. Complete Orchestration Pattern

A robust workflow may look like:

```text
Pipeline
   |
   +---- Parameter: Environment
   |
   +---- Variable: Runtime State
   |
   +---- Copy Data
   |        |
   |      Failure
   |        ↓
   |    Teams Alert
   |
   +---- Dataflow Gen2
   |        |
   |      Failure
   |        ↓
   |    Outlook Alert
   |
   +---- Notebook
            |
          Failure
            ↓
        Fail Activity
            |
       errorCode
            ↓
       Run History
```

For notebook-to-notebook orchestration:

```text
Parent Notebook
       |
       +---- runMultiple()
                |
                +---- Notebook A
                |
                +---- Notebook B
                |
                +---- Notebook C
                       |
                 Dependency DAG
```

---

# 33. Common Exam Traps

## Trap 1 — Parameter can change during a run

❌ Incorrect.

> Parameters are immutable per run.

Use a variable when the value must change.

---

## Trap 2 — Use variables for every input

❌ Not necessary.

If the value stays fixed, use a parameter.

---

## Trap 3 — Put huge nested expressions inline

❌ Avoid unreadable expressions.

Use a named Set Variable activity.

---

## Trap 4 — Use `%run` for complex orchestration

❌ Incorrect.

Use:

> `runMultiple` → DAG / dependencies / parallelism

---

## Trap 5 — `concurrency = 10` guarantees 10 parallel notebooks

❌ Incorrect.

Concurrency is a ceiling.

Actual parallelism is bounded by Spark driver cores.

---

## Trap 6 — `%run` and `run()` are identical

❌ Incorrect.

> `%run` = shared namespace

> `run()` = isolated child execution

---

## Trap 7 — `%run` supports unlimited nesting

❌ Incorrect.

Maximum nesting depth:

> **5**

No recursion.

---

## Trap 8 — `exit()` behaves identically everywhere

❌ Incorrect.

Its behavior differs between interactive and pipeline-invoked execution.

---

## Trap 9 — One shared failure path always means OR

❌ Incorrect.

Multiple activities connected through the same outcome condition must **all** satisfy that condition before the downstream activity fires.

---

## Trap 10 — Fail activity without a useful error code

❌ Avoid it.

Use a specific, greppable `errorCode`.

---

## Trap 11 — One notification path for everything

Not always appropriate.

For critical activities, use dedicated **Upon Failure** paths unless "all must fail" semantics are intended.

---

# 34. Exam Decision Table

| Scenario | Answer |
|---|---|
| Fixed run input | Parameter |
| Value changes during execution | Variable |
| Need activity output | `@activity('name').output` |
| Need pipeline parameter | `@pipeline().parameters.x` |
| Complex nested expression | Set Variable |
| Compose pipelines | Invoke Pipeline |
| Shared helper notebook code | `%run` |
| Notebook dependency DAG | `runMultiple` |
| Notebook parallelism | `runMultiple` |
| Isolated child notebook | `notebookutils.notebook.run` |
| Cross-workspace child execution | `run` / `runMultiple` |
| Custom deliberate pipeline failure | Fail activity |
| Fast failure triage | Specific `errorCode` |
| Notify on a specific critical activity failure | Dedicated Upon Failure path |
| Four activity outcomes | Success / Failure / Completion / Skip |

---

# 35. Key Takeaways

- Parameters are set once at trigger time and stay fixed.
- Variables mutate during the run using **Set Variable / Append Variable**.
- Use parameters for stable, auditable inputs.
- Use variables only when values genuinely need to change.
- The `@` expression language supports system variables, activity outputs, pipeline parameters, and functions such as `concat`, `utcNow`, and `formatDateTime`.
- Know these expression roots:
  - `@activity('name').output`
  - `@pipeline().parameters.x`
- Bracket `[]` syntax supports parameterized subfield access.
- Extract deeply nested expressions into a clearly named **Set Variable** activity.
- `Invoke Pipeline` composes pipelines.
- `notebookutils.notebook.run` and `runMultiple` compose notebooks in code.
- `runMultiple` supports a full dependency DAG.
- `runMultiple.concurrency` is a **ceiling**, not a guarantee.
- Actual notebook parallelism is bounded by Spark driver core count.
- `%run` provides a shared namespace.
- `%run` has a maximum nesting depth of **5** and no recursion.
- `%run` is same-workspace only.
- `notebookutils.notebook.run` provides isolated child execution and can be cross-workspace capable.
- `run()` returns the child's `exit()` value.
- `exit()` behavior differs between interactive and pipeline-invoked execution.
- Do not wrap `exit()` in try/except.
- Four outcome paths are:
  - Upon Success
  - Upon Failure
  - Upon Completion
  - Upon Skip
- Multiple activities on the same outcome path require **all** upstream conditions to be satisfied.
- Use **Fail** for deliberate custom failures.
- Give Fail activities specific, greppable `errorCode` values.
- Use dedicated Upon Failure paths for critical activity notifications.
- Teams and Outlook can be used for failure notifications.
- Use one shared failure path only when its semantics intentionally mean **all must fail**.

---

# 36. One-Minute Revision

```text
PARAMETER
   ↓
FIXED INPUT
IMMUTABLE PER RUN

VARIABLE
   ↓
CHANGING STATE
SET / APPEND VARIABLE
```

```text
ACTIVITY OUTPUT
   ↓
@activity('name').output

PIPELINE PARAMETER
   ↓
@pipeline().parameters.x
```

```text
COMPLEX EXPRESSION
   ↓
SET VARIABLE
   ↓
CLEAR NAME
   ↓
EASIER DEBUGGING
```

```text
PIPELINES
   ↓
Invoke Pipeline
   ↓
Child Pipeline
```

```text
NOTEBOOKS
   ↓
%run
   → Shared namespace
   → Same workspace
   → Max depth 5
   → No recursion

run()
   → Isolated child
   → Cross-workspace capable
   → Returns exit() value

runMultiple()
   → DAG
   → Dependencies
   → Parallelism
   → concurrency = ceiling
   → Driver cores limit actual parallelism
```

```text
OUTCOME PATHS
   ↓
SUCCESS
FAILURE
COMPLETION
SKIP
```

```text
FAILURE HANDLING
   ↓
Fail Activity
   ↓
Specific errorCode
   ↓
Fast Run-History Triage
```

```text
CRITICAL ACTIVITY
   ↓
Upon Failure
   ↓
Teams / Outlook
```

## Final Memory Formula

> **PARAMETER = FIXED**

> **VARIABLE = CHANGES**

> **INVOKE PIPELINE = COMPOSE PIPELINES**

> **%RUN = SHARED HELPERS**

> **RUN = ISOLATED CHILD**

> **RUNMULTIPLE = DAG + PARALLELISM**

> **CONCURRENCY = CEILING, NOT GUARANTEE**

> **FAIL = CUSTOM FAILURE**

> **ERRORCODE = FAST TRIAGE**

> **FAILURE PATH = TARGET THE CRITICAL ACTIVITY**

> **SUCCESS / FAILURE / COMPLETION / SKIP = FOUR OUTCOME PATHS**
