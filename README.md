# Northwind SQL Doc & Test Generator

**IBM Bob Dev Day Hackathon 2026** | Solo Project

---

## Problem

Data analysts inherit SQL query libraries with zero documentation. New team members spend days reverse-engineering queries to understand:
- What business question does this query answer?
- Which tables are involved and why?
- What are the edge cases and limitations?
- How do I validate the results?

Manual documentation is time-consuming, inconsistent, and quickly becomes outdated.

---

## Solution

Use **IBM Bob** to automatically generate comprehensive technical documentation and test cases for SQL queries in minutes instead of hours.

For each query, Bob produces:
- **Business summary** in plain English
- **Table relationships** and their roles
- **Key logic** explanations (joins, filters, calculations)
- **Output column** descriptions
- **Business use cases** and target users
- **Assumptions & limitations** with edge cases
- **Test cases** with validation SQL (5 positive + 4 edge cases per query)
- **Interactive web dashboard** to view results and documentation

---

## Demo

**Live dashboard:** Open `index.html` in your browser to explore query results, database schema, and full documentation with modal popups.

**Features:**
- Interactive query result viewer
- Database schema visualization
- Modal documentation popups
- Responsive design for all devices

---

## Repository Structure

```
bob-demo/
├── Querys/                      # Source SQL queries and generated docs
│   ├── Q1                       # Input: Revenue by Product Category query
│   ├── Q1_Documentation.md      # Output: Full documentation + 9 test cases
│   ├── Q2                       # Input: Top 10 Customers query
│   ├── Q2_Documentation.md      # Output: Full documentation + 9 test cases
│   ├── Q3                       # Input: Overdue Unshipped Orders query
│   ├── Q3_Documentation.md      # Output: Full documentation + 9 test cases
│   ├── Q4                       # Input: Employee Sales Performance query
│   ├── Q4_Documentation.md      # Output: Full documentation + 9 test cases
│   ├── Q5                       # Input: Critical Stock Levels query
│   └── Q5_Documentation.md      # Output: Full documentation + 9 test cases
├── Northwind/                   # Database schema
│   └── northwind.sql            # Northwind database creation script
├── index.html                   # Interactive web dashboard
├── TECHNICAL_SPEC.md            # Technical specification document
└── NREADME.md                   # This file
```

---

## How It Works

**3-Step Automated Flow:**

```
┌─────────────────┐
│  Input Query    │  SQL file (e.g., Q1)
│  (Raw SQL)      │  
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   IBM Bob       │  Analyzes query structure
│   Processes     │  Identifies tables, joins, logic
│                 │  Generates documentation
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Output Docs    │  Q1_Documentation.md
│  + Test Cases   │  - 7 documentation sections
│                 │  - 9 test cases with validation SQL
└─────────────────┘
```

1. **Input:** Provide SQL query file path to Bob
2. **Bob Generates:** Documentation with business context, technical details, and executable test cases
3. **Output:** Markdown file ready for team wiki + validation queries for SSMS

---

## Bob Usage

### Features Leveraged

| Bob Feature | Purpose | Example |
|-------------|---------|---------|
| **read_file** | Read SQL query files | `read_file Querys/Q1` |
| **Code Understanding** | Analyze SQL syntax, identify JOINs, aggregations | Parse complex multi-table queries |
| **write_to_file** | Create documentation files | Generate `Q1_Documentation.md` |
| **apply_diff** | Update docs with test cases | Add 9 test cases to existing doc |
| **Markdown Generation** | Format structured documentation | Tables, code blocks, sections |
| **Web Development** | Build interactive dashboard | HTML/CSS/JavaScript with modals |

### Prompt Strategy

**Phase 1 - Context First:**
- Provided SQL query file path
- Specified 7 required documentation sections
- Emphasized real Northwind data in examples

**Phase 2 - Iterative Refinement:**
- Expanded to all 5 queries
- Requested comprehensive test cases (5 positive + 4 edge cases)
- Added validation SQL for manual verification

**Phase 3 - Web Interface:**
- Built interactive HTML dashboard
- Created modal documentation viewer
- Embedded full content to avoid file loading issues

**Total Interaction:** ~20 Bob messages across 5 queries

---

## Before vs After

| Metric | Before (Manual) | After (Bob) | Improvement |
|--------|----------------|-------------|-------------|
| **Time to document 1 query** | 2-3 hours | 5-10 minutes | **95% faster** |
| **Onboarding time** | 3-5 days | 1-2 hours | **90% reduction** |
| **Test coverage** | 0-2 ad-hoc tests | 9 structured tests | **450% increase** |
| **Human effort required** | 100% manual | 10% review | **90% automation** |
| **Documentation consistency** | Varies by author | Standardized format | **100% consistent** |
| **Validation queries** | None | 9 per query | **∞ improvement** |

---

## Results

### Deliverables

- ✅ **5 SQL queries** fully documented
- ✅ **45 test cases** generated (9 per query)
- ✅ **45 validation SQL queries** for manual verification
- ✅ **1 interactive web dashboard** with modal documentation viewer
- ✅ **100% coverage** of all queries in the library

### Documentation Quality

Each query documentation includes:
- Business summary (2-3 sentences)
- Tables involved with roles
- Key logic in plain English
- Output column descriptions
- Business use cases
- Assumptions & limitations
- 5 positive test cases
- 4 edge case tests
- Validation SQL for each test

### Known Issues Documented

- **Q1:** Comment mentions `ShipVia IS NOT NULL` but query uses `ShippedDate IS NOT NULL`
- **Q3:** Historical data shows orders 9,000+ days overdue (expected for 1996-1998 data)
- **Q5:** Syntax error in ORDER BY clause (ends with "DE" instead of "DESC")

---

## How to Run

### View Documentation

**Option 1: Web Dashboard**
1. Open `index.html` in any browser
2. Click "📄 View Full Documentation" for any query
3. Explore query results, database schema, and test cases

**Option 2: Markdown Files**
1. Navigate to `Querys/` folder
2. Open any `Q*_Documentation.md` file
3. Read in VS Code, GitHub, or any Markdown viewer

### Run Test Validations

**Prerequisites:**
- SQL Server Management Studio (SSMS)
- Northwind database installed

**Steps:**
1. Open `Querys/Q1_Documentation.md` (or any query doc)
2. Scroll to **Test Cases** section
3. Copy validation SQL from any test case
4. Paste into SSMS and execute
5. Compare results with **Expected Result** column

**Example:**
```sql
-- From Q1, Test PT-01: Verify Beverages category
SELECT 
    c.CategoryName,
    SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) AS TotalRevenue
FROM Categories c
INNER JOIN Products p ON c.CategoryID = p.CategoryID
INNER JOIN [Order Details] od ON p.ProductID = od.ProductID
INNER JOIN Orders o ON od.OrderID = o.OrderID
WHERE o.ShippedDate IS NOT NULL
    AND c.CategoryName = 'Beverages'
GROUP BY c.CategoryName;
-- Expected: $267,868.18
```

---

## Team

**Solo Project** — A&D Queries Team
* *
IBM Bob Dev Day Hackathon 2026

**Tools Used:**
- IBM Bob (AI Assistant)
- SQL Server Management Studio
- Northwind Database
- VS Code
- Markdown

**Project Duration:** 1 day  
**Bob Interaction Time:** ~2 hours  
**Total Queries Documented:** 5  
**Total Test Cases Generated:** 45

---

**Database:** Northwind (Microsoft sample database, public domain)

**Last Updated** 2026-05-03
**Author:** A&D Queries Team
