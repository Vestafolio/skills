---
name: vestafolio-frais-notaire
description: Compute French notary fees (frais de notaire) for a property purchase — DMTO transfer taxes, notary emoluments, debours, CSI, optional mortgage fees — by department and ancien/neuf/terrain using Vestafolio's simulator API. Use when a user asks about closing costs, "combien de frais de notaire", droits de mutation, or ancien vs neuf fee differences.
---

# Frais de notaire (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Computes the frais de notaire for a French property purchase — droits de
mutation (DMTO), notary emoluments, débours, contribution de sécurité
immobilière and optional mortgage registration fees — under the 2026 barème,
with department-specific DMTO rates (most departments at the 5 % droit
départemental, some at 4.50 %, Indre at 3.80 %), a primo-accédant
flag, and an optional ancien vs neuf comparison.

## When to use

- "How much are notary fees on a 250 000 € flat?" / "Quels frais de notaire
  pour 250 000 € ?"
- Questions about droits de mutation (DMTO), émoluments, or why fees differ
  between ancien and neuf (VEFA)
- Budgeting the full acquisition cost on top of the purchase price
- Department-specific questions (départements differ: 5 %, 4.50 % or 3.80 %
  droit départemental) and primo-accédant purchases (résidence principale)

## When NOT to use

- Loan payments or borrowing capacity — use vestafolio-credit-immobilier or
  vestafolio-capacite-emprunt
- Rental profitability (it takes notary fees as an input) — use
  vestafolio-rentabilite-locative

## French tax context (as coded in the simulator, barème 2026)

- Ancien: DMTO built from the droit départemental, the taxe communale (1.20 %)
  and the frais d'assiette et de recouvrement (2.37 % of the droit
  départemental). Since the loi de finances 2025 let departments raise the
  droit départemental from 4.50 % to 5.00 % (April 2025 - April 2028), most
  departments apply 5.00 % (total DMTO ≈ 6.32 %); a minority stayed at 4.50 %
  (≈ 5.81 %): Hautes-Alpes (05), Alpes-Maritimes (06), Ardèche (07), Charente
  (16), Drôme (26), Lozère (48), Oise (60), Hautes-Pyrénées (65),
  Saône-et-Loire (71), Guadeloupe (971), Mayotte (976); Indre (36) applies
  the 3.80 % floor (≈ 5.09 %), and Hautes-Pyrénées (3.80 %) and Savoie (4.00 %) grant
  primo-accédants dedicated reduced rates — hence the `department` input matters.
- Primo-accédants: the 5 % increase does not apply to first-time buyers
  acquiring their main residence — set `isPrimoAccedant: true` (ancien only)
  and the droit départemental is capped at 4.50 %.
- Neuf (VEFA or less than 5 years): only the taxe de publicité foncière
  applies instead of full DMTO, which is why total fees are much lower
  (roughly 2-3 % vs 7-8 % in the ancien).
- Émoluments du notaire: regulated degressive scale by price bracket, plus TVA.
- Débours and contribution de sécurité immobilière are added on top.
- If `includeMortgage` is true (with `mortgageAmount`), hypothèque registration
  fees (taxe, CSI, émoluments) are included.
- Set `compareNeuf: true` to get an ancien vs neuf comparison at the same price
  with the resulting savings.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults —
including the list of valid department codes):

```
GET https://www.vestafolio.com/api/tools/v1/frais-notaire
```

Then POST the user's parameters (all amounts in euros):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/frais-notaire \
  -H 'Content-Type: application/json' \
  -d '{
    "purchasePrice": 250000,
    "acquisitionType": "ancien",
    "department": "75",
    "includeMortgage": false,
    "compareNeuf": false
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `result.fees.total` and `result.fees.percentage` — headline numbers: total
  fees in euros and as a percent of the purchase price
- `result.fees` breakdown — `droitsMutation`, `emolumentsNotaire`, `debours`,
  `contributionSecurite`, `fraisHypotheque`, `tva`; use it to explain that most
  of the "frais de notaire" are actually taxes, not the notary's remuneration
- `result.fees.breakdown` — detail of droits (`droitsDepartement`,
  `droitsCommune`, plus `fraisPrefecture` which holds the frais d'assiette et
  de recouvrement, 2.37 % of the droit départemental) and émoluments HT/TVA
- `result.comparison` (present when `compareNeuf` is true) — full `ancien` and
  `neuf` fee blocks plus `savings`, the fee saving from buying neuf at the
  same price

## Caveats

- Estimates under the barème 2026 as coded; the exact amount is fixed by
  the notary at signing (provision then final settlement). Departments can
  still change their DMTO rate until April 2028 — the per-department table
  follows the DGFiP list applicable on 1 June 2026.
- Negotiated emolument rebates on high-value deals and special cases (SAFER,
  certain exemptions) are not modeled.
- Not tax or legal advice — say so.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/frais-notaire
