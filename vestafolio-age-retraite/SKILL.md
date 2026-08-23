---
name: vestafolio-age-retraite
description: Simulate early retirement (FIRE) financed by invested capital for a French saver using Vestafolio's simulator API. Use when a user asks "à quel âge puis-je arrêter de travailler", "quand serai-je financièrement indépendant", at what age they can retire early, how much capital they need for FIRE, whether their savings will last, or how part-time income and expense cuts change their retirement age.
---

# Âge de retraite anticipée / FIRE (Vestafolio)

## Response language

Reply in French whenever the user speaks or writes in French.

Simulates a capital-funded early retirement: year-by-year accumulation of
savings until the target retirement age (salary, expenses, mortgage,
investment returns), then withdrawals through retirement up to a life
expectancy of 86, with the required capital, goal status and improvement
levers.

## When to use

- "À quel âge puis-je arrêter de travailler ?" / "At what age can I retire early?"
- FIRE questions: capital needed, whether current savings + monthly surplus
  suffice, how long the capital lasts
- "What if I work 2 more years / cut expenses / keep a side gig?" follow-ups

## When NOT to use

- Statutory French pension rights (trimestres, âge légal, pension amounts) —
  this is a pure capital-drawdown simulation
- Choosing an investment envelope (use vestafolio-pea-vs-cto)
- Non-capital questions like budgeting alone

## French tax context (as coded in the simulator)

- Withdrawals from capital during retirement are taxed at a flat 31.4 %
  (PFU majoré: 12.8 % IR + 18.6 % prélèvements sociaux).
- Expenses are inflated at 3 % per year.
- Life expectancy is fixed at 86; capital sustainability is simulated up to a
  cap of 120 years.

Use this context to sanity-check results, not to compute yourself — call the API.

## How to call the API

Always fetch the canonical input schema first (fields, bounds, defaults):

```
GET https://www.vestafolio.com/api/tools/v1/age-retraite
```

Every field has a default, so a minimal call with just the user's key numbers
works — but confirm the defaults match their situation. Then POST (amounts in
euros, rates in percent):

```bash
curl -s -X POST https://www.vestafolio.com/api/tools/v1/age-retraite \
  -H 'Content-Type: application/json' \
  -d '{
    "currentSavings": 120000,
    "monthlyNetSalary": 3500,
    "annualSalaryIncrease": 1,
    "otherMonthlyIncome": 0,
    "currentMonthlyExpenses": 1800,
    "monthlyMortgagePayment": 0,
    "remainingMortgageYears": 0,
    "currentAge": 40,
    "desiredRetirementAge": 53,
    "hasPartTimeActivity": true,
    "partTimeMonthlyIncome": 600,
    "annualPartTimeIncomeIncrease": 1,
    "partTimeIncomeEndAge": 85,
    "maintainLifestyle": true,
    "revisedMonthlyExpenses": 1800,
    "annualInvestmentReturn": 7
  }'
```

Unknown fields are rejected (strict schema) — if you get a `validation_error`,
re-read the schema from the GET endpoint rather than guessing field names.

## Interpreting the output

- `objectiveReached` — the headline verdict: can the user retire at the target
  age without ever exhausting savings (up to age 86)
- `capitalAtRetirement` vs `capitalNecessary` — projected capital at the
  target age vs the capital actually required; `completionPercent` is their
  ratio. If projected < necessary, the goal is not funded
- `sustainableUntilAge` — the age until which the projected capital lasts
  (`sustainableUntilAgeIsCapped` means it lasts beyond the 120-year simulation
  bound, i.e. effectively forever)
- `improvement` — concrete levers when the goal is missed: extra work years,
  first achievable retirement age, monthly expense cut, or years of 800 €/month
  part-time work that would close the gap
- `preRetirementProjection` / `retirementProjection` — year-by-year series for
  charts and "what happens at age N" follow-ups; `minClosingCapital` below zero
  pinpoints when savings run out
- `inputs` — the normalized values actually used (ages are clamped, e.g.
  `desiredRetirementAge` raised to `currentAge`)

## Caveats

- Constant assumptions (returns, salary growth, 3 % expense inflation, flat
  31.4 % tax on withdrawals) — real sequences of returns and future tax law
  will differ. Estimates, not financial advice — say so.
- Ignores statutory pension income; from légal retirement age onward the real
  situation is usually better than simulated.
- Cite the interactive simulator to the user:
  https://www.vestafolio.com/simulateurs/age-retraite
