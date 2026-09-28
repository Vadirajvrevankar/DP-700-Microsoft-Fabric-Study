# DP-700: Fabric Warehouse T-SQL Concepts — Complete Notes

> **Purpose:** Simple, exam-focused notes on Fabric Warehouse T-SQL concepts, based on the concepts and exam tips discussed.

> **Important:** Fabric's T-SQL support changes over time. Verify current support and preview/GA status in the [official T-SQL surface area documentation](https://learn.microsoft.com/en-us/fabric/data-warehouse/tsql-surface-area).

---

## 1. T-SQL Surface Area

### What is T-SQL?
**T-SQL (Transact-SQL)** is Microsoft's SQL language for querying and managing relational data.

```sql
SELECT EmployeeName, Salary
FROM dbo.Employees
WHERE Salary > 50000;
```

### What does “T-SQL surface area” mean?
It is the collection of SQL commands, functions, data types, and features supported by a platform.

Fabric Warehouse supports a **subset** of SQL Server's T-SQL. A SQL Server feature may not work in Fabric Warehouse.

**Exam tip:** Verify feature support instead of assuming SQL Server and Fabric Warehouse have identical capabilities.

---

## 2. Distributed `#temp` Tables

### What is a temporary table?
A temporary table stores intermediate data during a SQL workflow. A local temporary table commonly starts with `#`.

```sql
CREATE TABLE #TempSales
(
    ProductID INT,
    SalesAmount DECIMAL(18,2)
);
```

Insert and query rows:

```sql
INSERT INTO #TempSales
VALUES (101, 2500.00);

SELECT *
FROM #TempSales;
```

### What does `DISTRIBUTION = ROUND_ROBIN` mean?
`ROUND_ROBIN` distributes rows across distributions without choosing a particular column as the distribution key.

```sql
CREATE TABLE #TempSales
(
    ProductID INT,
    SalesAmount DECIMAL(18,2)
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN
);
```

### When is it useful?
- Small, trivial staging may not need special distribution.
- Larger intermediate datasets may benefit from distributed temporary tables and parallel processing.

**Exam tip:** Consider a distributed `#temp` table with `DISTRIBUTION = ROUND_ROBIN` for larger staging workloads.

---

## 3. Warehouse Collation

### What is collation?
Collation defines how text is compared and sorted. It can determine whether uppercase and lowercase letters are treated as equivalent.

### Case-insensitive (CI)
A case-insensitive comparison generally treats `Ravi`, `ravi`, and `RAVI` as equivalent.

```sql
SELECT *
FROM dbo.Customers
WHERE CustomerName = 'RAVI';
```

### Case-sensitive (CS)
A case-sensitive comparison distinguishes uppercase and lowercase letters. The query may match only the exact case-sensitive value `RAVI`.

### Why decide before creating the warehouse?
The warehouse's collation affects text comparison and sorting. **According to the supplied notes, warehouse collation cannot be changed after creation.**

**Exam tip:** Decide whether you need case-sensitive or case-insensitive behavior **before creating the warehouse**.

---

## 4. `MERGE` — Upsert Logic

### What is an upsert?
An **upsert** combines:
- **UPDATE:** Update an existing record.
- **INSERT:** Insert a record that does not already exist.

### Example
```sql
MERGE INTO dbo.TargetCustomers AS T
USING dbo.SourceCustomers AS S
ON T.CustomerID = S.CustomerID

WHEN MATCHED THEN
    UPDATE SET
        T.CustomerName = S.CustomerName,
        T.City = S.City

WHEN NOT MATCHED BY TARGET THEN
    INSERT (CustomerID, CustomerName, City)
    VALUES (S.CustomerID, S.CustomerName, S.City);
```

### Key clauses

| Clause | Meaning |
|---|---|
| `MERGE INTO` | Target table |
| `USING` | Source dataset |
| `ON` | Matching condition |
| `WHEN MATCHED` | Update matching rows |
| `WHEN NOT MATCHED BY TARGET` | Insert unmatched source rows |

**Exam tip:** In the supplied notes, `MERGE` is **GA (Generally Available)** in Fabric Warehouse—not preview-only. It expresses matching, updating, and inserting logic in one statement.

---

## 5. `IDENTITY` vs. `ROW_NUMBER()` — Surrogate Keys

### What is a surrogate key?
A surrogate key is an artificial identifier used to uniquely identify a row in a data warehouse.

| CustomerKey | CustomerID | CustomerName |
|---:|---|---|
| 1 | C101 | Ravi |
| 2 | C102 | Priya |
| 3 | C103 | Amit |

