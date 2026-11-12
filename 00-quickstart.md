# mini-RAG | From notebooks to the PRODUCTION

[Course Playlist Link](https://www.youtube.com/playlist?list=PLvLvlVqNQGHCUR2p0b8a0QpVjDUg50wQj)

> **Architecture Overview:** Enterprise-grade Retrieval-Augmented Generation systems designed to operate in real production environments, moving beyond the Proof of Concept (PoC) stage by building flexible modular services, decoupling retrieval mechanisms from generation, and automating asynchronous tasks to ensure performance stability and scalability.

---

## 🗂️ Course Map — Modules, Videos, Notes & Code Locations

| Module (Videos) | Target Concepts | Obsidian Notes | Source Code | Jupyter Lab Notebook |
| :--- | :--- | :--- | :--- | :--- |
| **01 · Ingestion & API Core**<br>(Videos 1–6) | FastAPI Architecture, MVC Pattern, Config Isolation, Streaming Uploads, Chunking & Overlap | [[01-MVC-FastAPI-Design]]<br>[[05-File-Validation-IO]]<br>[[04-Chunking-Strategies-Tradeoffs]] | `.gitignore`, `README.md`, `requirements.txt`, `.env.example`, `src/main.py`, `src/routes/base.py`, `src/helpers/config.py`, `src/controllers/data.py`, `src/models/enums/`, `src/routes/data.py` | `notebooks/01_document_parsing_chunking.ipynb` |
| **02 · Data Models & Migrations**<br>(Videos 7–9) | NoSQL Document Models, MongoDB/Motor, Schema Migration to Async Postgres, Declarative SQLAlchemy, Alembic | [[03-Database-Migration-Strategy]] | `src/models/db_schemes/`, `src/models/project.py`, `src/models/asset.py`, `src/models/db_schemes/mini_rag/`, `alembic/`, `alembic.ini` | `notebooks/02_database_schemas_and_indexing.ipynb` |
| **03 · Vector Storage & Retrieval**<br>(Videos 10–12) | Vector Indexing, Distance Metrics, Qdrant Integration, Native PGVector with HNSW Indexing | [[06-Cosine-vs-DotProduct]]<br>[[07-Qdrant-Engine-Internals]]<br>[[08-PGVector-and-HNSW-Indexing]]<br>[[99-Cheatsheets-and-Formulas/Vector-Distance-Metrics]] | `src/stores/vectordb/providers/qdrant.py`, `src/stores/vectordb/factory.py`, `src/stores/vectordb/providers/pgvector.py` | `notebooks/03_qdrant_vs_pgvector_benchmarks.ipynb` |
| **04 · LLM & Prompt Engineering**<br>(Videos 13–14) | Provider Factory, Interface Abstraction, Multi-lingual Prompt Templates, Semantic Search, Generation Decoupling | [[02-Factory-Interface-Patterns]]<br>[[11-Provider-Abstraction-Layer]]<br>[[09-Search-vs-Generation-Decoupling]]<br>[[10-Dynamic-Prompt-Localization]] | `src/stores/llm/`, `src/controllers/nlp.py`, `src/routes/nlp.py` | `notebooks/04_prompt_injection_evals.ipynb` |
| **05 · Async Orchestration & Distributed**<br>(Videos 15–16) | Celery Architecture, RabbitMQ Broker, Redis Backend, Late Ack, Idempotency Manager, Workflow Chains, Celery Beat | [[12-Celery-RabbitMQ-Lifecycle]]<br>[[13-Idempotency-and-Late-Ack]] | `src/tasks/`, `src/helpers/idempotency.py`, `docker/docker-compose.yml` | `notebooks/05_async_tasks_stress_test.ipynb` |
| **06 · DevOps & Observability**<br>(Videos 17–18) | Multi-Container Docker Compose, Nginx Proxy, Prometheus Telemetry, Grafana Dashboards, CI/CD Actions | [[14-Docker-Compose-Orchestration]]<br>[[15-Prometheus-Grafana-Monitoring]] | `docker/`, `.github/workflows/deploy.yml`, `nginx/` | `notebooks/06_infrastructure_benchmarks.ipynb` |

---

## 📐 Core Architectural Invariants (Non-Negotiable Engineering Rules)

1. **Decouple Search from Generation:** Do not merge the semantic search process with the LLM call into a single closed path; each stage must be independently callable, evaluable, and testable for latency and retrieval accuracy (Hit Rate / MRR) as a fully isolated unit[cite: 8].
2. **Strict Provider Abstraction:** Never call an external provider library (OpenAI, Cohere, Qdrant, PGVector) directly inside the execution logic of the Controllers; the call must pass exclusively through a unified `Interface` and `Factory Pattern` to ensure swappability without modifying the core code[cite: 2, 8, 14, 18].
3. **Idempotency by Design:** Any operation that consumes computational resources (file chunking, vector embedding generation) must be tied to a verification system based on input hashing (Input Arguments Hashing via SHA-256) to prevent reprocessing when a request is duplicated or distributed tasks are restarted[cite: 9, 18].
4. **Asynchronous Heavy Lifting:** The HTTP request lifecycle (FastAPI) is confined to input validation, initial data logging, and returning an immediate Task ID to the client; heavy operations (Disk IO, Tokenization, Remote Embedding Calls) are fully offloaded to Celery Workers behind a reliable message broker (RabbitMQ)[cite: 9, 16, 17].

---

## 🛠️ CLI Quick Reference

```bash
# Activate the working environment and run locally
conda activate mini-rag-app
uvicorn src.main:app --reload --host 0.0.0.0 --port 5000

# Manage database migrations via Alembic
alembic revision --autogenerate -m "create_initial_schema"
alembic upgrade head

# Start the infrastructure and service containers via Docker
cd docker
docker compose up -d postgres_vector qdrant rabbitmq redis
docker compose --profile monitoring up -d prometheus grafana

# Run Celery workers and scheduled tasks
celery -A src.tasks.celery_worker worker --loglevel=info -Q data_indexing,file_processing -E
celery -A src.tasks.celery_worker beat --loglevel=info
celery -A src.tasks.celery_worker flower --port=5555