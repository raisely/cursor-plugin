# Raisely Cursor plugin

Connect [Cursor](https://cursor.com) to [Raisely](https://raisely.com). This plugin ships the Raisely MCP server so the Cursor agent can query campaigns, donations, supporters, and the rest of your Raisely data straight from chat.

## What's inside

- `mcp.json`: registers the hosted Raisely MCP server at `https://mcp.raisely.com/mcp`.
- `.cursor-plugin/plugin.json`: plugin manifest.

The MCP server uses OAuth, so the first time you use it Cursor will open a browser tab to sign you in to Raisely.

## Install

### From the Cursor Marketplace

Search for `Raisely` in the Cursor marketplace panel and click Install. Cursor will register the MCP server automatically.

### From source (local development)

Symlink this repo into Cursor's local plugins folder:

```bash
mkdir -p ~/.cursor/plugins/local
ln -s "$(pwd)" ~/.cursor/plugins/local/raisely
```

Then reload Cursor (`Developer: Reload Window`) and confirm the `Raisely` MCP server shows up under `Settings -> Features -> Model Context Protocol`.

## Use it

Once connected, ask the agent things like:

- "List my active campaigns in Raisely."
- "Show the top 10 donations to the Spring Appeal this week."
- "Find supporters who signed up in the last 30 days but haven't donated."

The agent will pick the right Raisely MCP tool and walk you through the OAuth prompt the first time.

## Contributing

This plugin will grow to include Raisely-specific rules, skills, and agents. PRs welcome at [github.com/raisely/cursor-plugin](https://github.com/raisely/cursor-plugin).

## License

MIT