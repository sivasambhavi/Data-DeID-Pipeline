# 🚀 Product Requirements Document (PRD)

**Project Name:** Cloud-Native Tabular Data Deidentification Pipeline (ODIP-S3-Spark)
**Version:** 2.2 (XML Input, Parquet Output, Reversible Masking)
**Date:** December 21, 2025
**Type:** Personal Portfolio Project

---

## 1. Introduction

### Objective
To design and implement a scalable, automated pipeline that ingests XML datasets from an AWS S3 bucket, accurately identifies Personally Identifiable Information (PII) within the data, and generates masked versions of the datasets in **Parquet format** back to a secure S3 location. The system must support **reversible encryption** (demasking) to restore original values for authorized users.

### Personal Focus
This project demonstrates proficiency in **Cloud Computing (AWS)**, **Big Data Processing (Spark)**, **Data Engineering**, **Cryptography/Security**, **Natural Language Processing**, and **Infrastructure as Code (IaC)**.

---

## 2. Stakeholders

| Role | Responsibility |
|------|---------------|
| **Lead Developer (You)** | Architect the solution, implement S3 integration, tune PII detection logic, and deploy the system. |
| **Recruiters / Reviewers** | Evaluate cloud engineering skills, code security practices (encryption), big data capabilities, and data handling maturity. |

---

## 3. Goals & Success Metrics

| Goal Area | Success Metric |
|---------|----------------|
| **Cloud Integration** | Successful read/write operations with AWS S3 using `boto3` and Spark S3 connectors. |
| **Scalability** | Capable of processing large XML files efficiently using Apache Spark. |
| **Accuracy** | Correctly identify and mask PII (Emails, Phones, Names, SSNs) with **> 90%** precision. |
| **Reversibility** | Masked PII must be recoverable (demasked) to its original state using the correct decryption key. |
| **Optimization** | Output data is stored in **Parquet** (columnar format) for optimized analytics and reduced storage cost. |
| **Security** | Encryption keys are managed securely; sensitive data is never exposed in plain text during storage. |

---

## 4. User Stories

| User Story | Priority |
|-----------|----------|
| As the **Data Engineer**, I want the system to automatically pick up new XML files from an "Input" S3 bucket. | Must Have |
| As the **Compliance Officer**, I want specific PII types to be **encrypted** so unauthorized users see only ciphertext. | Must Have |
| As the **Authorized Auditor**, I want to be able to **demask** (decrypt) a processed file to verify specific records using a secure key. | Must Have |
| As the **Analyst**, I want the output in **Parquet** format to query it efficiently using tools like Athena or Spark SQL. | Must Have |

---

## 5. Constraints & Design Decisions

*   **Input Format:** `.xml` files.
    *   *Assumption:* XML is row-based (e.g., `<root><row><name>...</name></row>...</root>`).
    *   *Complexity:* Supports nested tags (e.g., `<address><city>...</city></address>`) and arrays.
*   **Output Format:** **Apache Parquet** (Columnar Storage).
*   **Processing Engine:** **Apache Spark (PySpark)** for scalability.
*   **Masking Strategy:** **Symmetric Encryption (AES)**.
*   **Key Management:** Keys will be stored in a `.env` file (local) or AWS Secrets Manager (simulated/future).
*   **Cost Control:** Uses standard S3 and local compute.

---

## 6. Technical Requirements

### 6.1 Core Pipeline Flow (Forward: Masking)

1.  **Ingestion:** Connect to AWS S3 bucket and list available `.xml` objects.
2.  **Load:** Read XML file into a Spark DataFrame.
3.  **PII Detection:**
    *   **Recursive Traversal:** The system will recursively traverse the DataFrame schema to identify PII even within nested `StructType` (nested tags) and `ArrayType` (repeated tags) fields.
    *   **Heuristic/Regex:** Detect patterns (Email, Phone, SSN) within leaf nodes.
    *   **Heuristic/Regex:** Detect patterns within leaf nodes.
    *   **NLP/NER:** Use `Microsoft Presidio` wrapped in Spark UDFs.
    *   **Target PII Entities:**
        *   `PERSON` (Name) -> Presidio
        *   `EMAIL_ADDRESS` -> Regex/Presidio
        *   `PHONE_NUMBER` -> Regex/Presidio
        *   `SSN` -> Regex
        *   `CREDIT_CARD` -> Regex/Presidio
4.  **Encryption:** Apply AES encryption to the identified PII values. The Output will be a base64 encoded ciphertext.
5.  **Output:** Write the masked (encrypted) DataFrame to **S3 in Parquet format**.
6.  **Upload:** Save to `s3://<project-name>-masked/`.

### 6.2 Core Pipeline Flow (Reverse: Demasking)

1.  **Input:** Read a masked **Parquet** file from S3.
2.  **Decryption:** Apply the reverse AES decryption transformation using the secret key.
3.  **Validation:** Ensure the decrypted text matches expected patterns (e.g., valid email format).
4.  **Output:** Restore original data (optionally write back as XML or CSV for viewing).

---

### 6.3 Technology Stack

| Layer | Tools |
|------|------|
| **Cloud Provider** | AWS (S3) |
| **Big Data Processing** | **Apache Spark (PySpark)** |
| **Input Processing** | `spark-xml` (Maven: `com.databricks:spark-xml`) |
| **Output Processing** | Built-in Spark Parquet writer |
| **Security/Crypto** | `Python Cryptography` library (Fernet/AES) inside Spark UDFs |
| **PII Detection** | `Microsoft Presidio` |
| **Configuration** | `python-dotenv` |
| **Orchestration** | **Apache Airflow** (Docker/MWAA) |
| **Infrastructure** | Terraform / AWS CLI |
| **Local Emulation** | **LocalStack** (S3 mocking) |
| **Containerization** | Docker |

---

### 6.4 Data Flow Specification

*   **Input Bucket:** `s3://<project-name>-raw/`
*   **Output Bucket (Masked):** `s3://<project-name>-masked/` (Parquet)
*   **Output Bucket (Demasked):** `s3://<project-name>-restored/`
*   **Quarantine Bucket (DLQ):** `s3://<project-name>-quarantine/` (For failed/malformed files)

---

## 7. Non-Functional Requirements

| Category | Requirement |
|--------|-------------|
| **Scalability** | Distributed encryption via Spark. |
| **Security** | **Key Management is paramount.** If the key is lost, data is lost. |
| **Reliability** | Retries on network errors; Dead Letter Queue isolation for bad data. |
| **Observability** | Structured JSON logging compatible with CloudWatch. |

---

## 8. Failure Management

*   **Decryption Failure:** If a field cannot be decrypted (wrong key or corrupted data), flag it as `[ERROR]` or keep ciphertext.
*   **Malformed data:** Log error using **Structured Logging** and move file to **Quarantine Bucket (DLQ)**.
*   **Schema Mismatch:** If incoming XML schema changes significantly (new nested structures), the pipeline should fail gracefully or alert.

---

## 9. Out of Scope

*   OCR for PDF/Images.
*   Asymmetric Encryption (Public/Private key infrastructure) - sticking to Symmetric for MVP simplicity.
*   **Schema Evolution:** Automatic handling of changing XML schemas is out of scope for MVP.
*   GUI.

---

## 10. Testing & Validation Strategy

### Scenarios
1.  **Round Trip:** Original (XML) -> Encrypt -> Parquet -> Decrypt -> Verify Match.
2.  **Wrong Key:** Attempting to decrypt with an invalid key should fail/raise error.
3.  **No PII:** Data un-modified.
