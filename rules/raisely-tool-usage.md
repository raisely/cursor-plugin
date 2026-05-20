---
description: When to use the Raisely MCP vs the Raisely CLI, preferred sequences, and read-only vs write tools
alwaysApply: true
---

# Raisely tool usage

Raisely work happens through three surfaces: the **Raisely MCP** (docs and read-only data with write operations as an opt-in configuration), the **Raisely CLI** (`raisely ...`, campaign edits and deploys), and **local file edits** in the campaign directory. Pick the right one for the job.

## Default rule

- **Reading data, docs, or campaign metadata** -> use the MCP.
- **Making changes beyond editing components or pages** -> use the MCP with the right configuration.
- **Editing campaigns, components, or pages** -> use the CLI plus local file edits.
- **Previewing changes** -> use `raisely local` (never `raisely start` unless the user explicitly asks to preview against production).

If a task can be done read-only first, prefer the MCP and do that before any write.

## Read-only (MCP and safe CLI)

Use these freely without confirmation.

| Tool                                          | Use for                                                             |
| --------------------------------------------- | ------------------------------------------------------------------- |
| Raisely MCP resources (`raisely-docs://...`)  | CLI reference, page JSON format, core blocks, custom component docs |
| `raisely list --json`                         | Find a campaign UUID, list campaigns the user has access to         |
| `raisely list-components` (when available)    | Inspect components in a campaign                                    |
| `git status` / `git diff` in the campaign dir | See what changed locally before any push                            |

## Write (MCP)

The Raisely MCP is **read-only by default**. Connecting to `/mcp` exposes only `GET`-style tools; no write tool is registered no matter what you send. To use MCP write tools (e.g. `create_campaign`, `update_campaign`), the MCP session must opt in at connection time.

**Three ways to opt in (set on the MCP server URL the client connects to):**

| Connection                                | Effect                                                                                  |
| ----------------------------------------- | --------------------------------------------------------------------------------------- |
| `/mcp`                                    | Read-only tools across all resources. Safe default.                                     |
| `/mcp/{resource}` (e.g. `/mcp/campaigns`) | All read tools + write tools for that one resource.                                     |
| `/mcp/all`                                | All tools across all resources, read + write. **Use only with explicit user approval.** |

**Header-based opt-in** (only when connecting to bare `/mcp`, not `/mcp/{resource}` or `/mcp/all`):

```
X-Raisely-Write: campaigns,tags
```

This grants write access to the listed resources while leaving everything else read-only. If a path segment is present, the header is ignored.

**Rules for the AI:**

- **Never silently change the connection URL** to gain write access. Treat the current MCP session's scope as fixed.
- If a task needs an MCP write tool and the current session is read-only, stop and ask the user to reconnect with the right path or `X-Raisely-Write` header. State exactly which scope is needed and why.
- Permissions are baked in at session `initialize`; you cannot escalate by resending headers mid-session.
- Prefer the narrowest scope that works. `/mcp/campaigns` over `/mcp/all`. Multiple resources via the header over `/mcp/all`.
- Treat `/mcp/all` as the equivalent of running CLI write commands without confirmation: it requires the same explicit approval as a `raisely deploy`.
- Unknown resources are silently ignored (path returns no write tools, header entry is dropped); if a write tool you expect is missing, double-check the resource name.

Once write tools are available in the session, the same confirmation rules from `raisely-guardrails.md` apply: summarize the change, confirm the environment (TEST vs LIVE), and never delete or unpublish without double confirmation.

## Write / destructive (CLI)

These change remote state. Always summarize first and follow `raisely-guardrails.md`.

| Tool                                         | What it does                                                                      | Confirmation needed                                             |
| -------------------------------------------- | --------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `raisely init --uuid <uuid>`                 | Pulls a campaign locally; overwrites local files for that campaign                | Confirm if a different campaign already exists in the directory |
| `raisely local`                              | Starts the local preview server. Read-only against the live API for content; safe | None                                                            |
| `raisely update`                             | Pulls latest remote campaign state into local files                               | Confirm; will overwrite uncommitted local edits                 |
| `raisely push` / `raisely deploy`            | Uploads local changes to the remote campaign                                      | Always summarize diff + confirm; validate first                 |
| `raisely create ...`                         | Scaffolds a new campaign or component                                             | Confirm name, target org, environment                           |
| `raisely start`                              | Runs against production data with live editor session                             | Only on explicit user request                                   |
| Any `delete`, `archive`, or `unpublish` flag | Destructive                                                                       | Double confirmation per `raisely-guardrails.md`                 |

## Preferred sequences

### Answer a question about a campaign

1. Read the relevant MCP doc (`raisely-docs://guides/page-json-format`, etc.).
2. If you need live data, use the Raisely MCP tools or inspect local campaign files.
3. Answer; do not run any write command.

### Edit a campaign locally

1. `raisely list --json` -> identify the UUID (skip if `.raisely.json` already matches).
2. `raisely init --uuid <uuid>` if not already pulled.
3. Start `raisely local` as a background task (see `raisely-start-local` skill).
4. Edit files locally. Validate JSON / components.
5. Verify in the local preview (screenshot).
6. Summarize the diff for the user. Wait for approval.
7. `raisely deploy` only after explicit approval.

### Add or modify a custom component

1. Read `raisely-docs://guides/introduction` and the relevant component docs.
2. Edit the component in the campaign repo.
3. Reload `raisely local` and verify in the preview.
4. Summarize the change, then deploy on approval.

### Switch which campaign you are working on

1. Confirm with the user before overwriting the current local campaign.
2. `raisely init --uuid <new-uuid>` in a clean directory or after committing/stashing local changes.

## Preview commands

- **`raisely local`** (default): local preview server on port 8015 (auto-increment if taken). Use this for every "show me the change" request. Run as a background task.
- **`raisely start`**: live preview against production. **Only run when the user explicitly asks to preview against production.**
- **Verification**: after starting either, confirm the URL is reachable and the campaign slug matches what the user asked for. Take a screenshot before reporting back.

## Don'ts

- Don't call write commands to "check" something; use the MCP or `raisely list` instead.
- Don't run `raisely deploy`, `raisely update`, or `raisely start` without explicit approval.
- Don't mix tools when one will do; for example, don't shell out to the API when the CLI has a command, and don't edit JSON by hand when a CLI scaffolder exists.
- Don't guess MCP tool arguments; read the tool descriptor under `mcps/<server>/tools/` first.
