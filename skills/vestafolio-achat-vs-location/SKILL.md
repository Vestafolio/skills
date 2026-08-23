---
name: vestafolio-achat-vs-location
version: 1.0.0
description: Compare final net wealth between buying a primary residence with a mortgage and renting while investing savings, over a chosen horizon, using Vestafolio's simulator API. Use when a user asks whether to buy or rent, "acheter ou louer", "est-ce rentable d'acheter ma résidence principale", rent vs buy break-even, or what owning really costs versus renting in France.
---

# Acheter vs louer (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Compares the final net wealth of buying a primary residence with a mortgage
versus staying a tenant and investing the savings, simulated month by month
over a chosen horizon: loan amortization, property appreciation, rent
increases, taxe foncière (indexed at 3 %/year) and capitalization of invested
savings.

## When to use

- "Should I buy or keep renting?" / "Acheter ou louer ma résidence
  principale ?"
- "After how many years does buying beat renting?" (break-even questions)
- Testing sensitivity to property appreciation, investment returns, rent
  growth or horizon

## When NOT to use

- Buy-to-let / rental-investment profitability — use
  vestafolio-rentabilite-locative
- Sizing the loan or its payment alone — use vestafolio-capacite-emprunt or
  vestafolio-credit-immobilier
- Choosing where to invest the savings themselves — use vestafolio-pea-vs-cto

## French housing context (as coded in the simulator)

- Buy scenario: `availableSavings` funds the apport (purchasePrice −
  loanAmount), the frais de notaire (`notaryFeePercent` of the price, e.g.
  7.5 % in the ancien) and the travaux; whatever savings remain are invested
  at `investmentReturnBuy`. The owner pays the mortgage (annuity, no
  insurance) plus taxe foncière indexed at 3 %/year.
- Rent scenario: the full `availableSavings` are invested at
  `investmentReturn`; rent grows at `rentIncreaseRate` per year.
- Fair comparison rule: each year, whichever side has the lower housing
  outflow invests the difference as an effort d'épargne — the owner's outflow
  is mensualités + taxe foncière, compared against the rent
  (`additionalSavingsInvested` on both sides). Savings interest starts
  compounding in Year 2; an effort saved in year N earns interest from N+1.
- Net wealth at horizon (buy) = sale proceeds (property value − outstanding
  principal, repaid at sale when the horizon ends before the loan) −
  cumulated expenses (taxe foncière and loan interest every year) +
  investment portfolio (savings after down payment, notary fees and works —
  acquisition costs counted once — plus efforts and interest). Intermediate
  years value the stake as property value × repaid share of the loan.
- Net wealth at horizon (rent) = investment portfolio (full savings +
  efforts + interest) − cumulated rents paid.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/achat-vs-location
```

Then POST the user's parameters (all amounts in euros, rates in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/achat-vs-location \
  -H 'Content-Type: application/json' \
  -d '{
    "purchasePrice": 300000,
    "loanAmount": 255000,
    "loanRate": 3.5,
    "loanYears": 20,
    "monthlyRent": 1200,
    "rentIncreaseRate": 2,
    "propertyAppreciation": 2,
    "investmentReturn": 6,
    "investmentReturnBuy": 6,
    "horizonYears": 20,
    "notaryFeePercent": 7.5,
    "taxeFonciereAnnual": 1500,
    "travaux": 0,
    "availableSavings": 80000
  }'
```

Note the input constraints: `loanAmount` must not exceed `purchasePrice`, and
`travaux` must fit within savings after apport and notary fees (the response's
`maxTravauxBudget` gives the ceiling). Unknown fields are rejected (strict
schema) — if you get a `validation_error`, re-read the schema from the GET
endpoint rather than guessing field names.

## Interpreting the output

- `result.buyWealth` / `result.rentWealth` / `result.difference` — final net
  wealth of each scenario and their gap (positive difference = buying wins)
- `result.recommendation` — "buy" or "rent" at the chosen horizon
- `result.breakEvenYear` — first year where the buy scenario's wealth catches
  up with renting, or null if never within the horizon; highlight it when the
  user's horizon is close to it (moving before break-even favours renting)
- `result.monthlyPayment` — the mortgage payment (hors assurance) driving the
  monthly-effort comparison
- `result.buyDetails` / `result.rentDetails` — decomposition (property value,
  remaining loan, cumulated interest, taxe foncière, portfolios,
  additionalSavingsInvested); use it to explain WHY one side wins
- `result.yearlyData` / `result.cashflowData` — year-by-year wealth curves and
  outflows, for "what if I sell after N years" follow-ups

## Caveats

- Sensitive to assumptions: the recommendation can flip with 1-2 points of
  propertyAppreciation or investmentReturn — present a sensitivity check, not
  a verdict.
- Not modeled: assurance emprunteur, copropriété charges and maintenance,
  selling costs, capital-gains rules, tax on investment returns.
- Estimates, not financial advice — say so.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/achat-vs-location
