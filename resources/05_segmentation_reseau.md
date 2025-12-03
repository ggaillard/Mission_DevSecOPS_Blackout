# 🌐 Segmentation Réseau — Fiche Concept

## 🎯 Objectif pédagogique
Comprendre l'importance de la segmentation réseau et les principes du Zero Trust pour limiter l'impact des cyberattaques.

---

## 🔍 Qu'est-ce que la segmentation réseau ?

La **segmentation réseau** consiste à diviser un réseau en zones isolées, chacune avec ses propres règles de sécurité.

### Analogie : Le sous-marin

```
Sans segmentation (navire classique) :
┌─────────────────────────────────────┐
│          COQUE UNIQUE               │
│   Une brèche = tout coule           │
└─────────────────────────────────────┘

Avec segmentation (sous-marin) :
┌───────┬───────┬───────┬───────┬───────┐
│Compart│Compart│Compart│Compart│Compart│
│   1   │   2   │   3   │   4   │   5   │
│       │       │       │       │       │
└───────┴───────┴───────┴───────┴───────┘
   Une brèche = un compartiment isolé
```

---

## ⚠️ Le problème : Réseau plat (non segmenté)

### Architecture dangereuse (cas LogiTrans)

```
┌──────────────────────────────────────────────────────────┐
│                    RÉSEAU PLAT                           │
│                   (même VLAN)                            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│   ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐       │
│   │ Poste  │──│Serveur │──│Serveur │──│Backup  │       │
│   │  RH    │  │  Web   │  │   DB   │  │Server  │       │
│   └────────┘  └────────┘  └────────┘  └────────┘       │
│       │           │           │           │             │
│       └───────────┴───────────┴───────────┘             │
│                       │                                  │
│              Tout communique librement                   │
│                                                          │
│   ⚠️ Un attaquant sur n'importe quel poste peut        │
│      accéder à TOUT le reste du réseau                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Conséquences

| Problème | Impact |
|----------|--------|
| Mouvement latéral facile | L'attaquant passe de machine en machine |
| Sauvegardes accessibles | Chiffrées en même temps que la prod |
| Données sensibles exposées | RH, finance, clients accessibles partout |
| Détection difficile | Le trafic "normal" masque l'attaque |

---

## ✅ Solution : Architecture segmentée

### Modèle en zones de confiance

```
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET                                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                    ┌───────┴───────┐
                    │   FIREWALL    │
                    │   EXTERNE     │
                    └───────┬───────┘
                            │
┌───────────────────────────┴───────────────────────────────────┐
│                         DMZ                                    │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│   │ Reverse  │  │   WAF    │  │  Mail    │                   │
│   │  Proxy   │  │          │  │ Gateway  │                   │
│   └──────────┘  └──────────┘  └──────────┘                   │
└───────────────────────────┬───────────────────────────────────┘
                            │
                    ┌───────┴───────┐
                    │   FIREWALL    │
                    │   INTERNE     │
                    └───────┬───────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────┴───────┐   ┌───────┴───────┐   ┌───────┴───────┐
