# ADR-002: Dedicated PostgreSQL storage

Status: Proposed

## Context

Memories require durable, queryable records, source links, relationships and audit history.

## Decision

Evaluate a dedicated locally hosted PostgreSQL container, separate from all consumer application databases. Begin with full-text search and structured filters; add pgvector only if benchmark results justify semantic search.

## Consequences

The service owns its own schema, migrations, least-privilege credentials, persistent volume and backup/restore procedure. No consumer project's tests may depend on the developer-owned instance. This ADR is not an implementation claim.
