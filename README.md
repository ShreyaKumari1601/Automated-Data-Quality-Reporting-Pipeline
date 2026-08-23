#Automated Data Quality & Reporting Pipeline

An end-to-end Python-based Data Quality and Reporting Pipeline that ingests messy multi-source e-commerce data, performs automated data validation, cleans and transforms the data, loads the processed data into **MySQL**, and generates recurring business reports.

This project demonstrates a practical **Data Analyst / Data Operations** workflow focused on data consistency, validation, documentation, automation, SQL, and reporting.

---

## Project Overview

In real-world organizations, data often comes from multiple operational systems and may contain:

* Missing values
* Duplicate records
* Inconsistent formats
* Invalid numeric values
* Invalid dates
* Referential integrity issues
* Inconsistent categorical values
* Outliers

Manually checking these issues before every reporting cycle is time-consuming and error-prone.

This project automates that process.

The pipeline:

1. Ingests raw CSV data
2. Validates data quality
3. Generates a data-quality scorecard
4. Cleans and transforms the data
5. Logs transformation decisions
6. Loads cleaned data into **MySQL**
7. Runs SQL-based business analysis
8. Generates an automated Excel report
9. Maintains a run-over-run changelog

---

## Architecture

```text
              Raw CSV Files
                    │
                    ▼
            ┌───────────────┐
            │ Data Ingestion│
            │    Pandas     │
            └───────┬───────┘
                    │
                    ▼
            ┌───────────────┐
            │ Data Validation│
            │ Missing Values │
            │ Duplicates     │
            │ Data Types     │
            │ Date Checks    │
            │ Range Checks   │
            │ Referential    │
            │ Integrity      │
            └───────┬───────┘
                    │
                    ▼
          Data Quality Scorecard
                    │
                    ▼
            ┌───────────────┐
            │ Transformation│
            │   & Cleaning  │
            └───────┬───────┘
                    │
                    ▼
             ┌─────────────┐
             │    MySQL    │
             │   Database  │
             └──────┬──────┘
                    │
                    ▼
             SQL Analysis
                    │
                    ▼
          Automated Excel Report
                    │
                    ▼
             Run Changelog
```

---

## Dataset

The project uses synthetic e-commerce data generated with intentional data-quality issues.

The pipeline contains four related datasets:

| Table       | Description          | Example Issues                                                           |
| ----------- | -------------------- | ------------------------------------------------------------------------ |
| `customers` | Customer information | Missing emails, duplicate records, inconsistent cities                   |
| `products`  | Product information  | Missing prices, negative prices, negative stock                          |
| `orders`    | Customer orders      | Missing customer IDs, duplicate order IDs, invalid product IDs, outliers |
| `returns`   | Product returns      | Missing reasons, orphan order IDs                                        |

The synthetic dataset is reproducible and is generated using the project's data-generation script.

---

## Tech Stack

### Programming & Data Processing

* Python
* Pandas
* NumPy
* Faker

### Database

* **MySQL**
* SQL
* SQLAlchemy
* PyMySQL

### Reporting

* Microsoft Excel
* OpenPyXL

### Automation

* Python Schedule

### Development

* Git
* GitHub
* Virtual Environment

---

# Data Quality Validation

The validation stage runs before the data is transformed.

This is important because the pipeline first measures the quality of the incoming raw data before modifying it.

## Validation checks

### 1. Schema Validation

Checks whether the expected columns exist in each source table.

### 2. Completeness

Measures missing values across columns.

Example:

```text
customers.email

Total records: 156
Missing records: 13
Missing percentage: 8.33%
```

### 3. Duplicate Detection

Checks for:

* Exact duplicate rows
* Duplicate primary keys

### 4. Date Validation

Detects inconsistent date formats and converts valid dates into a standardized format.

Example:

```text
DD/MM/YYYY
YYYY-MM-DD
MM-DD-YYYY
```

are standardized to:

```text
YYYY-MM-DD
```

### 5. Numeric Validation

Checks business rules such as:

```text
unit_price > 0
stock_qty >= 0
quantity > 0
```

### 6. Outlier Detection

Order quantities are checked using the IQR method.

Outliers are flagged rather than automatically deleted because a large order may represent a legitimate bulk purchase.

### 7. Referential Integrity

The pipeline checks relationships between tables.

For example:

```text
orders.customer_id
        ↓
customers.customer_id
```

and:

```text
orders.product_id
        ↓
products.product_id
```

and:

```text
returns.order_id
        ↓
orders.order_id
```

Invalid foreign-key relationships are identified before the cleaned data is loaded into MySQL.

---

# Data Quality Scorecard

The pipeline calculates three main quality metrics.

### Completeness Score

```text
100 - average percentage of missing values
```

### Uniqueness Score

```text
100 - percentage of duplicate records
```

### Referential Integrity Score

```text
100 - percentage of orphan foreign-key values
```

### Overall Quality Score

```text
(Completeness + Uniqueness + Referential Integrity) / 3
```

The formulas are intentionally simple and transparent so that business stakeholders can easily understand the results.

---

# Data Transformation

After validation, the pipeline applies documented cleaning rules.

Examples include:

* Removing duplicate records
* Standardizing city names
* Standardizing product categories
* Standardizing date formats
* Handling missing values
* Removing invalid product prices
* Correcting invalid inventory values
* Removing records with critical referential-integrity violations

Every transformation is logged so that changes to the source data can be audited.

---

# MySQL Database

The cleaned datasets are loaded into a **MySQL relational database** using SQLAlchemy and PyMySQL.

The database contains the following tables:

```text
customers
products
orders
returns
```

Relationships:

