# DP-700: PySpark and Delta Lake Concepts Explained

These notes cover working with data efficiently and correctly in Microsoft Fabric notebooks using PySpark and Delta Lake.

## Topics
1. Delta-native reads and writes
2. Broadcast joins
3. Deduplication using `row_number()` and window functions
4. Explicit data type casting
5. `na.fill()` vs. `na.drop()`
6. `partitionBy()` and physical storage partitioning
7. Exam tips and key takeaways

---

## 1. Delta-native reads and writes

### What is Delta Lake?

Delta Lake is a storage layer that adds a transaction log and table-management capabilities on top of data files, commonly Parquet files.

In Microsoft Fabric, Delta tables are commonly used in Lakehouses.

Delta Lake provides features such as:
- **ACID transactions:** Helps maintain consistent table updates.
- **Time travel:** Lets you query supported historical versions of a table.
- **Schema enforcement:** Helps prevent incompatible data from being written.
- **MERGE:** Supports inserting, updating, and deleting records based on matching conditions.

### A. Reading a Delta table

```python
df = spark.read.format("delta").load("Files/sales_delta")
```

- `spark.read` starts a read operation.
- `.format("delta")` specifies that the source is a Delta table.
- `.load()` loads the table from the given path.
- `df` stores the result as a Spark DataFrame.

### B. Writing a Delta table

```python
df.write \\
    .format("delta") \\
    .mode("overwrite") \\
    .save("Files/sales_delta")
```

- `.write` starts a write operation.
- `.format("delta")` writes data in Delta format.
- `.mode("overwrite")` replaces existing data at the target.
- `.save()` writes to the specified path.

**Important:** Overwrite can replace existing data. Use it only when that is intended.

### C. Using `saveAsTable()`

```python
df.write \\
    .format("delta") \\
    .mode("overwrite") \\
    .saveAsTable("Sales")
```

This saves a DataFrame as a named table in the current Spark catalog, depending on the write mode and table configuration.

### Why prefer Delta-native operations?

| Operation | Purpose |
|---|---|
| `spark.read.format("delta")` | Read Delta data |
| `saveAsTable()` | Save data as a named table |
| `MERGE` | Update, insert, or delete matching records |
| Time travel | Query supported historical table versions |
| Schema enforcement | Validate incoming data against the table schema |

**Exam tip:** When a question mentions Delta tables, transaction logs, time travel, or `MERGE`, think about Delta-native operations rather than treating the table as ordinary files.

---

## 2. Broadcast joins using `broadcast()`

### What is a join?

A join combines two datasets using a common key.

**Sales table**

| ProductID | Sales |
|---|---:|
| 101 | 500 |
| 102 | 700 |
| 103 | 300 |

**Product table**

| ProductID | ProductName |
|---|---|
| 101 | Laptop |
| 102 | Phone |
| 103 | Tablet |

Joining these tables on `ProductID` adds the product name to each sales record.

### What is a broadcast join?

A broadcast join sends a small DataFrame to worker nodes so Spark can join it locally with partitions of the larger DataFrame. This can avoid shuffling the large side of the join across the cluster.

### Example

```python
from pyspark.sql.functions import broadcast

result = sales_df.join(
    broadcast(product_df),
    on="ProductID",
    how="inner"
)
```

- `sales_df` is the large sales table.
- `product_df` is the small product dimension table.
- `broadcast(product_df)` tells Spark to broadcast the small table.
- `join()` combines both tables using `ProductID`.

### Why use broadcast?

Without a broadcast join, Spark may need to shuffle data across the cluster to match records. With a broadcast join, the small table is sent to the workers, which can reduce network communication and improve performance.

### When should you use it?

- The dimension table is genuinely small.
- The table can fit comfortably in memory on the relevant workers.
- The join would otherwise require an expensive shuffle.

**Exam warning:** Broadcast the small side of the join, not the large table. Broadcasting a large table can cause memory pressure or out-of-memory errors.

Spark can automatically choose broadcast joins when its optimizer has enough information. Use explicit `broadcast()` when you know the small table is suitable.

---

## 3. `row_number()` with window functions

### What is the problem?

Suppose customer data contains multiple records for the same customer.

| CustomerID | CustomerName | UpdatedAt |
|---|---|---|
| 101 | Ravi | 2026-09-20 |
| 101 | Ravi Kumar | 2026-09-25 |
| 102 | Priya | 2026-09-21 |
| 102 | Priya Sharma | 2026-09-26 |

You want to **keep only the latest record for each customer**.

- For Customer 101, keep Ravi Kumar.
- For Customer 102, keep Priya Sharma.

### What is a window function?

A window function performs calculations across a group of rows while keeping individual rows available.

- `Window.partitionBy()` divides rows into groups.
- `orderBy()` sorts rows within each group.
- `row_number()` assigns a sequential number to each row in the group.

### Example: Keep the latest record per customer

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

window_spec = (
    Window
    .partitionBy("CustomerID")
    .orderBy(F.col("UpdatedAt").desc())
)

result = (
    df.withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)
