---
name: raisely-ensure-cli
description: Ensures the Raisely CLI (`@raisely/cli`) is installed before running any Raisely campaign work. Checks for the `raisely` binary and installs the latest version via npm if missing, or installs a user-specified version when provided. Use this before any task that runs `raisely init`, `raisely update`, `raisely deploy`, `raisely create`, `raisely local`, `raisely start`, `raisely list`, or any other `raisely` command, or before pulling, pushing, syncing, deploying, previewing, or scaffolding a Raisely campaign or custom component locally.
---

# Ensure Raisely CLI

Run this skill at the very start of any task that touches a Raisely campaign locally, before invoking the `raisely` binary or guiding the user through one of its commands.

## When to run

Trigger this skill before any of the following:

- Running `raisely <anything>` (`init`, `update`, `deploy`, `create`, `local`, `start`, `list`, `login`, `logout`).
- Pulling, pushing, syncing, deploying, or previewing a Raisely campaign locally.
- Scaffolding or editing custom components when the next step needs the CLI.
- Writing instructions that ask the user to run a `raisely` command.

Skip this skill only when the task is purely about Raisely API calls or MCP tools and never shells out to `raisely`.

## Steps

### 1. Check if the CLI is already installed

Run:

```bash
raisely --version
```

If it prints a version, you're done. Continue with the user's task.

If the command fails with `command not found` (or exits non-zero because the binary is missing), continue to step 2.

### 2. Check the requested version

Default to the **latest** version on npm.

Use a different version only when the user explicitly asked for one (for example "install raisely cli 2.3.1" or "use @raisely/cli@2.3.1"). When unsure, ask the user once before installing a non-latest version.

### 3. Verify Node and npm are available

Run:

```bash
node --version && npm --version
```

If either is missing, stop and tell the user the Raisely CLI needs Node.js (v13+) and npm before it can be installed. Do not try to install Node yourself.

### 4. Install the CLI

For the latest version:

```bash
npm install -g @raisely/cli
```

For a specific version requested by the user:

```bash
npm install -g @raisely/cli@<version>
```

If the install fails with `EACCES` or another permission error, retry with `sudo`:

```bash
sudo npm install -g @raisely/cli
```

Ask the user before running anything with `sudo`.

### 5. Confirm the install

Run `raisely --version` again and confirm it prints a version. If it still fails, surface the install output to the user and stop.

### 6. Continue with the original task

Once the CLI is available, return to the work that triggered this skill (`raisely local`, `raisely deploy --force`, etc.). Do not re-run the check on every subsequent command in the same session; one successful check per session is enough.

## Notes

- The npm package is `@raisely/cli`; the binary is `raisely`.
- Always tell the user when you are running this skill
- Authentication (`raisely login`) is a separate concern handled by individual commands. This skill does **not** run `raisely login`; let the command that needs auth trigger the OAuth flow.
- Do not modify global npm config or Node version manager state. If the user is on `nvm`, install into the currently active Node version.
