# Idempotency and Late Ack

## Summary

Ensure background tasks can safely retry without duplicate side effects by storing task hashes, deduplicating inputs, and acknowledging only after durable success.

## Key ideas

- Treat this note as the flattened, root-level artifact created during the vault refactor.
- Keep links stable via root-level Obsidian references like [[00-quickstart]].
- Related notes:
  - [[01-MVC-FastAPI-Design]]
  - [[02-Factory-Interface-Patterns]]
  - [[03-Database-Migration-Strategy]]
  - [[04-Chunking-Strategies-Tradeoffs]]
  - [[05-File-Validation-IO]]
  - [[06-Cosine-vs-DotProduct]]
  - [[07-Qdrant-Engine-Internals]]
  - [[08-PGVector-and-HNSW-Indexing]]
  - [[09-Search-vs-Generation-Decoupling]]
  - [[10-Dynamic-Prompt-Localization]]
  - [[11-Provider-Abstraction-Layer]]
  - [[12-Celery-RabbitMQ-Lifecycle]]
  - [[14-Docker-Compose-Orchestration]]
  - [[15-Prometheus-Grafana-Monitoring]]

## Tags

`idempotency`, `reliability`, `celery`

---

