---
name: raisely-start-local
description: Start a local environment of your Raisely Campaign to preview changes
---

# Start local Raisely server

## Considerations

- The default port is 8015. Different ports can be used with the `--port` argument when running `raisely local`
- Do not update the campaign with `raisely update` unless explicitly requested by the user
- If you have access to an embedded browser, use the argument `--no-open` and open the campaign in the embedded browser. Otherwise, do not use `--no-open` and let the command open the URL in the default browser.
- When debug mode is enabled, state the skill name and every command with arguments before running them.
- If you used the embedded browser, after opening the URL, confirm the page is reachable and the campaign slug matches the requested campaign.
- Never assume or guess which campaign to start. The campaign must be confirmed by the user in the current session. If `.raisely.json` exists, you may propose its UUID, but you must show the campaign (name and UUID) and get explicit confirmation before using it. Otherwise, ask the user to pick from `raisely list --json`. Do not infer the campaign from prior chats, recent files, or other context without confirmation.

## Steps

1. Verify `raisely` is available using the `raisely-ensure-cli` skill
2. Verify if port 8015 is free. If not, keep incrementing the port by one (8016, 8017, 8018, 8019...) until you find one available. Do not stop existing process unless the user ask for it
3. Confirm the campaign the user wants. If `.raisely.json` exists, resolve its UUID to a name with `raisely list --json` and ask the user to confirm before using it. If `.raisely.json` is missing or the user rejects it, ask them to pick from `raisely list --json`.
4. Once the user has picked, use `raisely list --json` to resolve the UUID. If `.raisely.json` is missing or points to a different UUID, run `raisely init --uuid <uuid>` with the confirmed campaign's UUID
5. Run `raisely local` as a background task. Make sure to pass the right parameters based on the current task
6. If the process ends due to an error, notify the user and try to start it again following the previous steps