```text
customers
    │
    │ customer_id
    ▼
 orders
    │
    ├──────────► products
    │
    └──────────► returns
```

MySQL is used as the central storage layer for the cleaned and transformed data.

This allows SQL queries to be used for downstream reporting and business analysis.

---

# SQL Analysis

After loading the cleaned data into MySQL, SQL queries are used to generate business metrics such as:

* Total revenue
* Monthly revenue
* Revenue by product category
* Top-selling products
* Number of orders
* Customer activity
* Return counts
* Refund amounts
* Return reasons

Example analytical flow:

```text
Raw Data
   ↓
Python Validation
   ↓
Python Transformation
   ↓
MySQL
   ↓
SQL Queries
   ↓
Business Metrics
   ↓
Excel Report
```

---

# Automated Reporting

The reporting stage generates an Excel workbook containing multiple sheets.

## Data Quality Scorecard

Contains:

* Completeness score
* Uniqueness score
* Referential integrity score
* Overall quality score

## Sales Summary

Contains:

* Revenue by category
* Top products
* Monthly revenue trends

## Returns Summary

Contains:

* Return counts
* Refund amounts
* Return reasons

## Run Changelog

Tracks important metrics across multiple pipeline executions.

This allows changes in data quality and business metrics to be monitored over time.

---

# Pipeline Automation

The complete pipeline can be executed through:

```bash
python src/pipeline.py
```

The pipeline performs the following stages:

```text
1. Ingest
2. Validate
3. Transform
4. Load into MySQL
5. Generate reports
6. Record pipeline run
```

A lightweight scheduler is also included for recurring execution.

```bash
python src/scheduler.py --every 1
```

The scheduler can be replaced with production tools such as cron or Apache Airflow in a larger deployment.

---

# Project Structure

```text
automated-data-quality-reporting/
│
├── data/
│   ├── raw/
│   │   ├── customers.csv
│   │   ├── products.csv
│   │   ├── orders.csv
│   │   └── returns.csv
│   │
│   └── processed/
│
├── docs/
│   ├── data_dictionary.md
│   └── validation_rules.md
│
├── logs/
│   └── transform_log.jsonl
│
├── reports/
│   ├── validation_report.md
│   ├── ecommerce_report.xlsx
│   └── changelog.csv
│
├── src/
│   ├── generate_sample_data.py
│   ├── validate.py
│   ├── transform.py
│   ├── load.py
│   ├── report.py
│   ├── pipeline.py
│   └── scheduler.py
│
├── tests/
│
├── .gitignore
├── CHANGELOG.md
├── README.md
└── requirements.txt
```

---

# How to Run the Project

## 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/automated-data-quality-reporting.git

cd automated-data-quality-reporting
```

## 2. Create a virtual environment

### Windows

```bash
python -m venv .venv

.venv\Scripts\activate
```

### macOS / Linux

```bash
python3 -m venv .venv

source .venv/bin/activate
```

## 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

# MySQL Setup

Make sure MySQL Server is installed and running.

Create a database:

```sql
CREATE DATABASE ecommerce_data_quality;
```

Then configure the database connection using environment variables.

Example:

```text
DB_HOST=localhost
DB_PORT=3306
DB_NAME=ecommerce_data_quality
DB_USER=root
DB_PASSWORD=your_password
```

Do **not** commit your real password to GitHub.

Store credentials in a `.env` file and add `.env` to `.gitignore`.

---

# Generate Sample Data

```bash
python src/generate_sample_data.py
```

This generates the raw e-commerce datasets with intentional data-quality issues.

---

# Run the Complete Pipeline

```bash
python src/pipeline.py
```

The pipeline will:

```text
Read raw CSVs
     ↓
Validate data
     ↓
Generate quality score
     ↓
Clean data
     ↓
Load cleaned tables into MySQL
     ↓
Run reporting queries
     ↓
Generate Excel report
     ↓
Update run history
```

---

# Expected Outputs

After a successful pipeline run:

```text
reports/
├── validation_report.md
├── ecommerce_report.xlsx
└── changelog.csv

logs/
└── transform_log.jsonl
```

---

# Documentation

The project includes detailed documentation for data fields and validation rules.

* `docs/data_dictionary.md`
* `docs/validation_rules.md`
* `CHANGELOG.md`

The data dictionary documents each field, its validation rules, and the transformation applied during cleaning.

---

# Key Design Decisions

## Validate before cleaning

Raw data is validated before transformation so that the pipeline can measure the quality of the incoming source data.

## Audit transformations

Cleaning operations are logged instead of silently modifying records.

## Use MySQL as the data storage layer

Cleaned data is loaded into MySQL so that downstream analysis and reporting can be performed using SQL.

## Quality gate

The pipeline can distinguish between:

```text
Pipeline execution succeeded
```

and:

```text
Data quality met the required threshold
```

This makes the pipeline closer to a real data-operations workflow.

---

# Future Improvements

Potential improvements include:

* Add Pandera or Great Expectations
* Add automated email/Slack alerts
* Replace the lightweight scheduler with Apache Airflow
* Add Power BI integration
* Add unit and integration tests
* Add Docker support
* Deploy MySQL and the pipeline to AWS
* Add automated CI/CD using GitHub Actions

---

# Skills Demonstrated

This project demonstrates practical skills in:

* Python
* Pandas
* SQL
* MySQL
* SQLAlchemy
* Data Cleaning
* Data Validation
* Data Quality
* ETL
* Data Transformation
* Referential Integrity
* Data Analysis
* Automated Reporting
* Excel Reporting
* Documentation
* Pipeline Automation
* Git & GitHub

---

## Author

**Shreya Kumari**

GitHub: `https://github.com/YOUR_USERNAME`