- `CustomerKey`: Surrogate key
- `CustomerID`: Business key from the source system

### `IDENTITY`
`IDENTITY` generates numeric values for inserted rows.

```sql
CREATE TABLE dbo.Customers
(
    CustomerKey BIGINT IDENTITY,
    CustomerName VARCHAR(100)
);
```

**Limitations listed in the supplied notes:**
- Preview-only
- `BIGINT` only
- No custom seed or increment
- Cannot add an identity column using `ALTER TABLE`

### `ROW_NUMBER()`
`ROW_NUMBER()` assigns sequential numbers to query rows according to an ordering.

```sql
SELECT
    ROW_NUMBER() OVER (ORDER BY CustomerID) AS CustomerKey,
    CustomerID,
    CustomerName
FROM dbo.Customers;
```

**Important:** `ROW_NUMBER()` numbers query results; it does not automatically guarantee a permanent, stable key across repeated loads. Use a deterministic ordering, ideally including a unique tie-breaker.

**Exam tip:** For a broadly applicable, query-based sequential surrogate-key pattern, consider `ROW_NUMBER()`.

---

## 6. Constraints: PRIMARY KEY, FOREIGN KEY, UNIQUE

### PRIMARY KEY
Identifies a row.

```sql
CREATE TABLE dbo.Customers
(
    CustomerID INT NOT NULL,
    CustomerName VARCHAR(100),
    CONSTRAINT PK_Customers
        PRIMARY KEY (CustomerID) NOT ENFORCED
);
```

### FOREIGN KEY
Describes a relationship between tables, such as `Orders.CustomerID` referencing `Customers.CustomerID`.

### UNIQUE
Declares that a column or combination of columns should have unique values.

```sql
UNIQUE (Email) NOT ENFORCED
```

### What does `NOT ENFORCED` mean?
The constraint is declared but the warehouse does not automatically reject data that violates it. The constraints may inform the query optimizer, but they do **not** guarantee data quality.

| Constraint | Purpose |
|---|---|
| PRIMARY KEY | Identifies a row |
| FOREIGN KEY | Describes a relationship |
| UNIQUE | Declares uniqueness |
| `NOT ENFORCED` | Does not automatically prevent violations |

**Exam tip:** A declared constraint is not the same as a validated data-quality rule.

---

## 7. `COPY INTO` — Loading Data

### What is `COPY INTO`?
`COPY INTO` loads data from external files into a Fabric Warehouse table.

### Typical workflow
1. Source files in storage (CSV, JSONL, or Parquet).
2. Use `COPY INTO` to load data.
3. Transform, validate, and query the warehouse table.

### File types listed in the supplied notes

| `FILE_TYPE` | Meaning |
|---|---|
| `CSV` | Comma-separated values |
| `JSONL` | JSON Lines; one JSON record per line |
| `PARQUET` | Columnar file format |

The supplied notes state that `ORC` is not included in Fabric Warehouse's `COPY INTO` file types, unlike the Synapse dedicated SQL pool version.

### Illustrative example
```sql
COPY INTO dbo.Sales
FROM 'https://storage.example.com/sales/'
WITH
(
    FILE_TYPE = 'CSV'
);
```

This is illustrative; a real command requires a valid source location and any required authentication/configuration.

**Exam tip:** Remember the listed types: **CSV, JSONL, PARQUET**.

---

## 8. CTAS — `CREATE TABLE AS SELECT`

### What is CTAS?
CTAS creates a new table from the result of a `SELECT` query.

```sql
CREATE TABLE dbo.HighValueSales
AS
SELECT *
FROM dbo.Sales
WHERE SalesAmount > 10000;
```

### Use case
Create a new curated table from existing data—for example, a table containing only completed sales.

**Remember:** `SELECT` retrieves data; CTAS creates a new table from query results.

---

## 9. `INSERT ... SELECT`

### What is it?
`INSERT ... SELECT` inserts query results into an **existing** destination table.

```sql
INSERT INTO dbo.CuratedSales
(
    SaleID,
    SalesAmount
)
SELECT
    SaleID,
    SalesAmount
FROM dbo.RawSales
WHERE Status = 'Completed';
```

### CTAS vs. `INSERT ... SELECT`

| Feature | CTAS | `INSERT ... SELECT` |
|---|---|---|
| Creates a new table | Yes | No |
| Destination must already exist | No | Yes |
| Typical use | Create a transformed table | Add query results to an existing table |

**Memory trick:**
- CTAS = Create a table.
- `INSERT ... SELECT` = Insert into an existing table.

---

## 10. Window Functions

### What is a window function?
A window function calculates across a set of related rows while retaining individual rows in the result.

