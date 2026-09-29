# PySpark + Airflow ETL Pipeline with PostgreSQL

This project is a simple portfolio ETL pipeline that reads customer transaction data from a CSV file, transforms it with PySpark, and loads aggregated results into PostgreSQL. Apache Airflow orchestrates the workflow and runs the processing job in a containerized environment.

## Problem Statement

Many data engineering projects start with raw files that need cleaning, aggregation, and loading into a database for downstream analysis. This project demonstrates a practical ETL pattern using production-adjacent tools while keeping the design understandable enough to explain in an interview.

## Project Objective

The goal is to process customer spend data end-to-end and make the output easy to query in PostgreSQL after transformation. The flow is intentionally simple:

CSV input -> PySpark transformation -> PostgreSQL tables -> Airflow orchestration

## Architecture

The stack is composed of:

- Apache Airflow for orchestration and scheduling
- PySpark for transformation logic
- PostgreSQL for the warehouse tables
- Docker Compose for local orchestration of services

The processed data is stored in a separate PostgreSQL database named `customer_analytics` while the Airflow metadata database remains in the default `airflow` database.

## ETL Workflow

1. Read raw customer records from `data/customers.csv`.
2. Validate the CSV schema and convert date strings to dates.
3. Aggregate spending by customer and by product category.
4. Insert the results into PostgreSQL tables.
5. Confirm the data quality using Airflow task verification.

## Technologies Used

- Python
- PySpark
- Apache Airflow
- PostgreSQL
- Docker
- Docker Compose
- SQL

## Project Structure

```text
pyspark-airflow-etl-pipeline-with-postgresql-main/
├── airflow/
│   ├── dags/
│   │   └── customer_pipeline.py
│   ├── Dockerfile
│   └── requirements.txt
├── data/
│   └── customers.csv
├── postgres/
│   └── init.sql
├── spark/
│   ├── Dockerfile
│   └── scripts/
│       └── process_data.py
├── .gitignore
├── docker-compose.yml
├── README.md
└── images/
    ├── airflow.png
    └── postgres_tables.png
```

## How the Pipeline Works

### Input

The CSV contains customer-level transaction records with:

- `customer_id`
- `age`
- `gender`
- `marital_status`
- `income_level`
- `transaction_date`
- `amount_spent`
- `product_category`

### Transformation

The PySpark job:

- reads the CSV into a Spark DataFrame with a defined schema
- converts `transaction_date` to a proper date type
- aggregates total spend, transaction count, and average spend by customer
- aggregates revenue and category-level metrics by product category

### Load

The script writes records into PostgreSQL tables:

- `customer_summary`
- `product_category_stats`

These tables are created by the SQL initialization file in `postgres/init.sql`.

## Docker Setup

The project is designed to run with Docker Compose from the repository root.

```bash
docker compose up -d --build
```

This starts:

- PostgreSQL on port `5433`
- Airflow webserver on port `8080`
- Airflow scheduler

## Prerequisites

Before starting the stack, make sure the following are installed on your machine:

- Docker Desktop or Docker Engine
- Docker Compose
- Git
- Python 3 (for local validation outside Docker, optional)

## Installation and Setup

1. Clone the repository.
2. Change into the project directory.
3. Build the containers.

```bash
git clone <your-repository-url>
cd pyspark-airflow-etl-pipeline-with-postgresql-main

docker compose up -d --build
```

## How to Start the Pipeline

Once the containers are healthy:

1. Open the Airflow UI in a browser: http://localhost:8080
2. Log in with:
   - Username: `admin`
   - Password: `admin`
3. Activate the DAG named `customer_data_pipeline`.
4. Trigger the DAG manually.

The workflow runs in this order:

- `start_pipeline`
- `process_data_with_spark`
- `verify_postgres_data`
- `end_pipeline`

## How to Access Airflow

Open:

```text
http://localhost:8080
```

Use the default Airflow credentials configured in the init service:

- Username: `admin`
- Password: `admin`

## How to Verify PostgreSQL Results

After the DAG finishes, connect to the PostgreSQL database:

```bash
docker compose exec postgres psql -U airflow -d customer_analytics
```

Then run:

```sql
SELECT * FROM customer_summary ORDER BY total_spent DESC LIMIT 10;
SELECT * FROM product_category_stats ORDER BY total_revenue DESC;
```

## Example SQL Queries

```sql
-- View totals by customer
SELECT customer_id, total_spent, transaction_count, avg_transaction
FROM customer_summary
ORDER BY total_spent DESC;

-- View category revenue
SELECT product_category, total_revenue, transaction_count, avg_transaction_amount
FROM product_category_stats
ORDER BY total_revenue DESC;

-- Count records
SELECT COUNT(*) AS customer_rows FROM customer_summary;
SELECT COUNT(*) AS category_rows FROM product_category_stats;
```

## Configuration and Environment Variables

The default Docker Compose setup uses these values:

- PostgreSQL user: `airflow`
- PostgreSQL password: `airflow`
- PostgreSQL database: `airflow` for metadata and `customer_analytics` for ETL output
- Airflow database connection: `postgresql+psycopg2://airflow:airflow@postgres:5432/airflow`
- Airflow web UI: http://localhost:8080

## Troubleshooting

### Docker is not installed

This environment currently does not have Docker available, so the full containerized validation could not be run here. To validate the complete pipeline locally, install Docker Desktop or the Docker engine and Docker Compose first.

### Airflow DAG is missing

Check the container logs:

```bash
docker compose logs -f airflow-webserver
docker compose logs -f airflow-scheduler
```

### Database is not ready

Verify Postgres health:

```bash
docker compose ps
docker compose logs -f postgres
```

### PySpark job fails

Inspect the Airflow task logs and confirm the input file exists at:

```text
/opt/airflow/data/customers.csv
```

## Testing and Verification

This project was reviewed by checking:

- repository structure
- Docker Compose configuration
- Airflow DAG imports and orchestration logic
- PySpark transformation script
- PostgreSQL schema and database setup
- owner metadata and documentation cleanup

The full end-to-end Docker validation is pending installation of Docker on the local machine, because the current environment does not include Docker Compose.

## Future Improvements

Potential next steps include:

- adding unit tests for transformation logic
- parameterizing environment values in a `.env` template
- splitting the ETL job into reusable modules
- adding data quality checks and alerting
- extending the dashboard/reporting layer

## Author

Priyanshu Kumar Yadav

This version is a customized portfolio implementation that preserves the original ETL concept while updating documentation, ownership attribution, and maintainability for personal use.

## Key Features

- **Complete ETL Pipeline** - End-to-end data processing
- **Containerized** - Easy deployment and isolation
- **Orchestrated** - Automated workflow management
- **Scalable** - Spark-based distributed processing
- **Monitoring** - Web UIs for all components
- **Data Persistence** - PostgreSQL with volume storage
- **Error Handling** - Robust task failure management

## Use Cases

This project demonstrates patterns for:

- Customer analytics pipelines
- ETL workflow orchestration
- Docker-based data engineering
- PySpark data processing
- Airflow DAG development
- PostgreSQL data storage

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes and test with `docker compose up -d --build`
4. Submit a pull request
