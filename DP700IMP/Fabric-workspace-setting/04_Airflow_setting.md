# Apache Airflow Workspace Settings — DP-700

## 1. Starter Pool vs Custom Pool

### CONCEPT
Airflow needs **compute resources** to run DAGs.

```text
Airflow Compute
      │
      ├── Starter Pool
      │
      └── Custom Pool
```

### WHAT IT DOES
Lets you choose the compute model for an Apache Airflow job.

**Starter Pool**
- Development/testing
- Large compute node
- Starts quickly
- Resumes within about 5 minutes
- Shuts down after 20 minutes of inactivity

**Custom Pool**
- Production workloads
- Configurable node size
- Extra nodes
- Autoscaling
- Starts stopped and can remain running until manually paused

### KEYWORDS
`Starter Pool` · `Custom Pool` · `Development` · `Production` · `Node Size` · `Autoscale` · `Extra Nodes`

### TYPICAL SCENARIO
Developer testing a DAG → **Starter Pool**  
Production DAG needing scalable compute → **Custom Pool + autoscale**

### MEMORY TRICK
**STARTER = DEV 🧪**  
**CUSTOM = PROD 🏭**

---

## 2. Compute Node Size

### CONCEPT
**Compute Node Size**

### WHAT IT DOES
Controls the compute capacity available to the Airflow workload.

### KEYWORDS
`Node Size` · `Compute` · `Capacity` · `Performance`

### TYPICAL SCENARIO
Production workload needs more compute → configure an appropriate node size in a Custom Pool.

### MEMORY TRICK
**NODE SIZE = HOW POWERFUL IS THE MACHINE?**

---

## 3. Autoscale

### CONCEPT
**Autoscale**

### WHAT IT DOES
Allows a Custom Pool to adjust compute capacity according to workload demand.

### KEYWORDS
`Autoscale` · `Scale` · `Workload` · `Dynamic Capacity`

### TYPICAL SCENARIO
Workload varies and needs more/fewer workers automatically → **Custom Pool + autoscale**

### MEMORY TRICK
**AUTOSCALE = COMPUTE CHANGES WITH DEMAND 📈📉**

---

## 4. Extra Nodes

### CONCEPT
**Extra Nodes**

### WHAT IT DOES
Adds additional compute capacity/workers to a Custom Pool.

### KEYWORDS
`Extra Nodes` · `Workers` · `Custom Pool`

### TYPICAL SCENARIO
Production workload needs additional worker capacity → configure **extra nodes**.

### MEMORY TRICK
**EXTRA NODES = EXTRA WORKERS**

---

## 5. Airflow Runtime

### CONCEPT
**Apache Airflow Runtime**

### WHAT IT DOES
Fabric provides a managed Airflow environment for Python-based DAGs.

Current study-material runtime:
- Apache Airflow **2.10.5**
- Python **3.12**

### KEYWORDS
`Airflow Runtime` · `Airflow Version` · `Python Version` · `Managed Airflow`

### TYPICAL SCENARIO
Run Python-based Airflow DAGs without managing the underlying Airflow infrastructure → **Fabric Apache Airflow Job**

### MEMORY TRICK
**AIRFLOW JOB = MANAGED AIRFLOW FOR PYTHON DAGs**

---

## 6. Airflow Version Planning

### CONCEPT
**Airflow Version Planning**

### WHAT IT DOES
Requires planning the expected Airflow version early. An existing Airflow job cannot simply be upgraded in place to another version.

### KEYWORDS
`Airflow Version` · `Upgrade` · `Migration` · `New Job`

### TYPICAL SCENARIO
Need a different Airflow version → **create a new Airflow Job and migrate**.

```text
Existing Job
    ↓
New Airflow Job
    ↓
Migrate DAGs / Configuration
```

### MEMORY TRICK
**VERSION CHANGE = NEW JOB + MIGRATION**

---

## 7. requirements / Python Dependencies

### CONCEPT
**Airflow Requirements**

### WHAT IT DOES
Defines Python packages required by the Airflow workload.

Example:

```text
pandas==2.2.3
requests==2.32.3
```

Keep dependencies **minimal** and **version-pinned** to reduce dependency conflicts and environment drift.

### KEYWORDS
`requirements` · `Python Packages` · `Dependencies` · `Version Pinning` · `Environment Drift`

