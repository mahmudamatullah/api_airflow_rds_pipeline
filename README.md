# Books Data Pipeline (OpenLibrary API → Airflow → AWS RDS)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=flat&logo=amazon-aws&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-%235835CC.svg?style=flat&logo=terraform&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10.4-blue.svg?style=flat&logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-%230db7ed.svg?style=flat&logo=docker&logoColor=white)
![Airflow](https://img.shields.io/badge/Apache%20Airflow-%23017CEE.svg?style=flat&logo=apache-airflow&logoColor=white)

## Table of Contents

- [Project Overview](#project-overview)
- [Infrastructure / Architecture](#infrastructure--architecture)
- [Data Generation](#data-generation)
- [Data Pipeline Flow](#data-pipeline-flow)
- [Tech Stack](#tech-stack)
- [How to Run](#how-to-run)
- [Key Features](#key-features)
- [Future Improvements](#future-improvements)
- [What I Learnt](#what-i-learnt)
- [Next Projects](#next-projects)

---

## Overview

Ever wondered which book topics are trending each year? Or how authors and subjects shift over time?

This project is an end-to-end automated data pipeline that fetches book metadata from the OpenLibrary API, processes it, and stores it in a PostgreSQL database on AWS RDS, ready for analytics or dashboards.

The pipeline is fully orchestrated with Apache Airflow and deployed locally using Docker, while all cloud infrastructure (VPC, subnets, RDS) is provisioned via Terraform.

--- 

## Infrastructure / Architecture
- AWS resources provisioned via Terraform:
- Custom VPC with multiple subnets and route tables
- Internet Gateway and Security Groups
- PostgreSQL RDS instance (publicly accessible for testing)
- IAM roles and policies
- Local Airflow deployment using Docker:
  - DAG schedules data extraction and ingestion into RDS
  - Uses Airflow Variables for RDS credentials

## Data Generation
Book data is fetched dynamically from the OpenLibrary API.
- Each API call returns:
  - Book title
  - Subjects (topics)
  - Authors
  - Random sampling of subjects and books to simulate variability
  - Fully dynamic and repeatable for daily ingestion


## Data Pipeline Flow
1. Terraform provisions AWS infrastructure (VPC, subnets, RDS, security groups).
2. Airflow DAG – Fetch & Upload Books Data:
  - The DAG runs according to a schedule (daily).
  - Inside the DAG, the Python function upload_books_to_rds() is called.
  - upload_books_to_rds() internally calls fetch_books_data() from the Python module to fetch books from the OpenLibrary API.
3. The fetched data is processed and uploaded to the RDS Postgres instance.
4. Data is made available in RDS for analytics or downstream processing.

> Key point: The python script is not triggered manually. Airflow orchestrates the execution, making the workflow automated and scheduled.

## Tech Stack
- Python (requests, random, psycopg2)
- Apache Airflow, Airflow variables (storing sensitive info)
- AWS: VPC, Subnets, Security Groups, RDS (PostgreSQL)
- Terraform
- Docker
- OpenLibrary API

## How to Run

### 1. Prerequisites
Before running the pipeline, you need to have:

- An AWS account with access
- Terraform installed locally
- Docker installed
- Apache Airflow installed via Docker, or the script can be copied if it's on the cloud.
- Optional: Python (for local scripts/tests)

---

### 2. Provision AWS Infrastructure
``bash
terraform init
terraform apply``

> Terraform will create the necessary resources mentioned in the Architecture/ Infrastructure
> Note: Replace any placeholder variables in Terraform with your own AWS credentials and preferred naming.
> Ensure variables for RDS credentials (db_name, db_username, db_password) are set.

### 3. Start Airflow (Docker)
`docker-compose up` - This will launch the Airflow webserver and scheduler locally.

Access the Airflow UI at:
`http://localhost:8080`

4. Configure Airflow Variables

| Variable     | Description                   |
| ------------ | ----------------------------- |
| rds_endpoint | Endpoint of your RDS instance |
| rds_port     | RDS port (default 5432)       |
| db_name      | PostgreSQL database name      |
| db_username  | Database username             |
| db_password  | Database password             |

Use Airflow UI → Admin → Variables to add these.
You can use placeholder values to test or your own credentials.

4. Start Airflow (Docker)
`docker-compose up`
- Access the UI at http://localhost:8080
- Locate books_to_rds DAG and trigger it. Airflow will automatically execute the Python task, fetch data from the API, and load it into RDS.

5. Verify Data
Connect to RDS and query the books table to confirm records are inserted.

7. Notes
This project is intended to be run with your own AWS account.

## Key Features
- Fully automated infrastructure using Terraform
- Secure AWS network setup
- API-driven dynamic data ingestion
- Airflow DAG for scheduling and orchestration
- PostgreSQL database creation and insertion handled automatically
- Randomized sampling of book data for variety

## Future Improvements
- Deploy Airflow on AWS
- Extend pipeline to multiple external APIs or datasets
- Automated testing for Terraform and ETL scripts

## What I Learnt
- Building an end-to-end ETL pipeline with Airflow
- Fetching and transforming API data dynamically
- Provisioning AWS resources with Terraform
- Managing RDS PostgreSQL databases
- Integrating local Docker deployments with cloud infrastructure
