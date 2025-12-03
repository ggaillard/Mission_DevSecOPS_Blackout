# ⚙️ DevSecOps — Fiche Concept

## 🎯 Objectif pédagogique
Comprendre comment intégrer la sécurité dans le cycle de développement logiciel (CI/CD) pour détecter et corriger les vulnérabilités avant la production.

---

## 🔍 Qu'est-ce que le DevSecOps ?

**DevSecOps** = Development + Security + Operations

C'est l'intégration de la sécurité à **chaque étape** du cycle de développement, plutôt qu'un contrôle en fin de projet.

### Évolution historique

```
Avant (Waterfall) :
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│  Dev     │  Test    │  Sécurité│  Ops     │ Production│
│ (6 mois) │ (2 mois) │ (1 mois) │ (1 mois) │           │
└──────────┴──────────┴──────────┴──────────┴──────────┘
                              ↑
                    Audit sécurité tardif
                    = Corrections coûteuses

Maintenant (DevSecOps) :
┌─────────────────────────────────────────────────────────┐
│                      CI/CD Pipeline                      │
├─────────────────────────────────────────────────────────┤
│  Code → Build → Test → Sécu → Deploy → Monitor          │
│    ↑       ↑       ↑      ↑       ↑        ↑            │
│   Lint   SAST   Tests  Scan   DAST    Alertes           │
│                       deps                               │
└─────────────────────────────────────────────────────────┘
   Sécurité intégrée partout = Corrections rapides
```

---

## 🔧 Les outils du pipeline DevSecOps

### Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────┐
│                        PIPELINE CI/CD                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐│
│  │  Code  │──▶│  Build │──▶│  Test  │──▶│ Deploy │──▶│ Monitor││
│  │        │   │        │   │        │   │        │   │        ││
│  └────────┘   └────────┘   └────────┘   └────────┘   └────────┘│
│      │            │            │            │            │      │
│      ▼            ▼            ▼            ▼            ▼      │
│  ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐│
│  │ Lint   │   │ SAST   │   │  Scan  │   │ DAST   │   │  SIEM  ││
│  │Pre-comm│   │SonarQu.│   │Dépend. │   │OWASP   │   │Alertes ││
│  │it hook │   │Semgrep │   │Snyk    │   │ZAP     │   │        ││
│  └────────┘   └────────┘   └────────┘   └────────┘   └────────┘│
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │             🔑 DÉTECTION DES SECRETS                        │ │
│  │             (GitLeaks, git-secrets, TruffleHog)             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Étape 1 : Linting & Pre-commit

### Objectif
Détecter les erreurs de style et les problèmes évidents **avant** le commit.

### Outils

| Langage | Linter | Commande |
|---------|--------|----------|
| Python | pylint, flake8 | `flake8 src/` |
| JavaScript | ESLint | `eslint .` |
| Java | Checkstyle | `mvn checkstyle:check` |
| Go | golangci-lint | `golangci-lint run` |

### Configuration pre-commit

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: check-yaml
      - id: check-added-large-files

  - repo: https://github.com/PyCQA/flake8
    rev: 6.1.0
    hooks:
      - id: flake8

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
```

---

## 🛡️ Étape 2 : SAST (Static Application Security Testing)

### Objectif
Analyser le **code source** pour détecter les vulnérabilités sans exécuter l'application.

### Ce que SAST détecte

| Vulnérabilité | Exemple |
|---------------|---------|
| Injection SQL | `f"SELECT * FROM users WHERE id={user_input}"` |
| XSS | `innerHTML = userInput` |
| Secrets hardcodés | `password = "admin123"` |
| Path Traversal | `open(f"/data/{filename}")` |
| Désérialisation | `pickle.loads(user_data)` |

### Outils populaires

| Outil | Langages | Type |
|-------|----------|------|
| **SonarQube** | 30+ langages | Open source / Commercial |
| **Semgrep** | Python, JS, Go, Java... | Open source |
| **Bandit** | Python | Open source |
| **Brakeman** | Ruby on Rails | Open source |
| **Snyk Code** | Multi | Commercial |

### Exemple : Intégration SonarQube

```yaml
# .github/workflows/ci.yml
- name: SonarQube Scan
  uses: sonarsource/sonarqube-scan-action@master
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

