# SQL Query Documentation: Overdue Unshipped Orders

## 1. Summary

This query identifies orders that are past their required delivery date but have not yet been shipped. It answers the critical business question: **"Which orders are overdue, by how many days, and who is responsible for them?"** This is essential for SLA management, customer service escalation, and operational accountability.

## 2. Tables Involved

| Table | Role |
|-------|------|
| **Orders** | Primary fact table containing order dates, shipping status, and required delivery dates |
| **Customers** | Dimension table providing customer company names for communication |
| **Employees** | Dimension table identifying the sales representative responsible for each order |
| **Order Details** | Fact table used to calculate total order value (quantity, pricing, discounts) |

## 3. Key Logic

- **Joins**: Four-table join connecting orders → customers, employees, and order details
- **Primary Filter**: `WHERE o.ShippedDate IS NULL` identifies unshipped orders
- **Date Filter**: `AND o.RequiredDate < GETDATE()` restricts to orders past their required date
- **Date Calculation**: `DATEDIFF(day, o.RequiredDate, GETDATE())` computes days overdue (positive number = late)
- **Value Calculation**: Sums line-item values with discount formula: `Quantity × UnitPrice × (1 - Discount)`
- **Aggregation**: Groups by order and related attributes to calculate total order value
- **Sorting**: Orders by DaysOverdue descending to prioritize most critical delays

## 4. Output Columns

| Column | Description |
|--------|-------------|
| **OrderID** | Unique order identifier for tracking and reference |
| **Customer** | Company name of the customer awaiting delivery |
| **SalesRep** | Full name of the employee/sales representative responsible for the order |
| **OrderDate** | Date when the order was originally placed |
| **RequiredDate** | Customer's requested/promised delivery date |
| **DaysOverdue** | Number of days past the required date (calculated from current date) |
| **OrderValue** | Total monetary value of the order after discounts |

## 5. Business Use Case

**Primary Users:**
- **Operations Managers**: Prioritize fulfillment and shipping activities to reduce backlog
- **Customer Service Teams**: Proactively contact customers about delays and manage expectations
- **Sales Representatives**: Take accountability for their overdue orders and escalate issues
- **Logistics Coordinators**: Identify bottlenecks in the supply chain or warehouse
- **Executive Leadership**: Monitor SLA compliance and operational performance

**When to Use:**
- Daily operational reviews and stand-up meetings
- Customer service escalation workflows
- Performance reviews for sales and operations teams
- Root cause analysis for fulfillment delays
- SLA breach reporting and remediation planning

## 6. Assumptions & Limitations

**Assumptions:**
- `ShippedDate IS NULL` accurately indicates unshipped status (not data entry lag)
- `RequiredDate` represents the customer's actual expectation or contractual SLA
- `GETDATE()` provides accurate current date/time for calculation
- Sales representative assignment (EmployeeID) is current and accurate

**Limitations:**
- **Dynamic calculation**: DaysOverdue changes daily; results are point-in-time snapshots
- **No root cause**: Query doesn't explain *why* orders are delayed (inventory, logistics, etc.)
- **No priority weighting**: All overdue orders treated equally regardless of customer importance or order value
- **Timezone considerations**: GETDATE() uses server time; may not match customer's timezone
- **Historical data**: If run on old Northwind sample data, all orders may appear overdue

**Edge Cases:**
- Orders placed today with RequiredDate in the past will appear (unusual but possible)
- Orders with NULL RequiredDate are excluded (no way to determine if overdue)
- Partially shipped orders (if system allows) still appear as unshipped
- Orders cancelled but not marked as shipped will appear as overdue
- Very old unshipped orders may indicate data quality issues rather than operational problems

## 7. Test Cases

**Note**: The Northwind sample database contains historical data from 1996-1998. When run today, all unshipped orders will show as severely overdue (9,000+ days). The test cases below assume the query is run during the database's time period for realistic scenarios.

### 7.1 Positive Test Cases