│  VLAN USERS   │   │  VLAN PROD    │   │  VLAN ADMIN   │
│  (Postes)     │   │  (Serveurs)   │   │  (IT/Backup)  │
├───────────────┤   ├───────────────┤   ├───────────────┤
│ ┌───┐ ┌───┐  │   │ ┌───┐ ┌───┐  │   │ ┌───┐ ┌───┐  │
│ │PC │ │PC │  │   │ │Web│ │DB │  │   │ │DC │ │Bkp│  │
│ └───┘ └───┘  │   │ └───┘ └───┘  │   │ └───┘ └───┘  │
└───────────────┘   └───────────────┘   └───────────────┘
```

### Règles de flux typiques

| Source | Destination | Ports | Action |
|--------|-------------|-------|--------|
| USERS | PROD (Web) | 443 | ✅ Autorisé |
| USERS | PROD (DB) | 3306 | ❌ Refusé |
| USERS | ADMIN | * | ❌ Refusé |
| PROD (Web) | PROD (DB) | 3306 | ✅ Autorisé |
| PROD | ADMIN (Backup) | 22, 873 | ✅ Autorisé (sens unique) |
| ADMIN | * | * | ✅ Autorisé (avec MFA) |

---

## 🔒 Zero Trust : "Ne jamais faire confiance"

### Principe fondamental

> *"Ne jamais faire confiance, toujours vérifier"*

Contrairement au modèle traditionnel (château fort avec muraille), le Zero Trust part du principe que **toute connexion est potentiellement malveillante**, même depuis l'intérieur du réseau.

### Les 3 piliers du Zero Trust

```
┌─────────────────────────────────────────────────────────────┐
│                       ZERO TRUST                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. VÉRIFIER              2. ACCÈS MINIMAL                 │
│   EXPLICITEMENT            (Least Privilege)                │
│   ┌─────────────┐          ┌─────────────┐                  │
│   │ • Identité  │          │ Accès JIT   │                  │
│   │ • Appareil  │          │ (Just In    │                  │
│   │ • Lieu      │          │  Time)      │                  │
│   │ • Service   │          │             │                  │
│   └─────────────┘          └─────────────┘                  │
│                                                              │
│   3. SUPPOSER LA BRÈCHE                                     │
│   ┌─────────────────────────────────────────────┐           │
│   │ Segmentation fine + Détection + Chiffrement │           │
│   └─────────────────────────────────────────────┘           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Implémentation concrète

| Composant | Technologie | Rôle |
|-----------|-------------|------|
| **Identité** | Azure AD, Okta | Authentification centralisée + MFA |
| **Appareil** | Intune, JAMF | Vérification conformité du poste |
| **Réseau** | Micro-segmentation | Isolation application par application |
| **Accès** | ZTNA (Zscaler, Cloudflare) | Remplace le VPN traditionnel |
| **Données** | DLP, Chiffrement | Protection même si exfiltrées |

---

## 🛠️ Technologies de segmentation

### 1. VLANs (Virtual LANs)

Séparation logique au niveau Ethernet (couche 2).

```
Switch manageable
┌─────────────────────────────────────┐
│  Port 1-10  │  Port 11-20  │  Port 21-30  │
│   VLAN 10   │   VLAN 20    │   VLAN 30    │
│   (Users)   │   (Serveurs) │   (Admin)    │
└─────────────────────────────────────┘
```

**Avantages** : Simple, natif sur les switchs managés
**Limites** : Segmentation grossière, bypass possible

### 2. Pare-feu de nouvelle génération (NGFW)

Filtrage au niveau applicatif (couche 7).

| Fonctionnalité | Exemple |
|----------------|---------|
| Inspection SSL | Déchiffre et analyse le trafic HTTPS |
| IPS/IDS | Détecte les signatures d'attaque |
| Application Control | Bloque WhatsApp mais autorise Slack |
| User-based rules | Jean peut accéder à X, Marie non |

**Leaders** : Palo Alto, Fortinet, Check Point

### 3. Micro-segmentation

Isolation au niveau workload (VM, container).

```
┌───────────────────────────────────────────────────────────┐
│                    CLUSTER KUBERNETES                      │
├───────────────────────────────────────────────────────────┤
│                                                            │
│   ┌─────────────┐    ╳    ┌─────────────┐                │
│   │  Frontend   │    │    │   Backend   │                │
│   │   (pods)    │◄───┴───►│   (pods)    │                │
│   └─────────────┘ Network └─────────────┘                │
│          │         Policy        │                        │
│          │                       │                        │
│          ╳         ╳             ▼                        │
│   ┌──────┴──────┐        ┌─────────────┐                 │
│   │  Autre NS   │        │  Database   │                 │
│   │  (bloqué)   │        │   (pods)    │                 │
│   └─────────────┘        └─────────────┘                 │
│                                                            │
└───────────────────────────────────────────────────────────┘
```

**Outils** : Calico, Cilium, VMware NSX

