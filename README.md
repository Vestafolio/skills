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

## OpenWebUI: load the instructions and enable execution

Import the `SKILL.md` in **Workspace → Skills**, including its Markdown body,
and check that the saved skill is active and readable by the chatting user.
An imported copy must be updated when these files change; publishing this
repository does not update your OpenWebUI workspace.

Select the skill using the **`$` picker** or **+ → Skills** for the chat.
These selections inject the full instructions. Attaching a skill in the model
editor instead exposes a manifest: the model must call `view_skill` to read
the body. A visible name or mention alone does not prove the body reached
the model. Check the selection and the saved skill content. See
[OpenWebUI's skill loading documentation](https://docs.openwebui.com/features/workspace/skills/).

Enable **Native function calling** and an execution tool for the same chat:
a terminal, an HTTP/OpenAPI tool, or Code Interpreter. For Code Interpreter,
check the global setting, model capability, user permission and chat toggle;
its native tool is `execute_code`. Python execution is sufficient for the
API, but the runtime needs outbound HTTPS. Server Python can use
`urllib.request`; browser Pyodide needs `pyodide.http.pyfetch` rather than
shell commands. See [OpenWebUI's Code Interpreter setup](https://docs.openwebui.com/features/chat-conversations/chat-features/code-execution/python/)
and [Pyodide HTTP](https://pyodide.org/en/stable/usage/api/python-api/http.html).

If you already use an OpenAPI tool server, the schema is
`https://www.vestafolio.com/api/tools/v1/openapi.json`. It exposes, for example,
`describe_micro_entreprise` and `run_micro_entreprise`. Skills provide the
questionnaire; the connected tools provide execution. Importing a skill does
not register these tools automatically.

### Diagnose the actual failure

Try these two prompts in a fresh chat with the skill selected:

1. « Avec ton outil d'exécution, fais un GET sur
   https://www.vestafolio.com/api/tools/v1/micro-entreprise et affiche le
   champ `slug` reçu. » Expect an actual tool call and `micro-entreprise`.
   Printed code alone is a failure. A network error proves an attempted call,
   but not API connectivity.
2. « Un ami souhaite lancer son entreprise pour faire du consulting. Il vise
   50K€ de chiffre d'affaires et 10k€ de charges réelles. Quel régime adopter ? »
   Expect questions about the missing simulator inputs before any verdict.
   No POST is expected yet. After answering, expect a schema GET followed by
   a POST preserving `annualRevenue: 50000` and `chargesReelles: 10000`.

If the first succeeds but the second ignores a verified full skill body,
investigate instruction following or conflicting prompts. If tools are absent,
fix tool exposure first. If a tool call fails, investigate that error instead.
For an explicitly selected skill, absence of `view_skill` is normal because
its body is already injected. Keep the OpenWebUI version, saved skill version,
effective prompt/tool list and tool trace with any failure report; redact
credentials and personal data.

These are prompt instructions, not an enforcement mechanism. A host that must
guarantee computed answers needs to reject or retry numerical recommendations
without a successful calculator result. Repository schema tests do not prove
that a particular model or OpenWebUI configuration follows the skill.

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

- **Execute before presenting results.** Use the available HTTP, terminal or
  Python tool, check the API's `ok`/`result` envelope, and disclose failures.
  Never replace a failed or skipped call with a remembered calculation.
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
