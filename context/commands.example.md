# Commands

Copy this file to `.opencode/context/commands.md` in your project and fill it in.

| Task | Command | Notes |
| --- | --- | --- |
| Install deps | `npm install` | or `bun install` |
| Build | `npm run build` | outputs to `dist/` |
| Typecheck | `tsc --noEmit` | |
| Lint | `npm run lint` | eslint + prettier check |
| Unit tests | `npm test` | vitest / jest |
| Single test | `npm test -- <path>` | |
| Dev server | `npm run dev` | port 3000 |
| Full verify | `npm run build && npm test` | run before reporting done |
