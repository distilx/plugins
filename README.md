# DistilX plugins

Plugins that connect Claude and Cursor to [DistilX](https://distilx.ai/): read projects and DistilLang code, evaluate tables, and change projects through the DistilX agent. You sign in with your DistilX account on first use.

## Install

**Claude Code**

```
/plugin marketplace add distilx/plugins
/plugin install distilx@distilx
```

Get new versions with `/plugin marketplace update distilx`, or turn on auto-update for the marketplace in `/plugin`.

**claude.ai, Claude Desktop and Cowork:** an organization owner syncs this repository under Organization settings, Plugins.

**Cursor:** a team admin imports this repository under Dashboard, Settings, Plugins.

## Contents

| Plugin | What it is |
|---|---|
| `distilx` | The DistilX MCP server and skills for working with DistilX projects |

Everything under `plugins/` is published by DistilX CI with each release; changes made here by hand are overwritten.

## License

[MIT](LICENSE)
