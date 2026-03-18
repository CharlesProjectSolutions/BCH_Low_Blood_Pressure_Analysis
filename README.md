# 📌 Project Overview
During pediatric surgeries, every moment is critical. Monitoring patient signs, especially blood pressure, can mean the difference between life and death. This project analyzes blood pressure data to identifies cases where **systolic blood pressure dropped below a safe threshold** for at least **14 consecutive minutes**, helping **BCH** assess risks and quickly spot potentially dangerous trends in real-time.

## ⚠️ Problem Statement
BCH face a complex challenge: identifying prolonged periods of low blood pressure across different pediatric age groups, using data from multiple sources.  Without clear tracking, they may miss critical cases, which can lead to potential health risks for children undergoing pediatric surgery. 

## 🎯 Objective & Requirements
Our project aimed to:
- Develop an automated system to detect low blood pressure events
- Create age-specific thresholds for identifying critical BP drops
- Find cases where blood pressure dropped below age-appropriate thresholds for 14+ continuous minutes.
- Generate comprehensive a final report containing Person ID, Service Date, and duration (in minutes) of each low blood pressure episode.


  ## 🛠️ Methodology
We developed a Python-powered ETL (Extract, Transform, Load) solution designed to:
1. Import necessary libraries (pandas) needed for our analysis
2. Read both CSV files into a Pandas DataFrames
3. Convert date/time fields to proper datetime format
4. Integrate and preprocess patient demographic and blood pressure data
5. Implement dynamic, age-based blood pressure thresholds calculations based on patient age
6. Analyze and identify continuous periods of low blood pressure
7. Generate detailed, clear reports for medical review

## 📊 Key Result Findings & Insights
Our analysis revealed critical insights:
- **5 unique patients identified with prolonged low blood pressure events** during surgery.
- **Low BP periods ranging from 14 to 24 continuous minutes**.
- **Patient 987 had the longest duration (24 minutes)**.
- **Potential critical moments that might have gone unnoticed without advanced analysis**.

## 🔮 Future Enhancements
- Implement **real-time monitoring System** when low BP is detected 🚨.
- Expand dataset for **deeper analysis** 🏥.
- Proactive patient risk identification 🤖.

## 🚀 Skills Demonstrated
- **ETL (Extract, Transform, Load)** – Cleaning and merging raw data.
- **Data Analysis** – Analyzing & identifying patterns in healthcare data.
- **Visualization** – Creating graphs for insights.
- **Reporting** – Presenting findings in PowerPoint.
- **Problem-Solving** – Data analysis, handling missing, and invalid data.

## 🔧 Technologies Used
- **Python** (Pandas, Matplotlib)
- **Excel/CSV** (Data Storage)
- **PowerPoint** (Report Presentation)





# Hospital Analytics — Data Model Document

**Version:** 1.0 | **Schema:** Star Schema | **Target:** MS SQL Server — HospitalDW
**Date:** March 2026 | **Status:** Approved

---

## 1. Architecture Decision: Star Schema vs Snowflake

### Decision: Star Schema Selected

After evaluating both approaches against the specific characteristics of this dataset, the Star Schema was selected as the appropriate architecture. The rationale is documented below.

### Comparison Table

| Dimension | Star Schema | Snowflake Schema |
|---|---|---|
| Normalisation Level | Denormalised dimensions (1NF/2NF) | Fully normalised dimensions (3NF) |
| Joins per Query | Minimal — fact joins directly to dimension; 1 join per dimension | Multiple — parent/child dimension tables require chained joins |
| Storage | Higher (some redundancy in dimension tables) | Lower (normalised, less repetition) |
| BI Tool Compatibility | Excellent — Tableau, Power BI, Chart.js all work natively with star | Moderate — BI tools can handle it but require more complex relationship definitions |
| Query Performance | Fast — fewer joins, dimension tables fit in memory | Slower for ad-hoc queries — additional joins add execution time |
| Maintenance | Simpler — update one dimension table | More complex — updates may cascade across parent/child tables |
| ETL Complexity | Lower — single insert/update per dimension | Higher — must maintain parent dimension keys before child dimensions |
| When to Use | Analytics-first workloads, BI dashboards, < 10M rows, single-domain datasets | Data vault migrations, very large dimensions with high cardinality attributes, strict storage constraints |

### Specific Reasons for this Dataset

