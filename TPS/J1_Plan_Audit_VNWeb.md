# J1, Plan d'audit & analyse de sécurité de l'information et des données
### VNWeb, RNCP37173, M1.4

---

## 1. Contexte et objectifs de l'audit

Cet audit est mené dans le cadre de la certification RNCP37173 (ESDI Niveau 7), en situation professionnelle réelle au sein de VNWeb. L'objectif est d'identifier les écarts organisationnels et techniques par rapport à des bonnes pratiques reconnues, afin de produire un rapport d'audit, un plan d'action arbitré, et une base pour une éventuelle démarche d'accréditation.

L'audit est mené par l'apprentie elle-même (auditrice interne), sur son propre environnement de travail.

---

## 2. Périmètre

Le périmètre couvre quatre volets. Le détail de ce qui est testé dans chacun est donné en section 3.

| Volet | Inclus | Exclu (déclaré) |
|---|---|---|
| **Réseau** | Accès VPN d'entreprise, exposition des domaines/services publics, segmentation staging/preprod/prod | Réseau des serveurs preprod/prod externes, accès non disponible |
| **Serveur** | Serveur de staging interne (OS, comptes, SSH, durcissement) ; hébergement AWS de weevus au niveau infrastructure | Serveurs preprod/prod externes ; code applicatif |
| **Pipelines** | Processus de déploiement weevus (CodeCommit → AWS) et des dépôts Gitea, gestion des secrets associés | Contenu et qualité du code lui-même |
| **Organisationnel** | Communication d'équipe, gestion des accès, gestion des départs, gouvernance sécurité | - |

**Actifs et sites concernés** : Extranet, Gitea (tous dépôts hors weevus), weevus (AWS), serveur de staging.
**Hors périmètre global** : serveurs preprod et prod externes, faute d'accès de l'auditrice, déclaré explicitement plutôt que passé sous silence.

---

## 3. Méthodologie d'audit, ce qui est testé, et comment

### A. Réseau

| Élément testé | Comment |
|---|---|
| Connexion VPN | Vérifier que l'accès VPN d'entreprise fonctionne et comprendre son périmètre de couverture |
| Exposition des domaines | Résoudre les domaines exposés (`vnweb.freeboxos.fr`, domaine Gitea, domaine Extranet) et vérifier ce qui est accessible publiquement vs. interne |
| Segmentation des environnements | Entretien équipe : les flux entre staging/preprod/prod sont-ils cloisonnés, ou communiquent-ils librement ? |
| Pare-feu périmétrique | Vérifier s'il existe une règle de filtrage au niveau réseau (pas seulement au niveau serveur) |

### B. Serveur

| Élément testé | Comment |
|---|---|
| Authentification SSH | Vérifier le mode d'authentification configuré sur le serveur (mot de passe vs. clé) |
| Protection anti-bruteforce | Vérifier la présence et l'état d'un service de protection anti-bruteforce (type fail2ban) |
| Pare-feu hôte | Vérifier l'état et les règles du pare-feu local du serveur |
| Niveau de mise à jour | Vérifier si les paquets système disposent de mises à jour en attente |
| Comptes utilisateurs | Lister les comptes existants, identifier s'ils sont individuels ou partagés |
| AWS, statut Amplify | Confirmer la version (Gen 1) et l'échéance de fin de support (mai 2027) |
| AWS, localisation des données | Vérifier la région de chaque ressource (DynamoDB, S3, Cognito) et repérer les écarts par rapport à la région principale (Paris) |
| AWS, IAM (niveau accès) | Revue des rôles/permissions au niveau infrastructure, pas du code |
| Logging / Monitoring du SI | Vérifier l'existence d'un système de journalisation centralisée et de supervision (alerting) sur les serveurs et services ; entretien équipe : comment un incident serait-il détecté aujourd'hui, en combien de temps |

### C. Pipelines

| Élément testé | Comment |
|---|---|
| Déploiement weevus | Entretien équipe : le passage CodeCommit → AWS est-il automatisé ou manuel ? |
| Déploiement Gitea | Vérifier si un CI/CD (Gitea Actions ou autre) est configuré sur les dépôts |
| Gestion des secrets dans le pipeline | S'il existe un pipeline, vérifier comment les secrets y sont injectés (variables sécurisées vs. en dur) |
| Secrets commités par erreur | Vérifier dans l'historique Gitea/CodeCommit si des fichiers de configuration sensibles ou des identifiants ont déjà été poussés (recherche dans l'historique, pas seulement l'état actuel) |

### D. Organisationnel

| Élément testé | Comment |
|---|---|
| Communication d'équipe | Entretien : fréquence et canaux de communication, existence d'un point de synchronisation régulier |
| Gestion des accès | Entretien : qui a accès à quoi, existe-t-il une revue périodique des accès |
| Gestion des départs | Entretien : existe-t-il une procédure de retrait d'accès et de transfert de connaissance au départ d'un membre |
| Gouvernance sécurité | Entretien : existe-t-il un responsable sécurité (RSSI) désigné, une politique écrite, un processus de validation des mises en production |
| Sensibilisation sécurité de l'équipe | Entretien : l'équipe connaît-elle les pratiques de base (ne pas commiter de fichiers de configuration sensibles, ne pas partager d'identifiants en clair) ; existe-t-il un rappel, une doc, ou une formation sur ces sujets, même informelle |

---

## 4. Cadrage de l'audit

| Critère | Constat |
|---|---|
| **Moyens** | Temps disponible dans le cadre de l'alternance ; pas d'outillage d'audit automatisé (scanner, SIEM) disponible |
| **Ressources** | Équipe de 4 personnes ; pas de RSSI (Responsable de la Sécurité des Systèmes d'Information) désigné ; l'auditrice cumule le rôle d'auditrice et de développeuse |
| **Organisation** | Pas de gouvernance sécurité formalisée ; validation fonctionnelle des livraisons faite par le patron (sans compétence technique) |
| **Contraintes réglementaires** | RGPD potentiellement applicable (données clients sur l'Extranet, données utilisateurs sur weevus), à confirmer côté nature des données traitées |

---

## 5. Analyse de sécurité de l'information et des données (C2)

### Actifs de données identifiés

| Actif | Localisation | Nature |
|---|---|---|
| Identifiants / secrets | Circulent en clair sur Discord | Confidentiel, critique |
| Données de tâches / projet | Extranet | Possiblement à caractère personnel (clients) |
| Code source | Gitea + weevus (AWS) | Propriété intellectuelle de l'entreprise |
| Données utilisateur weevus | AWS DynamoDB | Données personnelles (app familiale) |

### Classification (sensibilité simplifiée)

- **Identifiants/secrets**, confidentialité critique, actuellement **non protégée** (canal non maîtrisé)
- **Données clients (Extranet)**, confidentialité/intégrité importante, RGPD potentiellement engagé
- **Code source**, confidentialité/intégrité importante, aucun contrôle de revue avant mise en production
- **Données weevus**, confidentialité/disponibilité importante, données personnelles d'utilisateurs finaux

### Menaces identifiées (analyse préliminaire, à affiner en EBIOS RM si le format l'exige)

- Compromission de comptes par fuite d'identifiants (canal Discord non sécurisé)
- Introduction de code défectueux ou malveillant en production, faute de revue
- Incident de sécurité non détecté, faute de supervision (logging/monitoring absent)
- Perte de continuité opérationnelle en cas de départ d'un membre (pas de documentation, pas de suppléance sur le rôle admin)

Ces menaces et actifs constituent la base d'entrée pour la qualification des écarts en J2.

---

*Document de travail, à compléter/valider avant intégration au rapport final.*
