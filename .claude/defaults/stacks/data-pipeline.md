# Data Pipeline Stack

ETL processes, data analytics, and machine learning projects. Use this for batch processing, data transformation, analytics dashboards, and ML/AI applications.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

### Python (Default for Data)

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Python | 3.11 | Minor |
| pandas | 2.x | Minor |
| polars | 0.20 | Moderate |
| scikit-learn | 1.4 | Minor |
| PyTorch | 2.2 | Moderate if 3+ |
| TensorFlow | 2.15 | Minor |

### Data Infrastructure

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Apache Airflow | 2.8 | Minor |
| dbt | 1.7 | Minor |
| Dagster | 1.6 | Minor |

**Recommendation**: Python is the standard for data work. Use pandas for small-medium data, polars or Spark for large data, and established ML frameworks.

## Default Stack by Use Case

### ETL / Data Processing

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Python 3.11+ | Data ecosystem |
| **Processing** | pandas / polars | Data manipulation |
| **Large Scale** | Apache Spark | Distributed processing |
| **Orchestration** | Airflow / Dagster | Pipeline scheduling |
| **Transformation** | dbt | SQL transformations |
| **Storage** | PostgreSQL / S3 | Data storage |
| **Testing** | pytest | Pipeline testing |
| **Source Control** | GitHub | Repository hosting |

### Analytics / BI

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Python / SQL | Analysis |
| **Notebooks** | Jupyter | Exploration |
| **Visualization** | Plotly / Altair | Charts |
| **Dashboards** | Streamlit / Metabase | Interactive dashboards |
| **Database** | PostgreSQL / DuckDB | Query engine |
| **Warehouse** | Snowflake / BigQuery | Cloud warehouse |
| **Source Control** | GitHub | Repository hosting |

### Machine Learning

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Language** | Python 3.11+ | ML ecosystem |
| **Framework** | PyTorch / scikit-learn | Model training |
| **Experiment Tracking** | MLflow / Weights & Biases | Tracking |
| **Feature Store** | Feast | Feature management |
| **Model Serving** | FastAPI / BentoML | API deployment |
| **Compute** | AWS SageMaker / GCP Vertex | Training infrastructure |
| **Testing** | pytest | Model testing |
| **Source Control** | GitHub | Repository hosting |

## Testing Stack

| Tool | Purpose |
|------|---------|
| pytest | Unit testing |
| great_expectations | Data validation |
| pandera | DataFrame validation |
| hypothesis | Property-based testing |

## Logging Stack

| Tool | Purpose |
|------|---------|
| structlog | Structured logging |
| Sentry | Error tracking |
| OpenTelemetry | Distributed tracing |

## Project Structure

### ETL Pipeline
```
{project}/
├── pipelines/
│   ├── __init__.py
│   └── {pipeline_name}/
│       ├── extract.py
│       ├── transform.py
│       └── load.py
├── dags/              # Airflow DAGs
├── models/            # dbt models (if using)
├── tests/
├── config/
├── pyproject.toml
└── README.md
```

### ML Project
```
{project}/
├── src/
│   ├── data/          # Data loading
│   ├── features/      # Feature engineering
│   ├── models/        # Model definitions
│   ├── training/      # Training scripts
│   └── serving/       # API serving
├── notebooks/         # Exploration
├── experiments/       # Experiment configs
├── tests/
├── pyproject.toml
└── README.md
```

### Analytics Project
```
{project}/
├── notebooks/
├── dashboards/
├── queries/           # SQL queries
├── src/
│   └── {package}/
├── data/              # Sample data
├── pyproject.toml
└── README.md
```

## Infrastructure Options

### Orchestration

| Tool | Best For | Notes |
|------|----------|-------|
| Airflow | Complex DAGs | Industry standard |
| Dagster | Modern pipelines | Asset-based |
| Prefect | Simple workflows | Easy setup |
| dbt | SQL transforms | Warehouse-focused |

### Compute

| Service | Best For | Notes |
|---------|----------|-------|
| AWS EMR | Spark workloads | Managed Spark |
| Databricks | Unified platform | Spark + notebooks |
| Google Dataflow | Streaming | Apache Beam |
| Modal | Serverless ML | Easy scaling |

### Storage

| Service | Best For | Notes |
|---------|----------|-------|
| S3 / GCS | Object storage | Data lake |
| Snowflake | Analytics warehouse | SQL focused |
| BigQuery | Analytics warehouse | GCP native |
| Delta Lake | ACID on data lake | Lakehouse |
| DuckDB | Local analytics | Embedded OLAP |

## For Non-Technical Users

When the user is non-technical, clarify the use case:

> "Data projects can mean different things. Could you help me understand:
> - Are you trying to move data from one place to another? (ETL)
> - Do you want to analyze data and see charts? (Analytics)
> - Are you building something that learns from data? (Machine Learning)
>
> This helps me set up the right tools."

## For Technical Users

Ask about specifics:

> "What type of data work is this? (ETL, analytics, ML/AI)"

Then dive into:
- Data scale? (GB, TB, PB)
- Batch vs streaming?
- Cloud provider preference?
- Orchestration needs?
- ML framework preference? (PyTorch, TensorFlow, JAX)
