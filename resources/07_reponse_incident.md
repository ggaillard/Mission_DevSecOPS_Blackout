# 🚨 Réponse à Incident — Fiche Concept

## 🎯 Objectif pédagogique
Comprendre les procédures et bonnes pratiques pour gérer efficacement un incident de sécurité informatique.

---

## 🔍 Qu'est-ce qu'un incident de sécurité ?

Un **incident de sécurité** est tout événement qui compromet la confidentialité, l'intégrité ou la disponibilité des systèmes d'information.

### Exemples d'incidents

| Type | Exemple | Gravité |
|------|---------|---------|
| **Malware** | Ransomware, cheval de Troie | 🔴 Critique |
| **Intrusion** | Accès non autorisé aux systèmes | 🔴 Critique |
| **Fuite de données** | Base clients publiée | 🔴 Critique |
| **Phishing réussi** | Credentials volés | 🟠 Élevée |
| **DoS/DDoS** | Service indisponible | 🟠 Élevée |
| **Défiguration** | Site web modifié | 🟡 Moyenne |
| **Tentative bloquée** | Attaque détectée et stoppée | 🟢 Faible |

---

## 📋 Le cycle de réponse à incident (NIST)

```
┌─────────────────────────────────────────────────────────────────┐
│                    CYCLE DE RÉPONSE À INCIDENT                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│        ┌──────────────┐                                         │
│        │ 1. PRÉPARER  │                                         │
│        │   (avant)    │                                         │
│        └──────┬───────┘                                         │
│               │                                                  │
│               ▼                                                  │
│        ┌──────────────┐      ┌──────────────┐                   │
│        │ 2. DÉTECTER  │─────▶│ 3. CONTENIR  │                   │
│        │  & ANALYSER  │      │  & ÉRADIQUER │                   │
│        └──────────────┘      └──────┬───────┘                   │
│                                      │                           │
│                                      ▼                           │
│                              ┌──────────────┐                   │
│                              │ 4. RÉCUPÉRER │                   │
│                              │              │                   │
│                              └──────┬───────┘                   │
│                                      │                           │
│                                      ▼                           │
│                              ┌──────────────┐                   │
│                              │  5. RETEX    │                   │
│                              │(Post-mortem) │                   │
│                              └──────────────┘                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Phase 1 : Préparation (AVANT l'incident)

### Équipe de réponse à incident (CSIRT)

| Rôle | Responsabilité |
|------|----------------|
| **Incident Manager** | Coordination globale, communication |
| **Analyste Sécurité** | Investigation technique |
| **Admin Systèmes** | Actions sur les systèmes |
| **Juriste** | Conformité, notification CNIL |
| **Communication** | Relations presse, clients |
| **Direction** | Décisions stratégiques |

### Documentation à préparer

- [ ] **Plan de réponse à incident** documenté
- [ ] **Contacts d'urgence** (internes et externes)
- [ ] **Procédures d'escalade** claires
- [ ] **Outils forensiques** prêts à l'emploi
- [ ] **Sauvegardes** testées et hors-ligne
- [ ] **Exercices de simulation** réguliers

### Kit d'intervention

```
📦 Kit de réponse à incident
├── 💻 Poste d'analyse forensique (isolé)
├── 💾 Disques durs vierges (pour images)
├── 🔌 Bloqueurs d'écriture (write blockers)
├── 📝 Formulaires de chaîne de custody
├── 📋 Checklist de réponse
├── 📞 Liste de contacts actualisée
└── 🔑 Accès administrateur d'urgence
```

---

## 2️⃣ Phase 2 : Détection & Analyse

### Sources de détection

| Source | Exemples |
|--------|----------|
| **SIEM** | Corrélation de logs, alertes |
| **EDR** | Comportement suspect sur postes |
| **IDS/IPS** | Signatures d'attaque réseau |
| **Utilisateurs** | Signalement comportement anormal |
| **Externe** | CERT, partenaire, client |

### Questions critiques

```
┌─────────────────────────────────────────────────────────────┐
│                 ANALYSE INITIALE (15 min)                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  QUOI ?                                                      │
│  ├── Quels systèmes sont touchés ?                          │
│  ├── Quel type d'incident ?                                 │
│  └── Quelle est la gravité estimée ?                        │
│                                                              │
│  QUAND ?                                                     │
│  ├── Quand l'incident a-t-il commencé ?                     │
│  ├── Quand a-t-il été détecté ?                             │
│  └── Est-il toujours en cours ?                             │
│                                                              │
│  COMMENT ?                                                   │
│  ├── Quel est le vecteur d'attaque ?                        │
│  ├── Quelles vulnérabilités exploitées ?                    │
│  └── Mouvements latéraux observés ?                         │
│                                                              │
│  QUI ?                                                       │
│  ├── Comptes compromis identifiés ?                         │
│  └── Indicateurs d'attribution ?                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Indicateurs de compromission (IoC)

