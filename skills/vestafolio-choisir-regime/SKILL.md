---
name: vestafolio-choisir-regime
description: Determine which French business legal forms (micro-entreprise, EI, EURL, SASU, SARL, SAS, SELARL, SELAS) an entrepreneur is eligible for and get a recommendation using Vestafolio's simulator API. Use when a user asks "quel statut juridique", which legal structure to start a business in France, micro-entreprise eligibility, or whether to create a company for liability or assurance chômage reasons.
---

# Choisir son régime d'entreprise (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Determines the eligible French legal forms for an entrepreneur's profile
(activity, projected turnover, partners, liability and unemployment-insurance
wishes) and recommends the most suitable one, with advantages, disadvantages
and reasoning. This is a qualitative orientation tool: it computes no
cotisations and no taxes.

## When to use

- "Quel statut juridique choisir ?" / "Should I start as micro-entreprise or
  create a company?"
- Checking micro-entreprise eligibility (turnover thresholds, existing
  business, regulated professions)
- Orienting a profile with partners, limited-liability needs, or a wish for
  assurance chômage toward the right family of structures

## When NOT to use

- Quantitative net-income comparisons — follow up with
  vestafolio-sasu-vs-eurl or vestafolio-micro-entreprise instead
- Tax or cotisation amounts (this tool returns none by design)
- Non-French business structures

## French tax context (as coded in the simulator, 2026)

- Micro-entreprise thresholds 2026: 203 100 € for vente and commerce,
  83 600 € for services and activités libérales.
- An existing business (`hasExistingBusiness`) makes a new micro-entreprise
  ineligible.
- Partners (`hasPartners`) orient toward pluripersonal forms (SARL, SAS)
  rather than unipersonal ones (EURL, SASU).
- Limited-liability needs orient toward a société; a wish for assurance
  chômage orients toward assimilé-salarié statuses (SASU, SAS, SELAS).
- Regulated professions libérales (médecin, avocat, architecte…) are oriented
  toward the SEL forms (SELARL/SELAS).

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/choisir-regime
```

Then POST the user's profile (amounts in euros):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/choisir-regime \
  -H 'Content-Type: application/json' \
  -d '{
    "activityCategory": "services_bnc",
    "projectedTurnover": 50000,
    "otherIncome": 0,
    "maritalStatus": "single",
    "numberOfChildren": 0,
    "hasExistingBusiness": false,
    "hasPartners": false,
    "needsLimitedLiability": false,
    "wantsUnemploymentInsurance": false
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `recommendedRegime` — the suggested structure with `recommendationReason`;
  may be null if nothing stands out
- `eligibleRegimes` — every eligible form with French `advantages` and
  `disadvantages` lists you can relay directly
- `microEntrepriseEligible`, `microEntrepriseThreshold`,
  `turnoverExceedsThreshold` — explain micro eligibility explicitly
- `explanations` — the reasoning trail in French, useful to justify the answer
- `nextSimulatorLink` on a regime — hand the user the matching Vestafolio
  simulator (e.g. /simulateurs/sasu-vs-eurl) for the quantitative follow-up

## Caveats

- Qualitative orientation only: no cotisation or tax math — for numbers,
  chain into vestafolio-sasu-vs-eurl or vestafolio-micro-entreprise.
- Eligibility rules as coded for 2026; legal thresholds evolve. Estimates,
  not legal advice — say so.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/choisir-regime
