# Customer SCD Type 2 Pipeline using Microsoft Fabric

## Project Overview

This project demonstrates the implementation of a Slowly Changing Dimension (SCD Type 2) solution using Microsoft Fabric Data Pipelines, Fabric Data Warehouse, and SQL Stored Procedures.

The solution automatically ingests customer files from Azure Data Lake Storage Gen2 (ADLS Gen2), loads them into a staging table, and maintains full historical customer changes inside a dimension table.

---

# Architecture

## Source
- Azure Data Lake Storage Gen2 (ADLS Gen2)
- CSV Files

## Technologies Used
- Microsoft Fabric Pipelines
- Fabric Data Warehouse
- SQL Stored Procedures
- Azure Data Lake Storage Gen2
- Get Metadata Activity
- ForEach Activity
- Copy Activity
- Script Activity
- SCD Type 2

---

# Pipeline Workflow

## Step 1 – Clear Staging Table

A Script Activity truncates the staging table before each load:

```sql
TRUNCATE TABLE stg_customer;
```

## Step 2 – Discover Source Files

A Get Metadata Activity retrieves all files from the source ADLS folder.

## Step 3 – Process Files

A ForEach Activity loops through every customer file found in the source folder.

## Step 4 – Load Data into Staging

Each CSV file is loaded into the staging table:

```sql
stg_customer
```

## Step 5 – Execute SCD Type 2 Procedure

After all files are loaded, the pipeline executes:

```sql
sp_upsert_customer_scd2
```

The procedure updates historical records and inserts new customer versions when changes are detected.

---

# Data Warehouse Tables

## Staging Table

Stores the latest incoming customer data.

```sql
CREATE TABLE stg_customer (
    customer_id INT,
    name VARCHAR(100),
    address VARCHAR(200),
    phone VARCHAR(20),
    last_updated DATETIME2(0)
);
```

## Dimension Table

Stores complete customer history.

```sql
CREATE TABLE dim_customer (
    customer_id INT,
    name VARCHAR(100),
    address VARCHAR(200),
    phone VARCHAR(20),
    start_date DATETIME2(0),
    end_date DATETIME2(0) NULL,
    is_current INT
);
```

---

# SCD Type 2 Logic

The stored procedure performs three main operations:

## 1. Deduplicate Records

Uses ROW_NUMBER() to select the latest record for each customer.

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY last_updated DESC
)
```

## 2. Close Existing Versions

If customer information changes:

- End Date is populated
- Current Flag becomes 0

Example:

| CustomerID | Address | Start Date | End Date | Is Current |
|------------|----------|------------|----------|------------|
| 1001 | Cairo | 2025-07-01 | 2025-07-15 | 0 |

## 3. Insert New Version

A new active record is inserted:

| CustomerID | Address | Start Date | End Date | Is Current |
|------------|----------|------------|----------|------------|
| 1001 | Giza | 2025-07-15 | NULL | 1 |

---

# Example

## Initial Customer

| CustomerID | Name | Address |
|------------|------|----------|
| 1001 | Ahmed | Cairo |

## New Incoming Record

| CustomerID | Name | Address |
|------------|------|----------|
| 1001 | Ahmed | Giza |

## Result

Historical Version:

| CustomerID | Address | End Date | Is Current |
|------------|----------|----------|------------|
| 1001 | Cairo | 2025-07-15 | 0 |

Current Version:

| CustomerID | Address | End Date | Is Current |
|------------|----------|----------|------------|
| 1001 | Giza | NULL | 1 |

---

# Key Features

- Automated Customer History Tracking
- SCD Type 2 Implementation
- Metadata-Driven File Processing
- Incremental Customer Updates
- Historical Data Preservation
- Fabric Pipeline Orchestration
- Fabric Data Warehouse Integration
- SQL Stored Procedure Automation

---

# Business Value

This solution enables organizations to:

- Track customer profile changes over time
- Maintain historical records for auditing
- Support regulatory and compliance requirements
- Build accurate customer analytics
- Enable time-based reporting and customer history analysis

---


# Project Outcome

Built a complete end-to-end SCD Type 2 solution in Microsoft Fabric that automatically ingests customer files, tracks customer changes, preserves historical versions, and maintains a fully auditable customer dimension.