1. **Small dimension cardinality:** The largest dimension (dim_date) has 4,053 rows; dim_payers has 10 rows; dim_encounter_class has 6 rows. There is no storage justification for snowflaking small reference tables.
2. **BI-first delivery:** Both the HTML/Chart.js dashboard and the planned Tableau workbook consume this model directly. Both tools perform significantly better against star schemas with direct fact-to-dimension joins.
3. **Single analytical domain:** This project covers one domain (hospital encounters). Snowflake is most beneficial in multi-domain enterprise data warehouses where dimension sharing across subject areas justifies normalisation.
4. **Query simplicity:** The analytical SQL library (03_analysis.sql) contains 15 queries. With a star schema, the most complex query requires 4 joins. A snowflake equivalent would require 7-9 joins for the same result, increasing maintenance burden.
5. **Synthetic data stability:** Synthea data does not change retroactively. There is no ongoing update stream where normalised storage would provide a meaningful benefit.

---

## 2. Source-to-Target Mapping

| Source File | Source Column | Target Table | Target Column | Data Type | Transformation Applied |
|---|---|---|---|---|---|
| patients.csv | Id | dim_patients | patient_id | CHAR(36) | REPLACE(Id, CHAR(65279), '') — strips UTF-8 BOM from first record |
| patients.csv | BIRTHDATE | dim_patients | birth_date | DATE | CAST(NVARCHAR -> DATE); format YYYY-MM-DD |
| patients.csv | DEATHDATE | dim_patients | death_date | DATE | CAST(NVARCHAR -> DATE); NULL if empty string |
| patients.csv | SSN | dim_patients | ssn | NVARCHAR(20) | Pass-through; masked in production views |
| patients.csv | DRIVERS | dim_patients | drivers_license | NVARCHAR(20) | Pass-through |
| patients.csv | PASSPORT | dim_patients | passport | NVARCHAR(20) | Pass-through |
| patients.csv | PREFIX | dim_patients | prefix | NVARCHAR(10) | Pass-through |
| patients.csv | FIRST | dim_patients | first_name | NVARCHAR(100) | Pass-through |
| patients.csv | LAST | dim_patients | last_name | NVARCHAR(100) | Pass-through |
| patients.csv | SUFFIX | dim_patients | suffix | NVARCHAR(10) | Pass-through |
| patients.csv | MAIDEN | dim_patients | maiden_name | NVARCHAR(100) | Pass-through |
| patients.csv | MARITAL | dim_patients | marital_status | CHAR(1) | COALESCE(NULLIF(TRIM(MARITAL),''), 'U') — null/blank mapped to 'U' (Unknown) |
| patients.csv | RACE | dim_patients | race | NVARCHAR(50) | Pass-through; lowercase normalisation |
| patients.csv | ETHNICITY | dim_patients | ethnicity | NVARCHAR(50) | Pass-through; lowercase normalisation |
| patients.csv | GENDER | dim_patients | gender | CHAR(1) | Pass-through |
| patients.csv | BIRTHPLACE | dim_patients | birthplace | NVARCHAR(200) | Pass-through |
| patients.csv | ADDRESS | dim_patients | address | NVARCHAR(200) | Pass-through |
| patients.csv | CITY | dim_patients | city | NVARCHAR(100) | Pass-through |
| patients.csv | STATE | dim_patients | state | CHAR(2) | Pass-through |
| patients.csv | ZIP | dim_patients | zip | CHAR(5) | RIGHT('00000' + CAST(CAST(ZIP AS FLOAT) AS INT), 5) — converts float-encoded ZIP (e.g. 2101.0) to zero-padded string (e.g. 02101) |
| patients.csv | (computed) | dim_patients | age | INT | DATEDIFF(YEAR, birth_date, GETDATE()) — or DATEDIFF to death_date if deceased |
| patients.csv | (computed) | dim_patients | is_deceased | BIT | CASE WHEN death_date IS NOT NULL THEN 1 ELSE 0 END |
| payers.csv | Id | dim_payers | payer_id | CHAR(36) | REPLACE — BOM strip on first record |
| payers.csv | NAME | dim_payers | payer_name | NVARCHAR(100) | TRIM; pass-through |
| payers.csv | ADDRESS | dim_payers | address | NVARCHAR(200) | Pass-through |
| payers.csv | CITY | dim_payers | city | NVARCHAR(100) | Pass-through |
| payers.csv | STATE_HEADQUARTERED | dim_payers | state | CHAR(2) | Pass-through |
| payers.csv | ZIP | dim_payers | zip | CHAR(5) | Same ZIP float->CHAR(5) transform as patients if needed |
| payers.csv | PHONE | dim_payers | phone | NVARCHAR(20) | Pass-through |
| encounters.csv | Id | fact_encounters | encounter_id | CHAR(36) | REPLACE — BOM strip |
| encounters.csv | START | fact_encounters | encounter_start | DATETIME2 | CAST datetimeoffset string -> DATETIME2; strip timezone offset |
| encounters.csv | STOP | fact_encounters | encounter_stop | DATETIME2 | CAST datetimeoffset string -> DATETIME2 |
| encounters.csv | (computed) | fact_encounters | encounter_duration_min | INT | DATEDIFF(MINUTE, encounter_start, encounter_stop) |
| encounters.csv | PATIENT | fact_encounters | patient_id | CHAR(36) | FK lookup to dim_patients.patient_id |
| encounters.csv | ORGANIZATION | fact_encounters | organization_id | NVARCHAR(100) | Pass-through |
| encounters.csv | PAYER | fact_encounters | payer_id | CHAR(36) | FK lookup to dim_payers.payer_id |
| encounters.csv | ENCOUNTERCLASS | fact_encounters | encounter_class_id | INT | FK lookup to dim_encounter_class.encounter_class_id via ENCOUNTERCLASS string |
| encounters.csv | CODE | fact_encounters | encounter_code | NVARCHAR(20) | Pass-through |
| encounters.csv | DESCRIPTION | fact_encounters | encounter_description | NVARCHAR(500) | Pass-through |
| encounters.csv | BASE_ENCOUNTER_COST | fact_encounters | base_encounter_cost | DECIMAL(12,2) | CAST NVARCHAR -> DECIMAL |
| encounters.csv | TOTAL_CLAIM_COST | fact_encounters | total_claim_cost | DECIMAL(12,2) | CAST NVARCHAR -> DECIMAL |
| encounters.csv | PAYER_COVERAGE | fact_encounters | payer_coverage | DECIMAL(12,2) | CAST NVARCHAR -> DECIMAL |
| encounters.csv | (computed) | fact_encounters | patient_out_of_pocket | DECIMAL(12,2) | total_claim_cost - payer_coverage |
| encounters.csv | REASONCODE | fact_encounters | reason_code | NVARCHAR(20) | CAST float string -> INT64 then NVARCHAR; NULL if missing |
| encounters.csv | REASONDESCRIPTION | fact_encounters | reason_description | NVARCHAR(500) | Pass-through; NULL for ~70% of rows |
| encounters.csv | START | fact_encounters | date_id | INT | FK lookup to dim_date.date_id via CAST(encounter_start AS DATE) |
| procedures.csv | START | fact_procedures | procedure_start | DATETIME2 | CAST datetimeoffset string -> DATETIME2 |
| procedures.csv | STOP | fact_procedures | procedure_stop | DATETIME2 | CAST datetimeoffset string -> DATETIME2 |
| procedures.csv | PATIENT | fact_procedures | patient_id | CHAR(36) | Pass-through; FK reference |
| procedures.csv | ENCOUNTER | fact_procedures | encounter_id | CHAR(36) | FK lookup to fact_encounters.encounter_id |
| procedures.csv | CODE | fact_procedures | procedure_code | NVARCHAR(20) | Pass-through |
| procedures.csv | DESCRIPTION | fact_procedures | procedure_description | NVARCHAR(500) | Pass-through |
| procedures.csv | BASE_COST | fact_procedures | procedure_base_cost | DECIMAL(12,2) | CAST NVARCHAR -> DECIMAL |
| procedures.csv | REASONCODE | fact_procedures | reason_code | NVARCHAR(20) | Same float->INT->NVARCHAR transform as encounters |
| procedures.csv | REASONDESCRIPTION | fact_procedures | reason_description | NVARCHAR(500) | Pass-through |
| procedures.csv | (computed) | fact_procedures | procedure_id | INT | ROW_NUMBER() OVER (ORDER BY encounter_id, procedure_start) — surrogate key |

