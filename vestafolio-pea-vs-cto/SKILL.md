---
name: vestafolio-pea-vs-cto
description: Compare PEA, CTO (compte-titres) and assurance-vie net-of-tax outcomes for a French investor using Vestafolio's simulator API. Use when a user asks which investment envelope to choose, about PEA vs CTO taxation, flat tax (PFU) on investments, assurance-vie abattement, or where to invest monthly savings in France.
---

# PEA vs CTO vs Assurance-vie (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Computes the net-after-tax capital of the same investment plan (initial amount +
monthly contributions) held in a PEA, a CTO or an assurance-vie, with
year-by-year projections, break-even years and a recommendation.

## When to use

- "Should I invest through a PEA or a CTO?" / "PEA ou assurance-vie ?"
- Comparing net outcomes of French investment envelopes over N years
- Questions about the PEA 150 000 € ceiling, PFU, or assurance-vie abattement

## When NOT to use

- Non-French tax residents (the rules are France-specific)
- Real-estate investing (use the immobilier skills instead)
- Pure compound-interest math without tax (use vestafolio-interets-composes)

## French tax context (as coded in the simulator, 2026)

- PEA and CTO: prélèvements sociaux at 18.6 %. PEA is exempt from income tax
  after 5 years (before that, PFU 31.4 % applies on gains at withdrawal).
- CTO: PFU 31.4 % or progressive barème depending on marginal rate (TMI) — the
  simulator picks the more favourable option.
- Assurance-vie: PFU 30 % (12.8 % IR + 17.2 % PS) before 8 years; after 8
  years, annual abattement of 4 600 € (9 200 € for a couple) and reduced 7.5 %
  IR under 150 000 € of premiums. Management fees reduce returns every year.
- PEA contribution ceiling: 150 000 € — excess contributions overflow to CTO in
  the simulation.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/pea-vs-cto
```

Then POST the user's parameters (all amounts in euros, rates in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/pea-vs-cto \
  -H 'Content-Type: application/json' \
  -d '{
    "initialInvestment": 10000,
    "monthlyContribution": 300,
    "holdingPeriodYears": 10,
    "annualReturn": 7,
    "marginalTaxRate": 30,
    "avManagementFees": 0.6,
    "isCouple": false
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `result.recommendation` — the envelope with the best net outcome
- Per-envelope blocks (`pea`, `cto`, `assuranceVie`): gross vs net capital,
  total tax paid, effective tax rate
- Break-even fields — the year from which one envelope overtakes another;
  highlight these when the user's horizon is near a threshold (5 or 8 years)
- Year-by-year projections — use for "what if I stop after N years" follow-ups

## Caveats

- Rules as coded for 2026; rates change with finance laws. Not tax advice —
  say so.
- The simulation assumes constant returns and contributions.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/pea-vs-cto
