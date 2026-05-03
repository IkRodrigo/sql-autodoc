# SQL Query Documentation: Critical Stock Levels vs Recent Demand

## 1. Summary

This query identifies products with critically low inventory levels and compares them against recent demand patterns. It answers the business question: **"Which products are at or below reorder levels, and what has their demand been in the last 90 days?"** This is essential for preventing stockouts, optimizing inventory replenishment, and maintaining customer satisfaction.

## 2. Tables Involved

| Table | Role |
|-------|------|
| **Products** | Primary dimension table containing inventory levels, reorder points, and product details |
| **Categories** | Dimension table providing product category names for grouping and analysis |
| **Order Details** | Fact table used to calculate recent demand (quantity ordered) |
| **Orders** | Transaction header table providing order dates for filtering last 90 days |

## 3. Key Logic

- **Joins**: Four-table join with LEFT JOINs to include products even without recent orders
- **Date Filter**: `o.OrderDate >= DATEADD(day, -90, GETDATE())` restricts demand calculation to last 90 days
- **Stock Filter**: `WHERE p.UnitsInStock <= p.ReorderLevel` identifies products at or below reorder threshold
- **Active Products**: `AND p.Discontinued = 0` excludes discontinued products from analysis
- **Demand Calculation**: Sums quantities ordered in last 90 days, using ISNULL to handle products with no recent orders
- **Status Logic**: CASE statement categorizes stock status (Sin stock, Stock crítico, OK)
- **Sorting**: Orders by UnitsInStock ascending and DemandLast90Days descending to prioritize critical items
- **Note**: Query has incomplete ORDER BY clause (ends with "DE" instead of "DESC")

## 4. Output Columns

| Column | Description |
|--------|-------------|
| **ProductID** | Unique product identifier for tracking and ordering |
| **ProductName** | Name of the product requiring attention |
| **CategoryName** | Product category for grouping and prioritization |
| **UnitsInStock** | Current inventory level (number of units available) |
| **ReorderLevel** | Minimum stock threshold that triggers reorder process |
| **UnitsOnOrder** | Quantity already ordered from suppliers (in transit) |
| **DemandLast90Days** | Total units sold in the last 90 days (0 if no recent sales) |
| **StockStatus** | Categorized status: "Sin stock" (0 units), "Stock crítico" (at/below reorder level), or "OK" |

## 5. Business Use Case

**Primary Users:**
- **Inventory Managers**: Prioritize purchase orders and prevent stockouts
- **Procurement Teams**: Identify urgent supplier orders and expedite deliveries
- **Supply Chain Analysts**: Optimize reorder points and safety stock levels
- **Sales Teams**: Communicate product availability to customers proactively
- **Operations Managers**: Balance inventory costs against stockout risks

**When to Use:**
- Daily inventory review meetings
- Weekly procurement planning sessions
- Stockout prevention and mitigation
- Demand forecasting and inventory optimization
- Supplier performance evaluation (frequent stockouts may indicate supplier issues)

## 6. Assumptions & Limitations

**Assumptions:**
- `ReorderLevel` is accurately set for each product based on lead time and demand
- `UnitsOnOrder` reflects current pending supplier orders
- `Discontinued = 0` accurately identifies active products
- 90-day window is appropriate for demand analysis (may vary by product lifecycle)
- `GETDATE()` provides accurate current date for calculations

**Limitations:**
- **Syntax Error**: Query has incomplete ORDER BY clause (ends with "DE" instead of "DESC")
- **Historical data**: Northwind sample data is from 1996-1998; 90-day lookback may return zero demand
- **Seasonality ignored**: 90-day demand may not reflect seasonal patterns or trends
- **Lead time not considered**: Doesn't account for supplier lead times in urgency calculation
- **No safety stock**: Reorder level may not include safety stock buffer
- **Pending orders**: UnitsOnOrder shown but not factored into urgency calculation

**Edge Cases:**
- Products with zero recent demand but low stock (slow-moving items vs. stockouts)
- Products with high UnitsOnOrder may not need immediate action
- Newly added products may have low stock but no historical demand
- Products transitioning to discontinued status may show as critical
- Bulk orders can skew 90-day demand (one large order vs. consistent demand)

## 7. Test Cases

**Note**: The Northwind sample database contains historical data from 1996-1998. When run today, DemandLast90Days will be 0 for all products. The test cases below assume the query is run during the database's time period for realistic scenarios. Also note: Q5 has a syntax error in the ORDER BY clause (ends with "DE" instead of "DESC").

### 7.1 Positive Test Cases

