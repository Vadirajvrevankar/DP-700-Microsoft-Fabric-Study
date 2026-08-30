# Lesson 5 – Operations: CI/CD, Monitoring, Security & Administration

## 5.1 – Implement CI/CD

### 1. Why CI/CD Matters
- Use separate environments such as **Development, Test, and Production**.
- Promote changes in a controlled way instead of building directly in production.
- Helps improve collaboration, reliability, and release management.

**Remember:**
> **CI/CD → Develop → Test → Deploy safely**

### 2. Git Integration
- Fabric workspaces can be connected to supported Git providers such as **Azure DevOps/GitHub**.
- Provides version control for supported Fabric items.
- Supports collaboration and change tracking.
- Enables commit, branch, pull request, merge, and synchronization workflows depending on the integration.

**Remember:**
> **Git = Version control**

### 3. Deployment Pipelines
- Fabric Deployment Pipelines provide a way to promote content across environments.
- Common stages:
  - Development
  - Test
  - Production
- Provides comparison and deployment capabilities.

**Remember:**
> **Deployment Pipeline = Promote content across environments**

### 4. Git vs Deployment Pipelines

| Git | Deployment Pipelines |
|---|---|
| Version control | Environment promotion |
| Tracks changes | Moves content between stages |
| Branching | Development → Test → Production |
| Pull requests/review | Compare and deploy |

**Most important memory:**
> **Git = Track**

> **Deployment Pipeline = Promote**

### 5. Deployment Rules
- Deployment rules can adjust environment-specific settings during promotion where supported.
- Useful when Development, Test, and Production use different connections/settings.

Example:

```text
Development Connection
        ↓
      Deploy
        ↓
Test Connection
        ↓
      Deploy
        ↓
Production Connection
```

**Remember:**
> **Deployment Rule → Adjust environment-specific settings**

### 6. Items That Can Be Deployed
Supported Fabric items can include:
- Notebooks
- Pipelines
- Dataflows
- Lakehouses
- Warehouses
- Reports
- Semantic models

**Important:**
- Deployment support varies by item type.

### 7. Branching and Pull Requests
Typical development workflow:

```text
main
 ↓
feature branch
 ↓
Develop changes
 ↓
Pull Request
 ↓
Review
 ↓
Merge
```

**Remember:**
> **Branch → Develop → PR → Review → Merge**

### 8. Typical CI/CD Workflow

```text
Development Workspace
        ↓
      Git
        ↓
 Commit / PR / Merge
        ↓
Deployment Pipeline
        ↓
       Test
        ↓
       Prod
        ↓
Deployment Rules
```

---

# 5.2 – Monitor Activities in Microsoft Fabric

## 1. Monitoring Hub
- **Monitoring Hub** provides a Fabric-wide view of supported activity/job runs.
- Can monitor activities such as:
  - Pipelines
  - Dataflow refreshes
  - Notebook/Spark runs
  - Semantic model refreshes
  - Other supported Fabric jobs

Can show:
- Status
- Start/end time
- Duration
- Trigger information
- Errors

**Remember:**
> **Monitoring Hub → Did my Fabric jobs run successfully?**

### 2. Item-Specific Run History
- Provides detailed information about a particular run.
- Useful for:
  - Input/output inspection
  - Error investigation
  - Activity-level troubleshooting
  - Understanding execution details

### 3. Spark Application Monitoring
Useful for investigating Spark workloads.

Can provide information about:
- Jobs
- Stages
- Executors
- Duration
- Resource usage
- Logs

**Remember:**
> **Spark monitoring → Investigate Spark execution**

### 4. Failure Notifications
- Supported monitoring/alerting capabilities can notify users about failures.
- Notifications may include:
  - Email
  - Teams
  - Other supported integrations

### 5. Capacity-Level Monitoring
- The **Fabric Capacity Metrics app** provides capacity-level information.
- Helps monitor:
  - Capacity utilization
  - CU/resource consumption
  - Workload impact
  - Capacity pressure

**Remember:**
> **Capacity Metrics → Is my Fabric capacity healthy?**

### 6. Monitoring vs Troubleshooting

> **Monitoring → Detect the problem**

> **Troubleshooting → Find the cause**

Example:

