---
name: vestafolio-lmnp-fiscalite
version: 1.0.0
description: Compare LMNP furnished-rental taxation between micro-BIC and régime réel with amortization using Vestafolio's simulator API. Use when a user asks "LMNP micro-BIC ou réel", how furnished rental income is taxed in France, about the 50 % / 30 % abattement, meublé de tourisme thresholds, building/furniture amortization, or which LMNP regime saves more tax.
---

# LMNP : micro-BIC vs régime réel (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Compares the taxation of a location meublée non professionnelle (LMNP) under
the micro-BIC (flat abattement) and the régime réel (real charges plus
amortization), with the recommended regime, annual and 10-year savings, and a
10-year amortization schedule.

## When to use

- "LMNP micro-BIC ou réel ?" / "Which tax regime for my furnished rental?"
- Questions about the micro-BIC abattement and thresholds, or meublé de
  tourisme classé vs non classé
- Estimating amortization (immeuble, mobilier, travaux) and its tax effect

## When NOT to use

- Unfurnished (nue) rentals — use vestafolio-micro-foncier-vs-reel
- Professional furnished landlords (LMP) — only LMNP is modeled
- Capital gains on the sale of the LMNP property (use
  vestafolio-impot-plus-value)

## French tax context (as coded in the simulator, Loi de Finances 2025)

- Micro-BIC: 50 % abattement and 83 600 € revenue threshold for standard
  furnished rentals and meublés de tourisme classés; 30 % abattement and
  15 000 € threshold for tourisme non classé. Above the threshold, micro-BIC
  is ineligible and the réel applies.
- Régime réel: real charges deducted, plus amortization of the building over
  30 years (85 % of value — the 15 % land share is not amortizable), furniture
  over 7 years, and works over 10 years. Amortization cannot create a deficit;
  the excess is carried forward.
- Prélèvements sociaux at 18.6 % on the taxable result; income tax at the
  household's TMI.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/lmnp-fiscalite
```

Then POST the user's parameters (all amounts annual and in euros, TMI in
percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/lmnp-fiscalite \
  -H 'Content-Type: application/json' \
  -d '{
    "annualRent": 15000,
    "purchasePrice": 200000,
    "notaryFees": 16000,
    "furnitureCosts": 8000,
    "renovationCosts": 10000,
    "charges": {
      "loanInterest": 4000,
      "propertyTax": 1200,
      "condoFees": 1800,
      "insurance": 200,
      "managementFees": 0,
      "accounting": 500,
      "entretien": 300
    },
    "marginalTaxRate": 30,
    "propertyType": "standard"
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `recommendation` — `micro_bic` or `reel`, whichever minimizes tax (réel is
  forced if micro-BIC is ineligible; check `isEligible` / `eligibilityReason`)
- `annualSavings` and `tenYearSavings` — the tax gap between regimes; lead
  with these when answering "which regime"
- Per-regime blocks (`microBic`, `reel`): `taxableIncome`, `incomeTax`,
  `socialContributions`, `totalTax`, `netIncome`, `effectiveRate`
- `details` — abattement and threshold in micro-BIC; charges, per-asset
  amortization, used vs carried amortization in réel
- `amortizationSchedule` — 10-year plan showing used vs carried amortization;
  a large `cumulativeCarried` means the réel advantage persists beyond year 10

## Caveats

- Rules as coded per the Loi de Finances 2025; thresholds and abattements
  change with finance laws. Estimates, not tax advice — say so.
- Assumes constant rents and charges over the 10-year projection; accounting
  fees for the réel are an input, not automatic.
- Amortization deducted au réel is reintegrated into the capital gain on sale
  (since 2025) — point users to vestafolio-impot-plus-value for exit planning.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/lmnp-fiscalite
