# AI Research Paper Curator

An AI engineering project for automatically ingesting, processing, storing, and retrieving academic research papers using a production-style Retrieval-Augmented Generation (RAG) architecture.

The system currently provides the infrastructure and ingestion foundation for a research assistant that can collect papers from arXiv, parse scientific PDFs, store structured paper data, and prepare that content for retrieval and LLM-based research workflows.

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.12+-blue.svg" alt="Python Version">
  <img src="https://img.shields.io/badge/FastAPI-0.115+-green.svg" alt="FastAPI">
  <img src="https://img.shields.io/badge/OpenSearch-2.19-orange.svg" alt="OpenSearch">
  <img src="https://img.shields.io/badge/Docker-Compose-blue.svg" alt="Docker">
  <img src="https://img.shields.io/badge/Status-Active%20Development-brightgreen.svg" alt="Status">
</p>

<p align="center">
  <img src="static/mother_of_ai_project_rag_architecture.gif" alt="RAG System Architecture" width="800">
</p>

## Overview

The goal of this project is to build an end-to-end AI research assistant around academic papers.

Rather than treating RAG as only an LLM prompt, the project focuses on the full engineering pipeline behind the system:

- research paper ingestion
- scientific PDF processing
- workflow orchestration
- structured data storage
- search infrastructure
- API development
- local LLM serving
- testing and monitoring
- retrieval and generation workflows

The current implementation includes the infrastructure layer and automated research-paper ingestion pipeline.

## Current Features

### Infrastructure

- FastAPI backend with health checks and interactive API documentation
- PostgreSQL for research-paper metadata and processed content
- OpenSearch for search and retrieval infrastructure
- Apache Airflow for workflow orchestration
- Ollama for local LLM serving
- Docker Compose for multi-service orchestration
- Pytest, Ruff, and MyPy for testing and code quality

### Research Paper Ingestion

- arXiv API integration
- Rate limiting and retry handling
- Automated paper metadata retrieval
- Scientific PDF processing using Docling
- Structured content extraction from research papers
- PostgreSQL persistence
- Automated Airflow ingestion workflows
- API endpoints for accessing stored papers

## System Architecture

The project is designed as a multi-stage AI data pipeline:

```text
                 arXiv
                   |
                   v
          +------------------+
          | Apache Airflow   |
          | Ingestion DAGs   |
          +------------------+
                   |
                   v
          +------------------+
          | Metadata Fetcher |
          +------------------+
             |           |
             v           v
       arXiv API     PDF Download
                         |
                         v
                  +---------------+
                  |    Docling    |
                  | PDF Processing|
                  +---------------+
                         |
                         v
                  +---------------+
                  |  PostgreSQL   |
                  | Paper Storage |
                  +---------------+
                         |
                         v
                  +---------------+
                  |  OpenSearch   |
                  | Search Layer  |
                  +---------------+
                         |
                         v
                  +---------------+
                  |    FastAPI    |
                  | Backend/API   |
                  +---------------+
                         |
                         v
                  +---------------+
                  |    Ollama     |
                  |  Local LLM    |
                  +---------------+
```

The retrieval and full RAG generation stages are being developed on top of this foundation.

## Tech Stack

| Area | Technologies |
|------|--------------|
| Programming | Python 3.12+ |
| Backend | FastAPI, Pydantic |
| Database | PostgreSQL 16 |
| Search | OpenSearch 2.19 |
| Workflow Orchestration | Apache Airflow 3.0 |
| PDF Processing | Docling |
| LLM Serving | Ollama |
| Infrastructure | Docker, Docker Compose |
| Package Management | uv |
| Testing | Pytest |
| Code Quality | Ruff, MyPy |
| Source Control | Git, GitHub |

## How It Works

### 1. Paper Discovery

The ingestion workflow queries the arXiv API for academic papers matching a research category or search query.

