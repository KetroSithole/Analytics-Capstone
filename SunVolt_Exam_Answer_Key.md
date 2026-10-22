# SunVolt Solar Exam: Memorandum (Lecturer Only)

**Total:** 100 marks (SQL 75 | Power BI 25)
**Do not distribute to students.**

**Source of figures:** every expected result below was produced by running the queries against the `SunVolt Solar Customer Data.xlsx` sheet (1,026 rows). The SQL was run in SQLite as a check. The SQL Server syntax (`CREATE DATABASE`, `GO`, `DECIMAL`) is standard T-SQL but was not run on SQL Server.

---

## Section A: Database and Tables (12 marks)

**A1 (2).**
```sql
CREATE DATABASE SunVolt;
GO
USE SunVolt;
GO
```

**A2 (4).** Award 1 mark for each correct group of columns (names, numeric types, text types, no primary key on `customer_id`).
```sql
CREATE TABLE customers (
    customer_id           INT          NOT NULL,
    marital_status        VARCHAR(10),
    gender                VARCHAR(10),
    annual_income         INT,
    household_dependents  INT,
    education_level       VARCHAR(30),
    occupation            VARCHAR(30),
    homeowner_status      VARCHAR(5),
    vehicles_owned        INT,
    distance_band         VARCHAR(15),
    region                VARCHAR(20),
    age                   INT,
    purchased_solar_system VARCHAR(5),
    purchase_flag         INT,
    age_group             VARCHAR(10),
    income_band           VARCHAR(10)
);
```
*Note:* no primary key is set on `customer_id` because the source file contains 26 duplicated rows. Accept `NOT NULL` with or without it.

**A3 (3).** 2 marks for the table with a primary key, 1 mark for the inserts.
```sql
CREATE TABLE distance_bands (
    distance_band            VARCHAR(15) PRIMARY KEY,
    installer_distance_order INT NOT NULL
);

INSERT INTO distance_bands (distance_band, installer_distance_order) VALUES
('0-1 km', 1), ('1-2 km', 2), ('2-5 km', 3), ('5-10 km', 4), ('10+ km', 5);
```

**A4 (2).** Award 1 mark for a correct import (Distance column mapped to `distance_band`, Installer Distance Order excluded), 1 mark for the count query.
```sql
SELECT COUNT(*) AS row_count FROM customers;   -- expected: 1026
```

**A5 (1).** Accept any answer that states: `DELETE` removes rows and can be rolled back or filtered with `WHERE`; `TRUNCATE` removes all rows quickly and resets the table without row-level logging; `DROP` removes the table structure and data.

---

## Section B: Selecting and Filtering (18 marks)

**B1 (3).** Expected: **170 rows**.
```sql
SELECT customer_id, age, annual_income AS income
FROM customers
WHERE region = 'Europe' AND age >= 40;
```

**B2 (2).** Expected: **5 rows**: Clerical, Management, Manual, Professional, Skilled Manual.
```sql
SELECT DISTINCT occupation
FROM customers
ORDER BY occupation;
```

**B3 (3).** Expected: **630 rows**.
```sql
SELECT customer_id, annual_income
FROM customers
WHERE annual_income BETWEEN 30000 AND 70000
ORDER BY annual_income DESC;
```

**B4 (3).** Expected: `total_rows` = **1026**, `unique_customers` = **1000**.
```sql
SELECT COUNT(*) AS total_rows,
       COUNT(DISTINCT customer_id) AS unique_customers
FROM customers;
```

**B5 (3).** Expected: **364 rows**.
```sql
SELECT customer_id, education_level, homeowner_status
FROM customers
WHERE education_level IN ('Bachelors', 'Graduate Degree')
  AND homeowner_status = 'Yes';
```

**B6 (2).** Expected: **300 rows** (Management + Manual).
```sql
SELECT customer_id, occupation
FROM customers
WHERE occupation LIKE 'Man%';
```

**B7 (2).** Expected: **273 rows**.
```sql
SELECT customer_id, marital_status, vehicles_owned, household_dependents
FROM customers
WHERE marital_status = 'Single'
  AND (vehicles_owned >= 2 OR household_dependents >= 3);
```

---

## Section C: Aggregation and Grouping (22 marks)

