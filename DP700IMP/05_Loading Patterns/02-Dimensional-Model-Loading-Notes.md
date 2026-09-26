# DP-700 — Dimensional Model Loading Notes

## 1. Load Dimensions Before Facts

A fact table needs dimension surrogate keys to identify the related dimension records.

**Correct order:**
1. Load or update dimension tables.
2. Resolve dimension surrogate keys.
3. Load fact tables.

**Memory trick:** Dimensions first → Facts second.

---

## 2. Surrogate Keys

A surrogate key is an artificial key generated for a dimension record.

Example:

| CustomerKey | CustomerID | Name |
|---|---|---|
| 101 | C101 | Ravi |
| 102 | C102 | Priya |

- `CustomerKey` = surrogate key.
- `CustomerID` = business key.

### Key-generation options

| Method | Meaning |
|---|---|
| `IDENTITY` | Automatically generates values; check Fabric Warehouse Preview limitations. |
| `ROW_NUMBER()` | Assigns sequential numbers to rows in a query result. |
| Hash key | Generates a key from selected input values using a hash function. |

**Exam tips from the study notes:**
- Fabric Warehouse `IDENTITY` is Preview in the referenced blueprint; it is bigint-only, has no reseed, and cannot be added using `ALTER TABLE ADD`.
- Prefer `ROW_NUMBER()`-based or hash-based surrogate keys when Preview limitations are unacceptable.
- `monotonically_increasing_id()` is not stable across retries or re-execution; do not use it as a real surrogate-key generator.

---

## 3. SCD Type 1 vs. SCD Type 2

SCD means Slowly Changing Dimension.

### SCD Type 1 — Overwrite

Updates the existing dimension row in place. It does not preserve the old value as a historical version.

Example: Ravi's city changes from Hubli to Bangalore.

| CustomerKey | Name | City |
|---|---|---|
| 101 | Ravi | Bangalore |

**Memory:** SCD1 = overwrite; no history.

### SCD Type 2 — Preserve history

Closes the old version and inserts a new version with a new surrogate key.

| CustomerKey | CustomerID | City | IsCurrent |
|---|---|---|---|
| 101 | C101 | Hubli | 0 |
| 205 | C101 | Bangalore | 1 |

**Memory:** SCD2 = new version + historical row retained.

### Implementation pattern

- **T-SQL:** Two steps — use `MERGE` to close the old version, then `INSERT` the new version.
- **Delta Lake:** `MERGE INTO` can implement SCD2 in one statement using the `mergeKey`-is-`NULL` technique described in the study notes.

**Exam trap:** A single `MERGE ... WHEN MATCHED THEN UPDATE` clause alone implements SCD1, not a complete SCD2 process.

---

## 4. Filter Dimension Lookups to the Current Version

SCD2 dimensions can contain multiple rows for the same business key.

Example:

| CustomerKey | CustomerID | City | IsCurrent |
|---|---|---|---|
| 101 | C101 | Hubli | 0 |
| 205 | C101 | Bangalore | 1 |

For fact loading, filter dimension lookups using:

```sql
WHERE IsCurrent = 1
```

This ensures the fact resolves to the current dimension version rather than a historical row.

**Memory:** Fact lookup → `IsCurrent = 1`.

---

## 5. Unknown Member vs. Reject/Quarantine

When a fact arrives but its dimension record cannot be found:

### Unknown Member
Reserve a special surrogate key (for example, `0`) for an Unknown Member. The fact can still load and remain visible in reports.

### Reject/Quarantine
Send the unresolved fact to a separate location for investigation. Use this when business requirements mandate rejection.

**Exam rule:** Default to an Unknown Member when the record should remain visible in reports. Use reject/quarantine only when required by business rules.

**Avoid NULL foreign keys:** NULL foreign keys can cause fact rows to disappear from join-based reports.

---

## 6. Late-Arriving Dimensions and Inferred Members

A fact can arrive before the complete dimension record.

1. Create an inferred-member stub using the known business key.
2. Load the fact using the stub's surrogate key.
3. When real details arrive, update the stub in place.

Example:

Before backfill:

| CustomerKey | CustomerID | Name |
|---|---|---|
| 105 | C105 | Unknown |

After backfill:

| CustomerKey | CustomerID | Name |
|---|---|---|
| 105 | C105 | Amit |

**Important:** Update the stub in place, not as a new SCD2 version, because the stub did not represent a real historical state.

**Memory:** Late dimension → Stub → Load fact → Update stub in place.

---

## 7. Best Practices — Quick Checklist

- [ ] Load dimensions before facts.
- [ ] Use stable surrogate-key strategies.
- [ ] Filter SCD2 lookups with `IsCurrent = 1` (or equivalent).
- [ ] Use an Unknown Member for unresolved lookups when facts should remain reportable.
- [ ] Use reject/quarantine only when business rules require it.
- [ ] Backfill inferred members with an in-place update.
- [ ] Avoid NULL foreign keys in fact tables.

---

## 8. Exam Tips — Quick Review

1. **Dimension before fact:** Facts need dimension surrogate keys.
2. **SCD1 vs. SCD2:** SCD1 overwrites; SCD2 versions records.
3. **T-SQL vs. Delta SCD2:** T-SQL uses `MERGE` + `INSERT`; Delta can use one `MERGE INTO` with the `mergeKey`-is-`NULL` technique.
4. **Late-arriving dimension:** Create an inferred-member stub and backfill it in place.
5. **Unresolved lookup:** Use a reserved Unknown Member key, not a NULL foreign key.
6. **Surrogate keys:** Avoid `monotonically_increasing_id()` for stable real keys.

---

## 9. One-Minute Revision

- **Dimensions → Facts**
- **SCD1 → Overwrite**
- **SCD2 → Close old + insert new**
- **Current lookup → `IsCurrent = 1`**
- **Unknown lookup → Reserved key**
- **Late dimension → Stub + in-place backfill**
- **T-SQL SCD2 → Two steps**
- **Delta SCD2 → One `MERGE INTO` using mergeKey trick**
