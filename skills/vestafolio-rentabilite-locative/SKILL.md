---
name: vestafolio-rentabilite-locative
description: Compute gross and net rental yield and monthly cash-flow for a French buy-to-let investment from acquisition cost, rent and annual charges using Vestafolio's simulator API. Use when a user asks about rental profitability, "quelle rentabilité locative", rendement brut vs net, cash-flow of an investissement locatif, or whether a rental property is a good deal.
---

# Rentabilité locative (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Computes the gross and net rental yield and the average monthly cash-flow
(excluding financing) of a buy-to-let investment, from the total acquisition
cost (price, frais de notaire, travaux), the rent and the annual charges
(taxe foncière, copropriété, gestion, vacance locative).

## When to use

- "Is 900 € rent on a 200 000 € flat a good deal?" / "Quelle rentabilité pour
  ce bien ?"
- Questions about rendement brut vs rendement net, or cash-flow of an
  investissement locatif
- Estimating the impact of vacancy, management fees or charges on profitability

## When NOT to use

- Buying a primary residence vs renting — use vestafolio-achat-vs-location
- Loan payments for financing the purchase — use vestafolio-credit-immobilier
- Computing the notary fees input precisely — use vestafolio-frais-notaire first

## French rental context (as coded in the simulator)

- Total acquisition cost = purchase price + frais de notaire + travaux; both
  yields are expressed as a percent of this total, not of the price alone.
- Gross yield = annual rent (12 × monthlyRent, no vacancy) / acquisition cost.
- Vacancy is modeled in weeks per year (`vacancyWeeks`): rent is scaled down
  pro rata, giving `annualNetRent` and `vacancyLoss`.
- Net yield deducts vacancy and all charges: taxe foncière, non-recoverable
  charges de copropriété, other charges (assurance PNO, entretien) and
  management fees (`managementFeesPercent`, applied to rent actually
  collected). It is the key decision indicator.
- Cash-flow is computed before any loan: financing and taxes on rental income
  (micro-foncier, LMNP, etc.) are out of scope here.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/rentabilite-locative
```

Then POST the user's parameters (all amounts in euros, rates in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/rentabilite-locative \
  -H 'Content-Type: application/json' \
  -d '{
    "purchasePrice": 200000,
    "notaryFees": 16000,
    "renovationCost": 0,
    "monthlyRent": 900,
    "propertyTax": 1200,
    "condoFees": 1800,
    "otherCharges": 0,
    "managementFeesPercent": 8,
    "vacancyWeeks": 2
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `result.grossYield` vs `result.netYield` — always present both; the gap
  shows how much charges and vacancy eat into the headline number. Net yield
  is the decision metric.
- `result.monthlyCashFlow` — average monthly net rental income before any
  loan; negative means charges exceed rents even without financing
- `result.netRent` / `result.annualCharges` / `result.vacancyLoss` — the
  components behind the net yield, useful to explain what to optimize
- `result.totalAcquisitionCost` — the denominator of both yields

## Caveats

- Excludes financing (mensualités) and taxation of rental income — a positive
  cash-flow here can turn negative once a loan and impôts are added.
- Estimates with constant rent and charges; not investment advice — say so.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/rentabilite-locative
