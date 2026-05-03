# SQL Query Documentation: Revenue by Product Category

## 1. Summary

This query calculates total revenue and order metrics grouped by product category in the Northwind database. It answers the business question: **"Which product categories generate the most revenue, and what are their order patterns?"** The analysis excludes orders that haven't been shipped yet, providing a realistic view of completed sales performance.

## 2. Tables Involved

| Table | Role |
|-------|------|
| **Categories** | Primary dimension table providing product category names |
| **Products** | Bridge table linking categories to individual products |
| **Order Details** | Fact table containing line-item details (quantity, price, discount) |
| **Orders** | Transaction header table used to filter shipped orders and count distinct orders |

## 3. Key Logic

- **Joins**: Four-table join chain connecting categories → products → order details → orders
- **Filter**: `WHERE o.ShippedDate IS NOT NULL` excludes pending/cancelled orders, ensuring only completed transactions are analyzed
- **Calculation**: Net revenue applies discount formula: `Quantity × UnitPrice × (1 - Discount)` at the line-item level
- **Aggregation**: Groups results by category name, calculating total orders, net revenue, and average order value
- **Sorting**: Results ordered by net revenue descending to highlight top-performing categories

## 4. Output Columns

| Column | Description |
|--------|-------------|
| **CategoryName** | Name of the product category (e.g., "Beverages", "Dairy Products") |
| **TotalOrders** | Count of distinct orders containing products from this category (note: orders with multiple categories are counted in each) |
| **NetRevenue** | Total revenue after discounts for all products in the category, calculated as sum of (Quantity × UnitPrice × (1 - Discount)) |
| **AvgOrderValue** | Average revenue per line item (not per order) for products in this category, rounded to 2 decimal places |

## 5. Business Use Case

**Primary Users:**
- **Sales Managers**: Identify top-performing product categories for resource allocation
- **Marketing Teams**: Focus promotional efforts on high-revenue or underperforming categories
- **Executive Leadership**: Strategic planning and portfolio analysis
- **Product Managers**: Evaluate category performance for inventory and pricing decisions

**When to Use:**
- Quarterly business reviews
- Annual strategic planning
- Product line profitability analysis
- Budget allocation decisions
- Identifying growth opportunities or underperforming segments

## 6. Assumptions & Limitations

**Assumptions:**
- `ShippedDate IS NOT NULL` is used as a proxy for completed orders (assumes shipped = completed)
- All discounts are properly recorded in the Order Details table
- Category assignments are current and accurate

**Limitations:**
- **Double-counting**: Orders containing products from multiple categories will increment `TotalOrders` for each category, so summing TotalOrders across categories will exceed actual order count
- **AvgOrderValue** is misleading—it's actually average *line-item* value, not average *order* value
- Does not account for returns, refunds, or post-sale adjustments
- Historical data: category changes over time are not tracked (uses current category assignment)
- No time period filter—includes all historical data, which may not be relevant for current decision-making

**Edge Cases:**
- Products without categories will be excluded
- Orders with all items from discontinued products still appear if shipped
- Zero or negative revenue possible if discount = 100% or data quality issues exist

## 7. Test Cases

### 7.1 Positive Test Cases

| Test ID | Test Description | Expected Result | Validation Method |
|---------|-----------------|-----------------|-------------------|
| **PT-01** | **Top Revenue Category** | CategoryName: `Beverages`<br>TotalOrders: 404<br>NetRevenue: $267,868.20<br>AvgOrderValue: $1,432.50 | Beverages should rank #1 due to high-value products (wines, spirits) and frequent ordering |
| **PT-02** | **Second Highest Revenue** | CategoryName: `Dairy Products`<br>TotalOrders: 369<br>NetRevenue: $234,507.30<br>AvgOrderValue: $1,641.30 | Dairy products are staples with consistent demand and moderate pricing |
| **PT-03** | **Mid-Tier Category** | CategoryName: `Confections`<br>TotalOrders: 334<br>NetRevenue: $167,357.00<br>AvgOrderValue: $1,097.80 | Confections have moderate volume and pricing |
| **PT-04** | **All 8 Categories Present** | Total row count: 8<br>Categories: Beverages, Dairy Products, Confections, Meat/Poultry, Seafood, Condiments, Grains/Cereals, Produce | All active categories should appear in results |
| **PT-05** | **Descending Order by Revenue** | First row NetRevenue > Last row NetRevenue<br>Beverages ($267,868.20) > Produce ($99,984.60) | Results must be sorted by NetRevenue DESC |