### TYPICAL SCENARIO
Keep the Airflow environment consistent across restarts → **minimal, version-pinned dependencies**

### MEMORY TRICK
**REQUIREMENTS = WHAT PACKAGES DO I NEED?**  
**PIN = KEEP THE VERSION CONSISTENT**

---

## 8. DAG Folder

### CONCEPT
**`dags` Folder**

### WHAT IT DOES
Contains Airflow DAG files.

```text
dags/
   sales_pipeline.py
   customer_pipeline.py
```

### KEYWORDS
`dags` · `DAG` · `Workflow` · `Python`

### TYPICAL SCENARIO
Where should an Airflow DAG Python file go? → **`dags`**

### MEMORY TRICK
**DAG → dags**

---

## 9. Plugins

### CONCEPT
**`plugins` Folder**

### WHAT IT DOES
Used for custom/private code and packages.

```text
plugins/
   custom_operator.py
   helper_functions.py
```

For no-Git custom libraries:

```text
plugins/libs
```

### KEYWORDS
`plugins` · `Custom Code` · `Private Packages` · `plugins/libs`

### TYPICAL SCENARIO
Need custom/private Python code → **plugins**  
No Git + custom libraries → **plugins/libs**

### MEMORY TRICK
**DAG → dags**  
**CUSTOM CODE → plugins**  
**NO GIT + LIBRARIES → plugins/libs**

---

## 10. Requirements Change with plugins/libs

### CONCEPT
**Restart after dependency changes**

### WHAT IT DOES
When requirements are uploaded/changed through the `plugins/libs` approach, explicitly restart the Airflow job so the updated dependencies are used.

### KEYWORDS
`plugins/libs` · `requirements` · `Restart` · `Dependencies`

### TYPICAL SCENARIO
Requirements changed in `plugins/libs` → **restart the Airflow job**

### MEMORY TRICK
**plugins/libs CHANGE → RESTART 🔄**

---

## 11. Git Synchronization

### CONCEPT
**Git-Synchronized Airflow Code**

### WHAT IT DOES
Allows Airflow project code to be synchronized from Git.

A Git-sync code change does not require a manual restart for the Git-sync change itself.

### KEYWORDS
`Git` · `Git Sync` · `Repository` · `Code Synchronization`

### TYPICAL SCENARIO
DAG/code changes are synchronized through Git → **no manual restart for the Git-sync change itself**

### MEMORY TRICK
**Git Sync → NO MANUAL RESTART**

---

## 12. Apache Airflow Job

### CONCEPT
**Apache Airflow Job**

### WHAT IT DOES
Provides a managed Apache Airflow environment in Fabric for Python-based DAG workflows.

```text
Python DAG
    ↓
Airflow Job
    ↓
Tasks execute
    ↓
Workflow completes
```

### KEYWORDS
`Apache Airflow` · `DAG` · `Python` · `Orchestration` · `Managed Service`

### TYPICAL SCENARIO
Existing Python-based Airflow DAG ecosystem → **Apache Airflow Job**

### MEMORY TRICK
**AIRFLOW = PYTHON DAG ORCHESTRATION**

---

## 13. Airflow Job vs Fabric Pipeline

### CONCEPT
**Choosing the Orchestration Tool**

### WHAT IT DOES
Helps choose orchestration based on the workload.

**Airflow:** Python, Airflow DAGs, operators, existing Airflow workflows.  
**Fabric Pipeline:** Fabric-native pipeline orchestration.

### TYPICAL SCENARIO
Existing Python Airflow DAG ecosystem → **Apache Airflow Job**

### MEMORY TRICK
**PYTHON DAG → AIRFLOW**  
**FABRIC-NATIVE → FABRIC PIPELINE**

---

## 14. Regional Availability

### CONCEPT
**Airflow Regional Availability**

### WHAT IT DOES
Requires checking that Airflow is supported in the target region before provisioning an Airflow-heavy workload. Some regions may show **Coming soon**.

### KEYWORDS
`Region` · `Availability` · `Coming Soon` · `Capacity` · `Deployment`

### TYPICAL SCENARIO
Need Airflow in a specific region → **check regional availability first**

### MEMORY TRICK
**AIRFLOW + REGION → CHECK AVAILABILITY FIRST 🌍**

---

## 15. Free and PPU Workspace Limitation

### CONCEPT
**Workspace Support Limitation**

