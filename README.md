# RetailPulse --- E-Commerce Data Warehouse & ETL Pipeline

An end-to-end data engineering project for transforming raw e-commerce
data into a structured analytics warehouse. The pipeline extracts
marketplace data, cleans and transforms it with Python, loads it into
PostgreSQL using a dimensional model, and makes the resulting data
available for SQL analysis and a Streamlit dashboard.

> **Project note:** This repository is a personal fork/adaptation of the
> open-source project credited below. The underlying implementation and
> features are based on the upstream repository. Any additional changes
> made in this version should be documented in the sections below.

## Project Overview

E-commerce platforms generate data across orders, customers, products,
sellers, payments, and deliveries. These datasets can contain missing
values, inconsistent categories, duplicate records, and records that
need to be connected before meaningful analysis is possible.

RetailPulse demonstrates a batch ETL workflow that turns raw marketplace
files into a structured data warehouse for business analysis.

## Architecture

``` text
Raw CSV files
    |
    v
Extract with Python
    |
    v
Staging schema in PostgreSQL
    |
    v
Clean and transform with pandas
    |
    v
Dimensional warehouse (star schema)
    |
    +----> Automated data-quality tests
    |
    +----> SQL business analysis
    |
    v
Streamlit dashboard
```

The Airflow workflow coordinates the pipeline tasks:

``` text
extract_raw
    |
clean_data
    |
load_dim_date
    |
load dimensions (geography, product, seller, customer)
    |
load_fact
    |
run_data_quality_checks
    |
refresh_dashboard_data
```

See `docs/er_diagram.md` for the warehouse entity-relationship diagram.

## Technology Stack

-   **Language:** Python 3.11
-   **Data processing:** pandas, SQLAlchemy
-   **Database:** PostgreSQL 16
-   **Orchestration:** Apache Airflow
-   **Testing:** pytest
-   **Dashboard:** Streamlit, Plotly
-   **Environment:** Docker Compose
-   **CI:** GitHub Actions

## Main Capabilities

-   Extracts source CSV files into a PostgreSQL staging schema.
-   Supports incremental extraction for sources with suitable timestamps
    and full refreshes for other sources.
-   Cleans missing values, inconsistent types, duplicate records, and
    category names.
-   Loads data into a dimensional model containing date, customer,
    product, seller, geography, and order fact tables.
-   Uses conflict-aware upsert patterns to make warehouse loads safe to
    rerun.
-   Runs automated data-quality checks for row counts, referential
    integrity, nulls, duplicates, and customer-history rules.
-   Includes SQL queries for revenue trends, product categories,
    delivery performance, customer cohorts, payment mix, and
    review/delivery analysis.
-   Provides a Streamlit dashboard with KPI cards and revenue and
    delivery visualisations.

## Data Model

The warehouse uses a **star schema**:

-   `dim_date`
-   `dim_customer`
-   `dim_product`
-   `dim_seller`
-   `dim_geography`
-   `fact_orders`

The customer dimension uses Slowly Changing Dimension Type 2 (SCD Type
2) to preserve historical customer-location changes. This allows
historical orders to remain associated with the customer details that
applied at the time of the order.

## Data Quality and Reliability

The pipeline includes checks for:

-   Unexpected row counts
-   Referential-integrity failures
-   Missing required values
-   Duplicate records
-   SCD Type 2 consistency

The transformation logic also accounts for source-data issues. For
example, delivery dates may be absent for orders that were not
delivered, and some product categories may be missing. These cases are
handled explicitly rather than inventing delivery dates or silently
discarding records.

## Dataset

The project uses the **Brazilian E-Commerce Public Dataset by Olist**,
containing anonymised marketplace transactions from 2016--2018.

Dataset: [Olist Brazilian E-Commerce Dataset on
Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

A sample-data generator is also included for an initial run without
downloading the full dataset:

``` bash
python scripts/generate_sample_data.py
```

Place the full dataset CSV files in `data/raw/` when using the original
Olist data.

## Getting Started

### Option 1: Run with Docker

1.  Clone your repository and move into the project directory.
2.  Create the environment file:

``` bash
cp .env.example .env
```

3.  Start PostgreSQL:

``` bash
docker compose up -d postgres
```

4.  Generate the sample dataset, or place the Olist CSV files in
    `data/raw/`:

``` bash
python scripts/generate_sample_data.py
```

5.  Initialise Airflow and start the services:

``` bash
docker compose run --rm airflow-init
docker compose up -d airflow-webserver airflow-scheduler dashboard
```

6.  Open the interfaces:

-   Airflow: `http://localhost:8080`\
    Default local credentials in the supplied configuration: `admin` /
    `admin`
-   Dashboard: `http://localhost:8501`

Trigger the `ecommerce_dw_pipeline` DAG from the Airflow interface.

### Option 2: Run with a local PostgreSQL installation

Install the dependencies using the repository's requirements file and
the Airflow constraints file:

``` bash
pip install -r requirements.txt \
  --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.9.1/constraints-3.11.txt"
```

Create and configure the environment file:

``` bash
cp .env.example .env
```

Create the staging and warehouse schemas:

``` bash
psql "$DATABASE_URL" -f sql/ddl/01_staging_schema.sql
psql "$DATABASE_URL" -f sql/ddl/02_warehouse_schema.sql
```

Generate sample data and run the ETL steps:

``` bash
python scripts/generate_sample_data.py
python -m etl.extract
python -m etl.clean
python -m etl.load
```

Run tests and launch the dashboard:

``` bash
pytest tests/ -v
streamlit run dashboard/app.py
```

## Repository Structure

``` text
.
├── sql/
│   ├── ddl/             # Staging and warehouse schema definitions
│   └── transforms/      # Dimension and fact-table load scripts
├── etl/                 # Extraction, cleaning, and loading modules
├── dags/                # Airflow DAG definition
├── tests/               # Data-quality and unit tests
├── analysis/             # Business-analysis SQL queries
├── dashboard/            # Streamlit dashboard
├── scripts/              # Sample-data generation utilities
└── docs/                 # ER diagram and data dictionary
```

## Screenshots

Add or retain screenshots from your own verified local run:

-   `docs/dashboard_screenshot.png`
-   `docs/airflow_dag_graph.png`

``` markdown
![Dashboard](docs/dashboard_screenshot.png)
![Airflow DAG](docs/airflow_dag_graph.png)
```

## Further Improvements

Potential extensions for this project include:

-   Add a dbt transformation layer.
-   Add Great Expectations for richer data-quality reporting.
-   Add cloud storage and processing with AWS S3 and AWS Glue.
-   Add Amazon Redshift as a cloud data warehouse.
-   Add pipeline monitoring and alerting.
-   Add documentation of pipeline execution time and data-volume
    behaviour.

Only mark an extension as implemented after it has been built and
tested.

## Upstream Attribution

This project is adapted from:

-   **Original repository:**
    [codykhalifa88-sys/ecommerce-data-warehouse](https://github.com/codykhalifa88-sys/ecommerce-data-warehouse)
-   **Original project:** E-Commerce Data Warehouse & ETL Pipeline

The upstream README identifies the project as MIT-licensed. Preserve the
original `LICENSE` file and required copyright/permission notices when
redistributing. Review the licence file included in the downloaded
repository before publishing your fork.

## Author Contributions

Document your own work here before publishing. For example:

-   Set up and ran the project locally.
-   Reviewed the extraction, transformation, and loading workflow.
-   Added or modified specific SQL queries, data-quality rules, or
    pipeline tasks.
-   Tested the changes and documented the results.

Do not list work here until you have completed it.