| Test ID | Test Description | Expected Result | Validation Method |
|---------|-----------------|-----------------|-------------------|
| **PT-01** | **Zero Stock Products** | Products with UnitsInStock = 0<br>StockStatus: 'Sin stock'<br>Examples: Chef Anton's Gumbo Mix, Alice Mutton | CASE statement correctly identifies zero stock |
| **PT-02** | **Critical Stock Products** | Products with 0 < UnitsInStock <= ReorderLevel<br>StockStatus: 'Stock crítico'<br>Examples: Mascarpone Fabioli, Gravad lax | CASE statement correctly identifies critical stock |
| **PT-03** | **Only Active Products** | All returned products have Discontinued = 0 | WHERE clause filters out discontinued products |
| **PT-04** | **Only Below Reorder Level** | All products have UnitsInStock <= ReorderLevel | WHERE clause filters correctly |
| **PT-05** | **Sorted by Stock Level** | Products sorted by UnitsInStock ASC (lowest first) | ORDER BY prioritizes most critical items first |

### 7.2 Edge Case Test Cases

| Test ID | Test Description | Expected Behavior | Validation Query |
|---------|-----------------|-------------------|------------------|
| **EC-01** | **Product with Zero Reorder Level** | Products with ReorderLevel = 0 and UnitsInStock = 0 should appear as 'Sin stock' | Verify CASE logic handles ReorderLevel = 0 |
| **EC-02** | **Product with High UnitsOnOrder** | Products with UnitsOnOrder > 0 may not need immediate action | Consider UnitsOnOrder in prioritization |
| **EC-03** | **No Recent Demand (90 days)** | Products with DemandLast90Days = 0 may be slow-moving | LEFT JOIN with date filter returns 0 for no demand |
| **EC-04** | **Syntax Error in ORDER BY** | Query has incomplete ORDER BY (ends with "DE") | Fix: Change "ORDER BY ... DE" to "DESC" |

### 7.3 Manual Validation Queries

#### Sanity Check: Count Products by Stock Status

```sql
-- Get overview of all products by stock status
SELECT
    CASE
        WHEN p.UnitsInStock = 0 THEN 'Sin stock'
        WHEN p.UnitsInStock <= p.ReorderLevel THEN 'Stock crítico'
        ELSE 'OK'
    END AS StockStatus,
    COUNT(*) AS ProductCount,
    SUM(p.UnitsInStock) AS TotalUnitsInStock,
    SUM(p.UnitsOnOrder) AS TotalUnitsOnOrder,
    AVG(p.ReorderLevel) AS AvgReorderLevel
FROM Products p
WHERE p.Discontinued = 0
GROUP BY
    CASE
        WHEN p.UnitsInStock = 0 THEN 'Sin stock'
        WHEN p.UnitsInStock <= p.ReorderLevel THEN 'Stock crítico'
        ELSE 'OK'
    END
ORDER BY
    CASE StockStatus
        WHEN 'Sin stock' THEN 1
        WHEN 'Stock crítico' THEN 2
        ELSE 3
    END;

-- Expected Result:
-- Sin stock: ~5 products, 0 units in stock
-- Stock crítico: ~4 products, ~46 units in stock
-- OK: ~68 products, ~3,000+ units in stock
```

#### Validation: Simulate Query with 90-Day Demand (1998 Context)

```sql
-- Simulate running the query on May 6, 1998 to see realistic demand
DECLARE @SimulatedDate DATE = '1998-05-06';
DECLARE @DaysBack INT = 90;

SELECT
    p.ProductID,
    p.ProductName,
    c.CategoryName,
    p.UnitsInStock,
    p.ReorderLevel,
    p.UnitsOnOrder,
    ISNULL(SUM(od.Quantity), 0) AS DemandLast90Days,
    CASE
        WHEN p.UnitsInStock = 0 THEN 'Sin stock'
        WHEN p.UnitsInStock <= p.ReorderLevel THEN 'Stock crítico'
        ELSE 'OK'
    END AS StockStatus
FROM Products p
JOIN Categories c ON c.CategoryID = p.CategoryID
LEFT JOIN [Order Details] od ON od.ProductID = p.ProductID
LEFT JOIN Orders o ON o.OrderID = od.OrderID
                  AND o.OrderDate >= DATEADD(day, -@DaysBack, @SimulatedDate)
                  AND o.OrderDate <= @SimulatedDate
WHERE p.Discontinued = 0
  AND p.UnitsInStock <= p.ReorderLevel
GROUP BY p.ProductID, p.ProductName, c.CategoryName,
         p.UnitsInStock, p.ReorderLevel, p.UnitsOnOrder
ORDER BY p.UnitsInStock ASC, DemandLast90Days DESC;

-- Expected: Shows realistic 90-day demand for products
-- High demand + low stock = urgent reorder
-- Low demand + low stock = less urgent
```

#### Validation: Check Specific Critical Product

