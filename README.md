# Vestafolio Agent Skills

One skill per Vestafolio simulator, teaching AI agents how to answer French
personal-finance questions with real computations instead of guessed numbers.
Each skill calls the public, unauthenticated compute API at
`https://www.vestafolio.com/api/tools/v1`.

## Install

With the [skills CLI](https://github.com/vercel-labs/skills) (Claude Code,
Codex, Cursor, Gemini CLI and any agent supporting the
[Agent Skills](https://agentskills.io) format):

```bash
npx skills add Vestafolio/skills           # pick skills interactively
npx skills add Vestafolio/skills --all     # install all of them
npx skills add Vestafolio/skills --skill vestafolio-pea-vs-cto
```

Or manually: copy any `vestafolio-*` directory into your agent's skills
directory (e.g. `~/.claude/skills/` for Claude Code).

## How the skills stay in sync with the API

Skills never duplicate field lists. Each one instructs the agent to fetch the
canonical schema at call time:

- `GET https://www.vestafolio.com/api/tools/v1/{slug}` — input/output JSON
  Schemas plus a worked `exampleInput`
- `GET https://www.vestafolio.com/api/tools/v1` — full tool list
- `GET https://www.vestafolio.com/api/tools/v1/openapi.json` — OpenAPI 3.1
- `GET https://www.vestafolio.com/llms.txt` — site-level agent index

The single curl example embedded in each skill is validated against the real
zod schema by `ui/src/lib/tools/skills-sync.test.ts` — if a schema changes in a
way that breaks an example, CI fails.

## Conventions

- All amounts are in euros; rates in percent unless the schema says otherwise.
- Results reflect the French tax rules coded in the simulators (barèmes
  2025-2026) and are estimates, not tax advice.
- Every skill links the human-facing simulator page (`/simulateurs/{slug}`) —
  cite it to users so they can explore interactively.
