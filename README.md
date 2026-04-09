# YouTube API ELT Pipeline

## Overview

This repository contains an end-to-end **ELT pipeline** that extracts data from the **YouTube Data API**, loads it into a **PostgreSQL** data warehouse, applies lightweight transformations, and validates the result with automated tests.

The project was built to practice core **data engineering** concepts in a realistic but manageable setup, including:

- workflow orchestration with **Apache Airflow**
- containerized development with **Docker** and **Docker Compose**
- data loading and transformation with **Python** and **SQL**
- database testing with **Soda Core**
- unit and integration testing with **pytest**
- CI/CD automation with **GitHub Actions**

The pipeline currently uses the **MrBeast** YouTube channel as the source, but it can be adapted to any public YouTube channel by changing the relevant channel configuration.

## Architecture

<p align="center">
  <img src="images/project_architecture.png" alt="Project architecture" width="700">
</p>

## Project Goal

The main goal of this project is to build a hands-on ELT workflow and understand how modern data tooling fits together in practice.

Instead of focusing only on data extraction, the project also covers the surrounding engineering work that makes a pipeline more robust:

- reproducible environments
- orchestration across multiple steps
- schema-based loading patterns
- automated tests
- CI/CD checks for pipeline changes

## Data Source

The source data is pulled from the **YouTube Data API v3**.

For each video, the pipeline extracts a focused set of metadata fields:

- `Video_ID`
- `Title`
- `PublishedAt`
- `Duration`
- `Video_Views`
- `Likes_Count`
- `Comments_Count`

The implementation is currently configured for the **MrBeast** channel, but the project can be reused for another channel by updating the API-related configuration.

## Pipeline Flow

The project is organized as an ELT process with three main stages:

1. **Extract**  
   Airflow tasks call the YouTube API and collect raw video metadata.

2. **Load**  
   The extracted data is first written into a **staging** layer in PostgreSQL.

3. **Transform**  
   The data is then cleaned and upserted into a **core** layer for downstream analysis.

The initial run performs a full load. Subsequent runs are designed to update existing records where needed and insert newly available videos.

## Airflow DAGs

The pipeline is orchestrated with three DAGs:

### `produce_json`
Extracts video data from the YouTube API and stores the raw payload in JSON format.

### `update_db`
Loads the JSON data into PostgreSQL and updates the `staging` and `core` schemas.

### `data_quality`
Runs data quality checks against the warehouse layers using Soda Core.

These DAGs can be viewed locally in the Airflow UI at:

```bash
http://localhost:8080
```

## Tech Stack

| Area | Tools |
|---|---|
| Containerization | Docker, Docker Compose |
| Orchestration | Apache Airflow |
| Storage | PostgreSQL |
| Programming | Python, SQL |
| Testing | pytest, Soda Core |
| CI/CD | GitHub Actions |


## Local Setup

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd <your-repo-name>
```

### 2. Configure environment variables

Create a `.env` file in the project root and define the variables required by Docker Compose and Airflow.

Typical variables include:

```env
API_KEY=your_youtube_api_key
CHANNEL_HANDLE=MrBeast
FERNET_KEY=your_fernet_key

POSTGRES_CONN_HOST=postgres
POSTGRES_CONN_PORT=5432
POSTGRES_CONN_USERNAME=your_postgres_admin_user
POSTGRES_CONN_PASSWORD=your_postgres_admin_password

ELT_DATABASE_NAME=your_elt_database
ELT_DATABASE_USERNAME=your_elt_user
ELT_DATABASE_PASSWORD=your_elt_password

METADATA_DATABASE_NAME=your_airflow_metadata_database
METADATA_DATABASE_USERNAME=your_metadata_user
METADATA_DATABASE_PASSWORD=your_metadata_password

CELERY_BACKEND_NAME=your_celery_database
CELERY_BACKEND_USERNAME=your_celery_user
CELERY_BACKEND_PASSWORD=your_celery_password

AIRFLOW_WWW_USER_USERNAME=airflow
AIRFLOW_WWW_USER_PASSWORD=airflow
AIRFLOW_UID=50000
DOCKERHUB_NAMESPACE=your_dockerhub_namespace
DOCKERHUB_REPOSITORY=your_image_name
```

### 3. Build and start the containers

```bash
docker compose up -d
```

### 4. Access Airflow

Open:

```bash
http://localhost:8080
```

Log in with the Airflow web credentials defined in your environment.

## Database Access

You can inspect the PostgreSQL database in two ways:

- connect from inside the Docker environment
- connect from a local database client such as **DBeaver**

If PostgreSQL is mapped like this:

```yaml
5433:5432
```

then:

- use `5432` **inside Docker containers**
- use `localhost:5433` **from your host machine**

## Testing

### Unit and integration tests

Tests are implemented with **pytest** and cover:

- Airflow variable and connection access
- DAG integrity
- database connectivity
- API connectivity

Run them with:

```bash
docker exec -t airflow-worker sh -c "pytest tests/ -v"
```

### Data quality tests

Data quality checks are implemented with **Soda Core** and executed through the `data_quality` DAG.

The checks validate the database layers after load and transformation.

## CI/CD

GitHub Actions is used to automate validation of pipeline changes.

The workflow is responsible for tasks such as:

- building the custom Docker image
- pushing the image to Docker Hub
- starting the Airflow stack in CI
- running unit and integration tests
- running DAG tests

This helps catch regressions when changing DAGs, dependencies, Docker configuration, or supporting pipeline code.

## Notes

- This project is intended for **learning and practice**, not production use.
- The current implementation is designed around a single YouTube channel configuration.
- The Docker Compose setup is based on the official Apache Airflow example and adapted for this project.

## License

This repository is intended for educational and portfolio purposes.

The `docker-compose.yaml` file is derived from the Apache Airflow project and remains subject to the **Apache License 2.0**. Please retain all required notices and attribution.


