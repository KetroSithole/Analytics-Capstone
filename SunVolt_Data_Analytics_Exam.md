# SunVolt Solar: First-Year Data Analytics Practical Exam

**Duration:** 3 hours
**Total marks:** 100 (SQL: 75 | Power BI: 25)
**Dataset:** `SunVolt Solar Customer Data.xlsx` (sheet: *Solar Customer Data*, 1,026 rows, 17 columns)
**Tools:** SQL Server (SSMS) and Power BI Desktop

---

## Instructions

1. Save all SQL in one file named `<student_number>_exam.sql`. Run it top to bottom.
2. Use the **table and column names exactly as given** in this paper.
3. Where a question asks for an **alias**, use the exact alias name given.
4. Every SQL question must return a result you can check. Put a comment above each answer, for example `-- A2`.
5. Submit your `.sql` file and your Power BI file `<student_number>_dashboard.pbix`.
6. Do not discuss answers with other students. Work independently.

---

## Dataset Dictionary

The Excel file holds one sheet. Load it into two SQL tables as described in Section A.

| Excel column | SQL column | Table | Description |
|---|---|---|---|
| Customer ID | `customer_id` | customers | Customer identifier |
| Marital Status | `marital_status` | customers | Married / Single |
| Gender | `gender` | customers | Male / Female |
| Annual Income | `annual_income` | customers | Yearly income (numeric) |
| Household Dependents | `household_dependents` | customers | Number of dependents |
| Education Level | `education_level` | customers | e.g. Bachelors, High School |
| Occupation | `occupation` | customers | e.g. Professional, Clerical |
| Homeowner Status | `homeowner_status` | customers | Yes / No |
| Vehicles Owned | `vehicles_owned` | customers | Number of vehicles |
| Distance from Nearest Installer | `distance_band` | customers | e.g. 0-1 km, 10+ km |
| Region | `region` | customers | North America / Europe / Pacific |
| Age | `age` | customers | Age in years |
| Purchased Solar System | `purchased_solar_system` | customers | Yes / No |
| Purchase Flag | `purchase_flag` | customers | 1 = purchased, 0 = did not purchase |
| Age Group | `age_group` | customers | e.g. 30-39, 60+ |
| Income Band | `income_band` | customers | e.g. <30k, 30k-49k, 100k+ |
| Installer Distance Order | `installer_distance_order` | distance_bands | Ranking of distance band (1 = nearest). **Not** stored in customers. |

---

## Section A: Database and Table Creation (12 marks)

**A1 (2 marks).** Create a database called `SunVolt` and switch to it.

**A2 (4 marks).** Create a table called `customers` with the 16 `customers` columns from the dictionary above.
- Use the SQL column names exactly.
- Choose suitable data types. Numbers must be numeric types and text must be character types.
- Do **not** add a primary key on `customer_id`.

**A3 (3 marks).** Create a table called `distance_bands` with:
- `distance_band` (text, primary key)
- `installer_distance_order` (integer, not null)

Then insert these five rows:

| distance_band | installer_distance_order |
|---|---|
| 0-1 km | 1 |
| 1-2 km | 2 |
| 2-5 km | 3 |
| 5-10 km | 4 |
| 10+ km | 5 |

**A4 (2 marks).** Import all rows from the Excel sheet into `customers`. Use the SSMS Import Data wizard or your own method. Map `Distance from Nearest Installer` to `distance_band`. Leave out `Installer Distance Order`.
Then write a query that returns the row count of `customers`.

**A5 (1 mark).** In one or two sentences, explain the difference between `DELETE`, `TRUNCATE` and `DROP`.

---

## Section B: Selecting, Filtering and Aliases (18 marks)

Use **aliases** where they are asked for. Every query must run without errors.

**B1 (3 marks).** Return `customer_id`, `age`, and `annual_income`. Alias `annual_income` as `income`. Include only customers in the **Europe** region who are **40 years or older**.

**B2 (2 marks).** List the **distinct** occupations, sorted alphabetically.

**B3 (3 marks).** Return `customer_id` and `annual_income` for customers earning **between 30,000 and 70,000** inclusive. Sort from highest to lowest income.

**B4 (3 marks).** Return two columns, aliased as `total_rows` and `unique_customers`:
- the total number of rows in `customers`
- the number of distinct `customer_id` values

**B5 (3 marks).** Return `customer_id`, `education_level` and `homeowner_status` for customers whose education is **Bachelors** or **Graduate Degree** and who are **homeowners**.

**B6 (2 marks).** Return `customer_id` and `occupation` for every customer whose occupation starts with `Man`.

**B7 (2 marks).** Return `customer_id`, `marital_status`, `vehicles_owned` and `household_dependents` for **single** customers who own **2 or more vehicles OR have 3 or more dependents**.