| Type | Exemple |
|------|---------|
| **IP** | 185.220.101.42 (TOR exit node) |
| **Domaine** | malware-c2.badsite.com |
| **Hash fichier** | SHA256: a1b2c3d4... |
| **Email** | phishing@fake-company.com |
| **Comportement** | Connexion à 2h du matin depuis TOR |

---

## 3️⃣ Phase 3 : Confinement & Éradication

### Stratégies de confinement

| Stratégie | Avantages | Inconvénients |
|-----------|-----------|---------------|
| **Isolation réseau** | Rapide, préserve preuves | Perte d'accès |
| **Isolation compte** | Ciblé, minimal impact | Mouvement latéral possible |
| **Arrêt système** | Total | Perte données volatiles |
| **Blocage IP/domaine** | Non intrusif | Facilement contournable |

### ⚠️ Erreurs à éviter

```
❌ NE PAS FAIRE :
├── Éteindre brutalement les machines (perte RAM)
├── Effacer les logs pour "nettoyer"
├── Restaurer immédiatement les sauvegardes
├── Informer l'attaquant qu'on l'a détecté
├── Négocier seul avec les attaquants
└── Cacher l'incident à la direction/CNIL
```

### Actions de confinement (exemple ransomware)

```bash
# 1. Isoler le segment réseau affecté
firewall-cmd --zone=infected --add-source=192.168.10.0/24
firewall-cmd --zone=infected --set-target=DROP

# 2. Désactiver les comptes compromis
net user admin_backup /active:no
Disable-ADAccount -Identity "admin_backup"

# 3. Bloquer les IoCs sur le firewall
firewall-cmd --add-rich-rule='rule family=ipv4 source address=185.220.101.42 reject'

# 4. Capturer la mémoire (avant toute autre action)
./winpmem.exe memdump.raw
```

### Éradication

| Étape | Action |
|-------|--------|
| 1 | Identifier tous les systèmes infectés |
| 2 | Supprimer les malwares/backdoors |
| 3 | Réinitialiser les credentials compromis |
| 4 | Patcher les vulnérabilités exploitées |
| 5 | Vérifier l'absence de persistance |

---

## 4️⃣ Phase 4 : Récupération

### Plan de reconstruction (LogiTrans)

```
Semaine 1 : Infrastructure critique
├── Jour 1-2 : Active Directory (nouveau domaine)
├── Jour 3-4 : Serveurs de fichiers
└── Jour 5-7 : Bases de données (depuis backup hors-ligne)

Semaine 2 : Applications métier
├── ERP (priorité 1)
├── CRM (priorité 2)
└── TMS (priorité 2)

Semaine 3-4 : Services secondaires
├── Messagerie
├── Intranet
└── Applications internes

Validation
├── Tests de non-régression
├── Audit de sécurité du nouveau système
└── Retour progressif des utilisateurs
```

### Critères de retour en production

- [ ] Tous les IoCs bloqués
- [ ] Vulnérabilités exploitées corrigées
- [ ] Nouveaux credentials déployés
- [ ] Monitoring renforcé actif
- [ ] Backup fonctionnel vérifié
- [ ] Communication aux utilisateurs effectuée

---

## 5️⃣ Phase 5 : Retour d'expérience (RETEX)

### Réunion post-mortem

**Participants** : Tous les intervenants + direction

**Objectif** : Apprendre, pas blâmer

### Questions à traiter

