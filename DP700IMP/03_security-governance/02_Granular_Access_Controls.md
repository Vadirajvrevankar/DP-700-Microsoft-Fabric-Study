# DP-700 — Granular Access Controls Notes

## 1. BIG PICTURE

Fabric has several security layers:

```text
                    DATA ACCESS
                        |
        +---------------+----------------+
        |               |                |
        v               v                v
       RLS             CLS              OLS
  Row Security    Column Security   Object Security
        |               |                |
        +---------------+----------------+
                        |
                   SQL Endpoint
                  / Fabric Warehouse

              OneLake Data Access Roles
                        |
              OneLake / Spark / Files

                 Dynamic Data Masking
                        |
                 Hide sensitive values
```

### Memory Trick

> **RLS = ROW**  
> **CLS = COLUMN**  
> **OLS = OBJECT**  
> **DDM = MASK VALUE**  
> **OneLake roles = CROSS-ENGINE DATA ACCESS**

---

# 2. RLS — ROW-LEVEL SECURITY

## What it does

RLS controls **which rows** a user can see or modify.

Example:

```text
Sales Table

India     ₹10,000
USA       ₹20,000
UK        ₹15,000
```

If the user is allowed to see only India:

```text
India     ₹10,000
```

The other rows are filtered.

## How RLS is implemented

RLS uses:

```sql
CREATE SECURITY POLICY
```

with a **filter predicate** implemented using a schema-bound inline table-valued function.

### Operations affected

- SELECT
- UPDATE
- DELETE

### Memory Trick

> **RLS = WHICH ROWS CAN I SEE?**

---

# 3. CLS — COLUMN-LEVEL SECURITY

## What it does

CLS controls access to **specific columns**.

Example:

```text
Customer
├── Name
├── Email
├── Phone
└── Salary
```

A user may access:

```text
Name
Email
Phone
```

but not:

```text
Salary
```

## How CLS works

CLS uses:

```sql
GRANT SELECT ON table(columns)
```

## Very Important Exam Point

CLS does **not** silently remove the unauthorized column.

If a query requests a column the user cannot access:

```sql
SELECT Name, Salary
FROM Customer;
```

the query gets a:

> **Permission error**

It does NOT silently return only `Name`.

### Memory Trick

> **CLS = WHICH COLUMNS CAN I ACCESS?**

---

# 4. OLS — OBJECT-LEVEL SECURITY

## What it does

OLS controls access to database objects.

Think:

```text
Database
├── Schema
├── Table
├── View
└── Other SQL objects
```

Access is controlled using:

```sql
GRANT
REVOKE
DENY
```

## VERY IMPORTANT: DENY WINS

Example:

```text
Role A → GRANT SELECT
Role B → DENY SELECT
```

If the user belongs to both roles:

```text
DENY wins
```

### Memory Trick

> **DENY ALWAYS WINS**

---

# 5. RLS vs CLS vs OLS

| Security | Controls | Question |
|---|---|---|
| **RLS** | Rows | Which rows? |
| **CLS** | Columns | Which columns? |
| **OLS** | Objects | Which objects? |

### Easy Example

Suppose:

```text
Sales Table

Country | Customer | Revenue | Salary
```

You could apply:

**RLS**

> User can see only India rows.

**CLS**

> User cannot access Salary.

**OLS**

> User cannot access the Sales table.

### Memory

> **ROW → COLUMN → OBJECT**

---

# 6. DDM — DYNAMIC DATA MASKING

## What it does

DDM **masks sensitive values**.

Example:

```text
Actual:
9876543210

Displayed:
xxxx
```

The important point:

> **DDM hides the value; it does not replace access control.**

---

# 7. DDM IS NOT A REPLACEMENT FOR SECURITY

DDM:

```text
User has access
      |
      v
Value is masked
```

It does NOT mean:

```text
User has no access
```

Therefore, DDM can be layered with:

```text
RLS + CLS + OLS
        +
       DDM
```

### Memory Trick

> **DDM = HIDE**

> **RLS/CLS/OLS = CONTROL ACCESS**

---

# 8. SQL-ENDPOINT SECURITY vs ONELAKE SECURITY

This is one of the **most important exam areas**.

SQL-endpoint security:

```text
RLS
CLS
OLS
 |
 v
SQL analytics endpoint / Warehouse
```

These SQL security controls apply to SQL queries through the SQL endpoint/Warehouse.

For access through:

```text
Spark
OneLake APIs
Files
```

use appropriate **OneLake data access roles** for equivalent OneLake-level restrictions.

---

# 9. ONELAKE DATA ACCESS ROLES

OneLake data access roles are useful when access needs to work across OneLake-based access paths.

Conceptually:

```text
                    OneLake
                       |
           +-----------+-----------+
           |           |           |
           v           v           v
         Spark     OneLake API   Files
           |           |           |
           +-----------+-----------+
                       |
             OneLake Data Role
```

### Exam Clue

> "The same restriction must apply when data is accessed by SQL and Spark/OneLake."

