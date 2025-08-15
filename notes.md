# 📚 Complete SQL Guide

## 1. Introduction to SQL

**Definition:**
SQL (Structured Query Language) is used to manage and manipulate relational databases.

**Why it’s used:**
- Create, read, update, and delete data (CRUD).
- Define database structures.
- Control access and ensure data integrity.

---

## 2. SQL Basics

### 2.1 SQL Data Types

**Common Data Types:**

| Category    | Examples                                  |
|-------------|-------------------------------------------|
| Numeric     | `INT`, `BIGINT`, `DECIMAL`, `NUMERIC`, `FLOAT`, `REAL` |
| String/Text | `CHAR`, `VARCHAR`, `TEXT`                   |
| Date/Time   | `DATE`, `TIME`, `DATETIME`, `TIMESTAMP`     |
| Boolean     | `BOOLEAN`                                 |
| Binary      | `BLOB`, `VARBINARY`                         |

**DBMS-specific Notes:**
- **SQL Server:** Use `NVARCHAR` to store Unicode characters (e.g., for multiple languages). `VARCHAR` stores non-Unicode characters.
- **MySQL:** `VARCHAR` is standard. For larger text, `TEXT` types are available. `DATETIME` stores date and time, while `TIMESTAMP` is often used for `created_at` or `updated_at` columns as it can automatically update.
- **PostgreSQL:** Use `TEXT` for variable-length strings, as `VARCHAR(n)` has no performance advantage. `TIMESTAMP` is the standard for date and time. Use `TIMESTAMPTZ` for time zone-aware timestamps.

---

### 2.2 CREATE, DROP, ALTER TABLE

**CREATE TABLE (Real-world Example)**
Here's a more realistic example of creating a table, including various constraints and data types.

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE,
    PhoneNumber VARCHAR(20),
    HireDate DATE NOT NULL,
    JobID VARCHAR(10) NOT NULL,
    Salary DECIMAL(10, 2) CHECK (Salary > 0),
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
);
```

**DROP TABLE**
This command permanently deletes the table and all its data. Use with caution!
```sql
DROP TABLE Employees;
```

**ALTER TABLE**
You can add, drop, or modify columns.
```sql
-- Add a column
ALTER TABLE Employees ADD Bonus DECIMAL(5, 2) DEFAULT 0;

-- Drop a column
ALTER TABLE Employees DROP COLUMN PhoneNumber;

