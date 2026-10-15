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