**C1 (4).** Expected: `customers` = 1026, `purchases` = 495, `conversion_pct` = **48.2**.
```sql
SELECT COUNT(*) AS customers,
       SUM(purchase_flag) AS purchases,
       ROUND(100.0 * SUM(purchase_flag) / COUNT(*), 1) AS conversion_pct
FROM customers;
```
*Marking:* deduct 2 marks if integer division is used (result 0 or 48).

**C2 (4).** Expected:

| region | customers | purchases | conversion_pct |
|---|---|---|---|
| Pacific | 202 | 119 | 58.9 |
| Europe | 316 | 156 | 49.4 |
| North America | 508 | 220 | 43.3 |

```sql
SELECT region,
       COUNT(*) AS customers,
       SUM(purchase_flag) AS purchases,
       ROUND(100.0 * SUM(purchase_flag) / COUNT(*), 1) AS conversion_pct
FROM customers
GROUP BY region
ORDER BY conversion_pct DESC;
```

**C3 (4).** Expected: 4 rows. Partial High School (78 customers) is excluded.

| education_level | customers | avg_income |
|---|---|---|
| Graduate Degree | 175 | 65943 |
| Bachelors | 311 | 63055 |
| Partial College | 278 | 54604 |
| High School | 184 | 47174 |

```sql
SELECT education_level,
       COUNT(*) AS customers,
       ROUND(AVG(CAST(annual_income AS DECIMAL(12,2))), 0) AS avg_income
FROM customers
GROUP BY education_level
HAVING COUNT(*) >= 100
ORDER BY avg_income DESC;
```

**C4 (4).** Expected: 3 rows (Under 30 and 60+ have 37 purchases each, so they are excluded).

| age_group | customers | purchases |
|---|---|---|
| 30-39 | 311 | 185 |
| 40-49 | 321 | 147 |
| 50-59 | 192 | 89 |

```sql
SELECT age_group,
       COUNT(*) AS customers,
       SUM(purchase_flag) AS purchases
FROM customers
GROUP BY age_group
HAVING SUM(purchase_flag) > 50
ORDER BY purchases DESC;
```

**C5 (6).** Expected:

| gender | youngest | oldest | avg_income |
|---|---|---|---|
| Female | 25 | 89 | 54331 |
| Male | 25 | 78 | 58000 |

```sql
SELECT gender,
       MIN(age) AS youngest,
       MAX(age) AS oldest,
       ROUND(AVG(CAST(annual_income AS DECIMAL(12,2))), 0) AS avg_income
FROM customers
GROUP BY gender;
```

---

## Section D: Joins and Subqueries (13 marks)

**D1 (4).** Expected: 5 rows.

| installer_distance_order | customers | purchases |
|---|---|---|
| 1 | 378 | 207 |
| 2 | 176 | 83 |
| 3 | 162 | 95 |
| 4 | 197 | 77 |
| 5 | 113 | 33 |

```sql
SELECT db.installer_distance_order,
       COUNT(*) AS customers,
       SUM(c.purchase_flag) AS purchases
FROM customers AS c
INNER JOIN distance_bands AS db
    ON c.distance_band = db.distance_band
GROUP BY db.installer_distance_order
ORDER BY db.installer_distance_order;
```

**D2 (3).** Expected: **533 rows**.
```sql
SELECT customer_id, annual_income
FROM customers
WHERE annual_income > (SELECT AVG(annual_income) FROM customers);
-- 533 rows
```

**D3 (3).** Expected: **518 rows** (Europe 316 + Pacific 202).
```sql
SELECT customer_id, region, purchase_flag
FROM customers
WHERE region IN (
    SELECT region
    FROM customers
    GROUP BY region
    HAVING AVG(CAST(purchase_flag AS DECIMAL(5,3))) > 0.45
);
```

**D4 (3).** Expected: 5 rows, all bands present.

| distance_band | customers |
|---|---|
| 0-1 km | 378 |
| 1-2 km | 176 |
| 2-5 km | 162 |
| 5-10 km | 197 |
| 10+ km | 113 |

```sql
SELECT db.distance_band,
       COUNT(c.customer_id) AS customers
FROM distance_bands AS db
LEFT JOIN customers AS c
    ON c.distance_band = db.distance_band
GROUP BY db.distance_band, db.installer_distance_order
ORDER BY db.installer_distance_order;
```
*Marking:* deduct 1 mark if `COUNT(*)` is used, because it counts the unmatched row as 1 for empty bands.

---

## Section E: Data Quality and CASE (10 marks)

**E1 (4).** Expected:

