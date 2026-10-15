# ADR-001: Agent-independent MCP boundary

Status: Proposed

## Context

Memory should serve Codex initially without being coupled to Codex internals or any consumer project's technology stack.

## Decision

Use MCP as the client integration boundary. Start with a local stdio transport for the proof of concept. Keep memory rules and persistence independent of client adapters.

## Consequences

Codex is the first integration target; other compatible agents may follow. Workspace context propagation and per-project scope must be verified, not assumed. This ADR is not an implementation claim.
