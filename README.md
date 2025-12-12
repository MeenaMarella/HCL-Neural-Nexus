## 🏦 Insurance Policy Data Engineering Pipeline

A complete end-to-end ETL + Data Warehouse solution built for multi-region, multi-day insurance policy data.
This project demonstrates real-world data engineering best practices including:

Daily batch ingestion

Standardized cleaning

Consolidation into a master dataset

Star Schema Data Warehouse

Business rule processing (late fees)

Automated analytical reporting

## 📂 Project Architecture

Raw data arrives from 4 U.S. regions (East, West, South, Central) for day0, day1, and day2.

Instead of merging everything at once, this pipeline follows day-wise ingestion, producing clean daily datasets first and then merging them globally.

This simulates how enterprise insurers load data into warehouses incrementally.

## 1️⃣ ingest.py — Raw Data Ingestion (Day-wise Merge Across Regions)

Each day has 4 regional CSVs:

Example (Day0):

US_East/day0.csv
US_West/day0.csv
US_South/day0.csv
US_Central/day0.csv

✔ What ingest.py does:
Step A — Combine all regions for each day

For each day (day0, day1, day2), the script:

Reads all region files

Extracts region name from folder

Adds source_file column

Performs light cleaning (trimming whitespace)

Merges all regions into a single dataset for that day

Outputs:

staging/cleaned_day0.csv
staging/cleaned_day1.csv
staging/cleaned_day2.csv

Step B — Merge all cleaned-day files
staging/raw_combined.csv


This becomes the single source of truth for the cleaning phase.

Command
python src/ingest.py

## 2️⃣ cleaning.py — Deep Cleaning & Standardization (DD-MM-YYYY Format)

This script performs full cleaning on the combined dataset (raw_combined.csv) to create a fully standardized dataset for DW loading.

Below are the detailed cleaning operations.

✔ Standardizing Column Names

Convert to lowercase

Replace spaces with underscores

Trim whitespace

Fix duplicate column names (region, region_1, etc.)

Ensures all files share a consistent schema.

✔ Normalizing Gender

Raw inputs:

m, M, male, MALE → Male
f, F, female → Female


Final values:

Male
Female

✔ Normalizing Marital Status

Examples:

Raw	Cleaned
single	Single
SINGLE	Single
married	Married
divorced	Divorced

Uses .str.title() to ensure consistency.

✔ Standardizing Country Name → ALWAYS “USA”

Variations such as:

United States, United States of America, US, U.S.A, America


All become:

USA

✔ Combining Customer Name Components

Constructs a clean full name:

Customer Name = First + Middle + Last


Handles missing middle names & fallback full name columns.

Examples:

John A Smith
Mary Johnson

✔ Cleaning Region Fields

Many files contain both Region and region.

Script merges all variants and standardizes:

east → East  
WEST → West  
south → South  
central → Central

✔ Converting All Date Columns → DD-MM-YYYY

All date fields are parsed with error handling and reformatted:

05-01-2012
31-12-2014
10-08-1990


Invalid or missing dates are set to:

NULL

✔ Cleaning Numeric Columns

Fixes formats like:

"10,000", "$800", " 4500 "


into:

10000.00
800.00
4500.00

✔ Removing Duplicate Rows

Ensures no repeated records after merging day0/day1/day2 data.

✔ Handling Empty or Invalid Values

Empty values like:

'', '   ', 'nan', 'None'


are replaced with:

NULL


Allowing clean loading into MySQL.

Output
staging/cleaned.csv
reports/cleaning_summary.json

Command
python src/cleaning.py

## 3️⃣ dw_loader.py — Build & Populate the Data Warehouse

This script creates a Star Schema and loads all data into MySQL.
![WhatsApp Image 2025-12-12 at 18 14 04](https://github.com/user-attachments/assets/b3e5398c-bb51-447e-955e-731be091b0e8)


✔ Dimension Tables

dim_customer

dim_policy

dim_location

dim_date

✔ Fact Table

fact_premium_payments

✔ What happens:

Unique values inserted into dimensions

Surrogate keys generated

Fact table populated with:

Customer Key

Policy Key

Location Key

Payment Date Key

Premium Amounts

Late Days

Command
python src/dw_loader.py

## 4️⃣ load_late_fee_rules.py — Load Excel Late-Fee Rules

Imports:

UseCase - Late_Fees_Calculation_Formula.xlsx


into the DW:

late_fee_rules


Rules include:

Flat fees

Percentage fees

Region-based rules

Month-based rules

Late-day slabs

Command
python src/load_late_fee_rules.py

## 5️⃣ compute_late_fees.py — Late Fee Calculation Engine

This script computes late fees by joining:

fact_premium_payments
late_fee_rules

✔ Computes:

late_days

flat fee

percentage fee

final late_fee_amount

Updates fact table for downstream reporting.

Command
python src/compute_late_fees.py

## 6️⃣ queries.sql — Generate Reports (a → g)

Produces business insights such as:

Customers who changed marital status

Policy changes over time

Auto policy customers

Total policy amounts by region

East & West quarterly customers (year 2012)

Full customer-policy-location mapping

Output CSVs:
reports/a.csv
reports/b.csv
...
reports/g.csv

Command
bash run_queries.sh

##🏁 Final Deliverables

This project delivers:

✔ Cleaned day-wise datasets

✔ Master consolidated dataset

✔ Fully cleaned standardized dataset

✔ Star Schema Data Warehouse in MySQL

✔ Late-fee business rule engine

✔ Fact table with computed late fees

✔ Automated analytical reporting

✔ Hackathon-ready ETL pipeline# HCL-Neural-Nexus
