# ADR-003: Mandatory project scoping

Status: Proposed

## Context

One memory service must support multiple unrelated repositories without cross-project contamination.

## Decision

Every MVP memory is owned by one stable project identity. The service validates project scope on reads and writes; no default global memories. Branch/task scope may further restrict visibility. Unknown project identity fails closed.

## Consequences

Workspace resolution and project registration are first-class concerns. A client-provided project label is not sufficient proof of identity. Isolation must be tested at the service and query boundaries. This ADR is not an implementation claim.