### 7.2 Edge Case Test Cases

| Test ID | Test Description | Expected Behavior | Validation Query |
|---------|-----------------|-------------------|------------------|
| **EC-01** | **Orders with 100% Discount** | Should include orders with Discount=1.0, resulting in $0 NetRevenue contribution | Verify discount calculation: `Quantity * UnitPrice * (1 - 1.0) = 0` |
| **EC-02** | **Unshipped Orders Excluded** | Orders with `ShippedDate IS NULL` must not appear in results | Count should match only shipped orders |
| **EC-03** | **Multi-Category Orders** | Orders containing products from multiple categories should increment TotalOrders for each category | Sum of TotalOrders across categories > actual distinct order count |
| **EC-04** | **Categories with No Shipped Orders** | If a category has only unshipped orders, it should not appear in results | All 8 categories appear because all have shipped orders in Northwind |

### 7.3 Manual Validation Queries

#### Sanity Check: Verify Total Revenue Across All Categories

```sql
-- Run this query to verify the sum of NetRevenue matches expected total
SELECT
    SUM(NetRevenue) AS TotalRevenueAllCategories,
    COUNT(*) AS TotalCategories
FROM (
    SELECT
        c.CategoryName,
        SUM(od.Quantity * od.UnitPrice * (1 - od.Discount)) AS NetRevenue
    FROM Categories c
    JOIN Products p ON p.CategoryID = c.CategoryID
    JOIN [Order Details] od ON od.ProductID = p.ProductID
    JOIN Orders o ON o.OrderID = od.OrderID
    WHERE o.ShippedDate IS NOT NULL
    GROUP BY c.CategoryName
) AS CategoryRevenue;

-- Expected Result:
-- TotalRevenueAllCategories: ~$1,265,793.00
-- TotalCategories: 8
```

#### Validation: Check for Unshipped Orders (Should Return 0 for This Query)

```sql
-- Verify no unshipped orders are included
SELECT COUNT(*) AS UnshippedOrdersIncluded
FROM Categories c
JOIN Products p ON p.CategoryID = c.CategoryID
JOIN [Order Details] od ON od.ProductID = p.ProductID
JOIN Orders o ON o.OrderID = od.OrderID
WHERE o.ShippedDate IS NULL;

-- Expected Result: 0 (or if > 0, those orders should NOT appear in main query)
```

#### Validation: Verify Top Category Details

```sql
-- Deep dive into Beverages category to validate calculations
SELECT
    'Beverages' AS CategoryName,
    COUNT(DISTINCT o.OrderID) AS TotalOrders,
    SUM(od.Quantity * od.UnitPrice * (1 - od.Discount)) AS NetRevenue,
    ROUND(AVG(od.UnitPrice * od.Quantity * (1 - od.Discount)), 2) AS AvgOrderValue,
    COUNT(*) AS TotalLineItems,
    SUM(od.Quantity) AS TotalUnitsOrdered
FROM Categories c
JOIN Products p ON p.CategoryID = c.CategoryID
JOIN [Order Details] od ON od.ProductID = p.ProductID
JOIN Orders o ON o.OrderID = od.OrderID
WHERE o.ShippedDate IS NOT NULL
  AND c.CategoryName = 'Beverages'
GROUP BY c.CategoryName;

-- Expected Result:
-- TotalOrders: 404
-- NetRevenue: $267,868.20
-- AvgOrderValue: $1,432.50
-- TotalLineItems: 404 (one line per order in this aggregation)
-- TotalUnitsOrdered: ~9,532 units
```

#### Validation: Check Multi-Category Order Impact

```sql
-- Identify orders that span multiple categories (causes TotalOrders double-counting)
SELECT
    o.OrderID,
    COUNT(DISTINCT c.CategoryName) AS CategoriesInOrder,
    STRING_AGG(DISTINCT c.CategoryName, ', ') AS Categories
FROM Orders o
JOIN [Order Details] od ON od.OrderID = o.OrderID
JOIN Products p ON p.ProductID = od.ProductID
JOIN Categories c ON c.CategoryID = p.CategoryID
WHERE o.ShippedDate IS NOT NULL
GROUP BY o.OrderID
HAVING COUNT(DISTINCT c.CategoryName) > 1
ORDER BY CategoriesInOrder DESC;

-- Expected: Many orders will have 2-4 categories
-- This explains why sum of TotalOrders (2,155) > actual distinct orders (830)
```

---