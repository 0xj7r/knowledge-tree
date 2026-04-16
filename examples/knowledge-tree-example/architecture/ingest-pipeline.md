---
title: Ingest pipeline
status: draft
last_updated: 2026-04-10
related: [architecture/memory-core.md]
---

# Ingest pipeline

## Purpose

The ingest pipeline consumes raw user events (chat turns, app actions, explicit facts) and converts them into normalized memory events ready for the Memory core. Throughput target: 10k events/sec burst, 2k sustained.

## Owners

Platform team. Repo: `contextlane/ingest`. Shared on-call with Memory core.

## Components

- **Gateway**: accepts raw events from producer services; validates envelope only.
- **Extractor**: LLM-driven; distills raw events into structured `MemoryEvent` candidates.
- **Deduper**: suppresses semantically equivalent facts within a per-user window.
- **Writer**: calls Memory core `POST /v1/memory` with idempotency keys.

## Key contracts

- Producer envelope (internal, not versioned yet).
- `MemoryEvent` on the write side.

## Open questions

- Should extraction run inline or async? Inline simplifies back-pressure; async buys throughput headroom. Not yet decided.
- What is the retention policy for raw events after extraction?

## Decision log

- (none yet; this node is still a draft.)
