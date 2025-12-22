# Project Tasks - ODIP-S3-Spark

## Backlog

### Phase 1: Setup
- [ ] Initialize Python project and create virtual environment <!-- id: 0 -->
- [ ] Install dependencies: `pyspark`, `boto3`, `faker`, `presidio-analyzer`, `cryptography` <!-- id: 1 -->
- [ ] Set up **Docker Compose** with **Airflow** and **LocalStack** <!-- id: 22 -->
- [ ] Verify local Spark installation and "Hello World" Spark session <!-- id: 2 -->
- [ ] Configure `boto3` for S3 access (check `.env` setup) to support **LocalStack** endpoint <!-- id: 3 -->
- [ ] Initialize Terraform `main.tf` for S3 Buckets and **EMR Cluster** (`emr.tf`) <!-- id: 20 -->
- [ ] Generate and store Symmetric Encryption Key (Fernet) <!-- id: 16 -->

### Phase 2: Data & Ingestion
- [ ] Create `generate_xml.py` to create synthetic XML data with PII <!-- id: 4 -->
- [ ] Implement `s3_client.py` to list and manage S3 objects <!-- id: 5 -->
- [ ] Implement Spark XML reader (verify `spark-xml` package dependency) <!-- id: 6 -->

### Phase 3: Processing (Spark - Encryption/Decryption)
- [ ] Define PII detection patterns (Regex) <!-- id: 7 -->
- [ ] Create Spark UDF for Presidio/NLP detection <!-- id: 8 -->
- [ ] Implement **Recursive Schema Traversal** to handle nested XML fields <!-- id: 19 -->
- [ ] Implement `encrypt_data(df)` transformation logic using Key <!-- id: 9 -->
- [ ] Implement `decrypt_data(df)` restoration logic using Key <!-- id: 17 -->
- [ ] Verify DataFrame schema preservation after transformations <!-- id: 10 -->

### Phase 4: Output & Integration
- [ ] Implement Spark **Parquet** writer logic <!-- id: 11 -->
- [ ] Create `masking_dag.py` in Airflow using `EmrAddStepOperator` <!-- id: 12 -->
- [ ] Create `restoration_dag.py` in Airflow using `EmrAddStepOperator` <!-- id: 18 -->
- [ ] Add Structured JSON Logging to pipeline <!-- id: 13 -->
- [ ] Implement DLQ Logic (Quarantine Bucket move) for failed files <!-- id: 21 -->

### Phase 5: Additional
- [ ] Write unit tests for encryption/decryption roundtrip <!-- id: 14 -->
- [ ] Create Dockerfile for the application <!-- id: 15 -->
