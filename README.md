🇫🇷 [Version française](README.fr.md)

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

The single curl example embedded in each skill is validated in CI against the
real zod schemas of the simulators — if a schema changes in a way that breaks
an example, the build fails before the skill is published.

## Conventions

- **Ask before computing.** Each skill carries a "Questions to ask before
  calling the API" checklist that mirrors the questions the web simulator asks,
  in the same order and with the same conditions (e.g. the micro-entreprise
  simulator asks whether the business benefits from ACRE, and only in its
  first year, before any computation). Agents must collect or confirm those
  answers instead of silently assuming defaults. CI checks that every input
  field of a simulator's schema is covered by its skill.
- **Rates come from the simulator code, nowhere else.** The tax rules and
  constants quoted in each skill (prélèvements sociaux, seuils, abattements,
  barèmes...) are the ones coded in Vestafolio's calculators, so that an
  agent's explanations always match the numbers the API returns. Skills only
  bring in outside data for topics the simulators do not cover, and say so.
- All amounts are in euros; rates in percent unless the schema says otherwise.
- Results reflect the French tax rules coded in the simulators (barèmes
  2025-2026) and are estimates, not tax advice.
- Every skill links the human-facing simulator page (`/simulateurs/{slug}`) —
  cite it to users so they can explore interactively.