| Test ID | Test Description | Expected Result (if run in 1998) | Validation Method |
|---------|-----------------|----------------------------------|-------------------|
| **PT-01** | **No Overdue Orders in Production** | Row count: 0<br>(All orders shipped on time in Northwind sample data) | In production Northwind, typically no unshipped orders exist past RequiredDate |
| **PT-02** | **Query Executes Successfully** | Query completes without errors<br>Returns 0 or more rows | Verify query syntax and joins are correct |
| **PT-03** | **DaysOverdue Calculation** | DaysOverdue = DATEDIFF(day, RequiredDate, GETDATE())<br>Positive integer for overdue orders | Verify date arithmetic is correct |
| **PT-04** | **Only Unshipped Orders** | All returned orders have ShippedDate IS NULL | Verify WHERE clause filters correctly |
| **PT-05** | **Sorted by Most Overdue First** | First row has highest DaysOverdue<br>Last row has lowest DaysOverdue | Verify ORDER BY DaysOverdue DESC |

### 7.2 Edge Case Test Cases

| Test ID | Test Description | Expected Behavior | Validation Query |
|---------|-----------------|-------------------|------------------|
| **EC-01** | **Order Required Today** | Orders with RequiredDate = GETDATE() should NOT appear (not yet overdue) | WHERE clause: `RequiredDate < GETDATE()` excludes today |
| **EC-02** | **Order with NULL RequiredDate** | Orders without RequiredDate should be excluded | NULL RequiredDate cannot be compared with GETDATE() |
| **EC-03** | **Multi-Line Order Aggregation** | Orders with multiple line items should show total OrderValue (sum of all lines) | GROUP BY OrderID aggregates correctly |
| **EC-04** | **Historical Data Scenario** | When run today (2026), all unshipped 1998 orders show ~9,800+ DaysOverdue | DATEDIFF calculates correctly across decades |

### 7.3 Manual Validation Queries

#### Sanity Check: Count All Unshipped Orders (Regardless of Date)

```sql
-- Check total unshipped orders in the database
SELECT
    COUNT(*) AS TotalUnshippedOrders,
    COUNT(CASE WHEN RequiredDate < GETDATE() THEN 1 END) AS OverdueUnshipped,
    COUNT(CASE WHEN RequiredDate >= GETDATE() THEN 1 END) AS FutureUnshipped,
    COUNT(CASE WHEN RequiredDate IS NULL THEN 1 END) AS NoRequiredDate
FROM Orders
WHERE ShippedDate IS NULL;

-- Expected Result in Northwind:
-- TotalUnshippedOrders: 21
-- OverdueUnshipped: 21 (if run today in 2026)
-- FutureUnshipped: 0
-- NoRequiredDate: 0
```

#### Validation: Simulate Query Run in 1998

```sql
-- Simulate running the query on June 1, 1998 to see realistic results
DECLARE @SimulatedDate DATE = '1998-06-01';

SELECT
    o.OrderID,
    c.CompanyName AS Customer,
    e.FirstName + ' ' + e.LastName AS SalesRep,
    o.OrderDate,
    o.RequiredDate,
    DATEDIFF(day, o.RequiredDate, @SimulatedDate) AS DaysOverdue,
    SUM(od.Quantity * od.UnitPrice * (1 - od.Discount)) AS OrderValue
FROM Orders o
JOIN Customers c ON c.CustomerID = o.CustomerID
JOIN Employees e ON e.EmployeeID = o.EmployeeID
JOIN [Order Details] od ON od.OrderID = o.OrderID
WHERE o.ShippedDate IS NULL
  AND o.RequiredDate < @SimulatedDate
GROUP BY o.OrderID, c.CompanyName, e.FirstName, e.LastName,
         o.OrderDate, o.RequiredDate
ORDER BY DaysOverdue DESC;

-- Expected: Shows orders that were overdue as of June 1, 1998
-- Typical result: 0-5 orders with DaysOverdue ranging from 1-30 days
```

#### Validation: Check Specific Overdue Order Details

