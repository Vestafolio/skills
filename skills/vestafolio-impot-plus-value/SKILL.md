---
name: vestafolio-impot-plus-value
version: 1.0.0
description: Compute French real-estate capital gains tax (impôt sur la plus-value immobilière) with holding-period abatements using Vestafolio's simulator API. Use when a user asks "plus-value immobilière", how much tax they owe when selling a property or résidence secondaire in France, about the abattement pour durée de détention, the 22/30-year exemptions, LMNP amortization reintegration, or whether to sell now or wait.
---

# Impôt sur la plus-value immobilière (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Computes the tax due on a French property sale: gross and net capital gain,
acquisition-fee and renovation deductions, holding-period abatements, IR at
19 %, prélèvements sociaux at 17.2 %, the high-gain surtax, exemptions, and
the savings from postponing the sale by one year.

## When to use

- "Combien d'impôt sur ma plus-value immobilière ?" / "How much tax if I sell my French property?"
- Questions about the abattement pour durée de détention, or the 22-year (IR)
  and 30-year (PS) exemptions
- Selling a résidence secondaire, a rental, or an LMNP au régime réel
  (amortization reintegration since 2025)
- "Should I sell now or wait a year?" decisions

## When NOT to use

- Capital gains on securities/shares (PFU territory — not this simulator)
- Professional sellers or companies at IS (only particulier and SCI à l'IR are
  modeled)
- Non-French property or non-French tax situations

## French tax context (as coded in the simulator, 2025-2026)

- Capital gain taxed at 19 % IR plus 17.2 % prélèvements sociaux, after
  abatements for holding duration: full IR exemption after 22 years, full PS
  exemption after 30 years.
- Surtax on high gains above 50 000 €.
- Exemptions: résidence principale (fully exempt), sale price under 15 000 €.
- Acquisition costs: forfait of 7.5 % of the purchase price, or actual
  documented amounts. Renovation works: forfait of 15 % of the purchase price
  (only if held >= 5 years), or actual documented amounts.
- LMNP au régime réel: amortization deducted during the rental is reintegrated
  into the taxable gain (rule since 2025).

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/impot-plus-value
```

Then POST the user's parameters (amounts in euros, dates as `YYYY-MM-DD`
strings — `saleDate` must be after `purchaseDate`):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/impot-plus-value \
  -H 'Content-Type: application/json' \
  -d '{
    "purchasePrice": 200000,
    "salePrice": 280000,
    "purchaseDate": "2015-01-01",
    "saleDate": "2026-06-01",
    "isMainResidence": false,
    "isRentalInvestment": false,
    "isRealRegime": false,
    "totalAmortization": 0,
    "acquisitionFeesMethod": "forfait",
    "acquisitionFeesReal": 16000,
    "renovationMethod": "forfait",
    "renovationReal": 20000,
    "sellerType": "particulier"
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `totalTax` — the key decision figure (IR + PS + surtax); `netProceeds` is
  the gain kept after tax, `effectiveTaxRate` the levy on the gross gain
- `grossCapitalGain` vs `netCapitalGain` — before/after acquisition-fee and
  renovation deductions
- `irAbatementRate` / `psAbatementRate` and the corresponding amounts — how the
  holding period shrank the taxable base
- `yearsUntilIRExemption` / `yearsUntilPSExemption` and `savingsIfWait1Year` —
  highlight these for "sell now or wait" questions
- `abatementSchedule` — the full 0-to-30-year abatement table by holding
  duration, useful to show when each exemption kicks in
- `exemptions` / `isFullyExempt` — applicable exemption reasons (French labels)
- `amortizationReintegration` / `usesLMNPRealTaxation` — LMNP réel specifics

## Caveats

- Rules as coded for 2025-2026; rates and abatement schedules change with
  finance laws. Estimates, not tax advice — say so.
- The forfait options (7.5 % fees, 15 % works) are simulator conventions the
  user may or may not be entitled to; actual documented amounts can differ.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/impot-plus-value
