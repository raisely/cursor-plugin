---
description: Hard safety rules for any Raisely campaign, payment, or publish operation
alwaysApply: true
---

# Raisely guardrails

These rules are non-negotiable. Follow them on every Raisely-related action, even if the user's request seems to imply otherwise. If a request conflicts with a guardrail, stop and ask before proceeding.

## 1. Never delete or unpublish without double confirmation

Destructive actions include: deleting a campaign, page, profile, donation, user, event, custom component, or any record; unpublishing a campaign or page; running `raisely deploy` with deletions; clearing data; or any CLI/API call with `delete`, `archive`, or `unpublish` semantics.

- State exactly what will be deleted or unpublished, and where (campaign UUID/path, environment).
- Ask the user to confirm twice: once on intent ("you want to delete X, correct?"), and a second time immediately before executing ("confirming: I will now delete X in <env>, proceed?").
- Never batch-delete across multiple records in a single confirmation. List them and confirm the full set.

## 2. Never switch TEST and LIVE without explicit confirmation

This applies to anything that changes payment mode, environment, or campaign mode flags: TEST -> LIVE, LIVE -> TEST, sandbox -> production, Stripe test keys -> live keys, mock mode toggles, etc.

- Detect the current mode before suggesting a change. State it back.
- Require an explicit "yes, switch from TEST to LIVE" (or vice versa) from the user. A general "deploy" or "publish" is not enough.
- Warn that LIVE mode means real donor charges and real money movement.

## 3. Never modify payment or Stripe config

Off-limits without an explicit, specific request from the user naming the exact change:

- Stripe account IDs, API keys, webhook secrets, publishable keys.
- Payment processor settings (Stripe, PayPal, Aplos Pay, Apple Pay, Google Pay).
- Currency, fee coverage, recurring billing, payout, or refund configuration.
- Donation amount presets, minimum/maximum amounts, processing fee logic.

If the user asks for a payment-related change, restate the exact field and value, confirm the environment (TEST vs LIVE), and only then apply. Never guess defaults.

## 4. Always validate before upload or deploy

Before `raisely deploy`, `raisely push`, any API write, or any file upload:

- Run the relevant validator (CLI validate command, JSON schema check, lint, type check) and report the result.
- Diff local vs remote and show the user what will change.
- If validation fails, stop. Do not "fix and push" silently; surface the errors and ask.
- Never deploy with unsaved or untracked secrets, half-edited JSON, or files the user did not explicitly include.

## 5. Always summarize changes before applying

Before any write, deploy, or destructive action:

- List the target (campaign, page, component, record) and the environment.
- Show a concise diff or bullet summary of what will change.
- State the command(s) you will run.
- Wait for the user to acknowledge before executing.

This applies to single-file edits, bulk JSON changes, CLI deploys, and API mutations alike. "Summarize, then act" is the default; silent execution is never acceptable for shared or production data.

## When in doubt

Stop and ask. A blocked action with a clear question is always better than an irreversible mistake on a live campaign.
