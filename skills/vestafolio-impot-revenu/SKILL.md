---
name: vestafolio-impot-revenu
version: 1.0.0
description: Compute French income tax (impôt sur le revenu) with the 2026 progressive barème, quotient familial and décote using Vestafolio's simulator API. Use when a user asks "combien d'impôt vais-je payer", how much income tax they owe in France, their TMI (marginal tax rate), taux moyen, parts fiscales, or the tax impact of marriage, PACS or children.
---

# Impôt sur le revenu (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Computes French income tax on the progressive barème for a household: bracket
by bracket detail, number of parts fiscales, quotient familial ceiling, décote,
réductions/crédits, net tax, marginal rate (TMI) and effective rate.

## When to use

- "Combien d'impôt sur le revenu vais-je payer ?" / "How much income tax will I owe in France?"
- Questions about TMI, taux moyen, tranches d'imposition, parts fiscales
- Estimating the tax effect of marriage/PACS, children, résidence alternée or parent isolé

## When NOT to use

- Non-French tax residents (the barème is France-specific)
- Social contributions (prélèvements sociaux), CEHR, or flat-taxed (PFU) investment
  income — this simulator computes barème income tax only
- Rental-income regime choices (use vestafolio-micro-foncier-vs-reel or
  vestafolio-lmnp-fiscalite)

## French tax context (as coded in the simulator, 2026)

- Progressive barème 2026 on 2025 income (article 197 du CGI): brackets at
  0 %, 11 %, 30 %, 41 % and 45 %.
- Quotient familial: household income divided by the number of parts; 0.5 part
  for each of the first 2 children, 1 part from the 3rd; +0.5 part per disabled
  child; shared-custody children count 0.25 part (0.5 from the 3rd); parent
  isolé (case T) adds 0.5 part. The QF advantage is capped at 1 807 € per
  demi-part.
- Décote reduces small tax amounts; réductions and crédits d'impôt are applied
  after it.
- Input income is the **revenu net imposable** — after the 10 % abattement (or
  frais réels), not gross salary.
- Excluded by design: prélèvements sociaux, CEHR, and PFU-taxed income.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/impot-revenu
```

Then POST the user's parameters (all amounts in euros):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/impot-revenu \
  -H 'Content-Type: application/json' \
  -d '{
    "revenuNetImposable": 40000,
    "familySituation": "celibataire",
    "jointDeclaration": true,
    "partnerIncome": 35000,
    "numberOfChildren": 0,
    "numberOfChildrenDisabled": 0,
    "sharedCustodyChildren": 0,
    "singleParent": false,
    "reductionsCredits": 0
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `netTax` — the key figure: net income tax due for the year (also given
  monthly as `monthlyTax`)
- `marginalRate` (TMI) — the rate on one additional euro of income, as a
  decimal (0.30 = 30 %); do not confuse it with `effectiveRate`, the average
  rate (net tax / total income, in percent), which is always lower
- `numberOfParts` and `quotientFamilial` — how the household splits income
  across the barème; `qfCeiling` shows how much QF advantage the 1 807 €
  per-demi-part cap clawed back
- `brackets` — per-bracket detail, useful to explain "why is my TMI 30 %"
- `decote`, `reductions` — adjustments between gross and net tax
- `incomeAfterTax` / `monthlyNetIncome` — what remains after income tax

## Caveats

- Rules as coded for the 2026 barème (2025 income); rates change with finance
  laws. Estimates, not tax advice — say so.
- Excludes prélèvements sociaux, the CEHR surtax and PFU-taxed investment
  income: the real total levy can be higher.
- Remind users the input is revenu net imposable, not gross salary.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/impot-revenu
