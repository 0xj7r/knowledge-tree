---
title: Glossary
status: approved
last_updated: 2026-04-16
related: []
---

# Glossary

Canonical terminology for the ContextLane project. Terms here are the single source of truth; node titles and prose elsewhere should match.

## Memory event

A single structured fact written to the Memory core. Carries a user id, an extracted text span, an embedding, and provenance metadata. Idempotent by `(user_id, content_hash)`.

## Memory core

The service that stores and retrieves memory events. See [architecture/memory-core.md](architecture/memory-core.md).

## Per-user filter

A vector-store query predicate that restricts results to a single user's memory events. Highly selective at scale; drives the choice of vector store.

## Post-filter

A vector search strategy that retrieves top-K candidates first, then applies predicates. Hostile to selective filters because most candidates are discarded. Contrast with pre-filter.

## Pre-filter

A vector search strategy that applies predicates before graph traversal, so candidates are drawn only from the allowed subset. Required for efficient selective filtering at the ContextLane scale.