---

## 3. ASCII Star Schema Diagram

```
                              +------------------+
                              |   dim_date       |
                              |------------------|
                              | date_id (PK)     |
                              | full_date        |
                              | year             |
                              | quarter          |
                              | month            |
                              | month_name       |
                              | week             |
                              | day_of_week      |
                              | day_name         |
                              | is_weekend       |
                              +--------+---------+
                                       |
                                       | FK: date_id
                                       |
+------------------+         +---------v---------+         +------------------+
|  dim_patients    |         |  fact_encounters  |         |   dim_payers     |
|------------------|         |-------------------|         |------------------|
| patient_id (PK)  +-------->+ encounter_id (PK) +-------->+ payer_id (PK)    |
| birth_date       | FK:     | patient_id (FK)   | FK:     | payer_name       |
| death_date       | patient | date_id (FK)      | payer   | address          |
| first_name       | _id     | payer_id (FK)     | _id     | city             |
| last_name        |         | encounter_class   |         | state            |
| gender           |         |   _id (FK)        |         | zip              |
| race             |         | encounter_start   |         | phone            |
| ethnicity        |         | encounter_stop    |         +------------------+
| marital_status   |         | encounter_duration|
| zip              |         |   _min            |
| age              |         | base_encounter    |
| is_deceased      |         |   _cost           |
+------------------+         | total_claim_cost  |
                              | payer_coverage    |         +------------------+
                              | patient_out_of   |         | dim_encounter    |
                              |   _pocket         |         |   _class         |
                              | reason_code       |         |------------------|
                              | reason_description+-------->+ encounter_class  |
                              +--------+----------+ FK:     |   _id (PK)       |
                                       |          enc cls   | encounter_class  |
                                       |          _id       |   _name          |
                                       | FK:                +------------------+
                                       | encounter_id
                                       |
                              +--------v----------+
                              |  fact_procedures  |
                              |-------------------|
                              | procedure_id (PK) |
                              | encounter_id (FK) |
                              | patient_id        |
                              | procedure_start   |
                              | procedure_stop    |
                              | procedure_code    |
                              | procedure_        |
                              |   description     |
                              | procedure_base    |
                              |   _cost           |
                              | reason_code       |
                              | reason_description|
                              +-------------------+
```

