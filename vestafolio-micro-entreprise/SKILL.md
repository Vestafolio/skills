---
name: vestafolio-micro-entreprise
description: Compare micro-entreprise tax regimes (versement libératoire, barème progressif, régime réel) for a French auto-entrepreneur using Vestafolio's simulator API. Use when a user asks about micro-entreprise vs régime réel, "micro-entreprise ou société", versement libératoire eligibility, auto-entrepreneur cotisations, or micro thresholds and abattements by activity (BNC, services BIC, commerce).
---

# Micro-entreprise (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Computes cotisations sociales, impôt and net income of a micro-entreprise
under the versement libératoire and under the barème progressif, and — when
real professional expenses are provided — compares with the régime réel, with
a recommendation.

## When to use

- "Should I opt for the versement libératoire?" / "Micro ou réel ?"
- "Micro-entreprise ou société ?" as a first quantitative step on the micro side
- Estimating net income of an auto-entrepreneur (BNC, services BIC, commerce)
- Checking versement libératoire eligibility against the RFR N-2 condition

## When NOT to use

- Comparing SASU vs EURL company structures (use vestafolio-sasu-vs-eurl)
- Choosing a legal form qualitatively (use vestafolio-choisir-regime)
- Revenue above the micro thresholds with no interest in the réel comparison

## French tax context (as coded in the simulator, 2025-2026)

- Exceeding a threshold does not end the micro regime immediately: the regime
  applies in year N unless the N-1 AND N-2 turnovers both exceeded the
  ceiling; the current year's overrun only counts towards next year. Set
  `previousYearAboveThreshold` / `twoYearsAgoAboveThreshold` accordingly, and
  `monthsOfActivity` in a creation year (the ceiling is prorated). For ACRE,
  `acreCreatedBeforeJuly2026` selects the legacy 50 % exemption for businesses
  created before 1 July 2026.
- Micro thresholds: 83 600 € (BNC and services BIC), 203 100 € (commerce /
  vente de marchandises).
- Abattements forfaitaires: 34 % BNC, 50 % services, 71 % commerce.
- Cotisations sociales on turnover: 23,4 % BNC CIPAV, 25,8 % BNC hors CIPAV,
  21,5 % services, 12,4 % commerce (URSSAF rates 23,2/25,6/21,2/12,3 % plus the CFP training contribution). With ACRE the first year, rates drop to
  75 % of the normal rates — the ACRE exemption was cut from 50 % to 25 % for
  businesses created from 1 July 2026 (17,6 % BNC CIPAV, 19,4 % BNC hors CIPAV,
  16,2 % services, 9,4 % commerce, CFP included).
- Versement libératoire de l'impôt sur le revenu: 2,2 % BNC, 1,7 % services,
  1 % commerce, on turnover — only if RFR N-2 ≤ 29 315 € per part fiscale.
- Régime réel (when `chargesReelles` > 0): cotisations TNS of 45 % (services)
  or 35 % (BNC, commerce) on the bénéfice (CA − charges), income taxed at the
  household TMI.
- Activity types in the schema: `bnc` (profession libérale), `services`
  (prestations BIC), `commerce` (vente de marchandises).

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/micro-entreprise
```

Then POST the user's parameters (all amounts annual, in euros; rates in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/micro-entreprise \
  -H 'Content-Type: application/json' \
  -d '{
    "annualRevenue": 50000,
    "activityType": "bnc",
    "isCipavAffiliated": true,
    "isFirstYear": false,
    "hasACRE": false,
    "marginalTaxRate": 30,
    "fiscalParts": 1,
    "previousYearIncome": 25000,
    "chargesReelles": 0
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `recommendation` — "micro_vl", "micro_ir" or "reel", whichever maximises net
  income among eligible options; `recommendationText` explains it in French
- `vlEligible` — whether the household qualifies for the versement libératoire
  (RFR N-2 condition); if false, ignore the `versementLiberatoire` scenario
- Scenario blocks (`versementLiberatoire`, `baremeProgressif`, and `reel` when
  charges were provided): `socialCharges`, `incomeTax`, `netIncome`,
  `effectiveRate`, and `isEligible` against the micro threshold
- `annualSavings` — net-income gap between the best and worst eligible option
- `details` — the applied abattement, cotisation rate and VL rate, useful to
  show the user how the numbers were built

## Caveats

- Rates and thresholds as coded for 2025-2026; they change with finance and
  social-security laws. Estimates, not tax advice — say so.
- `isCipavAffiliated` matters only for BNC; it is ignored for services and
  commerce.
- The réel scenario is a simplified TNS model, not a full accounting
  simulation.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/micro-entreprise
