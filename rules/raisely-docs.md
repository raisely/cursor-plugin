---
description: Use Raisely docs for CLI and local campaign work
alwaysApply: true
---

# Raisely local development and CLI

Before answering questions or acting on anything related to the **Raisely CLI**, **local campaigns**, **campaign JSON**, **custom components**, or **editor behaviour**, read the relevant documentation using the Raisely's MCP resources. Treat that as the source of truth for how things work.

## What to read first

- **CLI commands, workflows, local sync:** `raisely-docs://guides/cli`
- **Campaign pages / structure:** `raisely-docs://guides/page-json-format`
- **Custom components overview:** `raisely-docs://guides/introduction`
- **Block registry index:** `raisely-docs://index` and `raisely-docs://guides/components/README`

## Behaviour

- Prefer facts and commands from these resources over guesswork.
- ONLY run `raisely start` when the user ask to preview the changes live in production. For everything else, if the user wants to preview the campaign locally you MUST use `raisely local`
- If something is not covered there, say it is not documented and then ask.
- When suggesting edits to campaigns or components locally, align terminology and file shapes with `raisely-docs://guides/page-json-format`, `raisely-docs://guides/core-blocks`, and the component docs under `raisely-docs://index`.
- When the user asks to work with a **campaign locally** (pull, push, sync, init, open files, verify state, etc.), **run the Raisely CLI and shell steps yourself**; do not reply with a checklist or “run this command” instructions for them unless they explicitly asked for instructions only, or the environment cannot run commands (then say why).
- Only run `raisely start` or `raisely deploy` if the user asks for it.
