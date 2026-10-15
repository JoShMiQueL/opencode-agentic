<!-- Context: setup/mcp | Priority: high -->

# MCP servers

External tool servers (Model Context Protocol) that extend what agents can do. Configured under `mcp.servers` in `opencode.json(c)` — **not** in `agents/*.md`.

## Where they live

| Scope | File | When |
| --- | --- | --- |
| Every project | `~/.config/opencode/opencode.json` → `mcp.servers` | personal/global tools |
| This project | `<root>/opencode.json` or `<root>/.opencode/opencode.json` → `mcp.servers` | project-specific tools |

Global (user) config applies everywhere; project config overrides a server of the same name.

## The custom servers in this setup

- **rea** — reverse-engineering tools (native binaries, managed code, Electron, firmware, browser). Local stdio server: `node <rea-agents>/scripts/rea.mjs mcp` with `GHIDRA_INSTALL_DIR` pointing at Ghidra.
- **chrome-devtools** — browser automation/devtools. Local stdio: `npx -y chrome-devtools-mcp@latest`.

## Loading an MCP server that isn't configured

If a project needs an MCP server that isn't already configured:

1. **Check first** — `opencode mcp list` shows configured servers + connection state. If it shows `needs authentication`, run `/mcps` in the UI and sign in (don't run `mcp auth` via shell).
2. **Local stdio server** — add to `mcp.servers` with `type:"local"`, `command:[...]`, optional `environment`/`cwd`.
3. **Remote HTTP server** — `opencode mcp add <name> --url <url>` (project) or `--global` (everywhere).
4. **Secrets** — use `{env:NAME}` substitution, never hardcode keys. e.g. `"environment": {"GHIDRA_INSTALL_DIR": "{env:GHIDRA_INSTALL_DIR}"}`.
5. **This project's deps** — `rea` needs a local `rea-agents` checkout + `GHIDRA_INSTALL_DIR` env. If missing, the server won't start — flag it as a blocker, don't invent the path.
