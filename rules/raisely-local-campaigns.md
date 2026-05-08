---
description: Steps for working with Raisely campaigns locally
alwaysApply: true
---

# Working with Raisely campaigns locally

Follow these steps whenever the user wants to work on a Raisely campaign locally (preview, edit, iterate). Run the commands yourself; don't hand the user a checklist unless they ask for one.

## Steps

1. **Pick a campaign.** If the user didn't name one, run `raisely list --json` and ask them to pick from the results. Don't guess.
2. **Download the campaign.** If `.raisely.json` is missing or points to a different UUID, run `raisely init --uuid <uuid>` to pull it down.
3. **Start the local preview.** Use the `raisely-start-local` skill to run `raisely local` as a background task.
4. **Apply requested changes.** When the user asks for changes, edit the campaign files, then take a screenshot from the local preview to confirm the result before reporting back. Make sure to scroll to have the changes in the browser viewport
5. **Keep the preview alive.** If `raisely local` exits with an error or the process dies, restart it by repeating step 3. The local server must stay available for the whole session.
6. **Surface blockers.** If you can't keep the preview running (port conflicts you can't resolve, repeated crashes with the same error, missing auth, etc.), stop and explain the exact reason to the user.

## Notes

- For CLI install, port selection, and `--no-open` rules, defer to the `raisely-ensure-cli` and `raisely-start-local` skills.
- For docs on campaign JSON, components, and CLI behaviour, follow `raisely-docs.md`.