```sql
-- Examine a specific unshipped order in detail
SELECT
    o.OrderID,
    o.OrderDate,
    o.RequiredDate,
    o.ShippedDate,
    DATEDIFF(day, o.RequiredDate, GETDATE()) AS DaysOverdue,
    c.CustomerID,
    c.CompanyName,
    c.Phone AS CustomerPhone,
    e.EmployeeID,
    e.FirstName + ' ' + e.LastName AS SalesRep,
    e.Extension AS SalesRepExtension,
    COUNT(od.ProductID) AS LineItemCount,
    SUM(od.Quantity) AS TotalUnits,
    SUM(od.Quantity * od.UnitPrice * (1 - od.Discount)) AS OrderValue
FROM Orders o
JOIN Customers c ON c.CustomerID = o.CustomerID
JOIN Employees e ON e.EmployeeID = o.EmployeeID
JOIN [Order Details] od ON od.OrderID = o.OrderID
WHERE o.ShippedDate IS NULL
  AND o.OrderID = 11077  -- Example unshipped order
GROUP BY o.OrderID, o.OrderDate, o.RequiredDate, o.ShippedDate,
         c.CustomerID, c.CompanyName, c.Phone,
         e.EmployeeID, e.FirstName, e.LastName, e.Extension;

-- Expected: Detailed breakdown for escalation and customer contact
```

#### Validation: Compare Shipped vs Unshipped Order Counts

```sql
-- Get overview of order fulfillment status
SELECT
    COUNT(*) AS TotalOrders,
    SUM(CASE WHEN ShippedDate IS NOT NULL THEN 1 ELSE 0 END) AS ShippedOrders,
    SUM(CASE WHEN ShippedDate IS NULL THEN 1 ELSE 0 END) AS UnshippedOrders,
    SUM(CASE WHEN ShippedDate IS NULL AND RequiredDate < GETDATE() THEN 1 ELSE 0 END) AS OverdueOrders,
    CAST(SUM(CASE WHEN ShippedDate IS NOT NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS ShipmentRate
FROM Orders;

-- Expected Result:
-- TotalOrders: 830
-- ShippedOrders: 809
-- UnshippedOrders: 21
-- OverdueOrders: 21 (if run today)
-- ShipmentRate: 97.47%
```

#### Production Monitoring Query

```sql
-- Real-time monitoring query for operations dashboard
-- Shows current overdue orders grouped by severity
SELECT
    CASE
        WHEN DATEDIFF(day, o.RequiredDate, GETDATE()) >= 30 THEN 'Critical (30+ days)'
        WHEN DATEDIFF(day, o.RequiredDate, GETDATE()) >= 14 THEN 'High (14-29 days)'
        WHEN DATEDIFF(day, o.RequiredDate, GETDATE()) >= 7 THEN 'Medium (7-13 days)'
        ELSE 'Low (1-6 days)'
    END AS SeverityLevel,
    COUNT(*) AS OrderCount,
    SUM(od.Quantity * od.UnitPrice * (1 - od.Discount)) AS TotalValue,
    STRING_AGG(CAST(o.OrderID AS VARCHAR), ', ') AS OrderIDs
FROM Orders o
JOIN [Order Details] od ON od.OrderID = o.OrderID
WHERE o.ShippedDate IS NULL
  AND o.RequiredDate < GETDATE()
GROUP BY
    CASE
        WHEN DATEDIFF(day, o.RequiredDate, GETDATE()) >= 30 THEN 'Critical (30+ days)'
        WHEN DATEDIFF(day, o.RequiredDate, GETDATE()) >= 14 THEN 'High (14-29 days)'
        WHEN DATEDIFF(day, o.RequiredDate, GETDATE()) >= 7 THEN 'Medium (7-13 days)'
        ELSE 'Low (1-6 days)'
    END
ORDER BY
    CASE
        WHEN SeverityLevel LIKE 'Critical%' THEN 1
        WHEN SeverityLevel LIKE 'High%' THEN 2
        WHEN SeverityLevel LIKE 'Medium%' THEN 3
        ELSE 4
    END;

-- Use this for daily operations meetings to prioritize fulfillment
```

---
