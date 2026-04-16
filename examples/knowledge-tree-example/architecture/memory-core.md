---
title: Memory core
status: approved
last_updated: 2026-04-16
related: [architecture/ingest-pipeline.md, decisions/2026-04-16-qdrant-over-mongo.md]
---

# Memory core

## Purpose

The memory core stores and retrieves per-user contextual facts for downstream LLM applications. Callers write facts as structured events and retrieve them via vector similarity scoped to a single user. The service targets sub-150ms p95 retrieval latency at 2.5M active users.

## Owners

Platform team. Repo: `contextlane/memory-core`. On-call rotation: `#oncall-memory`.

## Components

- **Write path**: FastAPI ingress, LLM-driven fact extraction, idempotent upsert into the vector store.
- **Read path**: query embedder, vector store client, optional re-ranker.
- **Vector store**: Qdrant cluster (see [decisions/2026-04-16-qdrant-over-mongo.md](../decisions/2026-04-16-qdrant-over-mongo.md)).
- **Embedding gateway**: LiteLLM proxy; abstracts the embedding provider.

## Key contracts

- `MemoryEvent`: write envelope, accepted at `POST /v1/memory`.
- `MemoryQuery`: read envelope, accepted at `POST /v1/memory:search`.

Schemas are omitted from this example tree; in a real project they would live in `contracts/`.

## Open questions

- How should we handle multi-tenant quota enforcement at the vector layer? Tracked in `open-questions/`.

## Decision log

- **2026-04-16**: replaced MongoDB Atlas Vector Search with Qdrant for retrieval. See [decisions/2026-04-16-qdrant-over-mongo.md](../decisions/2026-04-16-qdrant-over-mongo.md).