### WHAT IT DOES
Apache Airflow Jobs are not supported in Free or PPU workspaces.

```text
Free Workspace → ❌ Airflow
PPU Workspace  → ❌ Airflow
Supported Capacity → ✅ Airflow
```

### KEYWORDS
`Free` · `PPU` · `Unsupported` · `Airflow Job`

### TYPICAL SCENARIO
Create Airflow Job in Free/PPU workspace → **Not supported**

### MEMORY TRICK
**FREE + PPU = NO AIRFLOW ❌**

---

## 16. Private Networks / VNet Limitation

### CONCEPT
**Private Network Support**

### WHAT IT DOES
Private networks / VNets are currently not supported for Apache Airflow Jobs.

### KEYWORDS
`Private Network` · `VNet` · `Network` · `Unsupported`

### TYPICAL SCENARIO
Requirement says Airflow must run inside a private VNet → **currently unsupported**

### MEMORY TRICK
**AIRFLOW + PRIVATE VNET = NOT SUPPORTED ❌**

---

## 17. Production Cutover

### CONCEPT
**Moving Airflow from Development to Production**

### WHAT IT DOES

```text
Development
     ↓
Starter Pool
     ↓
Test DAG
     ↓
Validate dependencies
     ↓
Production
     ↓
Custom Pool
     ↓
Large node + Autoscale
```

### KEYWORDS
`Development` · `Production` · `Starter Pool` · `Custom Pool` · `Autoscale`

### TYPICAL SCENARIO
DAG tested successfully and is ready for production → move to an appropriately configured **Custom Pool**.

### MEMORY TRICK
**DEV → STARTER**  
**PROD → CUSTOM + SCALE**

---

# Complete Airflow Folder Structure

```text
Airflow Job
│
├── dags/
│     └── sales_pipeline.py
│
├── plugins/
│     ├── custom_operator.py
│     └── helper.py
│
├── plugins/libs/
│     └── private_package
│
└── requirements
      ├── pandas==...
      └── requests==...
```

Remember:

```text
DAG files        → dags
Custom code      → plugins
No-Git libraries → plugins/libs
Dependencies     → requirements
```

---

# DP-700 Airflow Exam Cheat Sheet

| Scenario | Answer |
|---|---|
| Development/testing | **Starter Pool** |
| Production workload | **Custom Pool** |
| Need scalable compute | **Custom Pool + Autoscale** |
| Need more workers | **Extra Nodes** |
| Need compute capacity | **Node Size** |
| Python dependencies | **Requirements** |
| Avoid dependency drift | **Version-pin dependencies** |
| DAG files | **`dags`** |
| Custom/private code | **`plugins`** |
| No-Git custom libraries | **`plugins/libs`** |
| Change requirements through `plugins/libs` | **Restart job** |
| Git-sync code change | **No manual restart for the Git-sync change itself** |
| Need a different Airflow version | **Create new job + migrate** |
| Deploy in a region | **Check regional availability** |
| Free workspace | **Not supported** |
| PPU workspace | **Not supported** |
| Private VNet | **Not supported** |
| Python-based DAG orchestration | **Apache Airflow Job** |

---

# Ultimate Memory Trick

```text
DEV              → STARTER
PROD             → CUSTOM
MORE CAPACITY    → EXTRA NODES
DYNAMIC CAPACITY → AUTOSCALE

DAG              → dags
CUSTOM CODE      → plugins
NO-GIT LIBRARIES → plugins/libs
DEPENDENCIES     → requirements
VERSION PIN      → AVOID DRIFT

plugins/libs CHANGE → RESTART
Git SYNC CHANGE     → NO MANUAL RESTART

VERSION CHANGE   → NEW JOB + MIGRATION
REGION           → CHECK AVAILABILITY

FREE / PPU       → NO AIRFLOW
PRIVATE VNET     → NO AIRFLOW
```

# What to Master for DP-700

1. **Starter vs Custom Pool**
2. **Node size / Autoscale / Extra nodes**
3. **`dags` vs `plugins` vs `plugins/libs`**
4. **Requirements + version pinning**
5. **Restart behavior**
6. **Airflow version migration limitation**
7. **Free/PPU/VNet limitations**
8. **Regional availability**

You do **not** need to become an Apache Airflow expert. For DP-700, the goal is to recognize the scenario and choose the correct setting.