```

### Step-by-step

1. `Window.partitionBy("CustomerID")` creates a separate window for each customer.
2. `orderBy(F.col("UpdatedAt").desc())` puts the newest record first.
3. `row_number()` assigns row numbers within each customer group.
4. `filter(F.col("row_num") == 1)` keeps the latest record.
5. `.drop("row_num")` removes the helper column.

---

## 4. `dropDuplicates()` vs. `row_number()`

Both can be used for deduplication, but they solve different problems.

### A. `dropDuplicates()`

Use it when you want to remove duplicate rows or retain one row per key without specifying which record must win.

```python
result = df.dropDuplicates(["CustomerID"])
```

This keeps one record per `CustomerID`, but **does not guarantee that the latest record will survive**.

### B. `row_number()`

Use it when you need to control which record is retained.

Examples:
- Keep the latest record.
- Keep the highest-scoring record.
- Keep the most recent transaction.
- Keep the record with the highest priority.

### Comparison

| Feature | `dropDuplicates()` | `row_number()` |
|---|---|---|
| Remove duplicate records | Yes | Yes, with filtering |
| Keep latest record | No guarantee | Yes, with ordering |
| Control which record wins | No | Yes |
| Requires a window | No | Yes |
| Typical use | Remove duplicates | Latest/best record per key |

**Exam tip:** If the question says “keep the latest record per customer,” choose `row_number()` with a window ordered by timestamp descending, then filter for `1`.

**Important:** If multiple records have the same timestamp, add a tie-breaker column to the ordering to make the selection deterministic.

---

## 5. Explicit type casting after ingestion

### What is type casting?

Type casting means converting a column from one data type to another.

Examples:
- String → Integer
- String → Date
- String → Decimal
- Integer → String

### Why is it important?

Data arriving from source systems may have incorrect or inconsistent data types.

For example, a sales CSV file might contain:

| OrderID | Amount |
|---|---|
| 101 | "1500.50" |
| 102 | "2200.00" |
| 103 | "1800.75" |

The `Amount` column may be read as a string. If you need to calculate total sales, convert it to a numeric type.

### Convert a string to decimal

```python
from pyspark.sql import functions as F

df = df.withColumn(
    "Amount",
    F.col("Amount").cast("decimal(10,2)")
)
```

This converts `Amount` into a decimal type with precision 10 and scale 2.

### Convert a string to integer

```python
df = df.withColumn(
    "Quantity",
    F.col("Quantity").cast("int")
)
```

### Convert a string to date

```python
df = df.withColumn(
    "OrderDate",
    F.to_date(F.col("OrderDate"), "yyyy-MM-dd")
)
```

### Where should you cast types?

A common data engineering pattern is:

1. **Bronze layer:** Raw ingested data.
2. **Silver layer:** Clean data, correct data types, standardized values.
3. **Gold layer:** Business-ready, aggregated data.

**Exam tip:** Explicitly cast columns early in the cleaning process, commonly during Bronze-to-Silver transformation, so type mismatches do not propagate.

**Note:** Invalid values may become null during a cast. Validate results and handle conversion failures appropriately.

---

## 6. `na.fill()` vs. `na.drop()`

These are DataFrame methods for handling missing values in PySpark.

### A. `na.fill()`

Use `na.fill()` to replace null values with a specified value.

Example data:

| Name | Age |
|---|---:|
| Ravi | 25 |
| Priya | null |
| Amit | 30 |

```python
result = df.na.fill({"Age": 0})
```

The missing age is replaced with `0`.

You can also fill a specific column:

```python
result = df.na.fill(0, subset=["Age"])
```

### B. `na.drop()`

Use `na.drop()` to remove rows containing null values according to specified conditions.

```python
result = df.na.drop()
```

This drops rows that contain at least one null value.

### What is `subset=`?

`subset` specifies which columns to check when deciding whether to drop a row or where to apply a fill operation.

```python
result = df.na.drop(subset=["Age"])
```

This removes rows where `Age` is null, without requiring every other column to be non-null.

### What is `thresh=`?

`thresh` specifies the **minimum number of non-null values** a row must contain to be retained.

```python
result = df.na.drop(thresh=2)
```

This keeps rows with at least two non-null values across the DataFrame's columns.

### Comparison

| Method / parameter | Meaning |
|---|---|
| `na.fill()` | Replace null values |
| `na.drop()` | Remove rows with null values |
| `subset=` | Specify which columns to consider |
| `thresh=` | Minimum number of non-null values required to keep a row |

**Exam example:** A DataFrame has five columns. You want to keep rows with at least three non-null values:

```python
result = df.na.drop(thresh=3)
```

Answer: `thresh=3`.

---

## 7. `partitionBy()` — Physical Storage Partitioning

### What is partitioning?

Partitioning divides data into separate storage directories based on the values of one or more columns. It can help improve query performance by allowing Spark to skip irrelevant partitions when filters match the partition columns. This is called **partition pruning**.

### Example: Partition sales data by year

```python
df.write \\
    .format("delta") \\
    .partitionBy("Year") \\
    .mode("overwrite") \\
    .saveAsTable("Sales")
