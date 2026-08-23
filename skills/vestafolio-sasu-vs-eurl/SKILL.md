---
name: vestafolio-sasu-vs-eurl
version: 1.0.0
description: Compare SASU (président assimilé salarié) and EURL (gérant TNS) net director income for a French solo entrepreneur using Vestafolio's simulator API. Use when a user asks "SASU ou EURL", which company structure pays more, about cotisations sociales assimilé salarié vs TNS, dividend taxation in an EURL, or optimal salary vs dividendes split.
---

# SASU vs EURL (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Computes, for the same annual revenue, the total net income of the director in
a SASU (président assimilé salarié) versus an EURL at IS (gérant TNS):
cotisations sociales, impôt sur les sociétés, impôt sur le revenu, dividend
taxation, and a recommendation with the annual net-income gap.

## When to use

- "SASU ou EURL ?" / "Which structure should I pick for my solo business?"
- Comparing assimilé-salarié vs TNS social charges on the same revenue
- Questions about dividend taxation in an EURL (TNS on dividends above 10 % of
  capital social) or optimal salary/dividend mix

## When NOT to use

- Choosing among all legal forms including micro-entreprise (use
  vestafolio-choisir-regime for orientation first)
- Micro-entreprise regime comparisons (use vestafolio-micro-entreprise)
- Multi-partner companies (SARL/SAS) — the tool models a solo entrepreneur

## French tax context (as coded in the simulator, 2026)

- IS at 15 % up to 42 500 € of profit, then 25 %.
- Simplified cotisations sociales at 45 % of gross remuneration, for both the
  assimilé-salarié (SASU) and TNS (EURL) scenarios.
- Dividends: PFU (flat tax) at 31,4 % (12,8 % IR + 18,6 % prélèvements
  sociaux), or progressive barème with 40 % abattement if `preferPFU` is false
  (18,6 % PS in both cases).
- EURL specificity: dividends above 10 % of the capital social bear additional
  cotisations TNS.
- Remuneration is taxed at the barème progressif IR 2026; the leftover after
  remuneration is fully distributed as dividends.
- Salary amounts are optional: "si omis, le salaire maximal disponible après
  cotisations est versé" — omitted salary means the maximum available salary
  is paid out; provided amounts are capped at the maximum available.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/sasu-vs-eurl
```

Then POST the user's parameters (all amounts annual, in euros; rates in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/sasu-vs-eurl \
  -H 'Content-Type: application/json' \
  -d '{
    "targetRevenue": 100000,
    "capitalSocial": 10000,
    "marginalTaxRate": 30,
    "sasuMarginalTaxRate": 30,
    "eurlMarginalTaxRate": 30,
    "sasuSalaryAmount": 30000,
    "eurlSalaryAmount": 30000,
    "preferPFU": true,
    "chargesDeductibles": 0
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `result.recommendation` — "sasu" or "eurl", whichever maximises net income
- `annualSavings` — the annual net-income gap between the two structures
- Per-structure blocks (`sasu`, `eurl`): `netIncome`, `totalTax`,
  `effectiveRate`, plus `companyView` (CA → dividendes) and `directorView`
  (income received and taxes paid)
- `maxSalaryAvailable` — useful when the user asked for a salary above what
  the revenue can fund
- `alerts.salaryBelowSmic` and `alerts.highDividendRatio` — surface these
  warnings (requalification risk) to the user when true
- `breakdownComparison` — line-by-line SASU vs EURL table, good for a summary

## Caveats

- Rules as coded for 2026 with simplified 45 % cotisation rates; real
  assimilé-salarié charges differ from TNS in detail. Estimates, not tax or
  legal advice — say so.
- Assumes the full post-remuneration profit is distributed as dividends.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/sasu-vs-eurl
