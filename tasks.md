# Project Tasks - ODIP-S3-Spark

## Backlog

### Phase 1: Setup
- [ ] Initialize Python project and create virtual environment <!-- id: 0 -->
- [ ] Install dependencies: `pyspark`, `boto3`, `faker`, `presidio-analyzer`, `cryptography` <!-- id: 1 -->
- [ ] Verify local Spark installation and "Hello World" Spark session <!-- id: 2 -->
- [ ] Configure `boto3` for S3 access (check `.env` setup) <!-- id: 3 -->
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
- [ ] Create `pipeline_fwd.py` (Masking) to orchestrate List -> Process -> Upload <!-- id: 12 -->
- [ ] Create `pipeline_rev.py` (Demasking) to orchestrate Masked -> Process -> Restored <!-- id: 18 -->
- [ ] Add logging to pipeline <!-- id: 13 -->

### Phase 5: Additional
- [ ] Write unit tests for encryption/decryption roundtrip <!-- id: 14 -->
- [ ] Create Dockerfile for the application <!-- id: 15 -->
