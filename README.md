# 🛡️ Mission Blackout v3 — Package Complet

## 🌐 Accès en ligne

**Application en ligne** : [https://ggaillard.github.io/Mission_DevSecOPS_Blackout/](https://ggaillard.github.io/Mission_DevSecOPS_Blackout/)

L'application est déployée sur GitHub Pages et accessible directement depuis votre navigateur.

---

## 📦 Contenu du package

```
Mission_Blackout/
├── index.html                    # Application principale (déployée sur GitHub Pages)
├── mission_blackout_qcm.xml      # QCM 20 questions (Pronote/Moodle)
├── README.md                      # Ce fichier
│

└── resources/                     # Fiches en Markdown (référence)
    ├── 01_injection_sql.md
    ├── 02_hash_passwords.md
    ├── 03_gestion_secrets.md
    ├── 04_ransomware.md
    ├── 05_segmentation_reseau.md
    ├── 06_devsecops.md
    └── 07_reponse_incident.md
```

---

## 🚀 Installation rapide (10 min)

###  Utiliser l'application en ligne (recommandé)

1. Accéder à : [https://ggaillard.github.io/Mission_DevSecOPS_Blackout/](https://ggaillard.github.io/Mission_DevSecOPS_Blackout/)


---

## 🎓 Déroulement pédagogique

### Séance type (2h)

| Durée | Phase | Activité |
|-------|-------|----------|
| **0-10 min** | Lancement | Briefing, distribution des accès |
| **10-30 min** | Mission Blackout | Escape game en autonomie |
| **30-45 min** | Débriefing | Discussion sur les choix, erreurs découvertes |
| **45-75 min** | QCM | 20 questions avec ressources autorisées |
| **75-90 min** | Correction | Correction interactive du QCM |
| **90-120 min** | Approfondissement | Travail sur une fiche concept au choix |

### Organisation des groupes

- **Binômes recommandés** : Favorise la discussion et l'entraide
- **Écran partagé** : Un avec l'application, l'autre avec les ressources
- **Ressources autorisées** pendant le QCM : Encourage la recherche

---

## 💪 Valorisation du rôle du développeur

L'application met en avant le rôle **positif et essentiel** du développeur :

| Écran | Message clé |
|-------|-------------|
| **Alerte** | "Vos compétences sont essentielles : Investiguer, Sécuriser, Reconstruire, Innover" |
| **Investigation** | "Vous savez lire du code et des logs, une compétence rare et précieuse" |
| **Secure Coding** | "Un code bien sécurisé aurait empêché cette attaque" |
| **Pipeline** | "Ce pipeline sera votre héritage, protégeant l'équipe à chaque commit" |
| **Final** | "Le développeur : héros de la cybersécurité" |

---

## 📝 QCM — 20 questions

### Structure

| Partie | Questions | Thème | Fiche ressource |
|--------|-----------|-------|-----------------|
| 1 | Q1-Q5 | Ransomware et menaces | 04, 07 |
| 2 | Q6-Q9 | Gestion des secrets | 03, 06 |
| 3 | Q10-Q13 | Injection SQL & Secure Coding | 01, 02 |
| 4 | Q14-Q16 | Segmentation réseau | 04, 05 |
| 5 | Q17-Q19 | DevSecOps & Pipeline | 06 |
| 6 | Q20 | Réponse à incident (CNIL) | 07 |

### Import dans Pronote

1. Aller dans **Ressources pédagogiques → QCM**
2. **Importer → Fichier XML**
3. Sélectionner `mission_blackout_qcm.xml`

### Import dans Moodle

1. Aller dans **Banque de questions**
2. **Importer → Format XML Moodle**
3. Sélectionner le fichier (peut nécessiter une conversion)

---

## 📚 Contenu des fiches

### Fiche 01 — Injection SQL
- Définition et analogie simple
- Code vulnérable vs sécurisé
- Requêtes paramétrées, ORM
- Statistiques OWASP

### Fiche 02 — Hashage des mots de passe
- Pourquoi ne pas stocker en clair
- SHA-256 vs bcrypt vs Argon2
- Implémentation Python
- Erreurs courantes

### Fiche 03 — Gestion des secrets
- Définition des secrets
- Variables d'environnement (.env)
- HashiCorp Vault
- Détection avec GitLeaks

### Fiche 04 — Ransomware
- Fonctionnement d'une attaque
- Statistiques 2024
- Règle de sauvegarde 3-2-1-1
- Que faire en cas d'attaque

### Fiche 05 — Segmentation réseau
- Réseau plat vs segmenté
- VLANs et zones de confiance
- Zero Trust (3 piliers)
- Impact sur la sécurité

### Fiche 06 — DevSecOps
- Pipeline CI/CD sécurisé
- SAST vs DAST
- Scan des dépendances
- Exemple GitHub Actions

### Fiche 07 — Réponse à incident
- Cycle NIST (5 phases)
- Équipe CSIRT
- Erreurs à éviter
- Obligations CNIL (72h)

---

## 🔧 Personnalisation

### Modifier le scénario

Dans le fichier HTML, section `SCENARIO` :
```javascript
const SCENARIO = {
    company: { name: "VotreEntreprise", employees: 500 },
    attack: { ransom: "300 000€", deadline: 48 }
};
```

### Ajouter des questions au QCM

Éditer `mission_blackout_qcm.xml` et ajouter une structure :
```xml
<question id="21" type="single" points="1">
    <text>Votre question ici</text>
    <answers>
        <answer correct="false">Réponse A</answer>
        <answer correct="true">Bonne réponse</answer>
        <answer correct="false">Réponse C</answer>
    </answers>
    <feedback>Explication</feedback>
</question>
```

### Modifier les couleurs

Dans le CSS de l'application :
```css
:root {
    --accent-red: #ff4757;
    --accent-green: #2ed573;
    --accent-blue: #5352ed;
}
```


## 📞 Ressources complémentaires

### Pour les enseignants
- [ANSSI - Formations](https://www.ssi.gouv.fr/formation/)
- [CNIL - Ateliers RGPD](https://www.cnil.fr/fr/ateliers-rgpd)
- [Cybermalveillance.gouv.fr](https://www.cybermalveillance.gouv.fr/)

### Pour les étudiants
- [Root-Me](https://www.root-me.org/) — Challenges pratiques
- [TryHackMe](https://tryhackme.com/) — Parcours guidés
- [OWASP WebGoat](https://owasp.org/www-project-webgoat/) — Apprentissage

---

## 📄 Licence

Ce matériel pédagogique est destiné à un usage éducatif dans le cadre du BTS SIO.

---

*Mission Blackout v3 — BTS SIO SLAM — Cybersécurité*
*Créé avec ❤️ pour l'enseignement de la cybersécurité*
