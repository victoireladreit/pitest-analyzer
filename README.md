# pitest-analyzer

Analyse en profondeur un rapport PiTest pour un projet Maven Java/Kotlin.  
Deux formats : un **prompt standalone** à coller dans Claude, et un **skill installable** qui se déclenche automatiquement.

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

Copier le contenu de [`docs/index.html`](docs/index.html) → section "Maxi-prompt", ou directement depuis [la page web](https://victoireladreit.github.io/pitest-analyzer).

Coller dans une conversation Claude ouverte depuis la racine du projet Maven.

---

## Installer le skill

1. Télécharger [`pitest-analyzer.skill`](https://victoireladreit.github.io/pitest-analyzer/pitest-analyzer.skill)
2. Dans Claude.ai → Settings → Skills → Install from file
3. Ouvrir Claude depuis la racine du projet — le skill se déclenche automatiquement dès qu'on mentionne PiTest ou mutation testing

---

## Structure du repo

```
pitest-analyzer/
├── docs/
│   ├── index.html              ← GitHub Pages (landing page)
│   └── pitest-analyzer.skill   ← fichier d'installation du skill
└── README.md
```

---

## Prérequis

- Projet Maven (Java ou Kotlin)
- Plugin `pitest-maven` configuré dans le `pom.xml` (ou laisser le skill vérifier/lancer)
- Claude avec les skills activés (pour le `.skill`), ou n'importe quel accès Claude (pour le prompt)

Pour Kotlin : vérifier la présence du plugin `pitest-kotlin` dans les dépendances.

---

*Talk "Testez vos tests avant qu'ils ne vous trahissent : le mutation testing!" — le prompt c'est l'expérimentation, le skill c'est l'industrialisation.*
