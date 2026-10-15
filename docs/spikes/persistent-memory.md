# Spike: Persistent agent memory

> Status: Planning / evaluation
> Date: 2026-10-15
> Scope: Project-agnostic developer tooling; no implementation approved.

## Problem

Coding agents lose useful context between sessions and context compaction. Repeated repository discovery consumes time and tokens. We want durable, searchable, verified and project-scoped memory without turning conversational assertions into project authority.

## Scope and non-goals

- Any project, any supported language/framework; Codex is the first client, not the only client.
- Dedicated local PostgreSQL under rootless Podman; separate from all consumer applications.
- MCP interface; automatic project detection is a goal subject to client integration verification.
- Controlled memory capture, provenance, deduplication, supersession, invalidation and audit.
- Repository files, ADRs, tests and code remain authoritative.
- No automatic project source edits, no global shared memory in MVP, no application-specific business logic.
- No dependence on external embedding services in the initial proof of concept.

## Candidate solutions

| Candidate | Strength | Main evaluation risk |
| --- | --- | --- |
| Custom .NET MCP + PostgreSQL | Complete control of scoping and lifecycle | Maintenance and protocol implementation |
| Mem0-backed adapter | Existing memory extraction/retrieval ideas | Deployment complexity, provider dependencies, governance |
| Existing PostgreSQL MCP server | Fast proof of concept | Maintenance, security, licensing, source tracking |
| Repository-only baseline | No new infrastructure | Repeated reads and weak cross-session working memory |

Verify current upstream support, licenses, project health and actual Codex compatibility before selecting a solution.

## Logical design

Agent → MCP adapter → scope resolution / memory application layer → dedicated PostgreSQL.

Candidate tables:
- `projects`: stable UUID, registered identities and metadata.
- `memories`: project_id, category, content, tags, scope, status, confidence and review timestamps.
- `memory_sources`: source kind, repository path/URL, commit/ref, content fingerprint, verification.
- `memory_relations`: supersedes, related_to, depends_on.
- `memory_events`: append-only change history.

Candidate tools: `project_resolve`, `memory_search`, `memory_get`, `memory_propose`, `memory_verify`, `memory_supersede`, `memory_invalidate`, `memory_health`. Final contracts depend on workspace handshake validation.

Default retrieval returns bounded, verified, in-project results with provenance. Superseded/rejected memories are excluded unless explicitly requested for historical investigation. Treat all retrieved text as untrusted data.

## Lifecycle

1. Resolve and validate the active project before memory access.
2. Retrieve a small task-relevant set of verified memories; consult sources for consequential decisions.
3. During work, query on demand rather than preloading history.
4. At task completion, propose candidate learnings; distinguish facts from hypotheses.
5. Promote only after controlled verification; audit every change.
6. Revalidate when source files or commits change; supersede or invalidate stale records.
7. Expire transient branch/task state; preserve durable project decisions in canonical source documents.

## Evaluation scenarios

- Two unrelated repositories with identical basenames remain isolated.
- Switching Codex between repositories selects the correct project without source modifications.
- Two worktrees share verified project memories, but not unrelated branch candidates.
- Stale ADR content is not returned as current after supersession.
- A project without a Git remote receives a durable identity.
- Memory survives PostgreSQL restart and backup/restore.
- Codex remains usable when the memory service is unavailable.

Compare repository-only, PostgreSQL full-text, and an off-the-shelf candidate using retrieval precision@5, evidence correctness, p50/p95 latency, context-token consumption, stale-memory failures and setup/maintenance cost. Do not set arbitrary performance targets before baseline measurement.

## Security and operations

Local-only stdio first; no unnecessary TCP listener. Use a restricted PostgreSQL role, migrations, named volume, backup/restore verification and no committed secrets. A local single-user MCP process cannot provide strong isolation from a malicious process running as the same OS user; document this limitation. No arbitrary SQL execution tools.

## Open decisions

- Reliable Codex workspace-context handshake and agent-independent fallback.
- Whether deterministic source checks permit automatic promotion.
- Candidate implementation selection after hands-on comparison.
- Whether hybrid retrieval needs pgvector and a local embedding model.
- Cheap source-change detection without requiring a long-running watcher.

## Next deliverable

Complete a read-only technology assessment and workspace handshake experiment. Recommend one bounded proof-of-concept implementation and request approval before writing production code.