---

## 4. Table Definitions

### 4.1 dim_patients

| Attribute | Detail |
|---|---|
| **Purpose** | Conformed patient dimension; one row per unique patient; stores demographic and geographic attributes used in population health analysis |
| **Grain** | One row = one unique patient (identified by Synthea patient UUID) |
| **Row Count** | 974 |
| **Primary Key** | patient_id (CHAR(36), natural key from source) |
| **Foreign Keys** | None (dimension table; referenced by fact_encounters) |

| Column | Type | Nullable | Notes |
|---|---|---|---|
| patient_id | CHAR(36) | NOT NULL | BOM-stripped UUID from patients.csv Id column |
| birth_date | DATE | NOT NULL | |
| death_date | DATE | NULL | NULL = currently alive |
| is_deceased | BIT | NOT NULL | Computed: 1 if death_date IS NOT NULL |
| age | INT | NOT NULL | Computed at load time |
| first_name | NVARCHAR(100) | NULL | |
| last_name | NVARCHAR(100) | NULL | |
| prefix | NVARCHAR(10) | NULL | |
| suffix | NVARCHAR(10) | NULL | |
| maiden_name | NVARCHAR(100) | NULL | |
| marital_status | CHAR(1) | NOT NULL | Default 'U' for unknown |
| gender | CHAR(1) | NULL | M/F |
| race | NVARCHAR(50) | NULL | |
| ethnicity | NVARCHAR(50) | NULL | |
| birthplace | NVARCHAR(200) | NULL | |
| address | NVARCHAR(200) | NULL | |
| city | NVARCHAR(100) | NULL | |
| state | CHAR(2) | NULL | |
| zip | CHAR(5) | NULL | Zero-padded 5-digit ZIP |
| ssn | NVARCHAR(20) | NULL | Masked in production views |
| drivers_license | NVARCHAR(20) | NULL | |
| passport | NVARCHAR(20) | NULL | |

---

### 4.2 dim_payers

| Attribute | Detail |
|---|---|
| **Purpose** | Payer/insurance dimension; one row per payer; used to classify encounters by insurance type |
| **Grain** | One row = one payer organization |
| **Row Count** | 10 |
| **Primary Key** | payer_id (CHAR(36), natural key from source) |
| **Foreign Keys** | None (dimension table; referenced by fact_encounters) |

