# J2, Rapport d'audit des écarts, plan d'action arbitré, préparation à l'accréditation
### VNWeb, RNCP37173, M1.4

*Construit à partir du plan d'audit (J1) et des constats déjà collectés. Certaines cases restent "à compléter" faute d'observation confirmée : c'est volontaire, pas un oubli (cf. réflexe "déclarer ce qui n'est pas couvert").*

---

## 1. Rapport d'audit des écarts (C3)

| # | Volet | Procédure attendue | Observation | Preuve d'audit | Écart qualifié |
|---|---|---|---|---|---|
| 1 | Serveur | Authentification forte (clé SSH) sur accès admin distant, ANSSI hygiène informatique | Accès SSH au serveur de staging en mot de passe uniquement, pas de clé | Constat direct de l'auditrice lors d'une connexion au serveur | Authentification faible sur l'accès serveur, surface d'attaque brute-force |
| 2 | Serveur | Protection anti-bruteforce sur service SSH exposé | Pas de fail2ban ou équivalent identifié | Constat direct de l'auditrice | Absence de mitigation face aux tentatives de connexion répétées |
| 3 | Serveur | Filtrage réseau au niveau serveur | Statut du pare-feu non confirmé | À vérifier | À qualifier une fois la commande de vérification exécutée |
| 4 | Serveur | Plateforme d'hébergement maintenue et supportée | Application weevus sur AWS Amplify Gen 1, fin de support prévue mai 2027, migration non finalisée | Confirmé via documentation AWS/roadmap interne | Dépendance sur une techno en fin de vie proche, absence de correctifs de sécurité après l'échéance |
| 5 | Serveur | Localisation des données dans l'UE pour données à caractère personnel (RGPD Art. 44-49), ou base légale de transfert documentée | Table DynamoDB `users` hébergée en Ohio (us-east-2), reste des ressources en Paris (eu-west-3) | Constat direct dans la console AWS | Non-conformité de localisation, transfert hors UE sans garantie identifiée |
| 6 | Technique (transverse) | Secrets jamais transmis en clair hors d'un système dédié | Identifiants/mots de passe partagés en clair sur Discord | Témoignage direct de l'équipe | Absence de gestion des secrets, canal non maîtrisé et non audité |
| 7 | Pipelines | Revue technique avant mise en production (OWASP secure SDLC) | Pas de revue de code ; validation faite par le patron via test manuel, sans compétence technique | Témoignage équipe | Aucune validation technique indépendante avant livraison |
| 8 | Pipelines | Couverture de tests comme condition de mise en production | Pas de tests systématiques, manuels ou automatisés | Témoignage équipe | Aucune vérification de non-régression avant livraison |
| 9 | Pipelines | Build/déploiement automatisé et auditable (DevSecOps) | Pas de pipeline CI/CD | Témoignage équipe | Déploiements manuels, non tracés |
| 10 | Serveur/SI | Journalisation centralisée et supervision (ANSSI ; ISO 27001:2022 Annexe A 8.15/8.16) | Pas de logging/monitoring sur les serveurs, un incident n'est détecté que si quelqu'un le signale | Témoignage équipe | Absence de visibilité sur les incidents, délai de détection inconnu |
| 11 | Serveur | Sécurisation homogène sur toute la chaîne staging, preprod, prod | Serveur de staging non sécurisé | Constat direct de l'auditrice | Le maillon staging fragilise l'ensemble de la chaîne de promotion |
| 12 | Organisationnel | Cadence de coordination régulière, même légère, pour une équipe de 4 | Pas de réunion récurrente, seulement avant chaque fonctionnalité, sinon communication seulement si nécessaire | Témoignage équipe | Risque que des informations de sécurité ne circulent pas entre les membres |
| 13 | Organisationnel | Procédure de transfert de connaissance au départ d'un collaborateur (ISO 27001:2022, Annexe A 6.5) | Aucune documentation transmise en cas de départ | Témoignage équipe | Perte de connaissance et de continuité en cas de départ |
| 14 | Organisationnel | Séparation des rôles / continuité sur les rôles à privilèges | Pas de suppléant identifié sur le rôle admin | Témoignage équipe | Point unique de défaillance sur l'accès admin |

