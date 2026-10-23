# healthcare-dashboard-mysql-powerbi
End-to-end healthcare admissions analysis: cleaned and analysed 54,966 patient records in MySQL, then built an interactive Power BI dashboard covering billing, length of stay, admissions and test results.

# Healthcare Admissions Analysis: MySQL and Power BI

An end-to-end data analysis project: raw patient admission data is cleaned and analysed in **MySQL**, then presented in an interactive **Power BI** dashboard.

![Dashboard](dashboard/healthcare_dashboard.jpg)

---

## Table of contents
1. [Project overview](#1-project-overview)
2. [Dataset](#2-dataset)
3. [Tools used](#3-tools-used)
4. [Workflow](#4-workflow)
5. [Data cleaning](#5-data-cleaning)
6. [Analysis and key findings](#6-analysis-and-key-findings)
7. [Data quality findings](#7-data-quality-findings)
8. [Power BI dashboard](#8-power-bi-dashboard)
9. [Recommendations](#9-recommendations)
10. [Limitations](#10-limitations)
11. [Repository structure](#11-repository-structure)
12. [How to reproduce](#12-how-to-reproduce)

---

## 1. Project overview

**Objective:** to clean and analyse patient admission data using MySQL, and to build an interactive Power BI dashboard that gives hospital management a clear view of admissions, billing, length of stay and test outcomes.

**Business questions answered**
1. How many patients were admitted, and how has that changed over time?
2. How much revenue (billing) was generated, and which insurance providers contribute the most?
3. How long do patients stay on average?
4. What is the age and gender profile of the patients?
5. How are patients spread across conditions and admission types?
6. What share of test results are normal, abnormal or inconclusive?

**Intended audience:** hospital management and finance teams.

## 2. Dataset

| Item | Detail |
|---|---|
| Source | Kaggle, *Healthcare Dataset* (`healthcare_dataset.csv`) |
| Size | 55,500 rows and 15 columns |
| Period | May 2019 to May 2024 |
| Nature | **Synthetic** (generated for practice, not real patient records) |

**Columns:** Name, Age, Gender, Blood Type, Medical Condition, Date of Admission, Doctor, Hospital, Insurance Provider, Billing Amount, Room Number, Admission Type, Discharge Date, Medication, Test Results.

> The CSV file is not included in this repository. Download it from Kaggle and import it into MySQL (see [How to reproduce](#12-how-to-reproduce)).

## 3. Tools used

- **MySQL 8.0** and MySQL Workbench: data import, cleaning and SQL analysis
- **Power BI Desktop:** DAX measures, visuals and slicers
- **MySQL Connector/NET:** connects Power BI to MySQL

## 4. Workflow

```
Kaggle CSV  ->  MySQL (healthcare_dataset)  ->  Cleaning  ->  healthcare_clean
                                                                    |
                                          SQL analysis  <-----------+----------->  Power BI dashboard
```

## 5. Data cleaning

All cleaning was done in MySQL, and the raw table was left untouched. The cleaned data lives in a new table, `healthcare_clean`.

| Step | Action | Result |
|---|---|---|
| 1 | Removed exact duplicate rows with `SELECT DISTINCT` | 55,500 rows reduced to **54,966** (534 duplicates removed) |
| 2 | Renamed columns to `snake_case` (for example `Medical Condition` to `medical_condition`) | Simpler queries |
| 3 | Converted `Date of Admission` and `Discharge Date` from text to `DATE` using `STR_TO_DATE` | Date functions such as `YEAR()` and `DATEDIFF()` work |
| 4 | Rounded `Billing Amount` to 2 decimals | Clean currency values |

The full script is in [`sql/healthcare_analysis.sql`](sql/healthcare_analysis.sql).

---

## 6. Analysis and key findings

Each section below shows the query, the result screenshot and the key finding. All figures come from the cleaned table (54,966 records).

### 6.1 Total patients

```sql
SELECT COUNT(*) AS total_patients FROM healthcare.healthcare_clean;
```
![Total patients](screenshots/01_total_patients.jpg)

**Finding:** the cleaned data holds **54,966** admission records. Each row is one admission, so a person admitted twice would be counted twice.

### 6.2 Total billing

```sql
SELECT ROUND(SUM(billing_amount),2) AS total_billing FROM healthcare.healthcare_clean;
```
![Total billing](screenshots/02_total_billing.jpg)

**Finding:** total billing is **$1,404,068,337.89**, about **$1.40 billion**.

### 6.3 Average bill per patient

```sql
SELECT ROUND(AVG(billing_amount),2) AS avg_bill FROM healthcare.healthcare_clean;
```
![Average bill per patient](screenshots/03_avg_bill_per_patient.jpg)

**Finding:** the average bill is **$25,544.31** per admission.

### 6.4 Average length of stay

```sql
SELECT ROUND(AVG(DATEDIFF(discharge_date, date_of_admission)),1) AS avg_stay_days
FROM healthcare.healthcare_clean;
```
![Average length of stay](screenshots/04_avg_length_of_stay.jpg)

**Finding:** patients stay **15.5 days** on average. Combined with the average bill, a stay costs roughly **$1,650 per day**.

### 6.5 Patient age

```sql
SELECT ROUND(AVG(age),1) AS avg_age, MIN(age) AS youngest, MAX(age) AS oldest
FROM healthcare.healthcare_clean;
```
![Average age](screenshots/05_avg_age.jpg)

**Finding:** the average age is **51.5 years**, ranging from **13 to 89**.

### 6.6 Patients by medical condition

```sql
SELECT medical_condition, COUNT(*) AS total_patients
FROM healthcare.healthcare_clean
GROUP BY medical_condition ORDER BY total_patients DESC;
```
![Patients by condition](screenshots/06_patients_by_condition.jpg)

**Finding:** **Arthritis** has the most patients (9,218) and **Asthma** the fewest (9,095). The gap is only 123 patients (about 1.4%), so the six conditions are almost evenly spread.

### 6.7 Billing by insurance provider

```sql
SELECT insurance_provider, ROUND(SUM(billing_amount),2) AS total_billing
FROM healthcare.healthcare_clean
GROUP BY insurance_provider ORDER BY total_billing DESC;
```
![Billing by insurance](screenshots/07_billing_by_insurance.jpg)

**Finding:** **Cigna** generates the most billing (about $284.3M, 20.3% of the total) and **Aetna** the least (about $276.5M, 19.7%). The five insurers each hold roughly one fifth of revenue, so no single payer dominates.

### 6.8 Admissions by year

```sql
SELECT YEAR(date_of_admission) AS yr, COUNT(*) AS admissions
FROM healthcare.healthcare_clean
GROUP BY yr ORDER BY yr;
```
![Admissions by year](screenshots/08_admissions_by_year.jpg)

**Finding:** admissions peaked in **2020 (11,172)** and then held steady at about 10,800 to 10,900 a year through 2023. **2024 is lowest (3,827)**, and 2019 (7,300) is also low. This is because the data only covers **May 2019 to May 2024**, so 2019 and 2024 are partial years and do not represent a real decline.

### 6.9 Test results

```sql
SELECT test_results, COUNT(*) AS total
FROM healthcare.healthcare_clean GROUP BY test_results;
```
![Test results](screenshots/09_test_results.jpg)

**Finding:** **Abnormal** is the most common result (18,437) and **Inconclusive** the least (18,198). The three outcomes are each about one third of all tests.

### 6.10 Average bill by medical condition

```sql
SELECT medical_condition, ROUND(AVG(billing_amount),2) AS avg_bill
FROM healthcare.healthcare_clean
GROUP BY medical_condition ORDER BY avg_bill DESC;
```
![Average bill by condition](screenshots/10_average_bill_per_condition.jpg)

**Finding:** **Obesity** has the highest average bill ($25,804.36) and **Cancer** the lowest ($25,152.32). The difference is about $650, or 2.6%.

### 6.11 Admission type

```sql
SELECT admission_type, COUNT(*) AS total
FROM healthcare.healthcare_clean GROUP BY admission_type;
```
![Admission type](screenshots/11_admission_type.jpg)

**Finding:** **Elective** admissions are highest (18,473), then Urgent (18,391), and **Emergency** is lowest (18,102). The split is nearly equal.

### 6.12 Gender

```sql
SELECT gender, COUNT(*) AS total
FROM healthcare.healthcare_clean GROUP BY gender;
```
![Gender](screenshots/12_gender.jpg)

**Finding:** **Male** patients number 27,496 and **Female** patients 27,470, a difference of only 26. The data is balanced by gender.

---

## 7. Data quality findings

### 7.1 Negative bills

```sql
SELECT COUNT(*) AS negative_bills, MIN(billing_amount) AS lowest
FROM healthcare.healthcare_clean WHERE billing_amount < 0;
```
![Negative bills](screenshots/13_negative_bills.jpg)

**Finding:** **106 records** have a negative billing amount, and the lowest is **-$2,008.49**. A bill should not be negative. They could be refunds or entry errors and should be verified at the source. They were kept in the analysis because they change total billing by less than 0.01%.

### 7.2 Other issues found
- **534 exact duplicate rows** (about 1% of the file), removed during cleaning.
- **Inconsistent capitalization** in patient names (for example "Bobby JacksOn").
- **Messy hospital names** with stray commas and "and", plus about 39,900 distinct hospital names and 40,300 doctor names across 54,966 records. Almost every hospital and doctor appears only once or twice, so hospital-level and doctor-level comparisons are not meaningful.

---

## 8. Power BI dashboard

The dashboard reads the `healthcare_clean` table directly from MySQL.

**KPI cards:** Total Patients, Total Billing, Average Bill, Average Length of Stay, Average Age.

**Charts:**
- Patients by medication (clustered column)
- Total billing by insurance provider (clustered bar)
- Patients by year (line/area)
- Patients by test result (clustered column)

**Slicers:** gender, admission type, medical condition.

**DAX measures**
```DAX
Total Patients = COUNTROWS(healthcare_clean)
Total Billing = SUM(healthcare_clean[billing_amount])
Avg Bill = AVERAGE(healthcare_clean[billing_amount])
Avg Length of Stay = AVERAGEX(healthcare_clean, DATEDIFF(healthcare_clean[date_of_admission], healthcare_clean[discharge_date], DAY))
Avg Age = AVERAGE(healthcare_clean[age])
```

The report file is in [`dashboard/BI_INTELLECT.pbix`](dashboard/BI_INTELLECT.pbix).

---

## 9. Recommendations

Written as if the data were real hospital data.

1. **Plan capacity around a 15-day stay.** With about 11,000 admissions a year and a 15.5-day average stay, bed and staffing plans should be built on that figure. Reviewing the longest stays (up to 30 days) would show where costs build up.
2. **Keep payer relationships balanced.** No insurer holds more than about 20% of revenue, which lowers risk. Track each insurer monthly so any shift is seen early.
3. **Investigate the 106 negative bills.** Confirm whether they are refunds or errors, and add a validation rule to block negative amounts at entry.
4. **Standardize hospital and doctor records.** Use a reliable ID for each, so performance can be compared properly.
5. **Collect outcome data.** Test results are not linked to treatment or recovery, so questions about effectiveness need extra fields.

## 10. Limitations

- The dataset is **synthetic**, and its groups are almost perfectly balanced. Differences of 1 to 3% between groups (conditions, insurers, admission types) are too small to support firm conclusions. They are shown to demonstrate the method.
- Billing amounts are not linked to services or procedures.
- 2019 and 2024 are partial years.
- Each row is an admission, not a unique patient.

## 11. Repository structure

```
healthcare-admissions-analysis/
|-- README.md
|-- sql/
|   `-- healthcare_analysis.sql      cleaning and analysis queries
|-- screenshots/                     MySQL query results (01 to 13)
`-- dashboard/
    |-- healthcare_dashboard.jpg     dashboard preview
    `-- BI_INTELLECT.pbix            Power BI report
```

## 12. How to reproduce

1. Download `healthcare_dataset.csv` from Kaggle.
2. In MySQL Workbench, create a schema named `healthcare` and import the CSV as the table `healthcare_dataset`.
3. Run the cleaning section of [`sql/healthcare_analysis.sql`](sql/healthcare_analysis.sql) to create `healthcare_clean`. You should get **54,966** rows.
4. Run the analysis queries and compare your results with the screenshots.
5. Install **MySQL Connector/NET**, open `dashboard/BI_INTELLECT.pbix` in Power BI Desktop, and update the data source (server `localhost`, database `healthcare`) with your own MySQL login.

---

**Author:** [Your name] | [LinkedIn] | [Email]
