# Implementation Plan - ODIP-S3-Spark (XML Pipeline)

This document outlines the step-by-step implementation plan for the Cloud-Native Tabular Data Deidentification Pipeline using **Apache Spark** and **XML** data sources, featuring **reversible encryption**.

## Phase 1: Environment Setup & Project Initialization

- [ ] **Project Structure Setup**
    - Initialize Git repository (if not already done).
    - Create directory structure: `src/`, `tests/`, `config/`, `data/input`, `data/output`, `dags/`.
    - Create virtual environment and `requirements.txt`.
    - Install core dependencies: `pyspark`, `boto3`, `python-dotenv`, `presidio-analyzer`, `cryptography`, `apache-airflow`.
    - **Airflow & LocalStack Setup:** Configure `docker-compose.yaml` to include **Airflow** and **LocalStack** services.
    - Configure `aws-cli-local` or setup environment variables to point `boto3` to localhost (e.g., `endpoint_url='http://localhost:4566'`).

- [ ] **Spark Configuration**
    - Install Java (JDK 8/11) locally if needed for Spark.
    - Set up local Spark session in a test script.
    - Configure Spark to work with S3 (add `hadoop-aws` and `aws-java-sdk` JARs to Spark config).
    - **Crucial:** Download or configure `spark-xml_2.12-0.15.0.jar` (or compatible version) for local PySpark/Submit execution.
    - Ensure `spark-xml` and Parquet dependencies are resolved (Parquet is built-in).

- [ ] **Security & Key Management**
    - Generate a secure symmetric key (Fernet/AES).
    - Store the key safely in `.env` (not committed to git).
    - Create a configuration module to load the key during runtime.

- [ ] **AWS Configuration & Infrastructure (IaC)**
    - Set up AWS IAM user with S3 read/write permissions.
    - Configure local AWS credentials (use `.env` for secrets).
    - **Terraform:**
        - Create `s3.tf` for buckets (`raw`, `masked`, `restored`, `quarantine`).
        - Create `emr.tf` to provision **AWS EMR Cluster** (Master/Core nodes, Spark installed).
        - Create `iam.tf` for EMR EC2 instance profiles and roles.

## Phase 2: Data Ingestion (S3 & XML)

- [ ] **Synthetic Data Generation**
    - Create a script `generate_xml_data.py` using `Faker` to generate sample XML files with PII.
    - Upload sample XMLs to the `raw` S3 bucket.

- [ ] **S3 Ingestion Module**
    - specific functions to list XML files in S3 bucket using `boto3`.
    - Create a PySpark logic to read XML file directly from S3 (or download locally then read if easier for MVP).
    - *Note:* Ideally use `spark-xml` package (`com.databricks:spark-xml`) for reading XML into DataFrame.

## Phase 3: Core Logic (Encryption & Demasking)

- [ ] **PII Detection Logic (UDFs & Traversal)**
    - Implement Regex-based detection for structured PII (Email, Phone, SSN).
    - Integrate Presidio or spaCy as Spark UDFs for context-based NER (Names).
    - **Recursive Logic:** Create a function to traverse `df.schema` recursively. If a `StructType` is found, drill down; if `StringType` is found, apply detection.

- [ ] **Encryption Transformation (Masking)**
    - Implement `encrypt_val(value, key)` function using `cryptography.fernet`.
    - Wrap it as a Spark UDF.
    - Apply this to identified PII columns.

- [ ] **Decryption Transformation (Demasking)**
    - Implement `decrypt_val(value, key)` function.
    - Wrap it as a Spark UDF.
    - Create a separate pipeline/function to reverse the process given an authorized key.

- [ ] **Output (Parquet)**
    - Write the transformed (encrypted) DataFrame to **Parquet** format.
    - Ensure optimization (partitioning if dataset is large).

## Phase 4: Pipeline Assembly & Integration

- [ ] **Airflow Orchestration (DAGs)**
    - **DAG 1 (Forward):** Define a DAG to trigger the Masking **EMR Step**.
    - **DAG 2 (Reverse):** Define a DAG to trigger the Demasking **EMR Step**.
    - Implement `EmrAddStepOperator` and `EmrStepSensor` to submit and monitor jobs.
    - **Script Upload:** Add task to upload PySpark scripts to `s3://.../scripts/` before execution.
    - **Error Handling:** Implement catch blocks to move malformed data and move to `quarantine` bucket.
    - **Logging:** Implement structured JSON logging.

- [ ] **Restoration Pipeline Script (Reverse)**
    - Combine Ingestion (Masked Parquet S3) -> Decrypt -> Output (CSV/XML) steps.
    - Save result to `restored` S3 bucket.

## Phase 5: Testing & Documentation

- [ ] **Unit Testing**
    - Write tests for encryption/decryption (ensure A -> B -> A works).
    - Write integration tests for S3 upload/download mocks.
- [ ] **Final Review**
    - Update `Readme.md` with usage instructions for both masking and demasking.
    - Verify code quality and comments.
