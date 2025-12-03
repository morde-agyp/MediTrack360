# MouseOverMe Data Engineering Pipeline

## Overview

MouseOverMe is an end-to-end data engineering pipeline that extracts data from multiple sources, processes it through orchestrated workflows, and loads it into a data warehouse for analytics. The entire infrastructure is containerized with Docker and leverages AWS services for scalable data storage and processing.

## Architecture

```
Data Sources → Airflow (Orchestration) → S3 (Data Lake) → Redshift (Data Warehouse)
    ↓                    ↓                     ↓                    ↓
PostgreSQL          Docker Container      AWS Storage         AWS Analytics
CSV Files           Kubernetes Pods       Staging Area        Query Engine
REST APIs           DAG Scheduling        Object Storage      Dimensional Model
```

## Tech Stack

- **Orchestration**: Apache Airflow
- **Data Sources**: PostgreSQL, CSV files, REST APIs
- **Data Lake**: Amazon S3
- **Data Warehouse**: Amazon Redshift
- **Container Runtime**: Docker
- **Infrastructure**: Kubernetes
- **Cloud Provider**: AWS

## Project Structure

```
mouseoverme/
├── dags/
│   ├── postgres_to_s3.py
│   ├── csv_to_s3.py
│   ├── api_to_s3.py
│   └── s3_to_redshift.py
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── kubernetes/
│   ├── airflow-deployment.yaml
│   ├── postgres-deployment.yaml
│   └── services.yaml
├── scripts/
│   ├── extract_postgres.py
│   ├── extract_csv.py
│   ├── extract_api.py
│   └── load_redshift.py
├── config/
│   ├── airflow.cfg
│   └── connections.yaml
├── sql/
│   └── redshift_schema.sql
├── .env.example
├── requirements.txt
└── README.md
```

## Prerequisites

- Docker Desktop or Docker Engine (v20.10+)
- Kubernetes cluster (local via Minikube/Kind or cloud EKS)
- kubectl CLI tool
- AWS Account with configured credentials
- Python 3.9+

## Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/mouseoverme.git
cd mouseoverme
```

### 2. Configure Environment Variables

```bash
cp .env.example .env
# Edit .env with your credentials
```

Required environment variables:
```env
# AWS Credentials
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-east-1
S3_BUCKET=mouseoverme-data-lake

# Redshift
REDSHIFT_HOST=your-cluster.redshift.amazonaws.com
REDSHIFT_PORT=5439
REDSHIFT_DB=mouseoverme_dw
REDSHIFT_USER=admin
REDSHIFT_PASSWORD=your_password

# PostgreSQL Source
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=source_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres

# API Configuration
API_BASE_URL=https://api.example.com
API_KEY=your_api_key
```

### 3. Build Docker Images

```bash
docker-compose build
```

### 4. Deploy to Kubernetes

```bash
# Create namespace
kubectl create namespace mouseoverme

# Apply Kubernetes configurations
kubectl apply -f kubernetes/ -n mouseoverme

# Verify deployments
kubectl get pods -n mouseoverme
```

### 5. Access Airflow UI

```bash
# Port forward Airflow webserver
kubectl port-forward svc/airflow-webserver 8080:8080 -n mouseoverme
```

Navigate to `http://localhost:8080` (default credentials: admin/admin)

## Pipeline Overview

### Data Extraction

1. **PostgreSQL Extraction**: Extracts tables from PostgreSQL database using incremental loading strategies
2. **CSV Ingestion**: Reads CSV files from mounted volumes or S3 buckets
3. **API Integration**: Fetches data from REST APIs with pagination and rate limiting

### Data Loading

1. **S3 Staging**: Raw data is stored in S3 with partitioning by date and source
2. **Redshift Loading**: Data is copied from S3 to Redshift using COPY commands
3. **Data Transformation**: SQL transformations create dimensional models in Redshift

### Airflow DAGs

- **daily_postgres_extract**: Runs daily at 2:00 AM UTC
- **hourly_api_fetch**: Runs every hour
- **csv_batch_load**: Triggered manually or via file sensor
- **redshift_transform**: Runs after all extractions complete

## Running the Pipeline

### Start All Services

```bash
docker-compose up -d
```

### Trigger DAGs Manually

From Airflow UI or CLI:
```bash
kubectl exec -it <airflow-scheduler-pod> -n mouseoverme -- airflow dags trigger daily_postgres_extract
```

### Monitor Pipeline

```bash
# Check Airflow logs
kubectl logs -f <airflow-scheduler-pod> -n mouseoverme

# Check task logs in Airflow UI
# Navigate to DAG → Task Instance → Logs
```

## Data Flow

1. **Extract Phase**: Data is extracted from sources and converted to Parquet format
2. **Stage Phase**: Parquet files are uploaded to S3 with metadata
3. **Load Phase**: Redshift COPY command loads data from S3
4. **Transform Phase**: SQL views and tables create analytical models

## AWS Resource Provisioning

Kubernetes manages the provisioning of AWS resources through:

- **IAM Roles for Service Accounts (IRSA)**: Pods assume IAM roles to access S3 and Redshift
- **External Secrets Operator**: Manages secrets from AWS Secrets Manager
- **S3 CSI Driver**: Mounts S3 buckets as volumes (optional)

## Monitoring and Alerts

- Airflow email alerts on DAG failures
- CloudWatch logs for AWS service monitoring
- Kubernetes metrics via Prometheus/Grafana (optional)

## Troubleshooting

### Common Issues

**Airflow tasks stuck in queue**
```bash
kubectl scale deployment airflow-worker --replicas=3 -n mouseoverme
```

**S3 connection errors**
- Verify IAM role permissions include S3 read/write
- Check AWS credentials in environment variables

**Redshift connection timeout**
- Verify security group allows inbound traffic from Kubernetes cluster
- Check VPC peering if using private Redshift endpoint

## Development

### Adding New Data Sources

1. Create extraction script in `scripts/`
2. Define new DAG in `dags/`
3. Add connection in Airflow UI or `connections.yaml`
4. Update Redshift schema if needed

### Testing Locally

```bash
# Run tests
python -m pytest tests/

# Test individual extraction scripts
python scripts/extract_postgres.py --dry-run
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-source`)
3. Commit changes (`git commit -am 'Add new data source'`)
4. Push to branch (`git push origin feature/new-source`)
5. Create Pull Request

## License

MIT License - see LICENSE file for details

## Contact

For questions or support, contact the data engineering team at data-eng@example.com

---

**Last Updated**: December 2025  
**Version**: 1.0.0