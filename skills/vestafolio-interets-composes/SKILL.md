---
name: vestafolio-interets-composes
version: 1.1.0
description: Project compound-interest growth of an investment with initial capital and monthly contributions using Vestafolio's simulator API, after asking the simulator's questions (initial capital, monthly contribution, duration, annual return, compounding frequency). Use when a user asks about intérêts composés, compound interest projections, how much recurring savings will grow over N years, or wants a year-by-year table of invested amounts versus gains.
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

## Questions to ask before calling the API

The simulator asks these five inputs, in this order (card « Paramètres »).
Ask or confirm each in French; there is no conditional question.

1. « Capital initial » → `initialAmount`.
2. « Versement mensuel » → `monthlyContribution`.
3. « Durée » → `years` (the site allows 1 to 50).
4. « Rendement annuel » → `annualRate` (nominal percent; the site allows 0 to
   20).
5. « Capitalisation des intérêts » → `compoundingPeriod`: « Mensuelle » =
   `monthly`, « Trimestrielle » = `quarterly`, « Annuelle » = `yearly`.

## Model as coded in the simulator

- The nominal annual rate is divided by the number of periods per year (12,
  4 or 1); each period the balance grows by that rate and the contributions of
  the period (monthly amount × months in the period) are added at the end of
  it. No effective-rate conversion is made.
- `totalInvested` = initial + monthly × 12 × years; `gains` = total − invested.
- `yearlyEvolution` rounds invested, gains and total to the euro per year,
  starting with year 0.
- No tax and no fees: pure pre-tax compounding.

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

- `total` — final value (« Capital final »); `totalInvested` — sum of all
  contributions; `gains` — the compounding effect (« Plus-values »)
- `yearlyEvolution` — rounded per-year `invested`, `gains`, `total` (year 0 is
  the starting point); ideal for tables and "after N years" follow-ups
- The site also shows the gain as a percent of the invested amount

## Caveats

- Assumes a constant return and constant contributions — real returns vary.
- Pre-tax: no fiscalité applied (chain vestafolio-pea-vs-cto for the net
  result). Estimates, not investment advice — say so.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/interets-composes