```python
from src.services.arxiv.factory import make_arxiv_client

async def fetch_recent_papers():
    client = make_arxiv_client()

    papers = await client.search_papers(
        query="cat:cs.AI",
        max_results=10,
        from_date="20240801",
        to_date="20240807",
    )

    return papers
```

### 2. PDF Processing

The system downloads and parses research-paper PDFs using Docling.

```python
from src.services.pdf_parser.factory import make_pdf_parser_service

async def process_paper_pdf(pdf_url: str):
    parser = make_pdf_parser_service()
    parsed_content = await parser.parse_pdf_from_url(pdf_url)

    return parsed_content
```

This produces structured content that can later be used for indexing, chunking, retrieval, and LLM context.

### 3. Automated Ingestion

A metadata-fetching service coordinates the ingestion pipeline.

```python
from src.services.metadata_fetcher import make_metadata_fetcher

async def ingest_papers():
    fetcher = make_metadata_fetcher()

    results = await fetcher.fetch_and_store_papers(
        query="cat:cs.AI",
        max_results=5,
        from_date="20240807",
    )

    return results
```

Apache Airflow automates this workflow so new papers can be collected and processed on a schedule.

### 4. Storage

Processed research-paper metadata and content are stored in PostgreSQL.

OpenSearch provides the search infrastructure used as the project moves toward hybrid retrieval and full RAG functionality.

### 5. API Access

FastAPI exposes application functionality through REST endpoints and provides interactive documentation through Swagger UI.

## Quick Start

### Prerequisites

Make sure the following are installed:

- Docker Desktop with Docker Compose
- Python 3.12+
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- At least 8 GB RAM
- Approximately 20 GB of available disk space

### Clone the Repository

```bash
git clone https://github.com/alihatemm/ai-research-paper-curator.git
cd ai-research-paper-curator
```

### Install Dependencies

```bash
uv sync
```

### Start the Services

```bash
docker compose up --build -d
```

### Check System Health

```bash
curl http://localhost:8000/health
```

## Local Services

| Service | URL | Purpose |
|---------|-----|---------|
| FastAPI Docs | http://localhost:8000/docs | API testing and documentation |
| Airflow | http://localhost:8080 | Workflow orchestration |
| OpenSearch Dashboards | http://localhost:5601 | Search engine interface |

> Airflow credentials can be found in `airflow/simple_auth_manager_passwords.json.generated` after setup.

## Infrastructure

The project currently runs several services locally through Docker Compose.

### FastAPI

Provides the backend application and API layer.

**Port:** `8000`

### PostgreSQL

Stores paper metadata and processed research content.

**Port:** `5432`

### OpenSearch

Provides the search infrastructure for the RAG retrieval layer.

**Ports:** `9200`, `5601`

### Apache Airflow

Schedules and manages the research-paper ingestion pipeline.

**Port:** `8080`

### Ollama

Provides local LLM serving for the generation layer of the system.

**Port:** `11434`

## Airflow Data Pipeline

The ingestion pipeline currently includes:

```text
Scheduled Airflow DAG
        |
        v
Search arXiv
        |
        v
Fetch Paper Metadata
        |
        v
Download PDF
        |
        v
Parse PDF with Docling
        |
        v
Extract Structured Content
        |
        v
Store Paper + Content
        |
        v
PostgreSQL
```

The main components include:

- `MetadataFetcher` — coordinates the ingestion workflow
- `ArxivClient` — retrieves papers with rate limiting and retry logic
- `PDFParserService` — parses scientific PDFs using Docling
- `Airflow DAGs` — automate ingestion
- `PostgreSQL` — stores metadata and extracted content

## Project Structure

