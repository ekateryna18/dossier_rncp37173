# J1, Plan d'audit selon la méthode EBIOS Risk Manager (ANSSI)
### VNWeb, RNCP37173, M1.4

*Méthode officielle ANSSI, publiée en 2018, structurée en 5 ateliers : cadrage et socle de sécurité, sources de risque, scénarios stratégiques, scénarios opérationnels, traitement du risque (source : cyber.gouv.fr/securisation/analyse-des-risques/methode-ebios-rm/). Elle est adaptée ici à l'échelle d'une petite structure (4 personnes, pas de RSSI dédié) : les 5 ateliers sont conservés dans leur logique, mais avec un niveau de détail proportionné à l'exercice, pas celui d'une mission EBIOS RM complète en entreprise.*

---

## Contexte et objectifs de l'audit

Cet audit est mené dans le cadre de la certification RNCP37173 (ESDI Niveau 7), en situation professionnelle réelle au sein de VNWeb. L'objectif est d'identifier les risques et écarts, organisationnels et techniques, sur l'ensemble de l'environnement de l'entreprise, à savoir l'application weevus hébergée sur AWS et les autres applications/services hébergés sur l'infrastructure serveur de l'entreprise. L'audit est mené par l'apprentie elle-même (auditrice interne), sur son propre environnement de travail.

---

## Atelier 1, Cadrage et socle de sécurité

### 1.1 Valeurs métier

Ce que VNWeb cherche à protéger, indépendamment de la technique :

- Continuité de service des applications exploitées (weevus, Extranet, sites clients)
- Confidentialité des données clients et utilisateurs
- Intégrité du code source et des déploiements
- Réputation de VNWeb vis-à-vis de ses clients et partenaires

### 1.2 Biens supports