-- Modify a column's data type
ALTER TABLE Employees ALTER COLUMN JobID VARCHAR(15);
```

**DBMS-specific Syntax for ALTER TABLE:**
- **MySQL:** `ALTER TABLE Employees MODIFY COLUMN JobID VARCHAR(15);`
- **PostgreSQL:** `ALTER TABLE Employees ALTER COLUMN JobID TYPE VARCHAR(15);`
- **SQL Server:** `ALTER TABLE Employees ALTER COLUMN JobID VARCHAR(15);`

---

### 2.3 INSERT, UPDATE, DELETE

**Input Table: `Customers`**
| CustomerID | Name    | Country |
|------------|---------|---------|
| 1          | Alice   | USA     |
| 2          | Bob     | UK      |

**INSERT**
- **Single Row:**
```sql
INSERT INTO Customers (CustomerID, Name, Country)
VALUES (3, 'Charlie', 'USA');
```
- **Multiple Rows:**
```sql
INSERT INTO Customers (CustomerID, Name, Country) VALUES
(4, 'David', 'Australia'),
(5, 'Eve', 'Japan');
```
- **From another table:**
```sql
INSERT INTO EuropeanCustomers (CustomerID, Name)
SELECT CustomerID, Name FROM Customers WHERE Country IN ('UK', 'Germany');
```

**UPDATE**
- **Simple Update:**
```sql
UPDATE Customers
SET Country = 'Canada'
WHERE CustomerID = 2;
```
- **Complex Update with a JOIN (SQL Server Syntax):**
```sql
UPDATE o
SET o.Status = 'Shipped'
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
WHERE c.Country = 'USA';
```
**Note:** The `UPDATE` with `JOIN` syntax varies across DBMS. For example, in PostgreSQL it would be `UPDATE Orders SET Status = 'Shipped' FROM Customers WHERE Orders.CustomerID = Customers.CustomerID AND Customers.Country = 'USA';` and in MySQL it would be `UPDATE Orders o JOIN Customers c ON o.CustomerID = c.CustomerID SET o.Status = 'Shipped' WHERE c.Country = 'USA';`.

**DELETE**
- **Simple Delete:**
```sql
DELETE FROM Customers
WHERE CustomerID = 1;
```
- **Delete all rows (but keep the table structure):**
```sql
DELETE FROM Customers;
-- Or, for a faster, non-logged operation in some DBMS:
TRUNCATE TABLE Customers;
```
**Note:** `TRUNCATE TABLE` is a DDL command and cannot be rolled back in a transaction in some DBMS (like SQL Server and MySQL, though in PostgreSQL it is transaction-safe). It also resets identity columns.

---

### 2.4 SELECT & FROM
```sql
SELECT Name, Country
FROM Customers;
```

---

### 2.5 WHERE clause
```sql
SELECT * FROM Customers
WHERE Country = 'USA';
```

---

### 2.6 ORDER BY, LIMIT
```sql
SELECT * FROM Customers
ORDER BY Name ASC
LIMIT 5;
```

---

## 3. Filtering & Aggregation

### 3.1 DISTINCT
```sql
SELECT DISTINCT Country
FROM Customers;
```

**Performance Consideration:**
`DISTINCT` can be slow on large datasets because the database needs to sort the data to find the unique values. If you only need to check for the existence of a value, using `EXISTS` with a subquery might be more performant.

**Alternative to `DISTINCT` on multiple columns:**
Sometimes, `GROUP BY` can be a more flexible alternative to `SELECT DISTINCT` on multiple columns, as it allows for aggregation.

---

### 3.2 GROUP BY

**Input Table: `Orders`**
| OrderID | CustomerID | Amount | OrderDate |
|---------|------------|--------|-----------|
| 1       | 1          | 100    | 2023-01-05|
| 2       | 2          | 150    | 2023-01-05|
| 3       | 1          | 200    | 2023-01-06|
| 4       | 1          | 50     | 2023-02-10|
| 5       | 2          | 300    | 2023-02-12|


**Simple `GROUP BY`:**
```sql
SELECT CustomerID, SUM(Amount) AS TotalSpent
FROM Orders
GROUP BY CustomerID;
```
**Output:**
| CustomerID | TotalSpent |
|------------|------------|
| 1          | 350        |
| 2          | 450        |

**`GROUP BY` multiple columns:**
This example groups by both customer and the month of the order.
```sql
SELECT
    CustomerID,
    EXTRACT(MONTH FROM OrderDate) AS OrderMonth,
    SUM(Amount) AS MonthlyTotal
FROM Orders
GROUP BY CustomerID, OrderMonth
ORDER BY CustomerID, OrderMonth;
```
**Note:** `EXTRACT` is standard SQL, but date functions can vary. For example, in SQL Server you might use `MONTH(OrderDate)`.

---

### 3.3 HAVING
The `HAVING` clause is used to filter groups created by `GROUP BY`. `WHERE` filters rows before aggregation, `HAVING` filters groups after aggregation.

**Simple `HAVING`:**
```sql
SELECT CustomerID, SUM(Amount) AS TotalSpent
FROM Orders
GROUP BY CustomerID
HAVING SUM(Amount) > 400;
```
**Output:**
| CustomerID | TotalSpent |
|------------|------------|
| 2          | 450        |

**Complex `HAVING`:**
```sql
SELECT
    CustomerID,
    EXTRACT(MONTH FROM OrderDate) AS OrderMonth,
    SUM(Amount) AS MonthlyTotal
FROM Orders
GROUP BY CustomerID, OrderMonth
HAVING SUM(Amount) > 150 AND COUNT(OrderID) >= 1;
```

---

### 3.4 Aggregate Functions
```sql
SELECT COUNT(*) AS TotalOrders,
       AVG(Amount) AS AvgAmount,
       MAX(Amount) AS MaxAmount,
       MIN(Amount) AS MinAmount
