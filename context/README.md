# context/ — optional project context files

This directory is a template for the `.opencode/context/` pattern.

## What it is

In each project that uses this agent system, create a `.opencode/context/` directory
containing small (**< 200 lines each**) committed markdown files that describe the
project's:

- **Commands** — build, test, lint, typecheck, dev-server commands
- **Conventions** — naming, file layout, commit style, formatting rules
- **Architecture** — module boundaries, data flow, where things live

## Why

The orchestrator (or a scout like `explore`) can load one short context file instead
of re-discovering the same facts every session. It keeps prompts lean and prevents
agents from guessing commands or conventions.

## How to use

1. Copy the `*.example.md` files into your project's `.opencode/context/` directory.
2. Rename them (drop the `.example` suffix) and fill in your project's real values.
3. Commit them — context is part of the project, so it stays in sync with the code.
4. Point the orchestrator at them in your task prompt or `opencode.json`
   (e.g. "read `.opencode/context/commands.md` first").

Keep files under ~200 lines. If a file grows past that, split it — context that
costs too many tokens defeats the purpose.

## navigation.md — the context bridge

Each context directory should contain a `navigation.md` index file. It lists a
**Quick Routes** table mapping task types to context files and priorities:

```markdown
## Quick Routes

| Task type | File | Priority |
| --- | --- | --- |
| Any code change | `conventions.md` | critical |
| Verifying / running builds, tests, lint | `commands.md` | critical |
```

The `contextscout` subagent uses `navigation.md` to find relevant context
without hardcoding file names. Resolution order (project → global fallback):

1. `.opencode/context/navigation.md` in the current workspace — project context
   takes precedence.
2. `~/.config/opencode/context/navigation.md` — shared global standards (e.g.
   `code-quality.md`) that apply when a project has no navigation file.
3. Neither → scout reports "No context found".

The scout returns ranked **paths only** (never contents). The orchestrator reads
the files itself or forwards the paths in the delegation prompt as
`Context to load: <paths>`. A `navigation.md` may also link to deeper
`navigation.md` files for subdirectories with their own conventions.
