---
name: vestafolio-interets-composes
version: 1.0.0
description: Project compound-interest growth of an investment with initial capital and monthly contributions using Vestafolio's simulator API. Use when a user asks about intérêts composés, compound interest projections, how much recurring savings will grow over N years, or wants a year-by-year table of invested amounts versus gains.
---

# Intérêts composés (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Projects the final value of an investment with an initial capital and monthly
contributions, at a chosen compounding frequency, returning total invested,
gains and a year-by-year evolution table.

## When to use

- "Combien vaudront mes intérêts composés dans 20 ans ?" / "How much will
  300 €/month grow to?"
- Producing a year-by-year invested-vs-gains table that matches the Vestafolio
  website exactly — you could do this math yourself, but the API guarantees a
  consistent table (same rounding, same compounding) as the simulator the user
  may open

## When NOT to use

- Anything involving French taxation of the gains (use vestafolio-pea-vs-cto)
- Loan amortisation or real-estate projections (other Vestafolio skills)

## French tax context

None — this tool is pre-tax pure compounding. All amounts are in euros, the
rate is a nominal annual percentage, and interest capitalises monthly,
quarterly or yearly per `compoundingPeriod`.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/interets-composes
```

Then POST the user's parameters (amounts in euros, rate in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/interets-composes \
  -H 'Content-Type: application/json' \
  -d '{
    "initialAmount": 10000,
    "monthlyContribution": 300,
    "years": 20,
    "annualRate": 7,
    "compoundingPeriod": "monthly"
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `total` — final value; `totalInvested` — sum of all contributions;
  `gains` — the compounding effect (total − invested)
- `yearlyEvolution` — rounded per-year `invested`, `gains`, `total` (year 0 is
  the starting point); ideal for tables and "after N years" follow-ups

## Caveats

- Assumes a constant return and constant contributions — real returns vary.
- Pre-tax: no fiscalité applied. Estimates, not investment advice — say so.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/interets-composes