FROM Orders;
```

**Common Pitfalls & Notes:**
- **`COUNT(*)` vs `COUNT(column)`:** `COUNT(*)` counts all rows, while `COUNT(column)` counts only non-NULL values in that column.
- **`AVG()` with `NULL` values:** `AVG()` ignores `NULL` values. If you want to treat `NULL` as 0, you need to use `AVG(COALESCE(column, 0))`.
- **Mixing aggregate and non-aggregate columns:** Any non-aggregate column in the `SELECT` list must be included in the `GROUP BY` clause. This is a common error for beginners.
- **`DISTINCT` with Aggregates:** You can use `DISTINCT` inside aggregate functions, e.g., `COUNT(DISTINCT CustomerID)` to count the number of unique customers.

---

## 4. Joins & Relationships
Joins are used to combine rows from two or more tables, based on a related column between them.

**Sample Tables for Joins:**
**`Employees`**
| EmployeeID | Name    | ManagerID |
|------------|---------|-----------|
| 1          | Alice   | 3         |
| 2          | Bob     | 3         |
| 3          | Charlie | NULL      |
| 4          | David   | 3         |

**`Departments`**
| DepartmentID | DepartmentName |
|--------------|----------------|
| 1            | Engineering    |
| 2            | HR             |
| 3            | Sales          |

**`EmployeeDepartments`**
| EmployeeID | DepartmentID |
|------------|--------------|
| 1          | 1            |
| 2          | 2            |
| 3          | 2            |

### 4.1 INNER JOIN
Returns records that have matching values in both tables. This is the most common type of join.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
INNER JOIN EmployeeDepartments ed ON e.EmployeeID = ed.EmployeeID
INNER JOIN Departments d ON ed.DepartmentID = d.DepartmentID;
```
**Output:**
| Name    | DepartmentName |
|---------|----------------|
| Alice   | Engineering    |
| Bob     | HR             |
| Charlie | HR             |

---

### 4.2 LEFT JOIN (or LEFT OUTER JOIN)
Returns all records from the left table (`Employees`), and the matched records from the right table. The result is NULL from the right side if there is no match.

```sql
SELECT e.Name, ed.DepartmentID
FROM Employees e
LEFT JOIN EmployeeDepartments ed ON e.EmployeeID = ed.EmployeeID;
```
**Output:** (Notice David has a NULL DepartmentID as he is not in the `EmployeeDepartments` table)
| Name    | DepartmentID |
|---------|--------------|
| Alice   | 1            |
| Bob     | 2            |
| Charlie | 2            |
| David   | NULL         |

---

### 4.3 RIGHT JOIN (or RIGHT OUTER JOIN)
Returns all records from the right table, and the matched records from the left table. The result is NULL from the left side when there is no match.

```sql
SELECT d.DepartmentName, ed.EmployeeID
FROM Departments d
RIGHT JOIN EmployeeDepartments ed ON d.DepartmentID = ed.DepartmentID;
```
**Note:** `RIGHT JOIN` is less common than `LEFT JOIN`. You can usually rewrite a `RIGHT JOIN` as a `LEFT JOIN` by swapping the tables.

---

### 4.4 FULL OUTER JOIN
Returns all records when there is a match in either left or right table. It's effectively a `LEFT JOIN` and `RIGHT JOIN` combined.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
FULL OUTER JOIN EmployeeDepartments ed ON e.EmployeeID = ed.EmployeeID
FULL OUTER JOIN Departments d ON ed.DepartmentID = d.DepartmentID;
```
**Note:** `FULL OUTER JOIN` can produce large result sets. Not all DBMS support it (e.g., MySQL).

---

### 4.5 CROSS JOIN
Returns the Cartesian product of the two tables. This means it returns every possible combination of rows from the tables.

```sql
-- This will return 4 * 3 = 12 rows.
SELECT e.Name, d.DepartmentName
FROM Employees e
CROSS JOIN Departments d;
```
**Use Case:** Useful for generating all possible combinations for a report, e.g., all employees and all products.

---

### 4.6 SELF JOIN
A table is joined with itself. This is useful for querying hierarchical data or comparing rows within the same table.

```sql
SELECT
    e1.Name AS Employee,
    e2.Name AS Manager
