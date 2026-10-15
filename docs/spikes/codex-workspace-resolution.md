# Spike: Codex workspace resolution

> Status: Experimental plan — not yet executed
> Branch: `spike/workspace-resolution`
> Scope: Project identity handshake for a globally configured MCP memory service

## Objective

Determine how an agent-independent MCP server can safely identify the **active client project**, without requiring a file or custom MCP configuration in every consumer repository. Codex CLI and the Codex VS Code integration are the initial clients. Do not assume that MCP servers automatically receive a client workspace path.

## Questions to resolve

1. Does Codex launch a globally configured stdio MCP server with a predictable working directory? Does this change between CLI and VS Code, between workspaces, or across sessions?
2. What explicit workspace information, if any, does Codex supply to the server via MCP initialization, roots, resource metadata or tool calls? Is it stable and supported?
3. Can a client explicitly pass the active workspace to a tool, and what does the server have to verify independently?
4. How does a long-lived server behave if the user changes workspaces? Is a per-session scoped handle practical?
5. What fallback is viable when no trustworthy workspace context exists?

## Hypotheses (not verified)

- **H1: Process cwd.** Stdio launch cwd may correspond to the active repository. This is convenient but may reflect a global configuration directory, IDE process, or shared server; it is not an identity guarantee.
- **H2: MCP roots.** A client may expose workspace roots through MCP protocol capabilities. Actual Codex support and update semantics must be observed, not inferred from protocol support.
- **H3: Explicit workspace parameter.** A client may provide a path in a memory tool invocation. This is not proof of identity; the service must canonicalise and validate it against the active session and allowed roots.
- **H4: Explicit registration.** If automatic resolution cannot be established, an out-of-repository registration/selection step can safely fail closed without project-local files.

## Experiment setup

Use a temporary diagnostic MCP server that logs **metadata only**: process ID, cwd, MCP initialization capabilities, advertised roots if supported, session lifecycle, tool parameters and timestamps. Do not log repository content, credentials or environment variables. Keep diagnostics out of the permanent memory store.

Test with at least:
- Repository A: `ag-mem-on`.
- Repository B: a distinct repository with a different remote.
- Repository C: a different remote with the same basename as A, if feasible.
- Two worktrees of A on different branches.
- A local Git repository without a remote.
- A non-Git workspace.

Run each applicable case from Codex CLI and VS Code. Test startup, repeated calls, switching repositories, opening multiple workspaces and reconnecting the MCP server. Record client versions, OS, MCP configuration placement and transport, and whether the server is reused.

## Observations to capture

| Case | CLI cwd | IDE cwd | Roots available | Explicit path | Resolved identity | Outcome |
| --- | --- | --- | --- | --- | --- | --- |
| Repo A | Pending | Pending | Pending | Pending | Pending | Not run |
| Repo B | Pending | Pending | Pending | Pending | Pending | Not run |
| Same-name repo | Pending | Pending | Pending | Pending | Pending | Not run |
| Worktree A2 | Pending | Pending | Pending | Pending | Pending | Not run |
| Local/no remote | Pending | Pending | Pending | Pending | Pending | Not run |
| Non-Git | Pending | Pending | Pending | Pending | Pending | Not run |

**Evidence standard:** distinguish official documented capabilities from observed client behaviour. For each result, record a reproducible command/configuration, redacted diagnostic output, client version and conclusion. A successful call is not by itself evidence of secure scoping.

## Proposed identity algorithm for evaluation

1. Establish the client's workspace context through a tested, documented mechanism.
2. Canonicalise the workspace path, resolve symlinks and identify the Git worktree/repository root.
3. Resolve repository identity using canonical remote host and repository identity; distinguish forks and same-name repositories.
4. Map identity to a persistent internal project UUID; use an explicit external registration for local-only repositories.
5. Bind the resolved project to the client session or a validated scoped handle; never accept an arbitrary project UUID as authority.
6. Refuse memory reads/writes when context is missing, conflicting or ambiguous. No implicit fallback to the last project.

For a single-user local stdio server, this provides **accidental cross-project isolation**, not a hardened security boundary against a malicious agent running as the same OS user. Remote or multi-user use requires authentication and authorisation.

## Acceptance criteria

- **CLI and IDE:** demonstrate the supported resolution mechanism in each; if one fails, record a distinct fallback and do not claim parity.
- **Project switching:** no stale scope when moving from A to B.
- **Same-name repositories:** no identity collision.
- **Worktrees:** stable project identity with distinct branch context.
- **No remote:** stable identity after moving the local directory, via explicit registration.
- **Failure mode:** unknown, conflicting or unverifiable workspace fails closed.
- **Zero project changes:** no tracked files or settings added to consumer repositories.
- **Repeatability:** findings include versioned configuration and repeatable evidence.

## Decision gate

After the experiment, recommend one of:
1. Verified client-provided roots / supported workspace metadata.
2. Verified explicit workspace argument plus server-side scope checks.
3. Explicit local project registration and selection as the safe MVP fallback.

Do **not** select cwd-only discovery without evidence that it remains correct for all supported Codex entry points and concurrent sessions. Do not implement the durable memory schema until the project-scope handshake has a documented contract.

## Out of scope

PostgreSQL deployment, memory CRUD, embedding/vector search, automatic memory capture, cross-agent parity testing, and production MCP hosting.

## Proposed next step after approval

Build the smallest diagnostic MCP executable using the official .NET MCP SDK; test CLI and VS Code against the matrix above. Keep the probe disposable and free of persistence. Update this spike with observations and an ADR recommendation before building the actual memory service.