| Bien support | Porte quelle(s) valeur(s) métier | Hébergement |
|---|---|---|
| weevus (Amplify, Cognito, Lambda, DynamoDB, S3) | Continuité, confidentialité des données utilisateur | AWS, région Paris (eu-west-3), sauf table `users` en Ohio (us-east-2) |
| Extranet | Continuité, confidentialité des données clients | Interne à l'entreprise |
| Gitea | Intégrité du code source | Interne à l'entreprise, domaine VNWeb |
| Serveur de staging | Continuité, intégrité | Interne à l'entreprise |
| Serveur prod/preprod (VNWeb) | Continuité | Interne, accès disponible à l'auditrice, désormais dans le périmètre |
| Serveur des sites de voyance | Continuité | Externe, hors accès de l'auditrice (hors périmètre technique, cf. 1.4) |
| Canal Discord | Confidentialité (dans les faits, aujourd'hui vecteur de risque) | Service tiers, SaaS externe |
| Postes de travail des collaborateurs | Toutes les valeurs métier, en tant que point d'accès | Internes |

### 1.3 Événements redoutés

- Compromission d'un compte à privilèges (SSH, admin) menant à un accès non autorisé
- Fuite de données personnelles, notamment via la non-conformité de localisation constatée sur AWS
- Indisponibilité prolongée d'une application, faute de sauvegarde ou de plan de reprise
- Altération de code en production sans détection, faute de revue ou de tests
- Sanction réglementaire au titre du RGPD

### 1.4 Périmètre

| Volet | Inclus | Exclu (déclaré) |
|---|---|---|
| **Réseau** | Accès VPN d'entreprise, exposition des domaines/services publics, segmentation staging/preprod/prod (accès disponible sur le serveur hébergeant prod et preprod) | Réseau du serveur des sites de voyance, accès non disponible |
| **Serveur** | Serveur de staging interne (OS, comptes, SSH, durcissement) ; serveur hébergeant prod et preprod (accès disponible) ; hébergement AWS de weevus au niveau infrastructure | Serveur des sites de voyance ; code applicatif |
| **Pipelines** | Processus de déploiement weevus (CodeCommit vers AWS) et des dépôts Gitea, gestion des secrets associés | Contenu et qualité du code lui-même |
| **Organisationnel** | Communication d'équipe, gestion des accès, gestion des départs, gouvernance sécurité | - |
| **Conformité** | Localisation et souveraineté des données, base légale des transferts hors UE, applicabilité du RGPD | Conformité du serveur des sites de voyance, hors périmètre |

**Actifs et sites concernés** : Extranet, Gitea (tous dépôts hors weevus), weevus (AWS), serveur de staging, serveur hébergeant prod et preprod.
**Hors périmètre global** : serveur des sites de voyance, faute d'accès de l'auditrice, déclaré explicitement plutôt que passé sous silence.

### 1.5 Socle de sécurité retenu

En l'absence de politique de sécurité interne écrite chez VNWeb, le socle de référence est constitué de sources externes :

- ANSSI, *Guide d'hygiène informatique* (42 règles), pour le volet organisationnel et technique général
- ANSSI/CNIL, *Recommandations relatives à l'authentification multifacteur et aux mots de passe* (2021)
- OWASP (*Code Review Guide*, *DevSecOps Guideline*), pour le volet pipelines/développement
- RGPD, notamment Chapitre V (Art. 44 et suivants), pour le volet conformité
- ISO 27001:2022, Annexe A (contrôles organisationnels, humains, techniques), utilisée comme grille de comparaison, pas comme visée de certification formelle à ce stade

### 1.6 Ce qui doit être testé, et comment (écart au socle)

Le détail complet des vérifications par volet (réseau, serveur, pipelines, organisationnel, conformité) est conservé tel qu'établi précédemment. À noter : les résultats de ces vérifications alimentent aussi bien le rapport d'écarts classique (C3) que l'atelier 4 ci-dessous (scénarios opérationnels), puisque chaque écart technique est un maillon potentiel d'un chemin d'attaque.

### A. Réseau

| Élément testé | Comment |
|---|---|
| Connexion VPN | Vérifier que l'accès VPN d'entreprise fonctionne et comprendre son périmètre de couverture |
| Exposition des domaines | Résoudre les domaines exposés (Extranet, Gitea, weevus) et vérifier ce qui est accessible publiquement vs. interne |
| Segmentation des environnements | Entretien équipe : les flux entre staging/preprod/prod sont-ils cloisonnés, ou communiquent-ils librement ? |
| Pare-feu périmétrique | Vérifier s'il existe une règle de filtrage au niveau réseau (pas seulement au niveau serveur) |
| Chiffrement des flux (TLS/HTTPS) | Vérifier que tous les services exposés utilisent HTTPS/TLS, pas de flux en clair |
| Séparation réseau bureautique / administration | Vérifier si le poste utilisé pour administrer les serveurs est distinct du poste bureautique quotidien |
| Cartographie du réseau | Vérifier s'il existe un inventaire/schéma du réseau et des flux entre systèmes ; sinon, en établir un a minima pour cet audit |
| Filtrage sortant | Vérifier si des règles limitent aussi les connexions sortantes depuis les serveurs, pas seulement entrantes |

### B. Serveur

| Élément testé | Comment |
|---|---|
| Authentification SSH | Vérifier le mode d'authentification configuré sur le serveur (mot de passe vs. clé) |
| Protection anti-bruteforce | Vérifier la présence et l'état d'un service de protection anti-bruteforce (type fail2ban) |
| Pare-feu hôte | Vérifier l'état et les règles du pare-feu local du serveur |
| Niveau de mise à jour | Vérifier si les paquets système disposent de mises à jour en attente |
| AWS, périmètre des services | Services confirmés : Amplify (Gen 1), Cognito, Lambda, DynamoDB, S3 |
| AWS, statut Amplify | Confirmer la version (Gen 1) et l'échéance de fin de support (mai 2027) |
| AWS, localisation des données | Vérifier la région de chaque ressource (DynamoDB, Cognito, Lambda, S3) et repérer les écarts par rapport à la région principale (Paris) |
| AWS, IAM (niveau accès) | Revue des rôles/permissions au niveau infrastructure, pas du code |
| Logging / Monitoring du SI | Vérifier l'existence d'un système de journalisation centralisée et de supervision sur les serveurs et services |
| Sauvegardes | Vérifier l'existence, la fréquence et le test de restauration des sauvegardes du serveur de staging et des données critiques |
| Inventaire des actifs | Vérifier s'il existe une liste à jour des serveurs, services et logiciels installés |
| Comptes génériques / de service | Identifier les comptes partagés ou génériques et vérifier s'ils suivent une politique au moins aussi stricte que les comptes individuels |
| Séparation compte utilisateur / compte admin | Vérifier si chaque administrateur dispose d'un compte d'administration nominatif distinct de son compte utilisateur courant |
| Chiffrement au repos | Vérifier si les données stockées sont chiffrées au repos, pas seulement en transit |
| Plan de reprise/continuité | Vérifier s'il existe un plan écrit en cas d'indisponibilité prolongée du serveur ou de perte de données |

### C. Pipelines

| Élément testé | Comment |
|---|---|
| Déploiement weevus | Entretien équipe : le passage CodeCommit vers AWS est-il automatisé ou manuel ? |
| Déploiement Gitea | Vérifier si un CI/CD (Gitea Actions ou autre) est configuré sur les dépôts |
| Gestion des secrets dans le pipeline | S'il existe un pipeline, vérifier comment les secrets y sont injectés |
| Secrets commités par erreur | Vérifier dans l'historique Gitea/CodeCommit si des fichiers de configuration sensibles ou des identifiants ont déjà été poussés |
| Vulnérabilités des dépendances | Vérifier si les dépendances logicielles sont analysées pour des vulnérabilités connues, même manuellement |
| Protection des branches | Vérifier si des règles empêchent de pousser directement sur la branche de production |
| Traçabilité des déploiements | Vérifier s'il est possible de savoir qui a déployé quoi et quand |

### D. Organisationnel

| Élément testé | Comment |
|---|---|
| Communication d'équipe | Entretien : fréquence et canaux de communication, existence d'un point de synchronisation régulier |
| Gestion des accès | Entretien : qui a accès à quoi, existe-t-il une revue périodique des accès |
| Gestion des départs | Entretien : existe-t-il une procédure de retrait d'accès et de transfert de connaissance au départ d'un membre |
| Gouvernance sécurité | Entretien : existe-t-il un responsable sécurité (RSSI) désigné, une politique écrite, un processus de validation des mises en production |
| Sensibilisation sécurité de l'équipe | Entretien : l'équipe connaît-elle les pratiques de base ; existe-t-il un rappel, une doc, ou une formation sur ces sujets |
| Politique de sécurité écrite | Vérifier s'il existe un document, même court, formalisant des règles de sécurité |
| Gestion des tiers/sous-traitants | Identifier les prestataires externes ayant accès aux données ou aux systèmes, et vérifier s'il existe un contrôle sur ces accès |
| Procédure de gestion des incidents | Vérifier s'il existe une procédure décrivant que faire en cas d'incident détecté |
| Charte informatique / usage acceptable | Vérifier s'il existe des règles communiquées à l'équipe sur l'usage des outils |

### E. Conformité

| Élément testé | Comment |
|---|---|
| Localisation des données | Vérifier la région d'hébergement de chaque ressource et identifier celles situées hors de l'UE |
| Souveraineté / transfert hors UE | Pour chaque ressource hors UE, vérifier s'il existe une base légale de transfert ou si le transfert n'est pas justifié |
| Nature des données traitées | Identifier si les données sur l'Extranet et weevus incluent des données à caractère personnel |
| Applicabilité du RGPD | Déterminer si le RGPD s'applique formellement et à quel niveau de risque |
| Autres contraintes réglementaires sectorielles | Vérifier s'il existe une contrainte propre au secteur de VNWeb au-delà du RGPD |
| Registre des traitements (Art. 30 RGPD) | Vérifier s'il existe un registre listant les traitements de données personnelles |
| Durée de conservation des données | Vérifier si une durée de conservation est définie |
| Droits des personnes concernées | Vérifier s'il existe un moyen pour une personne de demander l'accès, la rectification ou la suppression de ses données |

---

## Atelier 2, Sources de risque et objectifs visés

| Source de risque (SR) | Objectif visé (OV) | Pertinence pour VNWeb |
|---|---|---|
| Cybercriminel opportuniste | Monétiser des accès compromis (revente d'identifiants, rançongiciel) via des identifiants exposés (Discord) ou un accès SSH faible | Élevée, correspond directement aux écarts déjà constatés |
| Négligence ou erreur interne (source non malveillante) | Exposition accidentelle de données ou de secrets, faute de procédure (partage sur Discord, absence de revue avant mise en production) | Très élevée, c'est le scénario le plus probable compte tenu du contexte observé |
| Ancien collaborateur ou tiers dont l'accès n'a pas été révoqué | Utiliser un accès résiduel après un départ non accompagné d'une procédure de retrait | Moyenne, dépend du turnover réel de l'équipe |
| Compromission de la chaîne logicielle (dépendances) | Introduire du code malveillant via une dépendance non vérifiée, faute d'analyse de vulnérabilités | Moyenne à faible, absence de CI/CD limite aussi la surface, mais aucune détection n'existerait si le cas se présentait |

**Couples SR/OV priorisés pour la suite de l'analyse** : négligence interne (le plus probable) et cybercriminel opportuniste (le plus grave en cas de succès), retenus pour les ateliers 3 et 4.

---

## Atelier 3, Scénarios stratégiques

### Cartographie de l'écosystème

- **Parties prenantes internes** : équipe de 4 personnes, patron (sans compétence technique, validateur fonctionnel)
- **Parties prenantes externes** : AWS (hébergeur cloud de weevus), Discord (outil de communication tiers, sans garantie de sécurité contractualisée pour un usage professionnel), fournisseur d'accès/Freebox (VPN)

### Chemins d'attaque stratégiques

**Chemin A** : Fuite d'identifiants sur Discord, accès au serveur de staging via SSH (authentification faible), pivot potentiel vers d'autres environnements faute de segmentation confirmée.

**Chemin B** : Absence de revue de code, introduction de code défectueux ou vulnérable, mise en production sans détection faute de tests ou de monitoring.

**Chemin C** : Compte d'un collaborateur parti non révoqué, accès résiduel à Gitea ou à l'Extranet.

Le chemin A est retenu pour l'atelier 4, car il combine plusieurs écarts déjà confirmés (secrets en clair, SSH par mot de passe, absence de fail2ban et de journalisation), ce qui en fait le scénario le plus vraisemblable et le plus documentable à ce stade.

---

## Atelier 4, Scénarios opérationnels

### Déroulé technique du chemin A

1. Un identifiant est partagé en clair sur Discord, canal sans garantie de confidentialité pour un usage professionnel, accessible à toute personne ayant accès à l'historique du salon
2. L'identifiant est récupéré et utilisé pour tenter une connexion SSH sur le serveur de staging
3. Aucune protection anti-bruteforce ne limite les tentatives de connexion (fail2ban absent)
4. Aucune journalisation centralisée ne permet de détecter la connexion, réussie ou non
5. Une fois connecté, l'accès obtenu est équivalent à celui d'un administrateur légitime, faute de séparation entre compte utilisateur et compte d'administration

### Estimation de la vraisemblance

Sur l'échelle à 4 niveaux utilisée par l'ANSSI (minime, significative, forte, maximale), ce scénario est estimé à un niveau **fort** : chaque maillon de la chaîne (partage du secret, authentification faible, absence de détection) est un écart déjà confirmé, sans mesure compensatoire en place à aucune étape.

---

## Atelier 5, Traitement du risque

### Stratégies de traitement envisagées

| Stratégie | Application chez VNWeb |
|---|---|
| **Réduire** | Stratégie retenue pour la majorité des écarts identifiés (cf. plan d'action détaillé en J2) |
| **Accepter** | Applicable au risque résiduel documenté après mise en œuvre d'une mesure, avec un responsable identifié |
| **Transférer** | Non retenu à ce stade, aucune assurance cyber ou clause de transfert de responsabilité identifiée |
| **Éviter** | Non pertinent ici, les biens supports concernés sont nécessaires à l'activité |

Le détail chiffré du plan d'action (mesures, responsables, échéances, arbitrage) est construit dans le rapport J2, qui découle directement des écarts identifiés en atelier 1 et du scénario opérationnel de l'atelier 4.

### Routines et bonnes pratiques à instaurer pour les employés

Au-delà des mesures ponctuelles, plusieurs routines sont proposées pour ancrer la sécurité dans le fonctionnement quotidien de l'équipe :

- **Point de synchronisation hebdomadaire court**, incluant un rappel ou un point sécurité (anomalie constatée, rappel d'une règle de base)
- **Interdiction formalisée du partage d'identifiants sur Discord**, avec un rappel explicite communiqué à toute l'équipe, pas seulement une pratique tacite
- **Checklist de départ d'un collaborateur** : révocation systématique des accès (Gitea, Extranet, AWS, VPN), transfert de la documentation liée à son périmètre
- **Checklist d'arrivée d'un collaborateur** : attribution d'accès nominatifs uniquement (pas de comptes partagés), rappel des règles de base dès l'arrivée
- **Revue périodique des accès**, à intervalle régulier (par exemple trimestriel), pour vérifier que chaque accès actif est encore justifié
- **Sensibilisation continue**, sous une forme légère et récurrente plutôt qu'une formation isolée : rappel des règles de base (pas de secrets en clair, activation du MFA partout où c'est possible, vigilance avant de pousser du code)

---

*Document de travail, à compléter/valider avant intégration au rapport final.*
