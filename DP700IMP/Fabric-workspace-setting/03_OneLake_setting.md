# Microsoft Fabric OneLake Settings — DP-700 Best Notes

## 1. OneLake

### CONCEPT
**Microsoft OneLake**

### WHAT IT DOES
OneLake is the **single, unified data lake for the entire Fabric tenant**.

Think:
> **One organization → OneLake → Many workspaces**

### KEYWORDS
`OneLake` · `Tenant-wide` · `Unified data lake` · `Workspace` · `Lakehouse`

### TYPICAL SCENARIO
A company has Sales, Finance, and HR workspaces. They don't need separate OneLake instances.

### ANSWER
> OneLake provides a single, unified data lake across the Fabric tenant.

### MEMORY TRICK
**ONE TENANT → ONE LAKE**

---

## 2. OneLake Workspace Settings

### CONCEPT
**Three Important Workspace-Level OneLake Settings**

### WHAT IT DOES
Controls three distinct OneLake behaviors:
1. External app access
2. Shortcut caching
3. OneLake diagnostics

### KEYWORDS
`External app access` · `Shortcut caching` · `Diagnostics`

### TYPICAL SCENARIO
During workspace provisioning, an administrator reviews external access, caching, and diagnostics requirements together.

### ANSWER
> Treat external app access, shortcut caching, and OneLake diagnostics as separate workspace-level settings.

### MEMORY TRICK
**ACCESS → CACHE → DIAGNOSE**

---

## 3. External App Access

### CONCEPT
**OneLake External App Access**

### WHAT IT DOES
Controls whether **external applications** can access the workspace's OneLake data.

### KEYWORDS
`External application` · `OneLake` · `Access` · `Security group`

### TYPICAL SCENARIO
An external application needs to access data in a Fabric workspace, but only a specific team should have this capability.

### ANSWER
> Enable external app access only when required and scope it to appropriate security groups.

### MEMORY TRICK
**External App Access = WHO can access OneLake externally**

⚠️ **Exam distinction:** External app access and OneLake File Explorer availability are different controls.

---

## 4. Security Group Scope

### CONCEPT
**Restrict External App Access to Security Groups**

### WHAT IT DOES
Limits external application access to specific security groups instead of the whole organization.

### KEYWORDS
`Security group` · `Least privilege` · `External access` · `Scope`

### TYPICAL SCENARIO
Only the Data Engineering team needs an external tool to access OneLake.

### ANSWER
> Scope external app access to specific security groups whenever possible.

### MEMORY TRICK
**Don't open ALL → Open WHO NEEDS IT**

---

## 5. Shortcut Caching

### CONCEPT
**OneLake Shortcut Cache**

### WHAT IT DOES
Caches data accessed through eligible OneLake shortcuts so repeated reads can be faster.

### KEYWORDS
`Shortcut` · `Cache` · `Performance` · `Remote source` · `Repeated reads`

### TYPICAL SCENARIO
A Lakehouse has a shortcut to Amazon S3 and a workload repeatedly reads the same files.

### ANSWER
> Enable shortcut caching when workloads repeatedly read eligible external shortcut sources.

### MEMORY TRICK
**REPEATED REMOTE READ → CACHE**

---

## 6. Eligible Shortcut Sources

### CONCEPT
**Shortcut Cache Source Restrictions**

### WHAT IT DOES
Shortcut caching applies to specific source types.

Eligible sources:
- **Google Cloud Storage (GCS)**
- **Amazon S3**
- **S3-compatible storage**
- **On-premises data gateway (OPDG)**

### KEYWORDS
`GCS` · `S3` · `S3-compatible` · `OPDG`

### TYPICAL SCENARIO
A shortcut points to S3 and is read repeatedly → caching can be useful.

A shortcut points to ADLS/Blob → don't select shortcut caching as the solution.

### ANSWER
> Shortcut caching is intended for eligible sources such as GCS, S3, S3-compatible storage, and OPDG sources.

### MEMORY TRICK
**G S S O**

> **G**CS · **S**3 · **S**3-compatible · **O**PDG

