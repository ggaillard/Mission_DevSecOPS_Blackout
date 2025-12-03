# 🔑 Gestion des Secrets — Fiche Concept

## 🎯 Objectif pédagogique
Apprendre à gérer les informations sensibles (mots de passe, clés API, tokens) de manière sécurisée dans vos projets.

---

## 🔍 Qu'est-ce qu'un "secret" ?

Un **secret** est toute information sensible qui donne accès à des ressources :

| Type | Exemples |
|------|----------|
| **Credentials** | Mots de passe, tokens d'accès |
| **Clés API** | Stripe, Twilio, Google Maps |
| **Certificats** | SSL/TLS, SSH |
| **Clés de chiffrement** | AES, RSA |
| **Connexions DB** | Chaînes de connexion |

---

## ⚠️ Le problème : secrets dans le code

### Exemple de code dangereux (comme dans Mission Blackout)

```python
# ❌ CATASTROPHE - Le code qui a causé l'attaque !
DB_CONFIG = {
    "host": "prod-db.logitrans.local",
    "user": "admin_backup",
    "password": "LogiTrans2023!",  # EN CLAIR!
    "sslmode": "disable"
}
```

### Pourquoi c'est dangereux ?

1. **Git conserve tout** : Même supprimé, le secret reste dans l'historique
2. **Fuites multiples** : Backups, logs, screenshots, partage de code
3. **Pas de rotation** : Impossible de changer sans modifier le code
4. **Accès non contrôlé** : Tous les développeurs voient le secret

### Statistiques alarmantes

| Fait | Chiffre |
|------|---------|
| Secrets exposés sur GitHub (2023) | 12,8 millions |
| Temps moyen avant exploitation | < 1 minute |
| Coût moyen d'une fuite de credentials | 4,5 M$ |

---

## ✅ Solution 1 : Variables d'environnement

### Principe

Les secrets sont stockés dans l'environnement d'exécution, pas dans le code.

### Mise en œuvre

**Fichier `.env`** (jamais commité !)
```bash
# .env - À ajouter dans .gitignore !
DB_HOST=prod-db.logitrans.local
DB_USER=admin_backup
DB_PASSWORD=LogiTrans2023!
STRIPE_API_KEY=sk_live_xxx
```

**Code Python**
```python
import os
from dotenv import load_dotenv

# Charger le fichier .env
load_dotenv()

# Utiliser les variables
DB_CONFIG = {
    "host": os.environ.get("DB_HOST"),
    "user": os.environ.get("DB_USER"),
    "password": os.environ.get("DB_PASSWORD"),
}

# Vérification au démarrage
if not all(DB_CONFIG.values()):
    raise RuntimeError("Variables d'environnement manquantes!")
```

**Important** : Le fichier `.gitignore`
```
# .gitignore
.env
.env.local
.env.production
*.pem
*.key
```

### Avantages / Limites

| ✅ Avantages | ⚠️ Limites |
|-------------|-----------|
| Simple à mettre en place | Pas de rotation automatique |
| Secrets hors du code | Pas d'audit des accès |
| Différent par environnement | Fichier .env peut être volé |

---

## ✅ Solution 2 : Gestionnaire de secrets (Vault)

### Principe

Un service centralisé stocke et distribue les secrets de manière sécurisée.

### HashiCorp Vault - Architecture

```
┌─────────────────────────────────────────────────┐
│                   VAULT                          │
│  ┌─────────────┐  ┌─────────────┐               │
│  │   Secrets   │  │   Policies  │               │
│  │   Engine    │  │   & Audit   │               │
│  └─────────────┘  └─────────────┘               │
└─────────────────────────────────────────────────┘
         │                    │
         ▼                    ▼
┌─────────────┐      ┌─────────────┐
│  App Web    │      │  App Mobile │
│  (lecture)  │      │  (lecture)  │
└─────────────┘      └─────────────┘
```

### Exemple d'utilisation

```python
import hvac

# Connexion à Vault
client = hvac.Client(url='https://vault.company.com')
client.token = os.environ.get('VAULT_TOKEN')

# Lecture d'un secret
secret = client.secrets.kv.v2.read_secret_version(
    path='database/prod'
)

db_password = secret['data']['data']['password']
```

### Fonctionnalités avancées

| Fonctionnalité | Description |
|----------------|-------------|
| **Rotation automatique** | Les mots de passe changent régulièrement |
| **Secrets dynamiques** | Credentials générés à la demande, expiration courte |
| **Audit complet** | Qui a accédé à quoi, quand |
| **Chiffrement transit** | Vault chiffre/déchiffre sans exposer la clé |

