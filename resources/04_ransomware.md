# 🦠 Ransomware — Fiche Concept

## 🎯 Objectif pédagogique
Comprendre le fonctionnement des ransomwares, les méthodes d'infection et les stratégies de protection.

---

## 🔍 Qu'est-ce qu'un ransomware ?

Un **ransomware** (ou rançongiciel) est un logiciel malveillant qui :
1. **Chiffre** les fichiers de la victime
2. **Exige une rançon** (souvent en cryptomonnaie) pour la clé de déchiffrement
3. Menace de **publier les données** volées (double extorsion)

### Évolution des tactiques

```
2013-2017 : Chiffrement simple
    └── Payer pour récupérer les fichiers

2018-2020 : Double extorsion
    └── Chiffrement + Vol de données
    └── "Payez ou on publie vos données"

2021-2024 : Triple extorsion
    └── Chiffrement + Vol + Attaque des clients/partenaires
    └── "On contacte vos clients si vous ne payez pas"
```

---

## ⚙️ Comment fonctionne une attaque ?

### Phase 1 : Accès initial (T0)

| Méthode | Fréquence | Exemple |
|---------|-----------|---------|
| **Phishing** | 36% | Email avec pièce jointe malveillante |
| **Vulnérabilité** | 28% | Serveur non patché (Log4j, Exchange) |
| **Credentials volés** | 19% | Mot de passe exposé (comme LogiTrans!) |
| **RDP exposé** | 12% | Bureau distant accessible sur Internet |
| **Supply chain** | 5% | Logiciel compromis (SolarWinds) |

### Phase 2 : Reconnaissance (T0 à T+7 jours)

L'attaquant explore le réseau :
```
- Cartographie du réseau
- Identification des serveurs critiques
- Localisation des sauvegardes
- Vol de credentials administrateurs
- Exfiltration de données sensibles
```

### Phase 3 : Déploiement (T+7 à T+14 jours)

```
┌─────────────────────────────────────────────────────────┐
│                    INFRASTRUCTURE CIBLE                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐            │
│   │   DC    │───▶│  File   │───▶│ Backup  │            │
│   │ (Admin) │    │ Server  │    │ Server  │            │
│   └─────────┘    └─────────┘    └─────────┘            │
│        │              │              │                  │
│        │              │              │                  │
│        ▼              ▼              ▼                  │
│   ┌─────────────────────────────────────────┐          │
│   │         RANSOMWARE DÉPLOYÉ              │          │
│   │   Tout est chiffré simultanément        │          │
│   └─────────────────────────────────────────┘          │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Phase 4 : Extorsion (T+14)

```
╔══════════════════════════════════════════════════════╗
║                                                      ║
║   ⚠️  VOS FICHIERS ONT ÉTÉ CHIFFRÉS  ⚠️              ║
║                                                      ║
║   Tous vos documents, bases de données et           ║
║   sauvegardes sont maintenant chiffrés avec         ║
║   AES-256 + RSA-4096.                               ║
║                                                      ║
║   Pour récupérer vos données :                      ║
║   💰 Payez 450 000€ en Bitcoin                      ║
║   ⏰ Délai : 72 heures                              ║
║                                                      ║
║   Si vous ne payez pas :                            ║
║   📢 Vos données seront publiées                    ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

## 📊 Statistiques clés (2024)

| Indicateur | Valeur |
|------------|--------|
| Rançon moyenne demandée | 1,5 M€ |
| Rançon moyenne payée | 400 K€ |
| % qui récupèrent toutes leurs données | **30%** |
| Temps d'arrêt moyen | 23 jours |
| Coût total moyen (rançon + arrêt + reconstruction) | **4,5 M€** |

### Groupes les plus actifs

| Groupe | Secteurs ciblés | Rançon moyenne |
|--------|-----------------|----------------|
| LockBit | Tous | 500K - 5M€ |
| BlackCat/ALPHV | Santé, Finance | 1M - 10M€ |
| Cl0p | Supply chain | 5M - 50M€ |
| Play | PME, Industrie | 100K - 1M€ |

---

## 🛡️ Comment se protéger ?

### 1. Sauvegardes robustes (Règle 3-2-1-1)

```
3 copies de vos données
├── 2 sur des supports différents
│   ├── Serveur local
│   └── NAS ou Cloud
└── 1 hors-ligne (déconnecté du réseau!)
    └── 1 immuable (impossible à modifier/supprimer)
```

**Test crucial** : Restaurer régulièrement pour vérifier que ça fonctionne !

### 2. Segmentation réseau