🔥 **DP-700 TRAP:** ADLS / Blob → **No shortcut cache value**

---

## 7. Cache Retention

### CONCEPT
**Shortcut Cache Retention**

### WHAT IT DOES
Controls how long cached files can remain available.

- Retention: **1–28 days**
- Files **larger than 1 GB are not cached**

### KEYWORDS
`1–28 days` · `1 GB` · `Cache retention` · `File size`

### TYPICAL SCENARIO
A 7-day cache retention is valid. A 2 GB file won't be cached.

### ANSWER
> Shortcut cache retention can be configured from 1 to 28 days, and files larger than 1 GB aren't cached.

### MEMORY TRICK
**CACHE = 1–28 DAYS + MAX 1 GB**

---

## 8. Cache Countdown Reset

### CONCEPT
**Cache Retention Reset on Access**

### WHAT IT DOES
When a cached file is accessed, the retention countdown is reset.

### KEYWORDS
`Access` · `Countdown` · `Retention` · `Reset`

### TYPICAL SCENARIO
A cached file has a 7-day retention period and users continue accessing it.

### ANSWER
> Accessing a cached file resets its retention countdown.

### MEMORY TRICK
**ACCESS → TIMER RESTARTS**

---

## 9. Reset Cache

### CONCEPT
**Reset Shortcut Cache**

### WHAT IT DOES
Removes cached files for the **workspace**.

It is not limited to one shortcut.

### KEYWORDS
`Reset cache` · `Workspace-wide` · `Cached files` · `All shortcuts`

### TYPICAL SCENARIO
An administrator wants to clear stale cached data from a workspace.

### ANSWER
> Treat Reset Cache as a deliberate workspace-wide operation because it removes cached files across the workspace.

### MEMORY TRICK
**RESET CACHE = RESET WORKSPACE CACHE**

🔥 **Exam trap:** It is not simply a reset for the currently selected shortcut.

---

## 10. OneLake Diagnostics

### CONCEPT
**OneLake Diagnostics**

### WHAT IT DOES
Provides diagnostic information useful for monitoring, troubleshooting, and compliance-related requirements.

### KEYWORDS
`Diagnostics` · `Troubleshooting` · `Monitoring` · `Compliance`

### TYPICAL SCENARIO
An organization has a compliance requirement that requires OneLake diagnostic information.

### ANSWER
> Enable OneLake diagnostics proactively when monitoring, troubleshooting, or compliance requirements require diagnostic information.

### MEMORY TRICK
**DIAGNOSTICS = TROUBLESHOOT + EVIDENCE**

---

## 11. OneLake File Explorer

### CONCEPT
**OneLake File Explorer**

### WHAT IT DOES
Allows users to interact with OneLake data through Windows File Explorer.

It uses **placeholders/metadata** rather than downloading the entire lake locally.

### KEYWORDS
`OneLake File Explorer` · `Windows` · `Placeholders` · `Metadata`

### TYPICAL SCENARIO
A user sees OneLake files in Windows File Explorer, but the complete files are not necessarily downloaded to the computer.

### ANSWER
> OneLake File Explorer uses local placeholders representing OneLake files rather than downloading the entire OneLake dataset locally.

### MEMORY TRICK
**FILE EXPLORER = PLACEHOLDERS, NOT FULL COPY**

---

## 12. OneLake File Explorer Sync

### CONCEPT
**OneLake File Explorer Synchronization**

### WHAT IT DOES
Handles synchronization between local changes and OneLake.

- Local changes → can be automatically pushed to OneLake
- Remote changes → require **Sync from OneLake**

### KEYWORDS
`Sync from OneLake` · `Local changes` · `Remote changes` · `Placeholder`

### TYPICAL SCENARIO
A remote user changes a file in OneLake. The local File Explorer view needs **Sync from OneLake** to pull the remote change.

### ANSWER
> OneLake File Explorer uses placeholders. Local changes can be pushed automatically, while pulling remote changes requires manual **Sync from OneLake**.

### MEMORY TRICK
**LOCAL → PUSH AUTOMATIC**  
**REMOTE → SYNC MANUALLY**