---

## 📊 Impact sur la sécurité

### Sans segmentation vs avec segmentation

| Métrique | Réseau plat | Réseau segmenté |
|----------|-------------|-----------------|
| Temps de propagation ransomware | ~15 minutes | Bloqué |
| Systèmes impactés | 100% | 5-20% |
| Temps de détection | Post-incident | En cours d'attaque |
| Coût de l'incident | 4,5 M€ | 500 K€ |

### Conformité réglementaire

| Norme | Exigence de segmentation |
|-------|--------------------------|
| **PCI-DSS** | Obligatoire pour isoler les données cartes |
| **ISO 27001** | A.13.1.3 - Séparation des réseaux |
| **NIS2** | Exigence renforcée pour secteurs critiques |
| **RGPD** | Implicite (mesures techniques appropriées) |

---

## 🎯 Checklist de segmentation

### Niveau 1 : Basique
- [ ] VLANs séparés (Users / Serveurs / Admin)
- [ ] Firewall entre les VLANs
- [ ] Sauvegardes sur un segment isolé

### Niveau 2 : Intermédiaire
- [ ] DMZ pour les services exposés
- [ ] Règles de flux documentées et auditées
- [ ] Logs centralisés (SIEM)
- [ ] Jump server pour l'administration

### Niveau 3 : Avancé (Zero Trust)
- [ ] Micro-segmentation applicative
- [ ] Authentification continue (MFA + contexte)
- [ ] ZTNA remplaçant le VPN
- [ ] Chiffrement du trafic interne
- [ ] Détection des mouvements latéraux (NDR)

---

## 📚 Pour aller plus loin

- [ANSSI - Guide Cloisonnement Système](https://www.ssi.gouv.fr/guide/recommandations-pour-la-mise-en-place-de-cloisonnement-systeme/)
- [NIST Zero Trust Architecture (SP 800-207)](https://csrc.nist.gov/publications/detail/sp/800-207/final)
- [Google BeyondCorp](https://cloud.google.com/beyondcorp)

---

## ✏️ Exercice pratique

Analysez cette architecture et identifiez les problèmes :

```
                INTERNET
                    │
            ┌───────┴───────┐
            │   FIREWALL    │
            └───────┬───────┘
                    │
    ┌───────────────┼───────────────┐
    │               │               │
┌───┴───┐     ┌─────┴─────┐   ┌─────┴─────┐
│  PC   │     │  Serveur  │   │  Backup   │
│ Users │     │    Web    │   │  Server   │
└───────┘     └───────────┘   └───────────┘
    │               │               │
    └───────────────┴───────────────┘
            (même réseau)
```

<details>
<summary>Voir la solution</summary>

**Problèmes identifiés** :

1. **Pas de DMZ** : Le serveur web est sur le même réseau que les postes et les backups
2. **Sauvegardes accessibles** : Un ransomware peut les atteindre depuis n'importe quel poste
3. **Pas de segmentation** : Un poste utilisateur compromis = accès à tout
4. **Pas de zone admin** : L'administration n'est pas isolée

**Architecture corrigée** :

```
                INTERNET
                    │
            ┌───────┴───────┐
            │   FIREWALL    │
            └───────┬───────┘
                    │
         ┌──────────┴──────────┐
         │        DMZ          │
         │   ┌───────────┐     │
         │   │  Serveur  │     │
         │   │    Web    │     │
         │   └───────────┘     │
         └──────────┬──────────┘
                    │
            ┌───────┴───────┐
            │   FIREWALL    │
            │   INTERNE     │
            └───────┬───────┘
                    │
    ┌───────────────┼───────────────┐
    │               │               │
 VLAN Users     VLAN Prod       VLAN Admin
┌───────┐     ┌─────────┐     ┌─────────┐
│  PC   │     │   DB    │     │ Backup  │
│       │     │         │     │ (isolé) │
└───────┘     └─────────┘     └─────────┘
```
</details>

---

*Fiche créée pour le BTS SIO SLAM — Mission Blackout*