### Exemple de détection Semgrep

```python
# Règle Semgrep pour détecter les injections SQL
# rules/sql-injection.yaml
rules:
  - id: sql-injection
    patterns:
      - pattern: |
          $QUERY = f"SELECT ... {$VAR} ..."
    message: "Possible SQL injection"
    severity: ERROR
```

---

## 📦 Étape 3 : Scan des dépendances (SCA)

### Objectif
Détecter les vulnérabilités dans les **bibliothèques tierces**.

### Pourquoi c'est critique

| Statistique | Valeur |
|-------------|--------|
| % du code qui est des dépendances | 80-90% |
| Vulnérabilités dans npm packages | 1 sur 3 projets |
| Temps moyen pour patcher | 400+ jours |

### Exemple : Log4Shell (CVE-2021-44228)

```
Décembre 2021 : Vulnérabilité critique dans Log4j
├── Score CVSS : 10.0 (maximum)
├── Impact : Exécution de code à distance
├── Projets affectés : Des millions (Java, Minecraft, iCloud...)
└── Leçon : Vos dépendances sont votre responsabilité
```

### Outils de scan

| Outil | Écosystèmes | Intégration |
|-------|-------------|-------------|
| **Snyk** | npm, pip, Maven, Gradle, Go... | CLI, CI/CD, IDE |
| **Dependabot** | npm, pip, Maven, Bundler... | GitHub natif |
| **OWASP Dependency-Check** | Java, .NET, Ruby, Python | CLI, Jenkins |
| **npm audit** | npm | CLI native |

### Exemple : GitHub Dependabot

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
```

---

## 🔑 Étape 4 : Détection des secrets

### Objectif
Empêcher les mots de passe, clés API et tokens de se retrouver dans le code.

### Outils

| Outil | Mode | Utilisation |
|-------|------|-------------|
| **git-secrets** | Pre-commit | Bloque les commits avec secrets |
| **GitLeaks** | CI/CD | Scanne l'historique complet |
| **TruffleHog** | CLI/CI | Détection par entropie |
| **detect-secrets** | Pre-commit/CI | Baseline des faux positifs |

### Configuration GitLeaks

```yaml
# .gitleaks.toml
[allowlist]
description = "Allowlisted files"
paths = [
  '''test/.*''',
  '''.*_test\.go''',
]

# Dans le pipeline CI
- name: Gitleaks
  uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 🌐 Étape 5 : DAST (Dynamic Application Security Testing)

### Objectif
Tester l'application **en cours d'exécution** pour détecter les vulnérabilités exploitables.

### Différence SAST vs DAST

| Critère | SAST | DAST |
|---------|------|------|
| Quand | Avant exécution | Application déployée |
| Quoi | Code source | Comportement runtime |
| Faux positifs | Élevés | Faibles |
| Couverture | 100% du code | Chemins testés |
| Exemple | SonarQube | OWASP ZAP |

### OWASP ZAP - Configuration basique

```yaml
# .github/workflows/dast.yml
- name: ZAP Scan
  uses: zaproxy/action-baseline@v0.9.0
  with:
    target: 'https://staging.monapp.com'
    rules_file_name: '.zap/rules.tsv'
```

### Types de scans DAST

| Type | Durée | Profondeur |
|------|-------|------------|
| **Baseline** | 1-2 min | Surface (spidering) |
| **Full** | 1-2h | Complet (fuzzing) |
| **API** | 30 min | Endpoints REST/GraphQL |

---

## 📊 Métriques DevSecOps

### Indicateurs clés (KPIs)

| Métrique | Cible | Signification |
|----------|-------|---------------|
| **MTTR** (Mean Time To Remediate) | < 7 jours | Temps moyen de correction |
| **Taux de faux positifs** | < 20% | Qualité des alertes |
| **Couverture SAST** | > 80% | % du code analysé |
| **Vulnérabilités critiques ouvertes** | 0 | Risque immédiat |
| **Âge moyen des vulnérabilités** | < 30 jours | Hygiène sécurité |