| Column | Type | Nullable | Notes |
|---|---|---|---|
| payer_id | CHAR(36) | NOT NULL | BOM-stripped UUID |
| payer_name | NVARCHAR(100) | NOT NULL | Includes 'NO_INSURANCE' as a valid value |
| address | NVARCHAR(200) | NULL | |
| city | NVARCHAR(100) | NULL | |
| state | CHAR(2) | NULL | |
| zip | CHAR(5) | NULL | |
| phone | NVARCHAR(20) | NULL | |

---

### 4.3 dim_encounter_class

| Attribute | Detail |
|---|---|
| **Purpose** | Encounter classification dimension; maps string class names to integer surrogate keys; enables consistent grouping |
| **Grain** | One row = one encounter class type |
| **Row Count** | 6 |
| **Primary Key** | encounter_class_id (INT, surrogate, assigned by ROW_NUMBER() on first load) |
| **Foreign Keys** | None (dimension table; referenced by fact_encounters) |

| Column | Type | Nullable | Notes |
|---|---|---|---|
| encounter_class_id | INT | NOT NULL | Surrogate key |
| encounter_class_name | NVARCHAR(50) | NOT NULL | Values: inpatient, outpatient, ambulatory, emergency, wellness, urgentcare |

---

### 4.4 dim_date

| Attribute | Detail |
|---|---|
| **Purpose** | Date dimension; one row per calendar date; pre-computed date attributes eliminate repeated date calculations in queries |
| **Grain** | One row = one calendar date |
| **Row Count** | 4,053 (spanning 2011–2022 encounter date range) |
| **Primary Key** | date_id (INT, format YYYYMMDD, e.g. 20150601) |
| **Foreign Keys** | None (dimension table; referenced by fact_encounters via encounter_start date) |

| Column | Type | Nullable | Notes |
|---|---|---|---|
| date_id | INT | NOT NULL | YYYYMMDD integer format |
| full_date | DATE | NOT NULL | Actual date value |
| year | INT | NOT NULL | |
| quarter | INT | NOT NULL | 1-4 |
| month | INT | NOT NULL | 1-12 |
| month_name | NVARCHAR(10) | NOT NULL | DATENAME(month, full_date) |
| week | INT | NOT NULL | ISO week: DATEPART(iso_week, full_date) |
| day_of_week | INT | NOT NULL | 0=Sunday through 6=Saturday |
| day_name | NVARCHAR(10) | NOT NULL | DATENAME(dw, full_date) |
| is_weekend | BIT | NOT NULL | 1 if Saturday or Sunday |

---

### 4.5 fact_encounters

| Attribute | Detail |
|---|---|
| **Purpose** | Central fact table at encounter grain; one row per patient encounter; holds all financial measures and FK references to all four dimensions |
| **Grain** | One row = one hospital encounter (visit) |
| **Row Count** | 27,891 |
| **Primary Key** | encounter_id (CHAR(36), natural key from source) |
| **Foreign Keys** | patient_id -> dim_patients.patient_id; payer_id -> dim_payers.payer_id; encounter_class_id -> dim_encounter_class.encounter_class_id; date_id -> dim_date.date_id |

| Column | Type | Nullable | Notes |
|---|---|---|---|
| encounter_id | CHAR(36) | NOT NULL | Natural key; BOM-stripped |
| patient_id | CHAR(36) | NOT NULL | FK to dim_patients |
| payer_id | CHAR(36) | NOT NULL | FK to dim_payers |
| encounter_class_id | INT | NOT NULL | FK to dim_encounter_class |
| date_id | INT | NOT NULL | FK to dim_date; derived from encounter_start |
| encounter_start | DATETIME2 | NOT NULL | Parsed from datetimeoffset source string |
| encounter_stop | DATETIME2 | NULL | Parsed from datetimeoffset source string |
| encounter_duration_min | INT | NULL | DATEDIFF(MINUTE, start, stop) |
| organization_id | NVARCHAR(100) | NULL | |
| encounter_code | NVARCHAR(20) | NULL | SNOMED-CT code |
| encounter_description | NVARCHAR(500) | NULL | |
| base_encounter_cost | DECIMAL(12,2) | NULL | |
| total_claim_cost | DECIMAL(12,2) | NULL | Primary revenue measure |
| payer_coverage | DECIMAL(12,2) | NULL | Amount paid by payer |
| patient_out_of_pocket | DECIMAL(12,2) | NULL | Computed: total_claim_cost - payer_coverage |
| reason_code | NVARCHAR(20) | NULL | NULL for ~70% of encounters |
| reason_description | NVARCHAR(500) | NULL | NULL for ~70% of encounters |