```
┌───────────────────────────────────────────────────┐
│                   RÉSEAU SEGMENTÉ                  │
├───────────────────────────────────────────────────┤
│                                                    │
│   VLAN Prod        VLAN Backup       VLAN Admin   │
│   ┌───────┐       ┌───────┐         ┌───────┐    │
│   │ Servs │       │Backup │         │  DC   │    │
│   └───────┘       └───────┘         └───────┘    │
│       │               │                 │         │
│       └───────────────┴─────────────────┘         │
│                       │                           │
│              ┌────────┴────────┐                  │
│              │    FIREWALL     │                  │
│              │  Règles strictes │                 │
│              └─────────────────┘                  │
│                                                    │
└───────────────────────────────────────────────────┘
```

### 3. Principe du moindre privilège

- Pas de comptes admin partagés
- MFA obligatoire pour tous les accès privilégiés
- Comptes de service avec droits limités
- Suppression des comptes inutilisés

### 4. Détection et réponse

| Outil | Fonction |
|-------|----------|
| **EDR** (Endpoint Detection & Response) | Détecte les comportements suspects sur les postes |
| **SIEM** | Corrèle les logs, alerte sur les anomalies |
| **NDR** | Analyse le trafic réseau |

### 5. Formation des utilisateurs

- Reconnaître le phishing
- Ne pas ouvrir les pièces jointes suspectes
- Signaler les emails douteux
- Comprendre les enjeux

---

## 🚨 Que faire en cas d'attaque ?

### ❌ Ce qu'il ne faut PAS faire

1. **Paniquer** et prendre des décisions hâtives
2. **Payer immédiatement** la rançon
3. **Éteindre tous les systèmes** (perte de preuves forensiques)
4. **Négocier seul** avec les attaquants
5. **Cacher l'incident** (obligations légales RGPD)

### ✅ La bonne procédure

```
1. ISOLER
   └── Déconnecter les systèmes infectés du réseau
   └── NE PAS éteindre (préserver les preuves)

2. ALERTER
   └── Équipe de direction
   └── Équipe sécurité / RSSI
   └── Prestataire réponse incident si besoin

3. ÉVALUER
   └── Quels systèmes sont touchés ?
   └── Les sauvegardes sont-elles intactes ?
   └── Quelles données ont été volées ?

4. DÉCLARER (obligations légales)
   └── CNIL (si données personnelles) : 72h max
   └── ANSSI (si OIV/OSE)
   └── Plainte police/gendarmerie

5. DÉCIDER
   └── Reconstruire sans payer (recommandé ANSSI)
   └── Négocier uniquement si pas d'alternative

6. RECONSTRUIRE
   └── Depuis des sauvegardes saines
   └── Corriger les failles exploitées
   └── Renforcer la sécurité
```

---

## 💡 Le cas LogiTrans - Leçons apprises

### Failles exploitées

| Faille | Conséquence |
|--------|-------------|
| Mot de passe dans le code | Point d'entrée initial |
| Pas de segmentation | Accès aux sauvegardes |
| Compte de service oublié | Mouvement latéral facile |
| SSL désactivé | Écoute réseau possible |

### Mesures correctives

| Mesure | Coût | Bénéfice |
|--------|------|----------|
| Vault pour secrets | 5K€/an | Élimine les credentials exposés |
| Segmentation réseau | 30K€ | Limite la propagation |
| EDR sur tous les postes | 15K€/an | Détection précoce |
| Sauvegardes hors-ligne | 10K€ | Récupération garantie |
| Formation utilisateurs | 5K€/an | -70% de clics phishing |

**Total : 65K€/an vs 2,3M€ de coût d'attaque = ROI de 35x**

---

## 📚 Pour aller plus loin

- [ANSSI - Guide Attaques par Rançongiciels](https://www.cert.ssi.gouv.fr/uploads/CERTFR-2020-CTI-001.pdf)
- [No More Ransom](https://www.nomoreransom.org/) - Outils de déchiffrement gratuits
- [CISA Ransomware Guide](https://www.cisa.gov/stopransomware)

---

## ✏️ Quiz rapide

**Question** : Une entreprise est victime d'un ransomware. Les sauvegardes existent mais sont sur le même réseau que les serveurs de production. Que s'est-il probablement passé ?

<details>
<summary>Voir la réponse</summary>

Les sauvegardes ont **également été chiffrées** par le ransomware.

C'est pourquoi la règle 3-2-1-1 inclut :
- **1 copie hors-ligne** (air gap) : déconnectée physiquement du réseau
- **1 copie immuable** : impossible à modifier même avec des droits admin

Sans ces protections, les sauvegardes ne servent à rien en cas de ransomware !
</details>

---

*Fiche créée pour le BTS SIO SLAM — Mission Blackout*