Examples:
- `ROW_NUMBER()`
- `RANK()`
- `DENSE_RANK()`
- `SUM() OVER()`
- `AVG() OVER()`

### Example: Rank employees by salary
```sql
SELECT
    EmployeeName,
    Salary,
    RANK() OVER (ORDER BY Salary DESC) AS SalaryRank
FROM dbo.Employees;
```

### Example: Latest order per customer
```sql
WITH RankedOrders AS
(
    SELECT
        CustomerID,
        OrderID,
        OrderDate,
        ROW_NUMBER() OVER
        (
            PARTITION BY CustomerID
            ORDER BY OrderDate DESC, OrderID DESC
        ) AS rn
    FROM dbo.Orders
)
SELECT *
FROM RankedOrders
WHERE rn = 1;
```

This returns one latest order per customer, using `OrderID` as a tie-breaker when dates are equal.

### Common uses
- Ranking records
- Finding the latest record per customer
- Deduplication using ranking logic
- Running totals
- Sequential row numbers

---

## 11. Three-Part-Name Cross-Database Queries

### What is a three-part name?
It identifies an object using:

```text
DatabaseName.SchemaName.ObjectName
```

Example:

```sql
SELECT *
FROM SalesWarehouse.dbo.Customers;
```

| Part | Meaning |
|---|---|
| `SalesWarehouse` | Database/warehouse name |
| `dbo` | Schema |
| `Customers` | Table |

### Why use it?
It can reference objects in another warehouse/database in supported cross-database query scenarios, subject to Fabric's access and support rules.

**Exam tip:** Three-part-name cross-database queries are included in the supplied notes as supported and exam-relevant.

---

## 12. Quick Revision Table

| Concept | Remember this |
|---|---|
| T-SQL surface area | Fabric supports a subset of SQL Server T-SQL |
| Distributed `#temp` | `ROUND_ROBIN` distributes rows across distributions |
| Collation | Decide case behavior before warehouse creation |
| `MERGE` | Combines matching update and insert logic |
| `IDENTITY` | Preview-only with limitations in the supplied notes |
| `ROW_NUMBER()` | Sequential row numbers in query results |
| Constraints | PK, FK, UNIQUE are `NOT ENFORCED` in the supplied notes |
| `COPY INTO` | Loads external data into a warehouse table |
| CTAS | Creates a new table from a query |
| `INSERT ... SELECT` | Inserts query results into an existing table |
| Window functions | Ranking and calculations across related rows |
| Three-part names | `Database.Schema.Object` |

---

## 13. DP-700 Practice Quiz

### Q1. Which statement creates a new table from a query?
- A. `INSERT ... SELECT`
- B. `CREATE TABLE AS SELECT`
- C. `UPDATE`

**Answer: B — CTAS creates a new table from query results.**

### Q2. What constraint behavior is described in the supplied notes for Fabric Warehouse?
- A. All constraints are fully enforced
- B. Constraints are `NOT ENFORCED`
- C. Constraints cannot be declared

**Answer: B — Constraints are declared but not enforced.**

### Q3. Which function can assign sequential row numbers to query results?
- A. `ROW_NUMBER()`
- B. `COUNT()` only
- C. `MERGE()`

**Answer: A — `ROW_NUMBER()` assigns numbers based on the specified ordering.**

### Q4. Which statement combines matching updates and unmatched inserts?
- A. `COPY INTO`
- B. `MERGE`
- C. CTAS

**Answer: B — `MERGE` handles matching and insert/update logic.**

### Q5. Which `FILE_TYPE` is included in the supplied notes for Fabric `COPY INTO`?
- A. ORC
- B. PARQUET
- C. XML

**Answer: B — PARQUET is one of the listed file types.**

### Q6. When should warehouse collation be decided, according to the supplied notes?
- A. After loading data
- B. Before creating the warehouse
- C. After creating all tables

**Answer: B — Decide collation before warehouse creation.**

---

## 14. Final Memory Trick

- **Load:** `COPY INTO`
- **Create:** CTAS
- **Append:** `INSERT ... SELECT`
- **Upsert:** `MERGE`
- **Number rows:** `ROW_NUMBER()`
- **Declare relationships:** Constraints
- **Control text comparison:** Collation

---

## Official Reference

[Fabric Warehouse T-SQL surface area — Microsoft Learn](https://learn.microsoft.com/en-us/fabric/data-warehouse/tsql-surface-area)

> **Status reminder:** These notes explain the concepts and preserve the support-status claims from the supplied material. Check the official documentation for the latest feature availability and limitations before the exam.