---

### 4.6 fact_procedures

| Attribute | Detail |
|---|---|
| **Purpose** | Procedure-level fact table; one row per procedure performed; child of fact_encounters; enables procedure-level cost and frequency analysis |
| **Grain** | One row = one procedure performed within one encounter |
| **Row Count** | 47,701 |
| **Primary Key** | procedure_id (INT, surrogate key assigned by ROW_NUMBER()) |
| **Foreign Keys** | encounter_id -> fact_encounters.encounter_id |

| Column | Type | Nullable | Notes |
|---|---|---|---|
| procedure_id | INT | NOT NULL | Surrogate key; ROW_NUMBER() assigned |
| encounter_id | CHAR(36) | NOT NULL | FK to fact_encounters |
| patient_id | CHAR(36) | NULL | Denormalised for convenience; FK reference |
| procedure_start | DATETIME2 | NULL | |
| procedure_stop | DATETIME2 | NULL | |
| procedure_code | NVARCHAR(20) | NULL | SNOMED-CT code |
| procedure_description | NVARCHAR(500) | NULL | |
| procedure_base_cost | DECIMAL(12,2) | NULL | |
| reason_code | NVARCHAR(20) | NULL | |
| reason_description | NVARCHAR(500) | NULL | |

---

## 5. Grain Management & Fan-Out Prevention

### The Fan-Out Problem

In a star schema, when a fact table at one grain (encounters) is joined to a related table at a finer grain (procedures, where multiple procedure rows exist per encounter), aggregations computed at the encounter level will be inflated — each encounter's financial measures will be summed once per procedure row instead of once per encounter. This is called fan-out or a "chasm trap."

### Concrete Example of What Goes Wrong Without Aggregation

Suppose encounter E001 has total_claim_cost = $1,000 and has 3 procedure rows.

Without grain management:
```sql
SELECT SUM(fe.total_claim_cost)
FROM fact_encounters fe
JOIN fact_procedures fp ON fe.encounter_id = fp.encounter_id
-- Result: $3,000 (wrong — counted 3 times due to 3 procedure rows)
```

With aggregation step (proc_agg):
```sql
WITH proc_agg AS (
    SELECT encounter_id,
           COUNT(*) AS procedure_count,
           SUM(procedure_base_cost) AS total_procedure_cost
    FROM fact_procedures
    GROUP BY encounter_id
)
SELECT SUM(fe.total_claim_cost)
FROM fact_encounters fe
LEFT JOIN proc_agg pa ON fe.encounter_id = pa.encounter_id
-- Result: $1,000 (correct — encounter grain preserved)
```

### Resolution Applied

All queries in 03_analysis.sql that join fact_encounters to fact_procedures first aggregate fact_procedures to encounter grain using a CTE named `proc_agg`:

```sql
WITH proc_agg AS (
    SELECT
        encounter_id,
        COUNT(*)              AS procedure_count,
        SUM(procedure_base_cost) AS total_procedure_cost
    FROM dbo.fact_procedures
    GROUP BY encounter_id
)
```

This CTE is then LEFT JOINed to fact_encounters on encounter_id, ensuring that the encounter grain is preserved and financial measures are not inflated. The `master_encounter.csv` export was also built using this pattern, explaining why it has 27,891 rows (one per encounter) despite 47,701 procedure rows existing in the source.

---

## 6. Derived / Enrichment Columns

