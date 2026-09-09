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

## OpenWebUI : charger les instructions et activer l'exécution

Importez le `SKILL.md` dans **Espace de travail → Skills**, avec tout son
contenu Markdown. Vérifiez que la skill enregistrée est active et accessible
à l'utilisateur du chat. Cette copie doit être mise à jour après modification
des fichiers : publier ce dépôt ne met pas à jour votre espace OpenWebUI.

Sélectionnez la skill avec le **sélecteur `$`** ou **+ → Skills** dans le chat
pour injecter ses instructions complètes. Une skill attachée au modèle
présente seulement son nom et sa description : le modèle doit appeler
`view_skill` pour lire le contenu. Un nom ou une mention visible ne prouve pas
que les instructions ont été transmises. Consultez la
[documentation des skills OpenWebUI](https://docs.openwebui.com/features/workspace/skills/).

Activez les **appels de fonctions natifs** et un outil d'exécution dans le même
chat : terminal, outil HTTP/OpenAPI ou interpréteur de code. Pour ce dernier,
vérifiez l'activation globale, la capacité du modèle, les droits de
l'utilisateur et l'activation dans le chat ; l'outil natif est `execute_code`.
Python suffit pour appeler l'API, à condition de pouvoir accéder au réseau
en HTTPS. Sur un serveur, utilisez `urllib.request` ; dans Pyodide côté
navigateur, utilisez `pyodide.http.pyfetch`. Voir la
[configuration de l'interpréteur OpenWebUI](https://docs.openwebui.com/features/chat-conversations/chat-features/code-execution/python/)
et la [documentation HTTP de Pyodide](https://pyodide.org/en/stable/usage/api/python-api/http.html).

Si vous utilisez déjà un serveur d'outils OpenAPI, le schéma est disponible à
`https://www.vestafolio.com/api/tools/v1/openapi.json`, avec notamment
`describe_micro_entreprise` et `run_micro_entreprise`. La skill fournit les
questions à poser ; les outils connectés exécutent les calculs. Importer une
skill n'ajoute pas automatiquement ces outils.

### Identifier la panne

Essayez ces deux demandes dans un nouveau chat avec la skill sélectionnée :

1. « Avec ton outil d'exécution, fais un GET sur
   https://www.vestafolio.com/api/tools/v1/micro-entreprise et affiche le
   champ `slug` reçu. » Il faut un appel d'outil réel et `micro-entreprise`
   dans le résultat. Du code simplement affiché ne suffit pas. Une erreur
   réseau prouve une tentative d'appel, mais pas l'accès à l'API.
2. « Un ami souhaite lancer son entreprise pour faire du consulting. Il vise
   50K€ de chiffre d'affaires et 10k€ de charges réelles. Quel régime adopter ? »
   Le modèle doit demander les informations manquantes avant tout verdict.
   Aucun POST n'est encore attendu. Après vos réponses et votre accord sur
   les données transmises, il doit lire le schéma puis envoyer un POST conservant `annualRevenue: 50000` et
   `chargesReelles: 10000`.

Si le premier test réussit mais que le second ignore des instructions dont
l'injection complète est vérifiée, examinez les conflits de consignes ou le
suivi des instructions par le modèle. Si les outils sont absents, corrigez
d'abord leur exposition. Si un appel échoue, examinez son erreur. L'absence
de `view_skill` est normale pour une skill explicitement sélectionnée : son
contenu est déjà injecté. Pour signaler un échec, conservez les versions
d'OpenWebUI et de la skill, les instructions et outils effectivement transmis
et la trace des appels, sans identifiants secrets ni données personnelles.

Les skills sont des instructions, pas un mécanisme de contrôle. Pour garantir
une réponse calculée, l'application hôte doit refuser ou relancer une
recommandation chiffrée sans résultat de calcul réussi. Les tests de schéma
du dépôt ne prouvent pas qu'un modèle ou une configuration OpenWebUI suit
les instructions.

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

- **Expliquer le partage avant l'envoi.** Résumer les données transmises à
  l'API externe de Vestafolio et obtenir l'accord de l'utilisateur. Réutiliser
  un accord explicite couvrant déjà cette destination et ces données. En cas
  de refus, proposer le simulateur en saisie manuelle, sans données personnelles
  préremplies. Omettre les champs inutilisés : `choisir-regime` n'a pas besoin
  de la situation familiale, des enfants ni des revenus du foyer.
- **Exécuter avant de présenter un résultat.** Utiliser l'outil HTTP, terminal
  ou Python disponible, vérifier l'enveloppe `ok`/`result` de l'API et signaler
  les échecs. Ne jamais remplacer un appel manquant par un calcul de mémoire.
- **Poser les questions avant de calculer.** Chaque skill contient une liste
  « Questions to ask before calling the API » qui reprend les questions posées
  par le simulateur web, dans le même ordre et avec les mêmes conditions (par
  exemple, le simulateur micro-entreprise demande si l'entreprise bénéficie de
  l'ACRE, et seulement en première année, avant tout calcul). L'agent doit
  recueillir ou confirmer ces réponses au lieu de supposer des valeurs par
  défaut, sauf pour les champs explicitement documentés comme inutilisés et
  omis. La CI vérifie que chaque champ d'entrée du schéma d'un simulateur est
  couvert par sa skill.
- **Les taux viennent du code des simulateurs, et de nulle part ailleurs.** Les
  règles et constantes fiscales citées dans chaque skill (prélèvements sociaux,
  seuils, abattements, barèmes...) sont celles codées dans les calculateurs de
  Vestafolio, afin que les explications de l'agent correspondent toujours aux
  chiffres renvoyés par l'API. Les skills ne mobilisent des données externes
  que pour des sujets non couverts par les simulateurs, en le précisant.
- Tous les montants sont en euros ; les taux en pourcentage sauf indication
  contraire du schéma.
- Les résultats reflètent les règles fiscales françaises codées dans les
  simulateurs (barèmes 2025-2026) et sont des estimations, pas un conseil
  fiscal.
- Chaque skill référence la page du simulateur destinée aux humains
  (`/simulateurs/{slug}`) — proposez-la si elle aide à explorer les hypothèses,
  en respectant les demandes de réponse sans liens.