```

This writes the table partitioned by the `Year` column.

Conceptually, the storage layout may look like:

```text
Sales/
├── Year=2024/
│   └── data files
├── Year=2025/
│   └── data files
└── Year=2026/
    └── data files
```

### How does partition pruning work?

```python
df = spark.table("Sales").filter("Year = 2026")
```

Spark may read only the relevant `Year=2026` partition rather than scanning every year.

### `partitionBy()` vs. `repartition()`

| Feature | `partitionBy()` | `repartition()` |
|---|---|---|
| Purpose | Organize data in storage | Change Spark DataFrame partitioning |
| Affects | Physical file and directory layout | In-memory/distributed execution layout |
| Used for | Storage organization and partition pruning | Distributing work across Spark tasks |
| Example | `.write.partitionBy("Year")` | `df.repartition(8)` |

**Exam tip:** `partitionBy()` in a write operation is a physical storage optimization. It is not the same as controlling the number of Spark execution partitions.

**Practical caution:** Avoid creating too many small storage partitions. Choose partition columns and granularity based on data size and query patterns.

---

## 8. Complete Practical Example

### Scenario

You receive customer records with:
- Duplicate customer IDs.
- Multiple updates per customer.
- Amounts stored as strings.
- A need to retain only the latest record.

### PySpark code

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# 1. Read Delta data
df = spark.read.format("delta").load(
    "Files/bronze_customers"
)

# 2. Explicitly cast data types
df = (
    df.withColumn(
        "Amount",
        F.col("Amount").cast("decimal(10,2)")
    )
    .withColumn(
        "UpdatedAt",
        F.to_timestamp("UpdatedAt")
    )
)

# 3. Handle missing amounts
df = df.na.fill({"Amount": 0})

# 4. Define a window to keep the latest record
window_spec = (
    Window
    .partitionBy("CustomerID")
    .orderBy(F.col("UpdatedAt").desc())
)

# 5. Keep the latest row per customer
result = (
    df.withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)

# 6. Write the cleaned data as a Delta table
result.write \\
    .format("delta") \\
    .mode("overwrite") \\
    .saveAsTable("SilverCustomers")
```

### Concepts used

| Step | Concept |
|---|---|
| Read raw Delta data | `spark.read.format("delta")` |
| Convert data types | `cast()` |
| Replace missing values | `na.fill()` |
| Group and sort records | `Window.partitionBy().orderBy()` |
| Keep latest record | `row_number()` |
| Save as a named Delta table | `saveAsTable()` |

---

## 9. DP-700 Exam Tips

### Tip 1 — Broadcast joins
`broadcast()` is for the small side of a join. Broadcasting a large table can cause memory errors rather than improve performance.

### Tip 2 — Latest record per key
`dropDuplicates()` does not guarantee which record survives. Use `row_number()` over an ordered window and filter to `1` when the latest or best record must be retained.

### Tip 3 — Missing values
- `na.fill()` replaces nulls.
- `na.drop()` removes rows based on null conditions.
- `thresh=` specifies the minimum non-null count.
- `subset=` specifies which columns to check.

### Tip 4 — Storage partitioning
`partitionBy()` in a write operation controls the physical storage layout, not the number of Spark execution partitions.

### Tip 5 — Delta-native operations
Prefer Delta-aware reads and writes when working with Delta tables to benefit from Delta features such as transaction-log-based consistency and table operations.

---

## 10. Key Takeaways

- `spark.read.format("delta")` reads Delta data.
- `saveAsTable()` writes data as a named table.
- `partitionBy()` organizes data into physical storage partitions.
- `broadcast()` can improve joins when the broadcast side is genuinely small.
- `Window.partitionBy().orderBy()` with `row_number()` is the reusable pattern for keeping the latest or best record per key.
- `dropDuplicates()` removes duplicates but does not guarantee which record survives.
- Explicitly cast data types early, commonly during Bronze-to-Silver transformation.
- `na.fill()` replaces nulls; `na.drop()` removes rows according to specified conditions.
- `thresh=` is the minimum non-null count; `subset=` specifies columns to consider.

---

## 11. Quick Memory Table

| Exam clue | Concept / answer |
|---|---|
| Read a Delta table | `spark.read.format("delta")` |
| Save as a named table | `saveAsTable()` |
| Small dimension + large fact | Broadcast join |
| Keep latest record per key | `row_number()` + ordered window |
| Remove duplicates without choosing latest | `dropDuplicates()` |
| Convert string to numeric | `cast()` |
| Replace missing values | `na.fill()` |
| Remove rows with nulls | `na.drop()` |
| Minimum non-null count | `thresh=` |
| Check specific columns | `subset=` |
| Organize storage by year | `partitionBy("Year")` |
| Change execution partition count | `repartition()` |

### Final exam memory trick

**Delta → Broadcast → Window → Cast → NA → Partition**

1. **Delta:** Read and write Delta tables using Delta-aware operations.
2. **Broadcast:** Broadcast the small side of a join.
3. **Window:** Use `row_number()` to keep the latest or best row.
4. **Cast:** Set correct data types early.
5. **NA:** Know how to fill or drop nulls.
6. **Partition:** Distinguish physical storage partitioning from Spark execution partitioning.