```text
Pipeline Failed
      ↓
Monitoring Hub
      ↓
Find failed activity
      ↓
Open run details
      ↓
Read error
      ↓
Fix problem
```

### 7. Two Different Monitoring Questions

| Question | Tool |
|---|---|
| Did my process run? | **Monitoring Hub** |
| Why did my process fail? | **Run details / item history** |
| How is my Spark job performing? | **Spark monitoring** |
| Is my capacity healthy? | **Capacity Metrics app** |

---

# 5.3 – Secure Data Access in Microsoft Fabric

## 1. Layered Security Model

A useful way to understand Fabric security:

```text
Tenant
  ↓
Workspace
  ↓
Item
  ↓
Data / Object
  ↓
Rows / Columns
```

**Important:**
- **Domains** are primarily an organizational/governance concept rather than simply another security layer.

**Remember:**
> **Higher level → Broader access**

> **Lower level → More granular access**

### 2. Tenant-Level Settings
Fabric administrators can control organization-wide settings such as:
- Fabric features
- External sharing/access
- Trial settings
- Git integration
- Other tenant capabilities

**Remember:**
> **Tenant settings → Organization-wide controls**

### 3. Workspace Roles
Main workspace roles:
- **Admin**
- **Member**
- **Contributor**
- **Viewer**

These determine what users can do within the workspace.

### 4. Item-Level Permissions
Fabric items can have permissions controlling access.

Examples include:
- Read
- Read all data
- Build

**Important:**
- Workspace roles and item/data permissions are different layers of access control.
- Effective access depends on the permissions assigned to the user.

### 5. OneLake Data Access Roles
- OneLake security can provide more granular data access for supported Lakehouse scenarios.
- Access can be scoped to supported data resources such as folders/tables.

Conceptually:

```text
Lakehouse
    ↓
OneLake Security
    ↓
Folder / Table
    ↓
Specific Users / Groups
```

**Remember:**
> **OneLake Security → Fine-grained data access**

### 6. Row-Level Security (RLS)
- RLS restricts which **rows** a user can see.
- Different users can see different rows from the same data.

**Remember:**
> **RLS = Which rows can I see?**

### 7. Column-Level Security (CLS)
- CLS restricts access to specific **columns**.

**Remember:**
> **CLS = Which columns can I see?**

### 8. RLS vs CLS

| Security | Controls |
|---|---|
| **RLS** | Rows |
| **CLS** | Columns |

### 9. Sensitivity Labels
- Sensitivity labels help **classify and protect** sensitive information.
- Fabric integrates with Microsoft Purview capabilities.
- Labels can persist through supported export scenarios.

Examples:
- Confidential
- Highly Confidential

**Remember:**
> **Sensitivity Label → Classify + Protect**

### 10. Endorsement
Fabric supports endorsement concepts such as:
- **Promoted**
- **Certified**

#### Promoted
- Recommended by the content owner/creator.

#### Certified
- Formally trusted/approved according to organizational processes.

**Remember:**
> **Endorsement → Help users find trusted content**

### 11. Data Lineage
- Lineage shows relationships and dependencies between Fabric items.
- Useful for:
  - Impact analysis
  - Troubleshooting
  - Governance
  - Understanding data flow

Example:

```text
Source
  ↓
Dataflow
  ↓
Lakehouse
  ↓
Warehouse
  ↓
Semantic Model
  ↓
Report
```

**Remember:**
> **Lineage → Where did the data come from and where does it go?**

### 12. Fabric-Wide Security vs Warehouse Security

#### Lesson 4.7 – Warehouse Security
Focuses mainly on:
- T-SQL permissions
- Object-level security
- RLS
- CLS
- DDM

#### Lesson 5.3 – Fabric-Wide Security
Focuses mainly on:
- Tenant settings
- Workspace permissions
- OneLake security
- Sensitivity labels
- Endorsement
- Lineage
- Governance

**Remember:**
> **Lesson 4.7 → Warehouse/data-level security**

> **Lesson 5.3 → Fabric-wide access + governance**

---

# 5.4 – Administer the Microsoft Fabric Environment

## 1. Fabric Administrator
- Fabric administrators manage **tenant-wide Fabric settings**.
- Usually handled by organization/platform/IT administrators.

**Remember:**
> **Fabric Admin → Tenant-wide administration**

### 2. Fabric Admin Portal
The Admin Portal contains administration areas such as:

