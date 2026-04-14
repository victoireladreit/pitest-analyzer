# pitest-analyzer

Analyse en profondeur un rapport PiTest pour un projet Maven Java/Kotlin.
Deux formats : un **prompt standalone** compatible avec tous les agents IA (Claude, ChatGPT, Gemini, Copilot…), et un **skill installable** pour Claude qui se déclenche automatiquement.

→ **[Page de téléchargement](https://victoireladreit.github.io/pitest-analyzer)**

---

## Ce que ça fait

Donné un projet Maven avec des tests, le skill :

1. **Localise le rapport XML** dans les chemins Maven standard — ou le génère si absent après vérification de la config `pom.xml`
2. **Lit le code source et les tests** des classes mutées avant d'analyser (jamais à l'aveugle)
3. **Produit 4 sections actionnables** :
   - 🐛 Mutations critiques avec score de risque (🔴/🟠/🟡) et explication de pourquoi le test ne tue pas le mutant
   - 🧟 Faux positifs à exclure avec la config `pom.xml` exacte
   - 🛡️ Snippets de tests à ajouter, dans le style de test détecté (AAA/BDD, AssertJ/Kotest...)
   - 📊 Bilan, estimation du gain de score, suggestions de config PiTest

---

## Utiliser le prompt standalone

Compatible avec tous les assistants IA : Claude, ChatGPT, Gemini, GitHub Copilot, Cursor…

- **Texte brut** : [`docs/prompt.md`](docs/prompt.md) ou [depuis GitHub Pages](https://victoireladreit.github.io/pitest-analyzer/prompt.md)
- **Avec aperçu** : [page de téléchargement](https://victoireladreit.github.io/pitest-analyzer) → section "Maxi-prompt"

Coller dans une conversation ouverte depuis la racine du projet Maven.

---

## Installer le skill (Claude Code)

Télécharger [`pitest-analyzer.skill`](https://victoireladreit.github.io/pitest-analyzer/pitest-analyzer.skill) et extraire dans `~/.claude/skills/` (global) ou `.claude/skills/` (projet).

## Rules pour éditeurs IA (Cursor, Windsurf, Copilot, Gemini)

Télécharger le fichier correspondant depuis [la page de téléchargement](https://victoireladreit.github.io/pitest-analyzer) et le placer à la racine du projet Maven.

| Éditeur | Nom cible |
|---|---|
| Cursor | `.cursorrules` |
| Windsurf | `.windsurfrules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Gemini CLI | `GEMINI.md` |

---

## Structure du repo

```
pitest-analyzer/
├── docs/
│   ├── index.html                              ← GitHub Pages (landing page)
│   ├── prompt.md                               ← prompt texte brut (tous agents IA)
│   ├── llms.txt                                ← index lisible par les agents IA
│   ├── pitest-analyzer.skill                   ← skill installable (Claude)
│   ├── pitest-analyzer.cursorrules             ← rules (Cursor)
│   ├── pitest-analyzer.windsurfrules           ← rules (Windsurf)
│   ├── pitest-analyzer.copilot-instructions.md ← instructions (GitHub Copilot)
│   └── pitest-analyzer.gemini.md              ← instructions (Gemini CLI)
└── README.md
```

---

## Prérequis

- Projet Maven (Java ou Kotlin)
- Plugin `pitest-maven` configuré dans le `pom.xml` (ou laisser le skill vérifier/lancer)
- N'importe quel assistant IA pour le prompt standalone, Claude avec les skills activés pour le `.skill`

Pour Kotlin : vérifier la présence du plugin `pitest-kotlin` dans les dépendances.

---

*Talk "Testez vos tests avant qu'ils ne vous trahissent : le mutation testing!"*