```sql
-- Deep dive into a specific product at critical stock level
SELECT
    p.ProductID,
    p.ProductName,
    c.CategoryName,
    s.CompanyName AS SupplierName,
    s.Phone AS SupplierPhone,
    p.UnitsInStock,
    p.ReorderLevel,
    p.UnitsOnOrder,
    p.UnitPrice,
    p.UnitsInStock * p.UnitPrice AS CurrentStockValue,
    p.UnitsOnOrder * p.UnitPrice AS IncomingStockValue,
    ISNULL(SUM(od.Quantity), 0) AS DemandLast90Days,
    CASE
        WHEN p.UnitsInStock = 0 THEN 'URGENT: Out of stock'
        WHEN p.UnitsInStock <= p.ReorderLevel * 0.5 THEN 'HIGH: Below 50% of reorder level'
        ELSE 'MEDIUM: At or near reorder level'
    END AS UrgencyLevel
FROM Products p
JOIN Categories c ON c.CategoryID = p.CategoryID
JOIN Suppliers s ON s.SupplierID = p.SupplierID
LEFT JOIN [Order Details] od ON od.ProductID = p.ProductID
LEFT JOIN Orders o ON o.OrderID = od.OrderID
                  AND o.OrderDate >= DATEADD(day, -90, GETDATE())
WHERE p.ProductID = 31  -- Example: Gorgonzola Telino
GROUP BY p.ProductID, p.ProductName, c.CategoryName,
         s.CompanyName, s.Phone,
         p.UnitsInStock, p.ReorderLevel, p.UnitsOnOrder, p.UnitPrice;

-- Expected: Detailed view for procurement decision-making
-- Includes supplier contact info for immediate reordering
```

#### Validation: Compare Stock Levels Across Categories

```sql
-- Analyze which categories have the most stock issues
SELECT
    c.CategoryName,
    COUNT(*) AS TotalProducts,
    SUM(CASE WHEN p.UnitsInStock = 0 THEN 1 ELSE 0 END) AS OutOfStock,
    SUM(CASE WHEN p.UnitsInStock > 0 AND p.UnitsInStock <= p.ReorderLevel THEN 1 ELSE 0 END) AS CriticalStock,
    SUM(CASE WHEN p.UnitsInStock > p.ReorderLevel THEN 1 ELSE 0 END) AS HealthyStock,
    SUM(p.UnitsInStock) AS TotalUnitsInStock,
    SUM(p.UnitsOnOrder) AS TotalUnitsOnOrder,
    CAST(SUM(CASE WHEN p.UnitsInStock <= p.ReorderLevel THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS PctAtRisk
FROM Products p
JOIN Categories c ON c.CategoryID = p.CategoryID
WHERE p.Discontinued = 0
GROUP BY c.CategoryName
ORDER BY PctAtRisk DESC, OutOfStock DESC;

-- Expected: Identifies categories with highest stock risk
-- Use for category-level procurement planning
```

#### Production Reorder Recommendation Query

```sql
-- Generate reorder recommendations with priority scoring
SELECT
    p.ProductID,
    p.ProductName,
    c.CategoryName,
    s.CompanyName AS Supplier,
    p.UnitsInStock,
    p.ReorderLevel,
    p.UnitsOnOrder,
    ISNULL(SUM(od.Quantity), 0) AS DemandLast90Days,
    -- Calculate recommended order quantity
    CASE
        WHEN p.UnitsInStock = 0 THEN p.ReorderLevel * 2  -- Double for out of stock
        ELSE p.ReorderLevel - p.UnitsInStock + (ISNULL(SUM(od.Quantity), 0) / 3)  -- Reorder level + 30 days demand
    END AS RecommendedOrderQty,
    -- Priority score (higher = more urgent)
    (CASE WHEN p.UnitsInStock = 0 THEN 100 ELSE 0 END) +  -- Out of stock = +100
    (p.ReorderLevel - p.UnitsInStock) * 10 +  -- Below reorder = +10 per unit
    (ISNULL(SUM(od.Quantity), 0) / 10) AS PriorityScore,
    CASE
        WHEN p.UnitsInStock = 0 AND ISNULL(SUM(od.Quantity), 0) > 50 THEN 'CRITICAL'
        WHEN p.UnitsInStock = 0 THEN 'HIGH'
        WHEN p.UnitsInStock <= p.ReorderLevel * 0.5 THEN 'MEDIUM'
        ELSE 'LOW'
    END AS Priority
FROM Products p
JOIN Categories c ON c.CategoryID = p.CategoryID
JOIN Suppliers s ON s.SupplierID = p.SupplierID
LEFT JOIN [Order Details] od ON od.ProductID = p.ProductID
LEFT JOIN Orders o ON o.OrderID = od.OrderID
                  AND o.OrderDate >= DATEADD(day, -90, GETDATE())
WHERE p.Discontinued = 0
  AND p.UnitsInStock <= p.ReorderLevel
GROUP BY p.ProductID, p.ProductName, c.CategoryName, s.CompanyName,
         p.UnitsInStock, p.ReorderLevel, p.UnitsOnOrder
ORDER BY PriorityScore DESC;

-- Use this for automated reorder workflows
-- PriorityScore helps rank which products to reorder first
```

---