FROM Employees e1
LEFT JOIN Employees e2 ON e1.ManagerID = e2.EmployeeID;
```
**Output:**
| Employee | Manager |
|----------|---------|
| Alice    | Charlie |
| Bob      | Charlie |
| Charlie  | NULL    |
| David    | Charlie |

---

### 4.7 `USING` vs `ON` clause
- `ON`: You specify the join condition explicitly, e.g., `ON t1.id = t2.id`. This is the most common and flexible way.
- `USING`: A shorthand for `ON` when the columns to join on have the same name in both tables, e.g., `USING (CustomerID)`. It's less verbose but requires column names to match. Not all DBMS support `USING`.

---

### 4.8 Join Performance
- **Indexes are key:** Joins on columns with indexes are significantly faster. Ensure your foreign key columns are indexed.
- **`INNER JOIN` is fastest:** It usually performs better than outer joins as it deals with a smaller dataset.
- **Avoid `CROSS JOIN` on large tables:** It can produce a massive number of rows and be very slow.
- **Filter early:** Use `WHERE` clauses to filter data as early as possible to reduce the number of rows that need to be joined.

---

## 5. Subqueries & Derived Tables
A subquery, or inner query, is a query nested inside another SQL query.

### 5.1 Subqueries in `WHERE` clause

**Single-row Subquery:** The inner query returns only one value.
```sql
-- Find employees who have the same job as 'Alice'
SELECT Name
FROM Employees
WHERE JobID = (SELECT JobID FROM Employees WHERE Name = 'Alice');
```

**Multi-row Subquery:** The inner query returns a list of values. Use with `IN`, `NOT IN`, `ANY`, `ALL`.
```sql
-- Find customers who have placed an order
SELECT Name
FROM Customers
WHERE CustomerID IN (SELECT CustomerID FROM Orders);
```

### 5.2 Correlated Subquery
A subquery that depends on the outer query for its values. It is evaluated once for each row processed by the outer query, which can be inefficient.

```sql
-- Find employees whose salary is above the average for their department
SELECT e1.Name, e1.Salary, e1.DepartmentID
FROM Employees e1
WHERE e1.Salary > (
    SELECT AVG(e2.Salary)
    FROM Employees e2
    WHERE e2.DepartmentID = e1.DepartmentID
);
```

### 5.3 Subqueries in `FROM` clause (Derived Tables)
A subquery in the `FROM` clause creates a temporary table (a "derived table" or "inline view") that the outer query can select from.

```sql
SELECT ds.DepartmentName, ds.AvgSalary
FROM (
    SELECT d.DepartmentName, AVG(e.Salary) as AvgSalary
    FROM Employees e
    JOIN Departments d ON e.DepartmentID = d.DepartmentID
    GROUP BY d.DepartmentName
) AS ds
WHERE ds.AvgSalary > 50000;
```

### 5.4 Subqueries in `SELECT` clause (Scalar Subqueries)
A subquery that returns a single value (one row, one column).

```sql
SELECT
    e.Name,
    (SELECT d.DepartmentName FROM Departments d WHERE d.DepartmentID = e.DepartmentID) AS Department
