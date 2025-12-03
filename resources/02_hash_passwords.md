# 🔐 Hashage des mots de passe — Fiche Concept

## 🎯 Objectif pédagogique
Comprendre pourquoi et comment stocker les mots de passe de manière sécurisée.

---

## 🔍 Le problème : stocker les mots de passe

### ❌ Ce qu'il ne faut JAMAIS faire

```python
# CATASTROPHE - Mot de passe en clair
users_table = {
    "alice": "MonMotDePasse123",
    "bob": "SecretBob456"
}
```

**Risques** :
- Si la base est volée → tous les mots de passe exposés
- Les admins peuvent lire les mots de passe
- Réutilisation sur d'autres sites (78% des utilisateurs)

---

## 🧮 Qu'est-ce qu'un hash ?

Un **hash** est une fonction mathématique à sens unique :
- Entrée de n'importe quelle taille → sortie de taille fixe
- Impossible de retrouver l'entrée à partir de la sortie
- Même entrée → toujours même sortie

### Exemple avec SHA-256
```
"password123" → "ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f"
"password124" → "5e884898da28047d9d6b6f4b7d1c6c50e7c3a9b2d4f5e6a7b8c9d0e1f2a3b4c5" (totalement différent!)
```

---

## ⚠️ Pourquoi SHA-256 ne suffit pas ?

### Attaque par dictionnaire / Rainbow Tables

Les attaquants précalculent les hashs de millions de mots de passe courants :

| Mot de passe | Hash SHA-256 |
|--------------|--------------|
| 123456 | 8d969... |
| password | 5e884... |
| azerty | 2c0c8... |

Si votre hash correspond → mot de passe trouvé en quelques secondes.

### Attaque par force brute

Avec un GPU moderne :
- **SHA-256** : ~10 milliards de hashs/seconde
- Un mot de passe de 8 caractères : quelques heures

---

## ✅ La solution : Fonctions de hashage lentes + sel

### Le "sel" (salt)

Un **sel** est une chaîne aléatoire unique ajoutée à chaque mot de passe :

```python
sel = "x7Kp9mN2qR"  # Généré aléatoirement pour chaque utilisateur
hash = hashage(mot_de_passe + sel)
```

**Avantages** :
- Les Rainbow Tables deviennent inutiles
- Deux utilisateurs avec le même mot de passe → hashs différents

### Fonctions de hashage lentes

| Fonction | Temps par hash | Sécurité |
|----------|---------------|----------|
| MD5 | 0.000001s | ❌ Cassé |
| SHA-256 | 0.000001s | ❌ Trop rapide |
| **bcrypt** | 0.1-0.5s | ✅ Recommandé |
| **Argon2** | 0.1-1s | ✅ Standard actuel |
| **scrypt** | 0.1-0.5s | ✅ Recommandé |

---

## 💻 Implémentation en Python

### Avec bcrypt (recommandé)

```python
import bcrypt

# Création du hash (inscription)
def hash_password(password):
    # bcrypt génère automatiquement le sel
    salt = bcrypt.gensalt(rounds=12)  # 12 = facteur de coût
    hashed = bcrypt.hashpw(password.encode('utf-8'), salt)
    return hashed

# Vérification (connexion)
def verify_password(password, hashed):
    return bcrypt.checkpw(password.encode('utf-8'), hashed)

# Utilisation
hashed = hash_password("MonMotDePasse123")
# Stocké en DB: $2b$12$LQv3c1yqBwE...

if verify_password("MonMotDePasse123", hashed):
    print("Connexion réussie!")
```

### Avec Werkzeug (Flask)

```python
from werkzeug.security import generate_password_hash, check_password_hash

# Création
hashed = generate_password_hash("MonMotDePasse123", method='pbkdf2:sha256')

# Vérification
if check_password_hash(hashed, "MonMotDePasse123"):
    print("Connexion réussie!")
```

### Avec Argon2 (le plus récent)

```python
from argon2 import PasswordHasher

ph = PasswordHasher()

# Création
hashed = ph.hash("MonMotDePasse123")

# Vérification
try:
    ph.verify(hashed, "MonMotDePasse123")
    print("Connexion réussie!")
except:
    print("Mot de passe incorrect")
```

---

## 📊 Comparatif des algorithmes

| Critère | bcrypt | Argon2 | scrypt |
|---------|--------|--------|--------|
| Année | 1999 | 2015 | 2009 |
| Résistance GPU | ✅ Bonne | ✅✅ Excellente | ✅ Bonne |
| Mémoire utilisée | Faible | Configurable | Élevée |
| Standard actuel | Oui | **Recommandé OWASP** | Oui |
| Facilité d'usage | ✅✅ Simple | ✅ Simple | ⚠️ Config complexe |

**Recommandation 2024** : Argon2id ou bcrypt avec rounds ≥ 12

---

## 🚫 Les erreurs courantes

### 1. Hasher côté client
```javascript
// ❌ MAUVAIS - Hash côté client
const hash = sha256(password);
fetch('/login', { body: { passwordHash: hash } });
```
Le hash devient le mot de passe ! Un attaquant peut le réutiliser.

### 2. Utiliser MD5 ou SHA-1
```python
# ❌ MAUVAIS - Algorithmes obsolètes
import hashlib
hash = hashlib.md5(password.encode()).hexdigest()
```

### 3. Sel identique pour tous
```python
# ❌ MAUVAIS - Même sel pour tous
GLOBAL_SALT = "monsel123"
hash = bcrypt.hashpw((password + GLOBAL_SALT).encode(), bcrypt.gensalt())
```

---

## 📚 Pour aller plus loin

- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Have I Been Pwned](https://haveibeenpwned.com/) - Vérifier si un mot de passe a fuité
- [Argon2 RFC](https://datatracker.ietf.org/doc/html/rfc9106)

---

## ✏️ Exercice pratique

Quel est le problème dans ce code ?

```python
import hashlib

def register(username, password):
    hash = hashlib.sha256(password.encode()).hexdigest()
    db.save(username, hash)

def login(username, password):
    hash = hashlib.sha256(password.encode()).hexdigest()
    stored = db.get_hash(username)
    return hash == stored
```

<details>
<summary>Voir la solution</summary>

**Problèmes** :
1. SHA-256 est trop rapide (10 milliards/s sur GPU)
2. Pas de sel → vulnérable aux Rainbow Tables
3. Pas de protection contre le timing attack

**Correction** :
```python
import bcrypt

def register(username, password):
    hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
    db.save(username, hashed)

def login(username, password):
    stored = db.get_hash(username)
    return bcrypt.checkpw(password.encode(), stored)
```
</details>

---

*Fiche créée pour le BTS SIO SLAM — Mission Blackout*