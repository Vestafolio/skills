---
name: vestafolio-credit-immobilier
version: 1.0.0
description: Compute French mortgage monthly payments (insurance included), total interest and the full amortization schedule using Vestafolio's simulator API. Use when a user asks about mortgage payments, loan cost, "quelle mensualité pour un crédit immobilier", "combien coûte un prêt de 250 000 €", amortization tables (tableau d'amortissement), or assurance emprunteur cost.
---

# Crédit immobilier (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Computes the monthly payment (assurance emprunteur included), the total cost of
credit, total interest, total insurance and the complete month-by-month
amortization schedule for a fixed-rate French mortgage with constant payments.

## When to use

- "What would my monthly payment be for a 250 000 € loan over 20 years?" /
  "Quelle mensualité pour 250 000 € sur 20 ans ?"
- "How much interest will I pay in total?" / "Combien coûte mon crédit ?"
- Requests for a tableau d'amortissement (principal vs interest split per month)
- Estimating the impact of the insurance rate (assurance emprunteur) on payments

## When NOT to use

- "How much CAN I borrow given my income?" — use vestafolio-capacite-emprunt
- Buy vs rent decisions — use vestafolio-achat-vs-location
- Notary/closing costs of the purchase — use vestafolio-frais-notaire

## French banking context (as coded in the simulator)

- Fixed-rate, constant-payment loan: the crédit part of the payment follows the
  standard annuity formula on the monthly rate (annualRate / 12).
- Assurance emprunteur is computed on the initial capital (capital initial),
  the common French bank convention: monthly insurance =
  principal × insuranceRate / 100 / 12, constant over the whole loan.
- The schedule shows, for each month, the payment, the principal repaid, the
  interest paid and the capital restant dû.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/credit-immobilier
```

Then POST the user's parameters (all amounts in euros, rates in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/credit-immobilier \
  -H 'Content-Type: application/json' \
  -d '{
    "principal": 250000,
    "annualRate": 3.5,
    "durationYears": 20,
    "insuranceRate": 0.3
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `result.monthlyPayment` — total monthly payment, insurance included; also
  provided split as `monthlyPaymentWithoutInsurance` + `monthlyInsurance`
- `result.totalPayment` — total cost of the credit over the full duration
- `result.totalInterest` / `result.totalInsurance` — the two cost components
  beyond the capital; useful to show what the loan really costs
- `result.schedule` — full monthly amortization table (month, payment,
  principal, interest, remainingBalance); use it for "how much will I still owe
  after N years" follow-ups rather than recomputing

## Caveats

- Estimates, not a loan offer — the actual TAEG also includes frais de dossier,
  garantie and other fees not modeled here.
- Fixed rate and constant insurance on initial capital assumed; banks may quote
  insurance on capital restant dû, which is cheaper over time.
- Not financial advice — say so.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/credit-immobilier
