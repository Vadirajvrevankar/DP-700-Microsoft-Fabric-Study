# DP-700 — OneLake Shortcuts: Explanation and Exam Notes

## 1. What Is a Shortcut?

A **shortcut** in OneLake is a pointer to data stored somewhere else. It behaves somewhat like a symbolic link: Fabric can access the referenced data without creating another copy.

**Why it matters:** If the source is already in a query-compatible format, a shortcut can reduce unnecessary cross-workspace or cross-cloud data copies.

A shortcut does **not** transform data. If you need transformation, format conversion, or a decoupled snapshot, consider a pipeline or mirroring instead.

## 2. When Should You Use a Shortcut?

Use a shortcut when:
- Data already exists in a query-compatible format.
- You want to reference data across workspaces or clouds without copying it.
- You want to avoid maintaining a duplicate copy.
- The calling user has the required permission on the target.

### Example
A team has query-compatible data in another workspace. Rather than building a pipeline to copy it into a second workspace, it can create a shortcut to reference the existing data—provided the target permissions allow access.

## 3. Internal vs External Shortcuts

### Internal shortcuts
Internal shortcuts reference other Fabric items.

Targets listed in these study notes:
- KQL database
- Lakehouse
- Mirrored Azure Databricks catalog
- Mirrored database
- Semantic model
- SQL database
- Warehouse

### External shortcuts
External shortcuts reference supported sources outside the relevant Fabric item/workspace.

Targets listed in these study notes:
- Azure Data Lake Storage Gen2 (ADLS Gen2)
- Azure Blob Storage
- Amazon S3
- S3-compatible storage
- Google Cloud Storage (GCS)
- Dataverse
- Iceberg
- OneDrive / SharePoint
- On-premises sources through a gateway

## 4. Shortcut Caching

Caching can improve repeated reads for supported shortcut sources, but **not every source supports caching**.

### Sources that support caching in these notes
- Google Cloud Storage (GCS)
- Amazon S3
- S3-compatible storage
- On-premises sources through a gateway

### Cache rules
- Retention window: **1–28 days**.
- Files larger than **1 GB** are not cached.
- Choose retention based on the read pattern:
  - **Longer retention** for stable reference data that is read repeatedly.
  - **Shorter retention** for frequently changing sources.

**Exam trap:** Do not assume every external shortcut supports caching. The supported list is limited to the sources above in these notes.

## 5. Permissions and Identity

A shortcut is only a pointer; it does not automatically grant access to the underlying data.

### Standard internal shortcut behavior
- Authorization checks use the **calling user's identity** against the target.
- The user needs actual permission on the target.
- Audit target permissions, not just who can see or access the shortcut.

### Delegated identity exception
For **Direct Lake over SQL / Delegated identity mode**, the calling item's **owner identity** is delegated instead.

**Exam memory aid:** Shortcut visible ≠ target access granted. Check permissions at the target.

## 6. Shortcut vs Pipeline vs Mirroring

| Option | Best fit |
|---|---|
| **Shortcut** | Reference already-query-compatible data without copying it |
| **Pipeline** | Copy, transform, or convert data; orchestrate data movement |
| **Mirroring** | Keep supported source data continuously replicated or available as a decoupled analytics copy |

Use a shortcut when no transformation or separate snapshot is required. Use a pipeline or mirroring only when the scenario genuinely needs those capabilities.

## 7. Practical Guidance

- Prefer shortcuts to eliminate cross-workspace or cross-cloud copies when the source is already query-compatible.
- Enable caching only for supported sources: GCS, S3, S3-compatible storage, and on-premises gateway sources.
- Set cache retention to match read patterns: longer for stable reference data, shorter for frequently changing data.
- Check target permissions because the caller's identity is evaluated against the target.
- Avoid building a copy pipeline when a shortcut meets the requirement.
- Choose pipelines or mirroring when transformation, format conversion, or a decoupled snapshot is genuinely needed.

## 8. Exam Tips

- **Internal targets:** KQL database, lakehouse, mirrored Azure Databricks catalog, mirrored database, semantic model, SQL database, warehouse.
- **External targets:** ADLS Gen2, Blob Storage, S3, S3-compatible storage, GCS, Dataverse, Iceberg, OneDrive/SharePoint, on-premises through a gateway.
- **Caching sources:** GCS, S3, S3-compatible storage, and on-premises gateway only.
- **Retention:** 1–28 days.
- **File-size ceiling:** Files over 1 GB are not cached.
- **Authorization:** Calling user's identity against the target, except Direct Lake over SQL / Delegated identity mode, which uses the calling item's owner identity.
- **No transformation:** Shortcuts reference data; they do not transform it.

## 9. Key Takeaways

- Shortcuts are pointer-like references in OneLake that avoid unnecessary data copies.
- Internal shortcuts reference other Fabric items; external shortcuts reach supported external sources.
- A shortcut does not remove the need for real target permissions.
- Only the listed source types support caching; retention is 1–28 days, and files over 1 GB are not cached.
- Shortcuts do not transform data. Use pipelines or mirroring for transformation, format conversion, or a decoupled snapshot.

## 10. Quick Revision Table

| Question clue | Answer |
|---|---|
| Avoid cross-workspace copy of compatible data | Shortcut |
| Need to transform or convert format | Pipeline |
| Need a decoupled replicated snapshot | Mirroring or pipeline, depending on scenario |
| Cache-supported sources | GCS, S3, S3-compatible, on-prem gateway |
| Cache retention | 1–28 days |
| File over 1 GB | Not cached |
| Normal internal shortcut identity | Calling user's identity |
| Delegated identity exception | Direct Lake over SQL / Delegated identity uses calling item's owner |
| User can see shortcut but cannot read target | Check target permissions |