FROM Employees e;
```

---

### 5.5 Joins vs. Subqueries
- **Readability:** Joins are often more readable than complex subqueries, especially correlated subqueries.
- **Performance:** Modern database optimizers can often rewrite subqueries as joins, making the performance similar. However, `JOIN`s are sometimes more performant as the optimizer has more information about the data relationships. A correlated subquery is often less performant than a `JOIN`.
- **Flexibility:** Subqueries can sometimes do things that are difficult with joins, e.g., when you need to aggregate data before joining.

**General Rule:** If you can write it easily as a `JOIN`, that's usually the better option. If you need to perform an aggregation on a table before you join it to another, a subquery (or a CTE) is the way to go.

---

## 6. Constraints & Keys
Constraints are rules enforced on data columns to ensure the accuracy and reliability of the data.

### 6.1 `PRIMARY KEY`
- Uniquely identifies each record in a table.
- Must contain unique values and cannot contain `NULL` values.
- A table can have only one primary key.

### 6.2 `FOREIGN KEY`
- A key used to link two tables together.
- It is a field (or collection of fields) in one table that refers to the `PRIMARY KEY` in another table.
- The table with the foreign key is called the child table, and the table with the primary key is called the referenced or parent table.

### 6.3 `UNIQUE`
- Ensures that all values in a column are different.
- A table can have many `UNIQUE` constraints.

### 6.4 `NOT NULL`
- Ensures that a column cannot have a `NULL` value.

### 6.5 `DEFAULT`
- Provides a default value for a column when none is specified.

### 6.6 `CHECK`
- Ensures that all values in a column satisfy a specific condition.

**Example with all constraints:**
```sql
CREATE TABLE OrderItems (
    OrderItemID INT PRIMARY KEY,
    OrderID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL CHECK (Quantity > 0),
    UnitPrice DECIMAL(10, 2) NOT NULL,
    Status VARCHAR(20) DEFAULT 'Pending',
    UNIQUE (OrderID, ProductID), -- A product can only appear once per order
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
```

### 6.7 Composite Keys
A composite key is a primary key composed of multiple columns. It's used when a single column is not sufficient to uniquely identify a record.

**Example:** In the `OrderItems` table above, `(OrderID, ProductID)` is a `UNIQUE` composite key. We could also define it as a composite `PRIMARY KEY`:
```sql
CREATE TABLE OrderItems (
    OrderID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    PRIMARY KEY (OrderID, ProductID) -- Composite Primary Key
);
```

---

## 7. Indexes & Performance
Indexes are special lookup tables that the database search engine can use to speed up data retrieval. Think of it like the index in the back of a book.

### 7.1 How Indexes Work
- When you create an index on a column, the database creates a sorted data structure (often a B-Tree) that stores the column values and a pointer to the corresponding row.
- When you query with a `WHERE` clause on an indexed column, the database can use the index to quickly find the data, instead of scanning the entire table.

### 7.2 Creating and Dropping Indexes
**Create a simple index:**
```sql
CREATE INDEX idx_lastname ON Employees (LastName);
```
**Create a composite index (on multiple columns):**
```sql
CREATE INDEX idx_name ON Employees (LastName, FirstName);
```
**Drop an index:**
```sql
DROP INDEX idx_lastname ON Employees;
```
**Note:** The syntax for dropping an index can vary. For example, in MySQL you might use `DROP INDEX idx_lastname ON Employees;`.

### 7.3 Clustered vs. Non-Clustered Indexes
- **Clustered Index:** Determines the physical order of data in a table. Because of this, a table can have only one clustered index. The primary key of a table is often the clustered index.
- **Non-Clustered Index:** Has a structure separate from the data rows. It contains the index key values and a pointer to the data row. A table can have multiple non-clustered indexes.

### 7.4 Index Best Practices & Performance
- **Index frequently queried columns:** Add indexes to columns that are often used in `WHERE` clauses and `JOIN` conditions.
- **Don't over-index:** Indexes speed up reads (`SELECT`) but slow down writes (`INSERT`, `UPDATE`, `DELETE`) because the index also needs to be updated.
- **Index foreign keys:** It's a very good practice to index all your foreign key columns.
- **Use composite indexes:** If you often query on multiple columns together, a composite index can be very effective. The order of columns in the index matters.
- **Index maintenance:** Over time, as data is inserted and deleted, indexes can become fragmented, which can reduce performance. Regularly rebuilding or reorganizing indexes can be necessary.
  - **SQL Server:** `ALTER INDEX ALL ON TableName REBUILD;`
  - **PostgreSQL:** `REINDEX TABLE TableName;`
  - **MySQL:** `OPTIMIZE TABLE TableName;`

---

## 8. Advanced SQL

### 8.1 CTE (Common Table Expressions)
A CTE allows you to create a temporary, named result set that you can reference within a `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement. CTEs help break down complex queries into simpler, more readable logical blocks.

**Recursive CTE:** A CTE that references itself. Useful for querying hierarchical data like organizational charts.
```sql
WITH RECURSIVE EmployeeHierarchy AS (
    -- Anchor member
    SELECT EmployeeID, Name, ManagerID, 1 AS Level
    FROM Employees
    WHERE ManagerID IS NULL
    UNION ALL
    -- Recursive member
    SELECT e.EmployeeID, e.Name, e.ManagerID, eh.Level + 1
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT * FROM EmployeeHierarchy;
```
**Note:** The syntax for recursive CTEs can vary (`RECURSIVE` keyword is not needed in SQL Server).

---

### 8.2 Window Functions
Window functions perform calculations across a set of rows that are related to the current row. Unlike aggregate functions, they do not collapse rows.

**Common Window Functions:**
- `ROW_NUMBER()`: Assigns a unique number to each row.
- `RANK()`, `DENSE_RANK()`: Assign a rank based on ordering. `RANK` may have gaps in ranking, `DENSE_RANK` will not.
- `LEAD()`, `LAG()`: Access data from subsequent or previous rows.
- `SUM()`, `AVG()`, `COUNT()` as window functions: e.g., `SUM(Amount) OVER (PARTITION BY CustomerID)`.

**Example: Running Total**
```sql
SELECT
    OrderDate,
    Amount,
    SUM(Amount) OVER (ORDER BY OrderDate) AS RunningTotal
FROM Orders;
```

---

### 8.3 PIVOT & UNPIVOT
- `PIVOT`: Rotates rows into columns.
- `UNPIVOT`: Rotates columns into rows.

**PIVOT Example (SQL Server Syntax):**
```sql
SELECT Department, [2022] AS Sales2022, [2023] AS Sales2023
FROM (
    SELECT Department, YEAR(OrderDate) AS OrderYear, Sales
    FROM SalesData
) AS SourceTable
PIVOT (
    SUM(Sales)
    FOR OrderYear IN ([2022], [2023])
) AS PivotTable;
```
**Note:** `PIVOT` syntax is not standard. In PostgreSQL and other DBMS, you often use `CASE` statements with `GROUP BY` to achieve the same result.

---

### 8.4 CASE WHEN
The `CASE` statement is SQL's way of handling if/then/else logic.

**Searched `CASE`:**
```sql
SELECT Name, Salary,
    CASE
        WHEN Salary < 50000 THEN 'Low'
        WHEN Salary >= 50000 AND Salary < 100000 THEN 'Medium'
        ELSE 'High'
    END AS SalaryBracket
FROM Employees;
```

---

### 8.5 Transactions
A transaction is a sequence of operations performed as a single logical unit of work. All statements must succeed, or none of them are applied. This is known as Atomicity.

**ACID Properties:**
- **Atomicity:** All or nothing.
- **Consistency:** The database remains in a consistent state.
- **Isolation:** Concurrent transactions do not affect each other.
- **Durability:** Once a transaction is committed, it will remain so.

```sql
BEGIN TRANSACTION;
UPDATE Accounts SET Balance = Balance - 100 WHERE AccountID = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE AccountID = 2;
-- If there was an error, we could ROLLBACK;
COMMIT;
```

---

### 8.6 Triggers
A trigger is a special type of stored procedure that automatically runs when an event (like `INSERT`, `UPDATE`, `DELETE`) occurs.

**Use Case:** Maintaining an audit trail.
```sql
CREATE TRIGGER trg_EmployeeUpdate
AFTER UPDATE ON Employees
FOR EACH ROW
BEGIN
    INSERT INTO EmployeeAudit (EmployeeID, OldSalary, NewSalary, ChangeDate)
    VALUES (OLD.EmployeeID, OLD.Salary, NEW.Salary, NOW());
END;
```
**Note:** `OLD` and `NEW` keywords allow you to access the row's state before and after the change. Syntax varies between DBMS.

---

### 8.7 Views
A view is a virtual table based on the result-set of an SQL statement.

**Use Cases:**
- Simplify complex queries.
- Provide a layer of security by restricting access to certain columns or rows.
- **Updatable Views:** In some cases, you can `INSERT` or `UPDATE` a view, and the changes will be applied to the underlying table.

```sql
CREATE OR REPLACE VIEW HighValueCustomers AS
SELECT CustomerID, Name, Country
FROM Customers
WHERE CustomerID IN (SELECT CustomerID FROM Orders GROUP BY CustomerID HAVING SUM(Amount) > 1000);
```

---

### 8.8 User-defined Functions (UDFs)
A function you can create and reuse in your queries.

**Types:**
- **Scalar UDF:** Returns a single value.
- **Table-valued UDF:** Returns a table.

**Example (PostgreSQL Syntax):**
```sql
CREATE FUNCTION get_employee_count_by_dept(dept_id INT)
RETURNS INT AS $$
DECLARE
    employee_count INT;
BEGIN
    SELECT COUNT(*) INTO employee_count FROM Employees WHERE DepartmentID = dept_id;
    RETURN employee_count;
END;
$$ LANGUAGE plpgsql;
```

---

### 8.9 Query Optimization & Execution Plans
- **Execution Plan:** The sequence of steps the database will take to execute your query. Reviewing the execution plan (e.g., using `EXPLAIN` in PostgreSQL/MySQL or checking the "Actual Execution Plan" in SQL Server) is crucial for diagnosing performance issues.
- **Common Issues:**
  - **Table Scans:** The database has to read every single row in a table. This is often fixed by adding an index.
  - **Bad Cardinality Estimates:** The optimizer misjudges the number of rows, leading to a suboptimal plan.
- **General Tips:**
  - `SELECT` only the columns you need, avoid `SELECT *`.
  - Use `EXISTS` instead of `IN` when you just need to check for existence and the subquery is large.
  - Understand your data and how the database works.
