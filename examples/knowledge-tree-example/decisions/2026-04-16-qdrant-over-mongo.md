---
title: Qdrant over MongoDB Atlas Vector Search
status: approved
last_updated: 2026-04-16
related: [architecture/memory-core.md]
---

# Qdrant over MongoDB Atlas Vector Search

## Context

Retrieval latency on the Memory core was consistently 800 to 900ms p95, against a 100 to 150ms target. Profiling pointed at MongoDB Atlas Vector Search: the HNSW index applies post-filtering, which is fundamentally hostile to the highly selective per-user filter the workload requires. At 2.5M active users, the effective recall-per-candidate collapses, forcing larger candidate sets and dominating latency.

## Decision

Replace MongoDB Atlas Vector Search with Qdrant as the underlying vector store. Qdrant is the default provider in the memory layer's abstraction, so the migration is a configuration change plus a one-time backfill, not a rewrite.

## Rationale

- **Pre-filtered HNSW**: Qdrant applies payload filters before graph traversal, which matches the per-user filter shape and restores candidate efficiency.
- **Benchmarks**: Synthetic benchmarks using NVIDIA DataDesigner data put Qdrant at ~90ms p95 under the same selective filter, versus ~850ms on MongoDB.
- **Abstraction fit**: The memory layer already speaks to vector stores through a provider-agnostic interface. No application code changes.
- **Alternatives considered**: Self-hosted Weaviate (heavier ops), pgvector with IVFFlat (recall too low at 2.5M users), staying on MongoDB with a separate filter index (complex, fragile, and still post-filter underneath).

## Consequences

- Introduces Qdrant to the ops surface. On-call runbook needs an entry.
- Benchmarks must be re-run on production traffic shape before the migration date.
- The MongoDB cluster stays on standby for 30 days as rollback insurance, then is decommissioned.
- No contract changes for callers of the Memory core.

## Follow-ups

- [ ] Production benchmark against real traffic shape (@platform)
- [ ] Qdrant runbook and dashboard (@platform)
- [ ] Data backfill job, idempotent, resumable (@platform)
- [ ] Decommission plan for MongoDB cluster after 30-day standby