#### Tenant Settings
- Control organization-wide Fabric features and capabilities.
- Examples:
  - Fabric features
  - Trial settings
  - External sharing
  - Git
  - Other tenant settings

#### Capacity Settings
Used to manage Fabric capacity where supported.

Activities can include:
- Create capacity
- Configure capacity
- Assign workspaces
- Scale/resize capacity
- Pause/resume supported capacity

#### Domains
- Organize workspaces by business area.
- Support delegated governance/administration where applicable.

#### Audit
- Helps track tenant-level activity for governance, investigation, and compliance scenarios.

### 3. Capacity Management
Important concepts:
- Assign workspaces to capacity.
- Monitor capacity usage.
- Scale capacity when required.
- Understand workload consumption.
- Pause/resume supported capacities where applicable.

Example:

```text
Fabric Capacity
      ↓
 ┌────┼──────┐
 ↓    ↓      ↓
DW  Spark  Dataflow
```

**Remember:**
> **Capacity = Shared compute resource**

### 4. Domains and Delegated Administration
- Domains organize workspaces according to business areas.
- Can support delegated governance/administration.

Example:

```text
Fabric
  ↓
Sales Domain
  ├── Sales Workspace
  └── CRM Workspace

Finance Domain
  ├── Finance Workspace
  └── Reporting Workspace
```

**Remember:**
> **Domain → Organize + Delegate**

### 5. Admin APIs / PowerShell
- Supported Fabric APIs and PowerShell can automate administrative tasks.
- Useful for:
  - Auditing
  - Capacity management
  - Workspace administration
  - Tenant administration
  - Large-scale operations

**Remember:**
> **APIs / PowerShell → Automate administration**

---

# Lesson 5 – Final Revision

| Topic | Main Idea |
|---|---|
| **5.1 CI/CD** | Git + Deployment Pipelines |
| **Git** | Version control |
| **Pull Request** | Review changes |
| **Deployment Pipeline** | Promote content |
| **Deployment Rule** | Adjust environment-specific settings |
| **5.2 Monitoring** | Monitor Fabric activity and jobs |
| **Monitoring Hub** | Monitor Fabric runs |
| **Capacity Metrics** | Monitor capacity |
| **Spark Monitoring** | Investigate Spark execution |
| **5.3 Security** | Fabric-wide access and governance |
| **OneLake Security** | Fine-grained data access |
| **RLS** | Restrict rows |
| **CLS** | Restrict columns |
| **Sensitivity Labels** | Classify/protect data |
| **Endorsement** | Identify trusted content |
| **Lineage** | Track data dependencies |
| **5.4 Administration** | Manage Fabric environment |
| **Fabric Admin** | Tenant-wide administration |
| **Capacity** | Shared compute |
| **Domains** | Organize/delegate governance |
| **Audit** | Track activity |
| **APIs/PowerShell** | Automate administration |

# 🧠 Lesson 5 in One Picture

```text
                  FABRIC OPERATIONS
                         ↓
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
      CI/CD          MONITORING         SECURITY
       ↓                 ↓                 ↓
     Git          Monitoring Hub     Workspace
       ↓                 ↓            OneLake
Deployment        Capacity Metrics    RLS/CLS
 Pipelines              ↓             Labels
       ↓             Health          Lineage
Dev → Test → Prod
                         ↓
                    ADMINISTRATION
                         ↓
                  Tenant + Capacity
                  Domains + Audit
                  APIs / PowerShell
```

# 🔑 Most Important Exam Memory

1. **Git → Version control**
2. **Pull Request → Review changes**
3. **Deployment Pipeline → Promote content**
4. **Deployment Rule → Environment-specific settings**
5. **Monitoring Hub → Monitor Fabric runs**
6. **Capacity Metrics → Monitor capacity**
7. **OneLake Security → Fine-grained data access**
8. **RLS → Rows**
9. **CLS → Columns**
10. **Sensitivity Label → Classify + Protect**
11. **Endorsement → Trusted content**
12. **Lineage → Data dependencies**
13. **Fabric Admin → Tenant-wide administration**
14. **Capacity → Shared compute**
15. **Domain → Organize + Delegate**
16. **APIs/PowerShell → Automate administration**

**Lesson 5 – Operations ✅ COMPLETED**