---

## 13. File Explorer Tenant-Level Control

### CONCEPT
**OneLake File Explorer Availability**

### WHAT IT DOES
A tenant-level setting can disable OneLake File Explorer for users.

### KEYWORDS
`Tenant setting` · `File Explorer` · `Disable` · `External app access`

### TYPICAL SCENARIO
An organization decides that users should no longer use OneLake File Explorer.

### ANSWER
> OneLake File Explorer availability is controlled separately at the tenant level and should not be confused with workspace-level external app access.

### MEMORY TRICK
**FILE EXPLORER = TENANT CONTROL**

---

## 14. Regional OneLake Endpoint

### CONCEPT
**Regional OneLake Endpoint**

### WHAT IT DOES
Provides a regional endpoint for tooling when geographic or data-residency requirements matter.

### KEYWORDS
`Regional endpoint` · `Data residency` · `Geography` · `Compliance`

### TYPICAL SCENARIO
A company has data-residency requirements and its tooling must connect through the appropriate regional OneLake endpoint.

### ANSWER
> Use the regional OneLake endpoint when tooling has data-residency or regional access requirements.

### MEMORY TRICK
**DATA RESIDENCY → REGIONAL ENDPOINT**

---

# DP-700 SCENARIO CHEAT SHEET

| Scenario says... | Think... |
|---|---|
| Unified Fabric data lake | **OneLake** |
| External application needs access | **External app access** |
| Restrict external apps to a team | **Security group** |
| Repeated reads from S3/GCS | **Shortcut caching** |
| Shortcut to ADLS/Blob | **No shortcut cache value** |
| Eligible cache sources | **GCS + S3 + S3-compatible + OPDG** |
| Cache retention | **1–28 days** |
| File > 1 GB | **Never cache** |
| File accessed again | **Countdown resets** |
| Clear cached data | **Reset Cache** |
| Reset cache scope | **Workspace-wide** |
| Troubleshooting/compliance evidence | **OneLake diagnostics** |
| Windows access to OneLake | **File Explorer** |
| File Explorer behavior | **Placeholders, not full copy** |
| Pull remote changes | **Sync from OneLake** |
| Disable File Explorer | **Tenant-level setting** |
| Data residency requirement | **Regional endpoint** |

# FINAL MASTER MEMORY TRICK

> **ONELAKE = ONE TENANT-WIDE LAKE**
>
> **EXTERNAL ACCESS → WHO CAN ACCESS**
>
> **CACHE → FASTER REPEATED REMOTE READS**
>
> **GCS + S3 + S3-Compatible + OPDG → CACHE ELIGIBLE**
>
> **1–28 DAYS + ≤1 GB → CACHE LIMITS**
>
> **RESET CACHE → WHOLE WORKSPACE**
>
> **DIAGNOSTICS → TROUBLESHOOT / COMPLIANCE**
>
> **FILE EXPLORER → PLACEHOLDERS**
>
> **REMOTE CHANGE → SYNC FROM ONELAKE**
>
> **DATA RESIDENCY → REGIONAL ENDPOINT**

# ⭐ Most Important DP-700 Traps

### Trap 1
**External app access ≠ File Explorer availability**

- External app access → workspace-level OneLake setting
- File Explorer availability → tenant-level control

### Trap 2
**Shortcut caching ≠ all shortcut sources**

Remember:

> **GCS + S3 + S3-compatible + OPDG**

Not ADLS/Blob.

### Trap 3
**Domain ≠ OneLake**

Domain organizes/governs workspaces.

OneLake is the unified tenant-wide data lake.

### Trap 4
**File Explorer ≠ full local copy**

It uses **placeholders/metadata**.

### Trap 5
**Reset Cache ≠ one shortcut**

It is a **workspace-wide cache reset**.

## ⭐ One-line Exam Answer

> **“OneLake settings control how a workspace interacts with the unified tenant-wide lake—external access controls access, shortcut caching improves eligible remote reads, and diagnostics supports monitoring and troubleshooting.”**
