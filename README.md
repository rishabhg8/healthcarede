# GCP Healthcare Data Engineering Project

End-to-end healthcare data engineering pipeline on Google Cloud Platform (GCP) for Revenue Cycle Management (RCM).

## Project Goal
Build a simple medallion-style data platform to ingest healthcare data from multiple sources, orchestrate workflows with Airflow, and deliver analytics-ready outputs in BigQuery.

## Architecture
![Architecture](images/architecture.png)

## Data Sources
- Cloud SQL (MySQL): EMR data from Hospital A and Hospital B
  - patients, providers, departments, encounters, transactions
- Claims flat files (CSV)
- CPT codes (CSV)
- External reference APIs (NPI, ICD)

## GCP Services Used
- Cloud Storage (landing and staging)
- Cloud SQL (MySQL)
- Dataproc (PySpark processing)
- BigQuery (bronze, silver, gold layers)
- Cloud Composer / Airflow (orchestration)
- Cloud Build (deployment support)
- IAM (service account and role permissions for Spark worker access)

## Project Flow
1. Set up a GCP project and enable required APIs (Dataproc, Composer, Cloud Build, Cloud SQL, BigQuery, and GCS).
2. Create a Cloud Storage bucket with ingestion folders and upload config CSV files.
3. Use config and watermark columns to control full vs incremental loads and capture delta changes.
4. Create two Cloud SQL hospital databases (Hospital A and Hospital B), apply DDL, and load source EMR data.
5. Create a Dataproc Spark cluster and validate ingestion jobs.
6. Store pipeline logs in GCS and BigQuery audit tables for monitoring and operational visibility.
7. Load raw data into BigQuery bronze, standardize both hospitals into a common model in silver, and apply SCD Type 2 on history-tracked tables to preserve changes over time.
8. Build gold-layer analytical tables so data is business-ready for dashboards and downstream users.
9. Enable CI/CD by connecting GitHub to a Cloud Build trigger so each push deploys the latest DAG code and starts the Composer parent DAG, which runs PySpark DAG (Extract) first and BigQuery DAG (Transform + Load) next.

## Key Outputs

### Landing Bucket Snapshot
![Landing Bucket](images/final-bucket.png)

### Airflow Final Run Status
![Airflow Final](images/airflow-final.png)

### BigQuery Gold Output
![BigQuery Gold Output](images/bq-gold-output.png)

## Dataproc Cluster Setup (CLI)
```bash
CLUSTER_NAME="my-demo-cluster"
REGION="us-east1"

gcloud dataproc clusters create "${CLUSTER_NAME}" \
  --region="${REGION}" \
  --num-workers=2 \
  --worker-machine-type=n1-standard-2 \
  --worker-boot-disk-size=50 \
  --master-machine-type=n1-standard-2 \
  --master-boot-disk-size=50 \
  --image-version=2.0-debian10 \
  --enable-component-gateway \
  --optional-components=JUPYTER \
  --initialization-actions="gs://goog-dataproc-initialization-actions-${REGION}/connectors/connectors.sh" \
  --metadata="bigquery-connector-version=1.2.0,spark-bigquery-connector-version=0.21.0"
```

## Challenges Faced
- We stored ingestion audit metadata in BigQuery to support incremental loading and run traceability.
- Dataproc could not connect to Cloud SQL at first. We fixed it using VPC and private IP.
- For patients and transactions, old files are archived first, then new data is loaded (incremental).
- Composer setup failed first because some APIs were disabled. We enabled them and it worked.
- Airflow DAGs did not show up at first because of wrong Composer bucket details.
- Bronze step failed because Hospital A audit table was missing. We recreated required landing/data setup and ran again.

## Repository Structure
```text
project/
  data/
    BQ/
    claims/
    cptcodes/
    EMR/
    INGESTION/
  workflows/
  utils/
  cloudbuild.yaml
```

## Author
Rishabh
