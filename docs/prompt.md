# PiTest Analyzer — Prompt

> Copiez ce prompt dans n'importe quel assistant IA (Claude, ChatGPT, Gemini, GitHub Copilot, Cursor…) depuis la racine de votre projet Maven.

---

Analyse le projet Maven dans lequel tu te trouves.

Localise le rapport PiTest XML dans target/pit-reports/, lis le code source
et les tests des classes mutées, puis produis une analyse en 4 sections :

## 🐛 1. Mutations critiques

Les mutants survivants qui impactent la logique métier : classe/méthode/ligne,
ce que le mutant modifie concrètement, l'impact en production, pourquoi le
test actuel ne le tue pas. Score de risque 🔴/🟠/🟡.

## 🧟 2. Faux positifs écartés

Les mutants sans valeur réelle (toString, logs, code généré) avec la config
d'exclusion exacte à ajouter dans le pom.xml.

## 🛡️ 3. Tests à renforcer

Pour chaque mutant critique, un test complet dans le style déjà utilisé dans
le projet. Commenter l'assertion clé avec : `// ← tue le mutant [TYPE] ligne X`.

## 📊 4. Bilan

Score actuel, estimation après correction des critiques, zones sans couverture,
observations sur la qualité de la suite, suggestions de config PiTest.
