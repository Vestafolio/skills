---
name: vestafolio-micro-foncier-vs-reel
version: 1.0.0
description: Compare unfurnished rental taxation between micro-foncier and régime réel with déficit foncier using Vestafolio's simulator API. Use when a user asks "micro-foncier ou régime réel", how location nue rental income is taxed in France, about the 30 % abattement, the 15 000 € micro-foncier ceiling, déficit foncier imputation on global income, or which regime saves more tax on revenus fonciers.
---

# Micro-foncier vs régime réel (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Compares the taxation of an unfurnished rental (location nue) under the
micro-foncier (flat 30 % abattement) and the régime réel (real charges and
déficit foncier), with the recommended regime, annual savings and the
break-even charges level.

## When to use

- "Micro-foncier ou régime réel ?" / "Which regime for my unfurnished rental income?"
- Questions about the 30 % abattement, the 15 000 € micro-foncier ceiling, or
  déficit foncier (creation, imputation, carry-forward)
- Estimating tax on revenus fonciers given real charges and loan interest

## When NOT to use

- Furnished rentals (LMNP) — use vestafolio-lmnp-fiscalite
- Property capital gains on sale (use vestafolio-impot-plus-value)
- Non-French rental income

## French tax context (as coded in the simulator, 2025-2026)

- Micro-foncier: flat 30 % abattement, eligible only up to 15 000 € of annual
  gross rents.
- Régime réel: real charges deducted (loan interest, taxe foncière, condo
  fees, insurance, management, repairs). A déficit foncier is imputable on
  global income up to 10 700 € per year — loan interest excluded from that
  imputation — and the remainder is carried forward against future revenus
  fonciers for 10 years.
- Taxable result bears income tax at the household's TMI plus prélèvements
  sociaux at 17.2 %.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/micro-foncier-vs-reel
```

Then POST the user's parameters (all amounts annual and in euros, TMI in
percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/micro-foncier-vs-reel \
  -H 'Content-Type: application/json' \
  -d '{
    "annualRent": 12000,
    "charges": {
      "loanInterest": 3000,
      "propertyTax": 1200,
      "condoFees": 1800,
      "insurance": 300,
      "managementFees": 0,
      "repairs": 500
    },
    "marginalTaxRate": 30,
    "existingDeficit": 0
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `recommendation` — `micro_foncier` or `regime_reel`; the réel is recommended
  when it taxes less, when a deficit is imputable on global income, or when
  micro-foncier is ineligible (rents above 15 000 €, see `isEligible`)
- `annualSavings` — yearly tax saved by the réel vs micro (0 if micro wins)
- `breakEvenCharges` — the charges level (30 % of rents) above which the réel
  becomes more advantageous; useful to explain the tipping point
- Per-regime blocks (`microFoncier`, `reel`): `deductions`, `taxableIncome`,
  `incomeTax`, `socialContributions`, `totalTax`, `netIncome`, `effectiveRate`
- `reel.deficit` — deficit created, the part imputed on global income (10 700 €
  cap, interest excluded) and the part carried forward 10 years

## Caveats

- Rules as coded for 2025-2026; ceilings and rates change with finance laws.
  Estimates, not tax advice — say so.
- Opting for the réel commits the taxpayer for several years in real life —
  the simulator compares a single year with constant figures.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/micro-foncier-vs-reel