### Dashboard type

```
┌─────────────────────────────────────────────────────────────┐
│                   DASHBOARD DEVSECOPS                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Vulnérabilités ouvertes          Tendance (30 jours)      │
│   ┌─────────────────────┐          ┌─────────────────────┐  │
│   │ Critiques:    2     │          │  ▁▂▃▂▄▃▂▁▂▃▂▁      │  │
│   │ Hautes:      12     │          │  ────────────────   │  │
│   │ Moyennes:   34      │          │  -15% ce mois       │  │
│   │ Faibles:    89      │          └─────────────────────┘  │
│   └─────────────────────┘                                    │
│                                                              │
│   MTTR par sévérité              Top 5 vulnérabilités       │
│   ┌─────────────────────┐        ┌─────────────────────────┐│
│   │ Critique: 2 jours   │        │ 1. SQL Injection (3)    ││
│   │ Haute:    5 jours   │        │ 2. XSS Stored (8)       ││
│   │ Moyenne: 14 jours   │        │ 3. IDOR (5)             ││
│   │ Faible:  45 jours   │        │ 4. CSRF (12)            ││
│   └─────────────────────┘        │ 5. Open Redirect (4)    ││
│                                  └─────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Pipeline complet - Exemple GitHub Actions

```yaml
# .github/workflows/devsecops.yml
name: DevSecOps Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint Python
        run: |
          pip install flake8
          flake8 src/ --count --show-source

  secrets-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: GitLeaks
        uses: gitleaks/gitleaks-action@v2

  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Snyk
        uses: snyk/actions/python@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  dast:
    needs: [lint, sast, secrets-scan, dependency-scan]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: ./deploy-staging.sh
      - name: OWASP ZAP
        uses: zaproxy/action-baseline@v0.9.0
        with:
          target: 'https://staging.app.com'

  deploy-prod:
    needs: [dast]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: ./deploy-prod.sh
```

---

## 📚 Pour aller plus loin

- [OWASP DevSecOps Guideline](https://owasp.org/www-project-devsecops-guideline/)
- [NIST Secure Software Development Framework](https://csrc.nist.gov/Projects/ssdf)
- [DevSecOps Maturity Model (DSOMM)](https://dsomm.timo-pagel.de/)

---

## ✏️ Exercice pratique

Votre pipeline CI actuel ressemble à ceci :

```yaml
jobs:
  build:
    steps:
      - checkout
      - npm install
      - npm run build
      - npm run test
      - deploy-to-prod
```

Transformez-le en pipeline DevSecOps complet.

<details>
<summary>Voir la solution</summary>

```yaml
jobs:
  # 1. Qualité du code
  lint:
    steps:
      - checkout
      - run: npm run lint

  # 2. Détection de secrets
  secrets:
    steps:
      - checkout
      - uses: gitleaks/gitleaks-action@v2

  # 3. SAST
  sast:
    steps:
      - checkout
      - uses: github/codeql-action/analyze@v2

  # 4. Scan des dépendances
  deps:
    steps:
      - checkout
      - run: npm audit --audit-level=high
      - uses: snyk/actions/node@master

  # 5. Build & Tests
  test:
    needs: [lint, secrets, sast, deps]
    steps:
      - checkout
      - run: npm install
      - run: npm run build
      - run: npm run test

  # 6. DAST (en staging)
  dast:
    needs: [test]
    steps:
      - run: deploy-staging
      - uses: zaproxy/action-baseline@v0.9.0
        with:
          target: 'https://staging.app.com'

  # 7. Déploiement prod (si tout passe)
  deploy:
    needs: [dast]
    if: github.ref == 'refs/heads/main'
    steps:
      - run: deploy-prod
```

**Étapes ajoutées** : Lint, Secrets, SAST, Deps scan, DAST
**Dépendances** : Le déploiement ne se fait que si tout passe
</details>

---

*Fiche créée pour le BTS SIO SLAM — Mission Blackout*