---

## ✅ Solution 3 : Secrets des plateformes cloud

### AWS Secrets Manager

```python
import boto3

client = boto3.client('secretsmanager')
response = client.get_secret_value(SecretId='prod/db/credentials')
secret = json.loads(response['SecretString'])
```

### Azure Key Vault

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

client = SecretClient(
    vault_url="https://mykeyvault.vault.azure.net",
    credential=DefaultAzureCredential()
)
secret = client.get_secret("db-password")
```

### Google Secret Manager

```python
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()
name = "projects/my-project/secrets/db-password/versions/latest"
response = client.access_secret_version(request={"name": name})
secret = response.payload.data.decode("UTF-8")
```

---

## 🛡️ Détection des secrets dans le code

### Outils de prévention

| Outil | Usage |
|-------|-------|
| **git-secrets** | Hook pre-commit, bloque les commits avec secrets |
| **TruffleHog** | Scanne l'historique Git |
| **GitLeaks** | Détection patterns + entropie |
| **detect-secrets** | Hook + baseline des faux positifs |

### Configuration git-secrets

```bash
# Installation
brew install git-secrets  # macOS
apt install git-secrets   # Linux

# Configuration du repo
cd mon-projet
git secrets --install
git secrets --register-aws  # Patterns AWS

# Ajouter des patterns personnalisés
git secrets --add 'password\s*=\s*.+'
git secrets --add 'api_key\s*=\s*.+'
```

### Dans le pipeline CI/CD

```yaml
# .github/workflows/security.yml
- name: Scan for secrets
  uses: trufflesecurity/trufflehog@main
  with:
    path: ./
    base: main
```

---

## 🚨 Que faire si un secret est exposé ?

### Procédure d'urgence

```
1. RÉVOQUER immédiatement le secret
   └── Désactiver la clé API / changer le mot de passe

2. NETTOYER l'historique Git
   └── git filter-branch ou BFG Repo-Cleaner

3. AUDITER les accès
   └── Logs : le secret a-t-il été utilisé ?

4. NOTIFIER les parties concernées
   └── Équipe sécurité, clients si données exposées

5. POST-MORTEM
   └── Comment éviter que ça se reproduise ?
```

### Nettoyage de l'historique Git

```bash
# Avec BFG Repo-Cleaner (plus rapide)
bfg --replace-text passwords.txt my-repo.git

# Fichier passwords.txt
LogiTrans2023!=>***REMOVED***
sk_live_xxx=>***REMOVED***
```

⚠️ **Attention** : Nettoyer l'historique ne suffit pas ! Le secret a peut-être déjà été copié. **Toujours révoquer en premier.**

---

## 📋 Checklist de sécurité

- [ ] Fichier `.env` dans `.gitignore`
- [ ] Pas de secrets dans le code source
- [ ] Variables d'environnement documentées (sans valeurs)
- [ ] Hook pre-commit configuré (git-secrets)
- [ ] Scan automatique dans la CI/CD
- [ ] Rotation régulière des secrets
- [ ] Principe du moindre privilège (accès limités)

---

## 📚 Pour aller plus loin

- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [HashiCorp Vault Documentation](https://developer.hashicorp.com/vault/docs)
- [12-Factor App - Config](https://12factor.net/config)

---

## ✏️ Exercice pratique

Transformez ce code dangereux en code sécurisé :

```python
import requests

API_KEY = "sk_live_SuperSecretKey123"
DB_PASSWORD = "Admin@2024!"

def call_api():
    return requests.get(f"https://api.example.com?key={API_KEY}")

def connect_db():
    return psycopg2.connect(password=DB_PASSWORD)
```

<details>
<summary>Voir la solution</summary>

```python
import os
import requests
from dotenv import load_dotenv

load_dotenv()

def call_api():
    api_key = os.environ.get("API_KEY")
    if not api_key:
        raise RuntimeError("API_KEY non configurée")
    return requests.get("https://api.example.com", headers={"Authorization": f"Bearer {api_key}"})

def connect_db():
    password = os.environ.get("DB_PASSWORD")
    if not password:
        raise RuntimeError("DB_PASSWORD non configurée")
    return psycopg2.connect(password=password)
```

Fichier `.env` :
```
API_KEY=sk_live_SuperSecretKey123
DB_PASSWORD=Admin@2024!
```

Fichier `.gitignore` :
```
.env
```
</details>

---

*Fiche créée pour le BTS SIO SLAM — Mission Blackout*