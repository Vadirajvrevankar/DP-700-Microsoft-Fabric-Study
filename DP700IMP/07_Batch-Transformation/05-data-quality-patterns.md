# DP-700: Data Quality Patterns in PySpark, T-SQL, and KQL

> **Purpose:** Exam-focused notes on deterministic deduplication, missing-data handling, late-arriving data, and denormalization across PySpark, T-SQL, and KQL.

---

## 1. Deterministic Deduplication — Keep the Latest Record

### What is deduplication?
Deduplication means removing duplicate records from a dataset.

Example:

| CustomerID | Status | Timestamp |
|---|---|---|
| 101 | Active | 10:00 |
| 101 | Inactive | 11:00 |
| 102 | Active | 09:00 |
| 102 | Active | 09:00 |

If the rule is **keep the latest record for each customer**, retain the 11:00 record for Customer 101.

### What is deterministic deduplication?
Deterministic deduplication uses a clearly defined rule to decide which record survives—for example, sort by timestamp newest-first and keep the first row.

### A. PySpark — `row_number()`

```python
from pyspark.sql import Window
from pyspark.sql.functions import row_number, col

window_spec = (
    Window.partitionBy("CustomerID")
    .orderBy(col("Timestamp").desc())
)

deduplicated = (
    df.withColumn("rn", row_number().over(window_spec))
      .filter(col("rn") == 1)
      .drop("rn")
)
```

How it works:
1. `partitionBy("CustomerID")` groups rows by customer.
2. `orderBy(Timestamp.desc())` puts the newest record first.
3. `row_number()` numbers rows within each group.
4. `filter(rn == 1)` keeps the first row.

**Important:** If timestamps can tie, add a unique tie-breaker to the ordering to make the selection deterministic.

### B. T-SQL — `ROW_NUMBER()`

```sql
WITH RankedCustomers AS
(
    SELECT *,
           ROW_NUMBER() OVER
           (
               PARTITION BY CustomerID
               ORDER BY Timestamp DESC
           ) AS rn
    FROM dbo.CustomerEvents
)
SELECT *
FROM RankedCustomers
WHERE rn = 1;
```

### C. KQL — `arg_max()`

```kusto
CustomerEvents
| summarize arg_max(Timestamp, *) by CustomerID
```

`arg_max()` returns the record associated with the maximum timestamp for each customer.

### Non-deterministic deduplication

| Language | Non-deterministic pattern | Deterministic keep-latest pattern |
|---|---|---|
| PySpark | `dropDuplicates()` | Window + `row_number()` |
| T-SQL | `DISTINCT` | `ROW_NUMBER()` |
| KQL | `take_any()` | `arg_max()` |

**Exam tip:** For “keep the latest record,” prefer a window function or `arg_max()` rather than arbitrary deduplication.

---

## 2. Missing-Data Handling — `COALESCE` vs. `ISNULL`

### What is missing-data handling?
A `NULL` represents a missing or unknown value. You may want to replace it with a fallback value such as zero.

### A. PySpark — `na.fill()` and `coalesce()`

Replace nulls in a column:

```python
df_filled = df.na.fill({"Bonus": 0})
```

Use `coalesce()` to return the first non-null expression:

```python
from pyspark.sql.functions import coalesce, col, lit

df_filled = df.withColumn(
    "Bonus",
    coalesce(col("Bonus"), lit(0))
)
```

### B. T-SQL — `COALESCE` and `ISNULL`

Using `COALESCE`:

```sql
SELECT
    EmployeeID,
    EmployeeName,
    COALESCE(Bonus, 0) AS Bonus
FROM dbo.Employees;
```

Using `ISNULL`:

```sql
SELECT
    EmployeeID,
    EmployeeName,
    ISNULL(Bonus, 0) AS Bonus
FROM dbo.Employees;
```

Both can replace a null with a fallback value in this example.

### Why prefer `COALESCE`?
- It is part of the ANSI SQL standard.
- It accepts multiple arguments.
- Its type-resolution behavior differs from `ISNULL`; the supplied notes recommend it for portability and predictable type resolution.

Example:

```sql
SELECT COALESCE(NULL, NULL, 100, 200) AS Result;
```

Returns `100`, the first non-null value.

### C. KQL — `coalesce()`

```kusto
Employees
| extend Bonus = coalesce(Bonus, 0)
```

### Missing-data mapping

| Language | Pattern | Purpose |
|---|---|---|
| PySpark | `na.fill()` | Replace nulls with specified values |
| PySpark | `coalesce()` | Return first non-null expression |
| T-SQL | `COALESCE()` | Return first non-null expression |
| T-SQL | `ISNULL()` | Replace null with a fallback |
| KQL | `coalesce()` | Return first non-null expression |

**Exam tip:** Prefer `COALESCE` over `ISNULL` for exam-safe T-SQL answers, as recommended in the supplied notes.

---

## 3. Late-Arriving Facts vs. Late-Arriving Dimensions

This distinction matters because the two situations require different handling.

### Facts and dimensions
- **Fact table:** Stores measurable business events, such as sales, orders, payments, or device readings.
- **Dimension table:** Stores descriptive information, such as customer, product, or store details.

### A. Late-arriving facts — Watermark

A late-arriving fact is a business event that arrives after its expected processing time or after the pipeline has processed an earlier time range.

Example: A Sunday transaction arrives on Tuesday because the source system was delayed.

A **watermark** tracks event-time progress and helps determine how long a system should wait for late events before treating a time window as sufficiently complete.

**Remember:** Late-arriving facts are primarily a **watermark and event-time processing** concern.

