🇬🇧 [English version](README.md)

# Vestafolio Agent Skills

Une skill par simulateur Vestafolio, pour apprendre aux agents IA à répondre
aux questions de finances personnelles françaises avec de vrais calculs plutôt
qu'avec des chiffres inventés. Chaque skill appelle l'API de calcul publique,
sans authentification, disponible sur
`https://www.vestafolio.com/api/tools/v1`.

## Installation

Avec la [CLI skills](https://github.com/vercel-labs/skills) (Claude Code,
Codex, Cursor, Gemini CLI et tout agent compatible avec le format
[Agent Skills](https://agentskills.io)) :

```bash
npx skills add Vestafolio/skills           # choisir les skills interactivement
npx skills add Vestafolio/skills --all     # tout installer
npx skills add Vestafolio/skills --skill vestafolio-pea-vs-cto
```

Ou manuellement : copiez n'importe quel dossier `vestafolio-*` dans le
répertoire de skills de votre agent (par exemple `~/.claude/skills/` pour
Claude Code).

## Comment les skills restent synchronisées avec l'API

Les skills ne dupliquent jamais les listes de champs. Chacune demande à
l'agent de récupérer le schéma canonique au moment de l'appel :

- `GET https://www.vestafolio.com/api/tools/v1/{slug}` — JSON Schemas
  d'entrée/sortie plus un `exampleInput` complet
- `GET https://www.vestafolio.com/api/tools/v1` — liste complète des outils
- `GET https://www.vestafolio.com/api/tools/v1/openapi.json` — OpenAPI 3.1
- `GET https://www.vestafolio.com/llms.txt` — index agent du site

L'unique exemple curl intégré à chaque skill est validé en CI contre les vrais
schémas zod des simulateurs — si un changement de schéma casse un exemple, le
build échoue avant que la skill ne soit publiée.

## Conventions

- Tous les montants sont en euros ; les taux en pourcentage sauf indication
  contraire du schéma.
- Les résultats reflètent les règles fiscales françaises codées dans les
  simulateurs (barèmes 2025-2026) et sont des estimations, pas un conseil
  fiscal.
- Chaque skill référence la page du simulateur destinée aux humains
  (`/simulateurs/{slug}`) — citez-la aux utilisateurs pour qu'ils puissent
  explorer interactivement.