| Column | Source Table | Formula | Business Purpose |
|---|---|---|---|
| patient_out_of_pocket | fact_encounters | `total_claim_cost - payer_coverage` | Quantifies patient financial burden; key input to OOP rate KPI |
| encounter_duration_min | fact_encounters | `DATEDIFF(MINUTE, encounter_start, encounter_stop)` | Enables duration analysis by class; proxy for resource consumption |
| age | dim_patients | `DATEDIFF(YEAR, birth_date, ISNULL(death_date, GETDATE()))` | Enables age cohort analysis in population health; computed at ETL load time |
| is_deceased | dim_patients | `CASE WHEN death_date IS NOT NULL THEN 1 ELSE 0 END` | Enables survival analysis and deceased patient filtering |
| date_id | fact_encounters | `CAST(CONVERT(DATE, encounter_start) AS INT) * ... formatted as YYYYMMDD` | Integer FK to dim_date; enables all calendar-based grouping without string parsing |
| procedure_id | fact_procedures | `ROW_NUMBER() OVER (ORDER BY encounter_id, procedure_start)` | Provides stable surrogate PK for procedure fact table (source has no natural key) |
| is_weekend | dim_date | `CASE WHEN DATEPART(dw, full_date) IN (1,7) THEN 1 ELSE 0 END` | Supports day-type analysis; pre-computed to avoid repeated DATEPART in queries |
| procedure_count | (aggregation) | `COUNT(procedure_id) per encounter_id` | Pre-aggregated in master_encounter.csv; used in procedures per encounter KPI |
| total_procedure_cost | (aggregation) | `SUM(procedure_base_cost) per encounter_id` | Pre-aggregated in master_encounter.csv; used in procedure cost analysis |
| marital_status (default) | dim_patients | `COALESCE(NULLIF(TRIM(source_value), ''), 'U')` | Ensures analytical completeness; avoids NULL exclusions in marital status grouping |

---

## 7. Data Quality Findings

| Issue | Column | Rows Affected | Root Cause | Resolution | Why It Mattered |
|---|---|---|---|---|---|
| UTF-8 BOM character (CHAR 65279) | All Id columns (first row of each CSV) | 1 row per file (4 total) | Windows CSV exports with UTF-8 BOM encoding prepend a zero-width no-break space (U+FEFF) to the first column of the first data row | `REPLACE(column_value, CHAR(65279), '')` applied in all usp_Load* procedures for Id columns | Without stripping, the first patient/payer/encounter/procedure Id would not match any FK lookup, creating 1 orphaned record in each dimension and cascading NULL FK failures in fact tables |
| ZIP code stored as float | ZIP column | All rows in patients.csv | Synthea generates ZIP as numeric; CSV export converts to float notation (e.g., 2101.0 instead of 02101) | `RIGHT('00000' + CAST(CAST(ZIP AS FLOAT) AS INT), 5)` — converts float to integer then zero-pads to 5 chars | Boston-area ZIP codes beginning with 0 (e.g., 02101) would be stored as 4-digit strings, breaking any geographic analysis or mapping |
| NULL MARITAL status | MARITAL column | Approx. 15% of patient rows | Synthea does not assign marital status to all patient records | `COALESCE(NULLIF(TRIM(MARITAL), ''), 'U')` — maps NULL and blank to 'U' (Unknown) | NULL marital status would cause those patients to be excluded from GROUP BY marital_status queries, understating totals |
| 70% null REASONCODE / REASONDESCRIPTION | REASONCODE, REASONDESCRIPTION | ~19,500 of 27,891 encounter rows | Synthea assigns reason codes only to encounters that resulted from a specific simulated health event; administrative and wellness visits have no reason code | Documented as accepted data gap; all condition queries include `WHERE reason_description IS NOT NULL` | If NULL reason codes were included in condition frequency counts without filtering, they would dominate the "top conditions" report as a blank category, masking all real condition insights |
| REASONCODE stored as float | REASONCODE | All non-null rows | Same float formatting issue as ZIP; SNOMED-CT codes stored as 1.27e+10 style floats in CSV | `CAST(CAST(NULLIF(REASONCODE,'') AS FLOAT) AS BIGINT)` applied before storing as NVARCHAR | Without conversion, SNOMED-CT codes used for condition grouping would not match lookup tables or be human-readable |
| Datetimeoffset string format | START, STOP (encounters and procedures) | All rows | Synthea outputs timestamps with timezone offset (e.g., '2015-06-01T14:32:00-04:00'); SQL Server BULK INSERT cannot auto-parse this as DATETIME2 | Staged as NVARCHAR(500), then parsed with `CAST(SWITCHOFFSET(CAST(value AS DATETIMEOFFSET), '+00:00') AS DATETIME2)` in stored procedures | Without proper parsing, encounter_start would be NULL for all rows, making the date dimension join, duration calculation, and all time-series analysis impossible |
| Orphaned procedure encounter references | ENCOUNTER column in procedures.csv | Small count (< 50 rows) | Some procedures in the source reference encounter IDs that do not appear in encounters.csv (likely data generation artefact) | LEFT JOIN used when loading fact_procedures; orphaned rows logged but not rejected; encounter_id stored even if no match in fact_encounters | Prevented hard failures during load; orphaned rows are excluded from encounter-joined queries naturally via inner join logic in analysis queries |