Think:

**OneLake data access role**

### Memory Trick

> **ONE ENGINE → SQL SECURITY**

> **MULTIPLE ENGINES → ONELAKE SECURITY**

---

# 10. SECURITY LAYER DECISION

Use this decision tree:

```text
What are you trying to restrict?
             |
     +-------+-------+
     |       |       |
     v       v       v
    ROW   COLUMN   OBJECT
     |       |       |
     v       v       v
    RLS     CLS     OLS
```

If you're trying to **hide values**:

```text
Sensitive value
      |
      v
     DDM
```

If the restriction must work across multiple OneLake engines:

```text
Cross-engine restriction
          |
          v
OneLake data access role
```

---

# 11. SQL ROLES — BEST PRACTICE

Do not create the same security grants individually for every user when a role can be used.

Prefer:

```text
Users
  |
  v
SQL Role
  |
  v
RLS / CLS / OLS grants
```

Example:

```text
User A ─┐
User B ─┼──> SalesAnalyst Role
User C ─┘             |
                       v
                  SQL Grants
```

This keeps permission management manageable at scale.

### Memory Trick

> **GRANTS → ROLES, NOT USERS**

---

# 12. SCHEMABINDING

For RLS predicate functions, use:

```sql
WITH SCHEMABINDING
```

as the default/best practice unless there is a specific reason not to.

### Memory Trick

> **RLS FUNCTION → SCHEMABINDING**

---

# 13. SECURITY LAYERS — COMPLETE COMPARISON

| Feature | Restricts | Main purpose |
|---|---|---|
| **RLS** | Rows | Filter which rows a user can access |
| **CLS** | Columns | Restrict specific columns |
| **OLS** | Objects | Restrict database objects |
| **DDM** | Values | Mask sensitive data |
| **OneLake Data Access Role** | OneLake data | Cross-engine/file-level access control |

---

# 14. COMMON EXAM SCENARIOS

## Scenario 1

> Users should see only rows belonging to their department.

**Answer → RLS**

---

## Scenario 2

> Users can access the table but must not access the Salary column.

**Answer → CLS**

---

## Scenario 3

> Users must not access a particular table/object.

**Answer → OLS**

---

## Scenario 4

> Sensitive values should be hidden, but the user still has access to the column.

**Answer → DDM**

---

## Scenario 5

> A query requests a column the user doesn't have CLS permission for.

**Answer → Hard permission error**

Not silently removed.

---

## Scenario 6

> User gets GRANT through one role and DENY through another.

**Answer → DENY wins**

---

## Scenario 7

> Restriction must apply to Spark and OneLake access as well as SQL.

**Answer → OneLake data access role**

---

## Scenario 8

> Hundreds of users need the same CLS/OLS permissions.

**Best practice → Assign permissions to a SQL role**

---

## Scenario 9

> RLS predicate function needs the recommended definition.

**Answer → Use `WITH SCHEMABINDING`**

---

# 15. COMMON EXAM TRAPS

## Trap 1

> DDM protects the data completely.

**WRONG**

DDM masks values but does not replace access control.

---

## Trap 2

> CLS removes unauthorized columns from `SELECT *`.

**WRONG**

The query fails with a permission error when an inaccessible column is requested.

---

## Trap 3

> SQL RLS automatically protects Spark.

**WRONG**

SQL-endpoint RLS/CLS/OLS apply to SQL analytics endpoint/Warehouse queries. Cross-engine OneLake access needs appropriate OneLake data access controls.

---

## Trap 4

> GRANT always wins over DENY.

**WRONG**

> **DENY wins.**

---

## Trap 5

> Give every user individual SQL grants.

**NOT BEST PRACTICE**

Prefer:

```text
Users → SQL Role → Grants
```

---

# 16. FINAL MEMORY SHEET

```text
RLS → ROW
CLS → COLUMN
OLS → OBJECT
DDM → MASK VALUE

RLS
→ CREATE SECURITY POLICY
→ Filter predicate
→ SELECT / UPDATE / DELETE

CLS
→ GRANT SELECT ON table(columns)
→ Missing column = permission error

OLS
→ GRANT / REVOKE / DENY
→ DENY WINS

DDM
→ Masks values
→ Does NOT replace access control

SQL RLS/CLS/OLS
→ SQL endpoint / Warehouse

CROSS-ENGINE
→ OneLake Data Access Role

BEST PRACTICE
→ Grants to SQL ROLES
→ RLS function → WITH SCHEMABINDING
```

---

# 17. ONE-LINE DP-700 EXAM TRICKS

> **Which rows? → RLS**

> **Which columns? → CLS**

> **Which objects? → OLS**

> **Hide the value? → DDM**

> **Spark + OneLake + SQL restriction? → OneLake Data Access Role**

> **CLS missing column? → Query fails**

> **GRANT + DENY? → DENY wins**

> **Many users with same permissions? → SQL Role**

> **RLS predicate function? → SCHEMABINDING**

> **DDM replaces RLS? → NO**