**Écarts restant à qualifier** (dépendent de vérifications techniques non encore faites) :
- Pare-feu du serveur de staging (écart #3)
- Niveau de mise à jour système du serveur
- Comptes utilisateurs (individuels ou partagés)
- Autres ressources AWS hors région Paris (Cognito, Lambda, S3, au-delà de la table `users` sur DynamoDB)
- WAF/CDN devant les parties exposées publiquement
- Chiffrement au repos confirmé sur DynamoDB/S3
- Secrets commités par erreur dans l'historique Gitea/CodeCommit
- Sensibilisation réelle de l'équipe aux pratiques de base

---

## 2. Plan d'action (C4)

| Écart # | Mesure | Type | Justification | Responsable | Échéance | Preuve attendue | Risque résiduel |
|---|---|---|---|---|---|---|---|
| 1 | Migrer vers une authentification par clé SSH, désactiver l'authentification par mot de passe | Corrective | ANSSI, guide d'hygiène informatique | À définir | À définir | Config `sshd_config` mise à jour, test de connexion par clé réussi | Perte de clé privée si mal gérée, à couvrir par une procédure de gestion des clés |
| 2 | Installer et configurer fail2ban sur le serveur de staging | Corrective + Préventive | ANSSI | À définir | À définir | Service actif, règle testée | Faux positifs possibles, à surveiller |
| 4 | Planifier la migration Amplify Gen 1 vers Gen 2 en environnement sandbox | Préventive | ANSSI (maintien en condition de sécurité) | À définir (déjà identifié comme chantier en cours) | Avant mai 2027 | Environnement sandbox validé, plan de bascule écrit | Risque de régression pendant la migration, à couvrir par des tests en sandbox |
| 5 | Migrer la table `users` vers la région Paris (eu-west-3), ou documenter une base légale de transfert | Corrective | RGPD Art. 44-49 | À définir | À définir | Ressource migrée ou documentation de transfert validée | Coût/temps de migration, indisponibilité potentielle pendant la bascule |
| 6 | Mettre en place un gestionnaire de secrets partagé (ex. coffre-fort type Bitwarden/Vault), interdire le partage sur Discord | Corrective + Préventive | ANSSI, guide d'hygiène informatique | À définir | À définir | Outil déployé, ancien canal purgé | Adoption par l'équipe à vérifier dans le temps |
| 7 | Introduire une revue de code obligatoire avant fusion, même légère | Corrective + Préventive | OWASP Code Review Guide / Secure Code Review Cheat Sheet | À définir | À définir | Trace de revue sur les fusions postérieures | Ralentissement du cycle de livraison, à arbitrer avec le patron |
| 9 | Mettre en place un pipeline CI/CD minimal (build + tests de base) | Préventive | OWASP DevSecOps Guideline | À définir | À définir | Pipeline fonctionnel, historique des runs | Effort de mise en place initial |
| 10 | Mettre en place une journalisation centralisée basique et une alerte minimale | Corrective + Préventive | ANSSI ; ISO 27001:2022 Annexe A 8.15/8.16 | À définir | À définir | Logs centralisés consultables, alerte testée | Volume de logs à gérer dans le temps |
| 12 | Instaurer un point de synchronisation court et régulier (ex. hebdomadaire) | Préventive | ISO 27001:2022, clause 7.4 (Communication) | À définir | À définir | Récurrence effective sur plusieurs semaines | Adoption dépend de la discipline d'équipe |
| 13 | Rédiger un document minimal de transfert de connaissance par rôle/projet | Préventive | ISO 27001:2022, Annexe A 6.5 | À définir | À définir | Document existant et à jour | Doit être maintenu dans le temps, sinon devient obsolète |
| 14 | Désigner un suppléant sur le rôle admin | Corrective | ISO 27001:2022, Annexe A 5.3 (Segregation of Duties) | Patron (décision d'organisation) | À définir | Suppléant identifié et testé sur un accès réel | Dépend de la disponibilité d'un second profil compétent |

### Détail des références citées

**Mesure #1 (authentification SSH)** — ANSSI/CNIL, *Recommandations relatives à l'authentification multifacteur et aux mots de passe* (2021, cosignée CNIL) : un compte à privilèges (administration) doit être protégé par une authentification plus robuste qu'un simple mot de passe, la sensibilité de l'accès l'exigeant particulièrement.

**Mesure #2 (fail2ban)** — ANSSI, *Guide d'hygiène informatique* : principe général de limitation de l'exposition et des tentatives d'accès sur les ressources d'administration, présentées comme des cibles privilégiées des attaquants.

**Mesure #5 (localisation DynamoDB)** — RGPD, Chapitre V (Art. 44 et suivants) : tout transfert de données personnelles hors UE doit reposer sur une décision d'adéquation ou des garanties appropriées ; le responsable du traitement reste pleinement responsable de la conformité, quel que soit le lieu où le traitement a lieu.

**Mesure #6 (gestion des secrets)** — ANSSI, *Guide d'hygiène informatique* : les ressources sensibles doivent être identifiées et les secrets systématiquement chiffrés avant transmission ou hébergement, via un canal distinct de celui utilisé pour les données elles-mêmes.

**Mesure #10 (journalisation/monitoring)** — ANSSI, *Guide d'hygiène informatique* : la journalisation liée aux comptes (connexions réussies et échouées) doit être activée dès que possible ; recommandation classée "renforcée" dans le guide.

**Mesure #4 (migration Amplify Gen 1)** — ANSSI, *Guide d'hygiène informatique* : principe général de maintien en condition de sécurité, un système sorti du support de son éditeur ne reçoit plus de correctifs de sécurité. Le calendrier précis (fin de support mai 2027) est une donnée éditeur AWS, pas une exigence normative en soi.

**Mesure #7 (revue de code)** — OWASP, *Code Review Guide* et *Secure Code Review Cheat Sheet* (cheatsheetseries.owasp.org) : la revue de code doit être intégrée au processus de développement (notamment au moment de la pull request), pas ajoutée après coup ; elle combine idéalement revue humaine et outillage automatisé.

**Mesure #9 (pipeline CI/CD)** — OWASP, *DevSecOps Guideline* (projet officiel OWASP) : recommande d'intégrer des étapes de sécurité dès les premières étapes d'un pipeline, par exemple la détection de secrets dans les dépôts, avant d'ajouter progressivement des contrôles supplémentaires (SAST/DAST, audit des dépendances).

**Mesure #12 (cadence de communication)** — ISO 27001:2022, clause 7.4 (Communication) : l'organisation doit déterminer les besoins de communication interne pertinents pour la sécurité de l'information, y compris leur fréquence. Il s'agit d'une clause du corps de la norme, pas de l'Annexe A.

**Mesure #13 (transfert de connaissance au départ)** — ISO 27001:2022, Annexe A, contrôle 6.5 (*Responsibilities after termination or change of employment*, qui remplace le contrôle 7.3.1 de la version 2013) : les responsabilités de sécurité et les accès qui doivent être transférés ou révoqués au départ d'un collaborateur doivent être définis et appliqués.

**Mesure #14 (suppléant sur le rôle admin)** — ISO 27001:2022, Annexe A, contrôle 5.3 (*Segregation of Duties*) : les fonctions et responsabilités conflictuelles doivent être séparées pour réduire le risque qu'une seule personne puisse compromettre un processus critique sans être détectée. Le contrôle prévoit explicitement que les petites structures, incapables d'une séparation complète, mettent en place des contrôles compensatoires (supervision, traçabilité) — pertinent pour VNWeb.

*Les mesures 3, 8, 11 et les écarts "restant à qualifier" ne peuvent pas être arbitrées tant que leurs preuves ne sont pas confirmées.*

---

## 3. Arbitrage

### Critères retenus

L'arbitrage s'appuie sur les quatre termes qui cadraient déjà le plan d'audit en J1 (moyens, ressources, organisation, contraintes réglementaires), complétés par la gravité et les dépendances entre mesures, conformément à la méthode décrite en séquence 3 du cours.

| Critère | Ce qu'il signifie pour VNWeb |
|---|---|
| **Gravité** | Impact sur la confidentialité, l'intégrité ou la disponibilité si l'écart n'est pas traité |
| **Dépendances** | Une mesure en bloque-t-elle ou en conditionne-t-elle une autre (ex. la journalisation, #10, doit passer tôt car elle permet de vérifier l'effet des autres mesures) |
| **Moyens** | Absence d'outillage d'audit automatisé et de temps dédié illimité, donc priorité aux mesures nécessitant peu d'outillage |
| **Ressources** | Équipe de 4, pas de RSSI désigné, l'auditrice cumule les rôles, donc certaines mesures dépendent de sa seule disponibilité |
| **Organisation** | Pas de gouvernance formalisée, le patron valide sans compétence technique, donc certaines mesures (ex. revue de code obligatoire, #7) nécessitent son adhésion avant de pouvoir démarrer |
| **Contraintes réglementaires** | Le RGPD engage prioritairement la mesure #5 (localisation DynamoDB), qui porte donc un poids réglementaire que d'autres écarts n'ont pas |

### Pondération retenue

Conformément à la méthode du cours ("poser les critères d'arbitrage et leur poids avant de classer"), votre ordre de priorité (gravité + dépendances, puis moyens + ressources, puis organisation + contraintes réglementaires) est converti ici en poids numériques, sur un total de 100 :

| Critère | Poids |
|---|---|
| Gravité | 30 |
| Dépendances | 20 |
| Moyens | 15 |
| Ressources | 15 |
| Organisation | 10 |
| Contraintes réglementaires | 10 |
| **Total** | **100** |

Les deux premiers critères (gravité + dépendances) pèsent 50 % du score, moyens + ressources 30 %, organisation + contraintes réglementaires 20 %, reflétant directement votre ordre de priorité tout en permettant à un écart de gravité très élevée de rester déterminant même si un critère plus bas dans l'ordre est défavorable, ce qu'un simple tri par tiers ne permettait pas.

**Méthode de notation** : chaque mesure est notée de 1 (faible) à 5 (fort) sur chaque critère. Pour Gravité et Dépendances, 5 = urgence/effet d'entraînement élevé. Pour Moyens, Ressources et Organisation, 5 = facilement mobilisable/aucun blocage. Pour Contraintes réglementaires, 5 = fort enjeu légal. Le score pondéré = somme de (note × poids / 100).

*Les notes ci-dessous sont une proposition de départ pour chaque mesure : à valider ou corriger avant de les considérer comme définitives, la notation elle-même relevant de votre jugement.*

### Classement des mesures (score pondéré)

Les mesures liées aux écarts #3, #8, #11 restent hors classement, faute de preuve confirmée (cf. section 1).

**Note de lecture** : dans le tableau ci-dessous, le chiffre entre parenthèses dans l'en-tête de chaque colonne (30, 20, 15, 15, 10, 10) est le **poids** du critère, sur un total de 100. Le chiffre dans chaque cellule est la **note de la mesure sur ce critère, sur une échelle de 1 à 5** ; ce n'est pas une fraction du poids.

| Mesure | Gravité (30) | Dépend. (20) | Moyens (15) | Ressources (15) | Organisation (10) | Réglem. (10) | Score pondéré | Rang |
|---|---|---|---|---|---|---|---|---|
| #1, Authentification SSH par clé | 5 | 2 | 5 | 5 | 5 | 1 | **4.00** | 1 |
| #10, Journalisation / monitoring | 4 | 5 | 3 | 4 | 5 | 2 | **3.95** | 2 |
| #6, Gestion des secrets (Discord) | 5 | 2 | 4 | 4 | 4 | 2 | **3.70** | 3 |
| #2, Fail2ban | 3 | 1 | 5 | 5 | 5 | 1 | **3.20** | 4 |
| #5, Migration région DynamoDB (RGPD) | 4 | 1 | 2 | 3 | 3 | 5 | **2.95** | 5 |
| #9, Pipeline CI/CD minimal | 3 | 3 | 2 | 3 | 5 | 1 | **2.85** | 6 |
| #4, Migration Amplify Gen 1 → Gen 2 | 3 | 2 | 2 | 3 | 4 | 1 | **2.55** | 7 |
| #12, Cadence de communication | 2 | 1 | 5 | 4 | 3 | 1 | **2.55** | 7 |
| #7, Revue de code obligatoire | 3 | 1 | 4 | 3 | 2 | 1 | **2.45** | 9 |
| #13, Document de transfert de connaissance | 2 | 1 | 4 | 4 | 3 | 1 | **2.40** | 10 |
| #14, Suppléant sur le rôle admin | 3 | 1 | 4 | 2 | 1 | 1 | **2.20** | 11 |

**Formule utilisée** : Score pondéré = Σ (note du critère × poids du critère) ÷ 100

**Exemple, mesure #1** : (5×30 + 2×20 + 5×15 + 5×15 + 5×10 + 1×10) ÷ 100 = (150 + 40 + 75 + 75 + 50 + 10) ÷ 100 = **4,00**

**Point de vigilance** : le score pondéré place les contraintes réglementaires (poids 10) plus bas que la gravité et les dépendances, mais cela ne doit pas se traduire par un délai excessif dans le calendrier réel : le risque RGPD (mesure #5) continue de courir tant qu'il n'est pas traité, indépendamment de son rang ici. Une échéance raisonnable doit lui être fixée malgré sa 5ᵉ place.

*Ce classement applique les poids que vous avez fixés à des notes proposées par défaut ; à ajuster si votre lecture d'un critère précis diffère pour une mesure donnée.*

---

## 4. Dossier de preuves pour l'accréditation (C5) — ébauche

**Norme cible proposée** : ANSSI, guide d'hygiène informatique (référentiel de pratiques, pas une norme de certification formelle, à confirmer avec l'enseignant si une norme certifiante type ISO 27001 est attendue à la place)

| Exigence (exemple) | Statut | Preuve | Renvoi plan d'action |
|---|---|---|---|
| Authentification forte sur les accès distants | Non couverte | Constat #1 | Mesure #1 |
| Gestion sécurisée des secrets | Non couverte | Constat #6 | Mesure #6 |
| Supervision et détection d'incident | Non couverte | Constat #10 | Mesure #10 |
| Continuité sur les rôles à privilèges | Non couverte | Constat #14 | Mesure #14 |

*Ébauche minimale, à étoffer une fois la norme cible confirmée et les autres écarts qualifiés.*

---

## 5. Synthèse (Résultat, Preuve, Limite, Suite)

- **Résultat** : 14 écarts qualifiés à ce stade, répartis entre serveur, pipelines et organisationnel ; plan d'action initial rédigé pour la majorité d'entre eux.
- **Preuve** : constats directs de l'auditrice pour les éléments techniques vérifiables (SSH, AWS), témoignages d'équipe pour les éléments organisationnels et certains éléments techniques non encore vérifiés en direct.
- **Limite** : plusieurs vérifications techniques restent à faire (pare-feu, mises à jour, comptes, secrets dans l'historique du code) ; les serveurs preprod et prod externes restent hors périmètre faute d'accès ; responsables et échéances du plan d'action restent à assigner avec l'équipe/le patron.
- **Suite** : compléter les vérifications techniques listées en fin de section 1, valider le plan d'action avec l'équipe, construire l'arbitrage une fois toutes les preuves confirmées.

---

## 6. Sources et références citées

| Source | Lien | Où le trouver dans le document source | Utilisé pour |
|---|---|---|---|
| ANSSI, *Guide d'hygiène informatique* | https://messervices.cyber.gouv.fr/documents-guides/guide_hygiene_informatique_anssi.pdf | Document de 42 règles ; catégories "authentification", "sauvegarde", "gestion des secrets" et "journalisation" | Mesures #2, #4, #6 ; écarts #2, #6 |
| ANSSI/CNIL, *Recommandations relatives à l'authentification multifacteur et aux mots de passe* (2021) | https://messervices.cyber.gouv.fr/documents-guides/anssi-guide-authentification_multifacteur_et_mots_de_passe.pdf | Section sur l'authentification des comptes à privilèges | Mesure #1 ; écart #1 |
| RGPD, Chapitre V (Art. 44 et suivants) | https://www.gdpr-expert.eu/article.html?id=44 | Article 44, "Principe général applicable aux transferts" | Mesure #5 ; écart #5 |
| OWASP, *Code Review Guide* | https://owasp.org/www-project-code-review-guide/ | Section 1, "why and how of code reviews" | Mesure #7 ; écart #7 |
| OWASP, *Secure Code Review Cheat Sheet* | https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html | Section sur l'intégration de la revue au cycle de développement (pull request) | Mesure #7 ; écart #7 |
| OWASP, *DevSecOps Guideline* | https://owasp.org/www-project-developer-guide/release/operations/devsecops_guideline/ | Étapes recommandées d'un pipeline de base, dont le scan des dépôts pour fuite de secrets | Mesure #9 ; écarts #9, #6 |
| ISO 27001:2022, clause 7.4 (Communication) | https://advisera.com/iso27001/clause-7-4-communication/ | Corps de la norme, clause 7.4 (hors Annexe A) | Mesure #12 ; écart #12 |
| ISO 27001:2022, Annexe A, contrôle 6.5 | https://www.isms.online/iso-27001/annex-a-2022/6-5-responsibilities-after-termination-change-of-employment-2022/ | Annexe A, thème "Humain", contrôle 6.5 (remplace 7.3.1 en version 2013) | Mesure #13 ; écart #13 |
| ISO 27001:2022, Annexe A, contrôle 5.3 (Segregation of Duties) | https://www.isms.online/iso-27001/annex-a-2022/5-3-segregation-of-duties-2022/ | Annexe A, thème "Organisationnel", contrôle 5.3 (remplace 6.1.2 en version 2013) | Mesure #14 ; écart #14 |
| ISO 27001:2022, Annexe A, contrôles 8.15 et 8.16 | https://www.isms.online/iso-27001/annex-a-2022/8-15-logging-2022/ | Annexe A, thème "Technologique", contrôles 8.15 (Logging) et 8.16 (Monitoring activities), nouveaux en 2022 | Mesure #10 ; écart #10 |

**Note sur l'accès aux sources** : le texte intégral officiel de la norme ISO 27001:2022 (y compris son Annexe A) est payant et publié par l'ISO ; les liens ci-dessus renvoient vers des explications de référence tierces (ISMS.online, Advisera) largement utilisées dans le secteur en l'absence d'un accès gratuit au texte normatif lui-même. Les sources ANSSI, RGPD et OWASP sont en accès libre aux liens indiqués.

---

*Document de travail, construit à partir des observations disponibles à ce stade. Sections marquées "à compléter" nécessitent une vérification ou une décision supplémentaire avant intégration au rapport final.*
