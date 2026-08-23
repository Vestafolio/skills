---
name: vestafolio-capacite-emprunt
version: 1.0.0
description: Estimate the maximum mortgage a French household can borrow from income, existing charges and the HCSF 35 % debt ratio using Vestafolio's simulator API. Use when a user asks how much they can borrow, "combien puis-je emprunter", borrowing capacity (capacité d'emprunt), taux d'endettement, or what property budget their salary allows.
---

# Capacité d'emprunt (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Estimates the maximum amount a household can borrow for a property purchase
from its net monthly income, existing charges, the envisaged loan rate and
duration, and the maximum debt ratio (taux d'endettement).

## When to use

- "How much can I borrow with a 4 000 € salary?" / "Combien puis-je emprunter
  avec 4 000 € par mois ?"
- Questions about capacité d'emprunt, taux d'endettement or the HCSF 35 % rule
- Sizing a property budget before searching (combine with
  vestafolio-frais-notaire for the full acquisition cost)

## When NOT to use

- Computing the payment of an already-sized loan — use
  vestafolio-credit-immobilier
- Buy vs rent decisions — use vestafolio-achat-vs-location
- Rental-investment profitability — use vestafolio-rentabilite-locative

## French banking context (as coded in the simulator)

- Maximum supportable payment = monthlyIncome × debtRatio / 100 −
  monthlyCharges. The default debtRatio of 35 % is the HCSF norm in France
  (assurance included).
- The maximum loan is derived by inverting the annuity formula, with the
  assurance emprunteur computed on the initial borrowed capital
  (loan × insuranceRate / 100 / 12) and included inside the maximum payment.
- Income is net monthly household income before tax; charges are existing
  recurring commitments (crédits en cours, pensions).

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/capacite-emprunt
```

Then POST the user's parameters (all amounts in euros, rates in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/capacite-emprunt \
  -H 'Content-Type: application/json' \
  -d '{
    "monthlyIncome": 4000,
    "monthlyCharges": 0,
    "rate": 3.5,
    "years": 20,
    "insuranceRate": 0.3,
    "debtRatio": 35
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `result.maxLoan` — the headline number: maximum borrowable capital in euros
- `result.maxMonthlyPayment` — the payment ceiling (crédit + assurance) implied
  by the debt ratio
- `result.monthlyInsurance` — estimated insurance share of that payment
- `result.currentDebtRatio` — debt ratio reached at that payment, in percent;
  useful when the user has existing charges pushing them near 35 %
- To translate maxLoan into a property budget, add the apport and subtract
  frais de notaire (vestafolio-frais-notaire)

## Caveats

- An estimate, not a bank pre-approval: banks also weigh reste à vivre, job
  stability, saut de charges and can derogate from the 35 % HCSF norm for a
  share of their production.
- Insurance modeled at a flat rate on initial capital; real quotes vary with
  age and health.
- Not financial advice — say so.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/capacite-emprunt
