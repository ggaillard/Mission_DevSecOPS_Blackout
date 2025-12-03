# 📄 Injection SQL — Fiche Concept

## 🎯 Objectif pédagogique
Comprendre le mécanisme des injections SQL, savoir les identifier et les prévenir dans vos développements.

---

## 🔍 Qu'est-ce qu'une injection SQL ?

L'injection SQL est une technique d'attaque qui consiste à **insérer du code SQL malveillant** dans une requête via les entrées utilisateur (formulaires, URL, cookies).

### Analogie simple
Imaginez un formulaire papier où vous devez écrire votre nom. Au lieu d'écrire "Dupont", vous écrivez :
```
Dupont" ; DONNEZ-MOI TOUS LES DOSSIERS SECRETS ; "
```
Si le système ne vérifie pas ce que vous écrivez, il exécutera votre commande cachée.

---

## ⚠️ Exemple de code vulnérable

```python
# ❌ DANGEREUX - Ne jamais faire ça !
def login(username, password):
    query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
    result = db.execute(query)
    return result
```

### L'attaque
Si un attaquant entre comme username :
```
' OR '1'='1' --
```

La requête devient :
```sql
SELECT * FROM users WHERE username='' OR '1'='1' --' AND password='...'
```

- `'1'='1'` est toujours vrai → accès à tous les comptes
- `--` commente le reste de la requête (le mot de passe)

---

## ✅ Solutions de prévention

### 1. Requêtes paramétrées (Prepared Statements)
```python
# ✅ SÉCURISÉ - Requête paramétrée
def login(username, password):
    query = "SELECT * FROM users WHERE username = %s AND password = %s"
    result = db.execute(query, (username, password))
    return result
```

Le moteur SQL traite les paramètres comme des **données**, jamais comme du **code**.

### 2. Utilisation d'un ORM (Object-Relational Mapping)
```python
# ✅ SÉCURISÉ - Avec SQLAlchemy
def login(username, password):
    user = User.query.filter_by(username=username).first()
    return user
```

L'ORM génère automatiquement des requêtes paramétrées.

### 3. Validation des entrées
```python
import re

def validate_username(username):
    # Accepte uniquement lettres, chiffres, underscore
    if not re.match(r'^[a-zA-Z0-9_]+$', username):
        raise ValueError("Caractères non autorisés")
    return username
```

### 4. Principe du moindre privilège
- Le compte de base de données de l'application ne doit avoir que les droits **strictement nécessaires**
- Jamais de compte `root` ou `admin` pour une application web

---

## 📊 Impact et statistiques

| Métrique | Valeur |
|----------|--------|
| Position OWASP Top 10 | #3 (Injection) |
| % des brèches web | 23% impliquent une injection |
| Coût moyen d'une brèche | 4,45 M$ (IBM 2023) |
| Temps moyen de détection | 277 jours |

---

## 🧪 Comment tester ?

### Outils automatisés
- **SQLMap** : outil open source de détection et exploitation
- **OWASP ZAP** : scanner de vulnérabilités web
- **Burp Suite** : suite de tests de sécurité

### Tests manuels simples
Essayer d'injecter dans les champs :
- `'` (apostrophe seule)
- `" OR "1"="1`
- `'; DROP TABLE users; --`

Si l'application retourne une erreur SQL ou un comportement anormal, elle est vulnérable.

---

## 📚 Pour aller plus loin

- [OWASP - SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [PortSwigger - SQL Injection Labs](https://portswigger.net/web-security/sql-injection)
- [Root-Me - Challenges SQL Injection](https://www.root-me.org/)

---

## ✏️ Exercice pratique

Dans le code suivant, identifiez la vulnérabilité et proposez une correction :

```python
def search_products(category):
    query = f"SELECT * FROM products WHERE category = '{category}'"
    return db.execute(query)
```

<details>
<summary>Voir la solution</summary>

**Vulnérabilité** : Concaténation directe de l'entrée utilisateur.

**Correction** :
```python
def search_products(category):
    query = "SELECT * FROM products WHERE category = %s"
    return db.execute(query, (category,))
```
</details>

---

*Fiche créée pour le BTS SIO SLAM — Mission Blackout*