```
┌─────────────────────────────────────────────────────────────┐
│                      POST-MORTEM                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CHRONOLOGIE                                                 │
│  └── Timeline détaillée de l'incident                       │
│                                                              │
│  CAUSES RACINES                                              │
│  ├── Cause technique : Mot de passe exposé                  │
│  ├── Cause processus : Pas de revue de code                 │
│  └── Cause humaine : Pression des délais                    │
│                                                              │
│  CE QUI A FONCTIONNÉ                                        │
│  ├── Détection rapide (4h)                                  │
│  └── Sauvegardes hors-ligne disponibles                     │
│                                                              │
│  CE QUI A ÉCHOUÉ                                            │
│  ├── Segmentation réseau insuffisante                       │
│  └── Pas de scan de secrets dans le CI/CD                   │
│                                                              │
│  ACTIONS CORRECTIVES                                        │
│  ├── Action 1 : Vault pour les secrets (Resp: X, Date: Y)   │
│  ├── Action 2 : Segmentation VLAN (Resp: Z, Date: W)        │
│  └── Action 3 : Pipeline DevSecOps (Resp: A, Date: B)       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Rapport d'incident

Le rapport doit contenir :
1. **Résumé exécutif** (1 page)
2. **Timeline détaillée**
3. **Impact** (systèmes, données, financier)
4. **Cause racine**
5. **Actions de remédiation**
6. **Leçons apprises**
7. **Recommandations**

---

## 📞 Contacts & Ressources

### En France

| Organisme | Rôle | Contact |
|-----------|------|---------|
| **ANSSI** | Autorité nationale cybersécurité | cert-fr@ssi.gouv.fr |
| **CNIL** | Notification violations données | cnil.fr (72h max) |
| **Police/Gendarmerie** | Dépôt de plainte | Commissariat local |
| **Cybermalveillance.gouv.fr** | Assistance PME/particuliers | 0 800 200 000 |

### Prestataires spécialisés

- **Réponse à incident** : Wavestone, Orange Cyberdefense, Intrinsec
- **Forensique** : Synacktiv, Quarkslab
- **Négociation ransomware** : Coveware (spécialiste)

---

## ⏱️ Obligations légales (RGPD)

### Notification CNIL

```
┌─────────────────────────────────────────────────────────────┐
│                 NOTIFICATION CNIL (72h)                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  OBLIGATOIRE si violation de données personnelles avec      │
│  risque pour les personnes concernées.                      │
│                                                              │
│  Contenu de la notification :                               │
│  ├── Nature de la violation                                 │
│  ├── Catégories et nombre de personnes concernées           │
│  ├── Catégories et nombre d'enregistrements                 │
│  ├── Nom du DPO                                             │
│  ├── Conséquences probables                                 │
│  └── Mesures prises ou envisagées                           │
│                                                              │
│  Délai : 72 heures après prise de connaissance              │
│  Formulaire : notifications.cnil.fr                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Notification aux personnes concernées

Obligatoire si **risque élevé** pour les droits et libertés :
- Informer les personnes concernées
- Décrire la nature de la violation
- Donner les recommandations (changer mot de passe, surveiller comptes...)

---

## 📚 Pour aller plus loin

- [ANSSI - Guide de gestion de crise cyber](https://www.ssi.gouv.fr/guide/organiser-un-exercice-de-gestion-de-crise-cyber/)
- [NIST SP 800-61 - Computer Security Incident Handling Guide](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)
- [FIRST - Forum of Incident Response and Security Teams](https://www.first.org/)

---

## ✏️ Exercice : Simulation d'incident

**Scénario** : Il est 14h, un utilisateur signale que ses fichiers ont une extension `.encrypted` et qu'un fichier `README_DECRYPT.txt` est apparu sur son bureau.

Établissez votre plan d'action pour les 2 prochaines heures.

<details>
<summary>Voir un plan d'action type</summary>

**14h00 - Alerte reçue**
- [ ] Isoler immédiatement le poste du réseau (câble débranché)
- [ ] NE PAS éteindre le poste
- [ ] Alerter l'équipe sécurité et le management

**14h05 - Évaluation initiale**
- [ ] Vérifier si d'autres postes sont touchés (scan réseau)
- [ ] Identifier le ransomware (message de rançon, extension)
- [ ] Vérifier l'état des sauvegardes (sont-elles accessibles ?)

**14h15 - Confinement**
- [ ] Isoler le segment réseau concerné
- [ ] Désactiver les partages réseau
- [ ] Bloquer les comptes potentiellement compromis
- [ ] Capturer la mémoire du poste infecté

**14h30 - Investigation**
- [ ] Analyser les logs (Quand ? Comment ? D'où ?)
- [ ] Identifier le vecteur d'infection (email, vulnérabilité...)
- [ ] Rechercher d'autres IoCs sur le réseau

**15h00 - Communication**
- [ ] Briefer la direction
- [ ] Préparer la notification CNIL (si données personnelles)
- [ ] Informer les utilisateurs (ne pas cliquer, signaler)

**15h30 - Décision**
- [ ] Évaluer l'étendue des dégâts
- [ ] Vérifier la disponibilité des sauvegardes hors-ligne
- [ ] Décider : reconstruire ou négocier ?

**16h00 - Plan de restauration**
- [ ] Prioriser les systèmes critiques
- [ ] Planifier la reconstruction depuis sauvegardes saines
- [ ] Préparer le renforcement sécurité post-incident

</details>

---

*Fiche créée pour le BTS SIO SLAM — Mission Blackout*