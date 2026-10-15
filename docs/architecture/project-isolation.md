# Project identity and isolation

> Status: Proposed design; implementation and client behaviour unverified.

## Requirement

The service must work with any source project without requiring project-specific source changes. Memories must not leak between unrelated projects, even when names, local paths or branches coincide.

## Identity model

- Give each project a stable internal UUID and an independently verified external identity.
- For hosted Git repositories, normalise the remote's host and repository identity; do not use basename alone.
- Handle remote URL aliases, remote changes and forks deliberately; a fork must not silently inherit another project's private memory.
- For local repositories without remotes, create a persistent project registration identifier outside the project's tracked source. A directory rename must not change identity.
- Different worktrees of the same repository should resolve to one project, while temporary branch/task memories retain separate scope.
- An explicit user-approved rebind or migration must be available for remote renames or identity changes.
- Treat global/shared memory as out of scope for the MVP.

## Client-to-service workspace handshake

MCP does not guarantee that the server sees the client's active workspace. Investigate Codex's supported MCP configuration and runtime context; avoid deriving project identity from the MCP process's own working directory unless proven safe.

A proposed handshake is: client supplies workspace information; server resolves Git metadata or a registered local identity; server returns a short-lived scoped session handle. Subsequent read/write calls use that handle, which the service validates. Do not trust arbitrary caller-provided project IDs as authorisation. If the workspace cannot be resolved confidently, fail closed and request explicit project selection/registration.

For multi-user or remotely exposed deployments, add authenticated caller identity and per-project authorisation. A local single-user stdio MVP still requires scope validation, though it is not a strong boundary against a malicious agent running with the user's OS privileges.

## Branch and worktree rules

Verified, branch-independent facts can be project-wide. Provisional findings may be tied to branch and commit; they must not be promoted across branches without verification. Worktrees share durable project identity, not transient working state.

## Isolation tests

1. Same-name repositories with distinct remotes never share results.
2. A memory from project A is not returned to project B, including when B passes A's identifier.
3. Two worktrees of the same repository share verified durable memories.
4. A branch-specific candidate is excluded from unrelated branches by default.
5. Moving a registered local repository preserves identity.
6. An unknown or ambiguous workspace fails closed.
7. Repository rename/fork/rebinding scenarios do not silently cross boundaries.

## Open validation

Determine the concrete workspace handshake that Codex and other MCP clients can support. Do not claim automatic switching works before testing it.