---

## Section C: Aggregation and Grouping (22 marks)

Round all percentages to **one decimal place**. Use aliases for every calculated column.

**C1 (4 marks).** Return one row with:
- `customers`: total rows
- `purchases`: the number of customers with `purchase_flag = 1`
- `conversion_pct`: purchases as a percentage of total customers

**C2 (4 marks).** For each `region`, return:
- `region`
- `customers`: number of customers
- `purchases`: number of purchases
- `conversion_pct`: conversion percentage

Sort by `conversion_pct`, highest first.

**C3 (4 marks).** For each `education_level` with **at least 100 customers**, return:
- `education_level`
- `customers`
- `avg_income`: average `annual_income`, rounded to 0 decimal places

Sort by `avg_income`, highest first.

**C4 (4 marks).** For each `age_group` that has **more than 50 purchases**, return:
- `age_group`
- `customers`
- `purchases`

Sort by `purchases`, highest first.

**C5 (6 marks).** For each `gender`, return:
- `gender`
- `youngest`: minimum `age`
- `oldest`: maximum `age`
- `avg_income`: average `annual_income`, rounded to 0 decimal places

---

## Section D: Joins and Subqueries (13 marks)

**D1 (4 marks).** Join `customers` to `distance_bands` on `distance_band`. For each `installer_distance_order`, return:
- `installer_distance_order`
- `customers`: number of customers
- `purchases`: number of purchases

Sort by `installer_distance_order`.

**D2 (3 marks).** Using a **subquery**, return `customer_id` and `annual_income` for customers earning **more than the average annual income** of all customers. Count how many rows are returned and put that count in a comment.

**D3 (3 marks).** Using a **subquery**, return `customer_id`, `region` and `purchase_flag` for customers who live in a region whose **purchase rate is above 45%**.
Hint: a purchase rate is the average of `purchase_flag`.

**D4 (3 marks).** Use a **LEFT JOIN** from `distance_bands` to `customers`. Return:
- `distance_band`
- `customers`: the number of customers in each band

Include every band, even if it has no customers. Sort by `installer_distance_order`.

---

## Section E: Data Quality and CASE (10 marks)

**E1 (4 marks).** Use a `CASE` expression to group customers into these segments by `age`:
- `Under 40` for age below 40
- `40-59` for age from 40 up to 59
- `60+` for age 60 and above

Return, for each segment:
- `age_segment`
- `customers`
- `purchases`
- `conversion_pct`

Sort by `age_segment`.

**E2 (3 marks).** Find every `customer_id` that appears **more than once** in `customers`. Return `customer_id` and a count aliased as `copies`. Sort by `customer_id`.

**E3 (2 marks).** Write a query that counts rows where `purchased_solar_system` and `purchase_flag` **disagree**. Alias the count as `mismatches`.

**E4 (1 mark).** Count the **distinct** customers who purchased a system. Alias the count as `unique_buyers`.

---

## Section F: Power BI Dashboard (25 marks)

Build a two-page dashboard in Power BI Desktop, using the same data as in Section A.

**F1 (4 marks): Data model.**
- Load both tables (`customers` and `distance_bands`).
- Create a relationship: `customers[distance_band]` to `distance_bands[distance_band]`, many-to-one, single direction.

**F2 (6 marks): DAX measures.** Create these measures in a measures table:
- `Total Customers` = count of rows in `customers`
- `Purchases` = sum of `purchase_flag`
- `Conversion Rate %` = `Purchases` divided by `Total Customers`, using `DIVIDE`, formatted as a percentage
- `Avg Annual Income` = average of `annual_income`

**F3 (10 marks): Page 1, "Overview".**
- Four KPI cards: `Total Customers`, `Purchases`, `Conversion Rate %`, `Avg Annual Income`
- A bar chart of `Conversion Rate %` by `region`
- A column chart of `Purchases` by `education_level`
- A donut chart of customers split by `purchased_solar_system`
- Two slicers: `region` and `age_group`
- Every visual must have a clear title and readable labels

**F4 (5 marks): Page 2, "Customer Profile".**
- A column chart of `Conversion Rate %` by `income_band`. Bands must display in logical order: `<30k`, `30k-49k`, `50k-69k`, `70k-99k`, `100k+`. Alphabetical order does not count.
- A bar chart of `Conversion Rate %` by `installer_distance_order` (from `distance_bands`). Include the band names on the axis.
- A matrix with `age_group` in rows, `homeowner_status` in columns and `Conversion Rate %` as the value.

**F5 (0 marks, but required): Insights.** On Page 2, add a text box with **three** short findings from your dashboard. Each finding must cite a number from one of your visuals.

---

**End of paper. Good luck.**