### B. Late-arriving dimensions — Inferred member

A late-arriving dimension occurs when a fact arrives before its corresponding dimension record is available.

Example: A sales record references `CustomerID = 501`, but customer 501 is not yet in the customer dimension.

An **inferred member** is a placeholder dimension record created when a fact arrives before the corresponding dimension details.

Example placeholder:

| CustomerID | CustomerName | City |
|---|---|---|
| 501 | Unknown | Unknown |

When the actual details arrive, the placeholder can be updated.

### Watermark vs. inferred member

| Situation | Pattern |
|---|---|
| Fact arrives late relative to event time | Watermark |
| Fact arrives before its dimension record | Inferred member |
| Problem is delayed event processing | Watermark |
| Problem is a missing dimension reference | Inferred member |

**Exam memory trick:**
- **Late fact = Watermark.**
- **Missing dimension = Inferred member.**

---

## 4. Denormalization — Join and Flatten Data

### What is denormalization?
Denormalization combines data from related tables into a wider table so queries can access required information more directly. It often joins a large fact table with a smaller dimension table.

Example:

**Sales fact table**

| SalesID | ProductID | Amount |
|---|---|---:|
| 1 | P101 | 5000 |
| 2 | P102 | 3000 |
| 3 | P101 | 2000 |

**Products dimension table**

| ProductID | ProductName | Category |
|---|---|---|
| P101 | Laptop | Electronics |
| P102 | Chair | Furniture |

After denormalization, sales rows include product name and category columns.

---

## 5. Performance-Optimized Joins in Each Language

When the fact table is large and the dimension table is small, choose the join strategy deliberately.

### A. PySpark — `broadcast()`

A broadcast join sends a small table to worker nodes so it can be joined with a large table without shuffling the large table in the same way as a conventional join.

```python
from pyspark.sql.functions import broadcast

result = (
    sales_df
    .join(
        broadcast(products_df),
        on="ProductID",
        how="left"
    )
)
```

- `sales_df`: Large fact table.
- `products_df`: Small dimension table.
- `broadcast(products_df)`: Signals that the small table should be broadcast.
- `join()`: Combines rows using `ProductID`.

Use a broadcast join when the dimension is small enough to fit safely in the relevant workers' memory.

### B. T-SQL — `JOIN` and CTAS

A denormalized result can be created using a join and CTAS:

```sql
CREATE TABLE dbo.DenormalizedSales
AS
SELECT
    S.SalesID,
    S.ProductID,
    S.Amount,
    P.ProductName,
    P.Category
FROM dbo.Sales AS S
LEFT JOIN dbo.Products AS P
    ON S.ProductID = P.ProductID;
```

- `Sales`: Fact table.
- `Products`: Dimension table.
- `LEFT JOIN`: Retains sales rows even if product details are missing.
- CTAS creates a new table from the joined result.

### C. KQL — `lookup`

```kusto
Sales
| lookup kind=leftouter (
    Products
) on ProductID
```

`lookup` enriches a large fact table with columns from a small dimension table.

### Denormalization mapping

| Engine | Pattern | Typical use |
|---|---|---|
| PySpark | `broadcast()` + `join()` | Large fact + small dimension |
| Fabric Warehouse T-SQL | `JOIN` + CTAS | Create a flattened table |
| KQL | `lookup` | Enrich a large fact table with a small dimension |

**Memory trick:**
- **PySpark:** `broadcast()`
- **T-SQL:** `JOIN` / CTAS
- **KQL:** `lookup`

---

## 6. Three-Language Data Quality Cheat Sheet

| Data quality pattern | PySpark | T-SQL | KQL |
|---|---|---|---|
| Keep latest record | Window + `row_number()` | `ROW_NUMBER() = 1` | `arg_max()` |
| Remove duplicates without a keep-latest rule | `dropDuplicates()` | `DISTINCT` | `take_any()` |
| Replace missing values | `na.fill()` / `coalesce()` | `COALESCE()` | `coalesce()` |
| Late-arriving facts | Watermark / event-time handling | Watermark / pipeline logic | Watermark / event-time handling |
| Late-arriving dimensions | Inferred member | Inferred member | Inferred member |
| Large fact + small dimension | `broadcast()` + `join()` | `JOIN` / CTAS | `lookup` |

---

## 7. DP-700 Exam Tips

1. Use deterministic deduplication when the requirement is to **keep the latest record**.
2. Prefer `COALESCE` over `ISNULL` in T-SQL for portability and predictable type resolution.
3. **Late-arriving fact = Watermark.**
4. **Missing dimension record = Inferred member.**
5. For large fact + small dimension, choose the performance-appropriate join primitive for the engine.
6. Memorize the mapping: `dropDuplicates()` / `ROW_NUMBER() = 1` / `arg_max()`.
7. Memorize the missing-value mapping: `na.fill()` / `COALESCE` / `coalesce()`.

---

## 8. Final Key Takeaways

- Deduplication, missing-data handling, late-arriving data, and denormalization have corresponding patterns in PySpark, T-SQL, and KQL.
- Non-deterministic deduplication is appropriate only when any surviving duplicate is acceptable; use a defined ordering when the latest record must survive.
- Watermarks and inferred members solve different problems.
- Denormalization combines fact and dimension data, and each engine offers different join strategies.
- `broadcast()`, `JOIN`/CTAS, and `lookup` are the key denormalization patterns.

### One-line memory trick
**Deduplicate deterministically, fill nulls with COALESCE, handle late facts with watermarks, fix missing dimensions with inferred members, and optimize fact-dimension joins.**
