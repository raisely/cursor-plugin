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

## Steps

1. Verify `raisely` is available by running the skill `/ensure-raisely-cli`
2. Verify if port 8015 is free. If not, keep incrementing the port by one (8016, 8017, 8018, 8019...) until you find one available. Do not stop existing process unless the user ask for it
3. Use `raisely list --json` to find the uuid of the desired campaign. If there is no .raisely.json or the uuid doesn't exist there, then run `raisely init --uuid` with the desired campaign's UUID
4. Run `raisely local` as a background task. Make sure to pass the right parameters based on the current task
5. If the process ends due to an error, notify the user and try to start it again following the previous steps
