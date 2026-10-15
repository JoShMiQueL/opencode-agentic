# opencode-agentic

A custom **orchestrator + subagents** system for [OpenCode](https://opencode.ai).

A primary orchestrator agent plans the work, delegates implementation to
specialized subagents, and gates delivery on independent review and verification.
The orchestrator never writes code itself — it routes, inspects, and verifies.

## The agents

| Agent | Mode | Role | Key constraints |
| --- | --- | --- | --- |
| `orchestrator` | primary | Plans, delegates, verifies, reports | `edit` denied; shell runs autonomously except a destructive-command denylist (`rm`, `del`, pipe-to-shell, `git push`, etc.) which asks; can only launch the subagents below |
| `coder` | subagent | Implements one well-scoped task at a time | No web access, no skills; can only spawn `explore` |
| `reviewer` | subagent | Reviews diffs for correctness, regressions, conventions, risks | Fully read-only: no edit, no shell, no web, no subagents |
| `researcher` | subagent | Web research, library docs, comparing approaches | No edit, no shell, no subagents (web access kept) |
| `qa` | subagent | Runs builds, typechecks, linters, test suites | No edit — failures go back to `coder` |
| `contextscout` | subagent | Discovers which `.opencode/context/` files are relevant to a task | Read-only: no edit, no shell, no web, no subagents. Returns ranked file paths, never contents |
| `explore` | subagent (built-in) | Fast codebase search and "where is X" | Built into OpenCode |
| `general` | subagent (built-in) | Escape hatch for anything else | Built into OpenCode |

Typical pipeline: `explore`/`researcher` → `contextscout` → `coder` → `reviewer` → `qa` → report.

## How the permission model works

Each agent's frontmatter carries an ordered `permissions:` list of rules:

```yaml
permissions:
  - action: subagent
    resource: "*"
    effect: deny
  - action: subagent
    resource: coder
    effect: allow
```

- Each rule is `{action, resource, effect}` where `action` is a tool class
  (`edit`, `shell`, `webfetch`, `websearch`, `skill`, `subagent`), `resource` is a
  glob matched against the tool's input (e.g. `"git diff *"` for shell commands,
  `coder` for a subagent name), and `effect` is `allow`, `deny`, or `ask`
  (prompts the user).
- **Last match wins** — rules are evaluated top to bottom, so put broad `deny`s
  first and specific `allow`s after them (or vice versa when locking down).
- **`subagent` gating** controls which agents an agent may launch. The
  orchestrator `deny`s `subagent: *` then `allow`s each team member by name, so
  it can never spawn an unlisted agent — and subagents deny `subagent: *`
  entirely, so delegation cannot chain.

## v2 changes

- **Locked orchestrator** — `edit: *` denied, so it can never author code;
  all changes go through `coder`.
- **Shell denylist** — shell runs autonomously (`allow`), EXCEPT a short
  denylist of destructive commands (`rm`, `rmdir`, `del`, pipe-to-shell,
  `format`, `shutdown`, `git push`/`reset --hard`/`clean -f`, etc.) which ask
  for approval.
- **Delivery gate** — code changes are not reported as done until `reviewer`
  has reviewed them and `qa` has verified them.
- **Cheaper researcher** — downgraded to `swe-2-medium`; research doesn't need
  the top-tier model.
- **coder → explore** — the coder can spawn `explore` to locate things without
  burning its own context on file hunts.
- **Context template** — `context/` documents the `.opencode/context/` pattern:
  small committed markdown files (commands, conventions, architecture) the
  orchestrator loads to stay lean.
- **Self-provisioning context** — `contextscout` reads only the project's
  `.opencode/context/navigation.md`. When a project has none, the orchestrator
  bootstraps once: `coder` copies `~/.config/opencode/context-templates/` into
  `.opencode/context/` and adapts it to the project, then the scout re-runs.
  The global template store is read-only seeds — never live context.
  `context/templates/` is the distribution copy of that store.
- **AGENTS.md vs context** — `AGENTS.md` at the project root is always
  auto-loaded (hard rules); `.opencode/context/` holds on-demand detail the
  scout routes to.

## MCP servers

Custom MCP servers (**rea** — reverse engineering, **chrome-devtools** — browser
automation) are configured under `mcp.servers` in `opencode.json(c)`, not in
`agents/*.md`. They can be loaded globally (`~/.config/opencode/opencode.json`)
or per-project (`<root>/opencode.json`). Details and the loading procedure are
documented in `context/templates/mcp.md`.

## Install

Copy `agents/` into `~/.config/opencode/agents/`:

```bash
cp -r agents/* ~/.config/opencode/agents/
```

(Optional) copy `context/` files into a project's `.opencode/context/` and fill
them in. See `context/README.md`.

Then set the orchestrator as the default agent — see `opencode.example.json`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "your-provider/your-strong-model",
  "default_agent": "orchestrator"
}
```

## Adapting to another provider

The `model:` fields in each agent's frontmatter reference Devin's SWE-2 family
(`devin/swe-2-max`, `devin/swe-2-high`, `devin/swe-2-medium`). To run on a
different provider, replace them with your provider's model IDs — keep the
tiering intent:

| Agent | Tier intent |
| --- | --- |
| `orchestrator` | strongest model (planning + routing quality) |
| `coder`, `reviewer` | strong model (implementation/judgment) |
| `researcher`, `qa` | cheaper/faster model (mechanical work) |

Also update the "Implementation agent powered by SWE-2" line in
`agents/coder.md` if you rebrand.
