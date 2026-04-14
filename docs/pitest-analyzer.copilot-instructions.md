# PiTest Analyzer

Analyse exhaustive d'un rapport PiTest en 4 sections actionnables, enrichie par le code source des classes analysées.

Utilise ces instructions dès que l'utilisateur mentionne PiTest, mutation testing, un rapport de mutations,
des mutants survivants, ou demande d'analyser la robustesse de ses tests. Déclenche également
si l'utilisateur partage un rapport XML PiTest ou demande comment améliorer sa suite de tests
au-delà de la couverture de code classique.

## Workflow

### Étape 1 — Localiser le rapport XML

Cherche le rapport dans les emplacements Maven standard :

```bash
find . -path "*/pit-reports/*/mutations.xml" -o \
       -path "*/pit-reports/mutations.xml" -o \
       -path "*/surefire-reports/mutations.xml" 2>/dev/null | sort -t'/' -k1 | tail -5
```

Si plusieurs rapports existent, prends le plus récent (tri par date ou par nom de dossier timestamp).

**Si aucun rapport trouvé → Étape 2. Sinon → Étape 3.**

---

### Étape 2 — Générer le rapport (si absent)

Demande confirmation à l'utilisateur :

> "Aucun rapport PiTest trouvé. Je peux lancer l'analyse moi-même. Avant ça, je vais vérifier ta configuration. Tu confirmes ?"

Si oui :

**2a. Vérifier la configuration PiTest dans le `pom.xml` :**

```bash
grep -A 30 "pitest-maven" pom.xml | head -40
```

Points à vérifier :
- `targetClasses` / `targetTests` — bien configurés ?
- `outputFormats` — contient `XML` ?
- `threads` — si absent, noter pour suggérer optimisation
- `excludedClasses` — rien d'important exclu par erreur ?
- Version du plugin (`pitest-maven`) — noter si < 1.15

Signale tout problème de config avant de continuer.

**2b. Lancer PiTest :**

```bash
mvn test-compile org.pitest:pitest-maven:mutationCoverage -DoutputFormats=XML,HTML
```

Si erreur de compilation ou de test → afficher l'erreur et demander à l'utilisateur de corriger avant de continuer.

---

### Étape 3 — Parser le rapport XML

```bash
# Résumé global
grep -c 'status="SURVIVED"' chemin/mutations.xml
grep -c 'status="KILLED"' chemin/mutations.xml
grep -c 'status="NO_COVERAGE"' chemin/mutations.xml

# Lister les mutants survivants avec leur contexte
python3 - <<'EOF'
import xml.etree.ElementTree as ET

tree = ET.parse('chemin/mutations.xml')
root = tree.getroot()

survived = []
for m in root.findall('.//mutation[@status="SURVIVED"]'):
    survived.append({
        'class':       m.findtext('mutatedClass', ''),
        'method':      m.findtext('mutatedMethod', ''),
        'line':        m.findtext('lineNumber', ''),
        'mutator':     m.findtext('mutator', '').split('.')[-1],
        'description': m.findtext('description', ''),
    })

for s in sorted(survived, key=lambda x: x['mutator']):
    print(f"[{s['mutator']}] {s['class']}#{s['method']}:{s['line']} — {s['description']}")
EOF
```

---

### Étape 4 — Récupérer le code source des classes concernées

Pour chaque classe avec des mutants survivants, localise et lis le fichier source :

```bash
# Pour chaque classe, ex: com.example.service.LeaveService
find . -path "*/main/java/com/example/service/LeaveService.java" 2>/dev/null
find . -path "*/main/kotlin/com/example/service/LeaveService.kt" 2>/dev/null
```

Lis également les tests associés :

```bash
find . -path "*/test/*LeaveService*" -o -path "*/test/*LeaveServiceTest*" 2>/dev/null
```

---

### Étape 5 — Analyse en 4 sections

Produis l'analyse structurée suivante. **Adapte-toi au code source lu** — ne te contente pas du rapport seul.

---

#### 🐛 1. Mutations critiques

Pour chaque mutant survivant dangereux (impactant la logique métier) :