| age_segment | customers | purchases | conversion_pct |
|---|---|---|---|
| 40-59 | 513 | 236 | 46.0 |
| 60+ | 117 | 37 | 31.6 |
| Under 40 | 396 | 222 | 56.1 |

```sql
WITH seg AS (
    SELECT purchase_flag,
           CASE
               WHEN age < 40 THEN 'Under 40'
               WHEN age < 60 THEN '40-59'
               ELSE '60+'
           END AS age_segment
    FROM customers
)
SELECT age_segment,
       COUNT(*) AS customers,
       SUM(purchase_flag) AS purchases,
       ROUND(100.0 * SUM(purchase_flag) / COUNT(*), 1) AS conversion_pct
FROM seg
GROUP BY age_segment
ORDER BY age_segment;
```

**E2 (3).** Expected: **26 rows**. Every duplicated ID has `copies = 2`.
```sql
SELECT customer_id, COUNT(*) AS copies
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1
ORDER BY customer_id;
```

**E3 (2).** Expected: `mismatches` = **0**.
```sql
SELECT COUNT(*) AS mismatches
FROM customers
WHERE (purchased_solar_system = 'Yes' AND purchase_flag <> 1)
   OR (purchased_solar_system = 'No'  AND purchase_flag <> 0);
```

**E4 (1).** Expected: `unique_buyers` = **481**.
```sql
SELECT COUNT(DISTINCT customer_id) AS unique_buyers
FROM customers
WHERE purchase_flag = 1;
```

---

## Section F: Power BI Dashboard (25 marks)

### Reference figures

Use these to mark the dashboard. Students who remove duplicates may get slightly different totals. Accept those if the method is stated and correct.

| Measure | Expected (raw 1,026 rows) |
|---|---|
| Total Customers | 1,026 |
| Purchases | 495 |
| Conversion Rate % | 48.2% |
| Avg Annual Income | 56,209 |

Conversion by region: Pacific 58.9%, Europe 49.4%, North America 43.3%.

Conversion by income band: 30k-49k 50.5%, 50k-69k 49.5%, 70k-99k 49.0%, 100k+ 48.1%, <30k 40.9%.

Conversion by education: Graduate Degree 54.3%, Bachelors 54.3%, Partial College 45.7%, High School 44.6%, Partial High School 28.2%.

Conversion by homeowner: Yes 47.6%, No 49.5%.

### Marking guide

**F1 (4).** 2 marks for both tables loaded. 2 marks for the correct relationship (many-to-one, `distance_band`, single direction).

**F2 (6).** 1 mark per measure, 1 mark for `DIVIDE` used in `Conversion Rate %`, 1 mark for percentage formatting.
```DAX
Total Customers = COUNTROWS(customers)
Purchases = SUM(customers[purchase_flag])
Conversion Rate % = DIVIDE([Purchases], [Total Customers])
Avg Annual Income = AVERAGE(customers[annual_income])
```
*Note:* `DIVIDE` is preferred because it avoids errors when the denominator is zero. Deduct 1 mark for `[Purchases] / [Total Customers]` without a zero check.

**F3 (10).** Page 1 "Overview":
- 4 KPI cards: 2 marks
- Conversion by region bar chart: 2 marks
- Purchases by education column chart: 2 marks
- Donut by `purchased_solar_system`: 1 mark
- Region and age_group slicers working: 2 marks
- Titles, labels and formatting: 1 mark

**F4 (5).** Page 2 "Customer Profile":
- Income band chart in logical order (sort-by-column or a numeric sort key): 2 marks. Alphabetical order gets 0 marks for this item.
- Conversion by installer distance order with band names: 1 mark
- Matrix with age_group rows and homeowner columns: 2 marks

**F5 (0, required).** Three findings with cited numbers. Accept any finding supported by the reference figures above. Deduct the full F5 mark if findings are not in the dashboard or have no numbers.

**Example insights a strong student might write:**
- "Pacific converts at 58.9% against 43.3% in North America."
- "Conversion falls from 50.5% (30k-49k) to 40.9% for incomes under 30k."
- "Customers with a Partial High School education convert at only 28.2%."

---

## Quick Marks Summary

| Section | Marks |
|---|---|
| A: Database and tables | 12 |
| B: Selecting and filtering | 18 |
| C: Aggregation | 22 |
| D: Joins and subqueries | 13 |
| E: Data quality and CASE | 10 |
| **SQL subtotal** | **75** |
| F: Power BI dashboard | 25 |
| **Total** | **100** |
