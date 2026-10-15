# ag-mem-on

**Project-scoped persistent memory for AI coding agents.**

ag-mem-on is a proposed self-hosted, project-agnostic memory service. It helps coding agents recover verified decisions, implementation lessons and useful task context across sessions without treating conversational recollections as authoritative project documentation.

## Principles

- **Project-agnostic:** no assumptions about a project's language, framework, directory layout or domain.
- **Project-scoped:** retrieval and writes are isolated by stable project identity, not by an agent's prompt alone.
- **Agent-independent:** standard Model Context Protocol (MCP) interface; Codex first, other compatible clients later.
- **Source-aware:** retain provenance, verification state, expiry and supersession; the repository remains authoritative.
- **Local-first:** dedicated PostgreSQL running in a local container, independent of client project databases.
- **Small context footprint:** targeted retrieval, not indiscriminate session replay.
- **Safe by default:** no secrets, no automatic promotion of unverified assertions, and no dependence on the memory service for ordinary project builds/tests.

## Status

**Architecture and evaluation phase. No server, database or MCP integration is implemented yet.**

Read [the architecture spike](docs/spikes/persistent-memory.md), [project isolation design](docs/architecture/project-isolation.md), and [initial decisions](docs/adr/).

## Intended flow

Coding agent → MCP memory service → dedicated PostgreSQL database.

The memory service identifies the current project through a verified workspace identity, retrieves only in-scope knowledge and preserves references to authoritative sources. The exact workspace handshake must be validated against real MCP clients before implementation.

## Next steps

Compare a custom .NET MCP implementation with Mem0 and maintained community servers, validate Codex workspace context propagation, and run a bounded proof of concept with measurable retrieval, isolation and failure tests.