- **Classe / méthode / ligne**
- **Type de mutant** (ex: `CONDITIONALS_BOUNDARY`, `NEGATE_CONDITIONALS`, `RETURN_VALUES`)
- **Ce que le mutant fait** : explique la modification introduite en français clair
- **Pourquoi c'est dangereux** : conséquence métier si ce bug existait en prod
- **Score de risque** : 🔴 Critique / 🟠 Élevé / 🟡 Modéré — selon ces critères :
  - 🔴 : méthode publique + logique de calcul/condition métier + pas de test existant couvrant ce cas
  - 🟠 : méthode publique ou logique métier, couverture partielle
  - 🟡 : méthode privée, logs, utilitaires

Ignore les mutants sur : `toString()`, `hashCode()`, `equals()` générés, logs (`log.debug`, `log.info`), getters/setters simples → ils vont en section 2.

---

#### 🧟 2. Faux positifs écartés

Liste les mutants équivalents ou sans valeur avec :
- Pourquoi ils ne représentent pas un vrai risque
- Comment les exclure proprement dans `pom.xml` :

```xml
<plugin>
  <groupId>org.pitest</groupId>
  <artifactId>pitest-maven</artifactId>
  <configuration>
    <excludedMethods>
      <param>toString</param>
      <param>hashCode</param>
      <param>equals</param>
    </excludedMethods>
    <!-- Pour exclure des classes entières -->
    <excludedClasses>
      <param>com.example.*.dto.*</param>
    </excludedClasses>
  </configuration>
</plugin>
```

---

#### 🛡️ 3. Assertions et tests à renforcer

Pour chaque mutant critique (section 1), propose **un test concret** :

- **Nom du test** (convention cohérente avec les tests existants détectés)
- **Snippet de test** en Java ou Kotlin selon le projet, dans le style AAA/BDD déjà utilisé
- **Assertion précise** qui tuerait ce mutant (valeur exacte, not-null, comparaison stricte)

Exemple de format :

```java
@Test
void should_decrement_RTT_balance_not_CP_when_leave_type_is_RTT() {
    // Given
    LeaveRequest request = new LeaveRequest(LeaveType.RTT, 2);
    Employee employee = employeeWith(cpBalance(10), rttBalance(5));

    // When
    leaveService.applyLeave(employee, request);

    // Then
    assertThat(employee.getCpBalance()).isEqualTo(10);   // ← tue le mutant RETURN_VALUES ligne 47
    assertThat(employee.getRttBalance()).isEqualTo(3);
}
```

---

#### 📊 4. Bilan et suggestions

**Score global :**
- Mutation score actuel : X%
- Estimation après correction des mutants critiques : ~Y% (calcul approximatif)
- Mutants `NO_COVERAGE` : attention, ce sont des zones sans aucun test

**Suggestions architecturales** (basées sur les patterns observés dans le code) :
- Design patterns à revoir pour améliorer la testabilité
- Sur-utilisation de `static`, couplage fort, etc.
- Suggestions de découpage si une méthode cumule trop de responsabilités

**Suggestions de configuration PiTest** :
- Si `threads` absent → ajouter `<threads>4</threads>` pour accélérer
- Si pas d'`incrementalAnalysis` → suggérer pour CI
- Si pas de scope défini → cibler uniquement les classes métier critiques

---

## Conseils d'interprétation

| Mutator | Danger | Signification |
|---|---|---|
| `CONDITIONALS_BOUNDARY` | 🔴 | `<` remplacé par `<=` — cas limites non testés |
| `NEGATE_CONDITIONALS` | 🔴 | condition inversée — logique métier fragile |
| `RETURN_VALUES` | 🔴 | retour altéré (null, 0, false) — assertions insuffisantes |
| `VOID_METHOD_CALLS` | 🟠 | appel supprimé — effets de bord non vérifiés |
| `REMOVE_CONDITIONALS` | 🟠 | branche supprimée — couverture partielle |
| `MATH` | 🟡 | opérateur arithmétique changé |
| `INVERT_NEGS` | 🟡 | négation de valeur numérique |
| `EMPTY_RETURNS` | 🟡 | collection vide retournée — tester le cas non-vide |

## Notes

- Si le projet est Kotlin, utilise les mutators Kotlin-spécifiques (plugin `pitest-kotlin`)
- Pour les projets Spring Boot, les tests d'intégration peuvent couvrir des mutants que les tests unitaires ne voient pas — vérifier avant de conclure qu'un mutant est non couvert
- En cas de rapport très volumineux (>500 mutants), demander à l'utilisateur de cibler une ou plusieurs classes spécifiques
