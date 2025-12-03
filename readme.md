# Meditrack360

An end-to-end data engineering pipeline that ingests data from multiple sources, orchestrates workflows with Airflow, and delivers analytics-ready data to Amazon Redshift.

## Architecture Overview

```
┌─────────────────┐     ┌─────────────┐     ┌──────────────┐     ┌───────────────┐
│  Data Sources   │     │   Airflow   │     │  Amazon S3   │     │   Redshift    │
│  ─────────────  │────▶│ Orchestrator│────▶│  Data Lake   │────▶│  Data Warehouse│
│ • PostgreSQL    │     └─────────────┘     └──────────────┘     └───────────────┘
│ • CSV Files     │
│ • REST APIs     │
└─────────────────┘
        │
        └──────────────── All running on Docker ──────────────────┘
```

## Tech Stack

| Component | Technology |
|-----------|------------|
| Data Sources | PostgreSQL, CSV, REST APIs |
| Orchestration | Apache Airflow |
| Storage | Amazon S3 |
| Data Warehouse | Amazon Redshift |
| Containerization | Docker |

## Project Structure

```
meditrack360/
├── dags/                     # Airflow DAGs
│   ├── postgres_ingestion.py
│   ├── csv_ingestion.py
│   └── api_ingestion.py
├── src/
│   ├── extractors/           # Data extraction modules
│   │   ├── postgres_extractor.py
│   │   ├── csv_extractor.py
│   │   └── api_extractor.py
│   ├── loaders/              # S3 and Redshift loaders
│   │   ├── s3_loader.py
│   │   └── redshift_loader.py
│   └── transformers/         # Data transformation logic
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── config/
│   └── config.yaml
├── tests/
├── requirements.txt
└── README.md
```

## Prerequisites

- Docker & Docker Compose
- AWS CLI configured
- Python 3.9+

## Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/your-org/meditrack360.git
cd meditrack360
```

### 2. Set Environment Variables
```bash
cp .env.example .env
# Edit .env with your credentials
```

### 3. Build Docker Images
```bash
docker-compose build
```

### 4. Deploy to Kubernetes
```bash
kubectl apply -f k8s/
```

### 5. Start Airflow
```bash
docker-compose up -d airflow
```

Access Airflow UI at `http://localhost:8080`

## Data Sources Configuration

### PostgreSQL
```yaml
postgres:
  host: your-postgres-host
  port: 5432
  database: source_db
  user: ${POSTGRES_USER}
  password: ${POSTGRES_PASSWORD}
```

### CSV Files
```yaml
csv:
  input_path: /data/input/
  file_patterns:
    - "*.csv"
```

### API
```yaml
api:
  base_url: https://api.example.com
  endpoints:
    - /users
    - /transactions
  auth_type: bearer
```

## AWS Resources (Kubernetes Provisioned)

- **S3 Bucket**: Raw and processed data storage
- **Redshift Cluster**: Data warehouse for analytics
- **IAM Roles**: Service accounts for secure access

## Pipeline Workflow

1. **Extract**: Pull data from PostgreSQL, CSV files, and APIs
2. **Load to S3**: Store raw data in S3 (landing zone)
3. **Transform**: Clean and transform data
4. **Load to Redshift**: Copy transformed data to Redshift tables

## Running Tests

```bash
docker-compose run --rm app pytest tests/
```

## Monitoring

- **Airflow UI**: DAG status and logs
- **CloudWatch**: AWS resource metrics
- **Kubernetes Dashboard**: Pod health and resource usage

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## License

MIT License