```text
ai-research-paper-curator/
├── airflow/
│   ├── dags/
│   │   ├── arxiv_ingestion/
│   │   └── arxiv_paper_ingestion.py
│   └── requirements-airflow.txt
│
├── notebooks/
│   ├── week1/
│   │   └── week1_setup.ipynb
│   └── week2/
│       └── week2_data_ingestion.ipynb
│
├── src/
│   ├── main.py
│   ├── routers/
│   ├── models/
│   ├── repositories/
│   ├── schemas/
│   ├── services/
│   │   ├── arxiv/
│   │   ├── pdf_parser/
│   │   ├── metadata_fetcher.py
│   │   └── ollama/
│   ├── db/
│   ├── config.py
│   └── dependencies.py
│
├── static/
├── tests/
├── .env.example
├── .gitignore
├── .pre-commit-config.yaml
├── Dockerfile
├── LICENSE
├── Makefile
├── README.md
├── compose.yml
├── pyproject.toml
└── uv.lock
```

## Running the Project

### Using the Makefile

```bash
make help
```

Common commands:

```bash
make start
make health
make test
make stop
```

| Command | Description |
|---------|-------------|
| `make start` | Start all services |
| `make stop` | Stop all services |
| `make restart` | Restart services |
| `make status` | Show service status |
| `make logs` | View service logs |
| `make health` | Check service health |
| `make setup` | Install dependencies |
| `make format` | Format code |
| `make lint` | Run linting and type checks |
| `make test` | Run tests |
| `make test-cov` | Run tests with coverage |
| `make clean` | Clean local resources |

### Direct Commands

```bash
docker compose up --build -d
docker compose ps
docker compose logs
uv run pytest
```

## Testing

The project includes a test suite under:

```text
tests/
```

Run all tests with:

```bash
uv run pytest
```

Or:

```bash
make test
```

For coverage:

```bash
make test-cov
```

## Development Roadmap

The infrastructure and ingestion pipeline provide the foundation for the full RAG system.

### Completed

- [x] Docker-based infrastructure
- [x] FastAPI application
- [x] PostgreSQL integration
- [x] OpenSearch infrastructure
- [x] Airflow orchestration
- [x] Ollama local LLM service
- [x] arXiv API integration
- [x] PDF processing with Docling
- [x] Automated paper ingestion
- [x] Metadata and content storage

### In Progress / Planned

- [ ] Hybrid retrieval using BM25 and semantic vectors
- [ ] Context-aware paper chunking
- [ ] Embedding generation
- [ ] Retrieval evaluation using ranking metrics such as nDCG
- [ ] Full retrieval-augmented generation pipeline
- [ ] Prompt optimization
- [ ] LLM response evaluation
- [ ] Langfuse observability
- [ ] A/B testing
- [ ] Production deployment

## What I'm Learning

This project is helping me build practical experience with:

- designing multi-service AI systems
- building backend APIs with FastAPI
- working with asynchronous Python
- building automated data pipelines
- orchestrating workflows with Airflow
- processing scientific documents
- designing database-backed applications
- using OpenSearch as retrieval infrastructure
- running LLMs locally
- containerizing services with Docker
- testing and debugging distributed application components
- understanding how production RAG systems are structured beyond just the LLM layer

## Troubleshooting

### Services are not starting

Wait a few minutes after starting the stack and inspect the logs:

```bash
docker compose logs
```

### Port conflicts

Check whether another service is using:

```text
8000
8080
5432
9200
5601
11434
```

### Docker memory issues

Increase the amount of memory allocated to Docker Desktop.

### Reset the Environment

```bash
docker compose down --volumes
docker compose up --build -d
```

## Project Origin

This project began from the **Jam With AI "Mother of AI" / arXiv Paper Curator learning project**.

I am using the original project as a structured foundation for learning production AI engineering while implementing, running, testing, understanding, and extending the system through each stage of the RAG pipeline.

Original educational material and architecture concepts are credited to **Jam With AI**.

My goal with this repository is to develop hands-on experience with the engineering behind production RAG systems rather than treating the project as a finished black-box implementation.

## Author

**Ali Hatem**

Computer Science @ Florida International University

Interested in software engineering, AI/ML, backend development, and AI infrastructure.

[LinkedIn](https://www.linkedin.com/in/alihatemm) | [GitHub](https://github.com/alihatemm)

## License

MIT License — see [LICENSE](LICENSE) for details.
