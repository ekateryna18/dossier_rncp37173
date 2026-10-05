# J2 — Rapport d'audit des écarts, plan d'action arbitré, préparation à l'accréditation
### VNWeb, RNCP37173, M1.4

*Version consolidée, mise à jour avec les preuves AWS collectées le 25/09 (Amplify, CodeCommit, Cognito, DynamoDB, IAM) et la vérification d'exposition réseau des domaines publics. Chaque écart qualifié suit le même format à quatre champs pour rester lisible et comparable. Les points non encore vérifiés ou non concluants sont listés séparément plutôt que passés sous silence, conformément au principe du cours. Numérotation des écarts revue le 25/09 pour suivre l'ordre du document (auparavant, certains écarts portaient un numéro attribué au moment de leur découverte plutôt que leur position dans le rapport ; toutes les références croisées — plan d'action, arbitrage, dossier de preuves, sources — ont été mises à jour en conséquence).*

---

## Comment lire ce document

Chaque écart qualifié suit ce format fixe :

> **Volet** — de quelle partie du périmètre il s'agit
> **Procédure attendue** — ce que la bonne pratique / le référentiel exige
> **Observation** — ce qui a été constaté
> **Preuve d'audit** — comment c'est prouvé, et quand
> **Écart qualifié** — la conséquence / le risque que cela crée

Trois autres statuts existent dans ce document :
- **Non qualifiable en l'état** — la preuve est insuffisante ou ambiguë ; ne pas conclure tant que la vérification complémentaire indiquée n'est pas faite.
- **Conforme (aucun écart)** — vérifié et jugé conforme ; noté explicitement car "déclarer ce qui est couvert" fait aussi partie de la méthode.
- **Non testé / non applicable** — prévu dans le plan d'audit (J1) mais pas encore abordé, ou sans objet compte tenu de la configuration constatée.

---

## 1. Rapport d'audit des écarts (C3)

### A. Serveur ScalarX (héberge prod et preprod, hostname système : vnw2)

#### Écart 1 — Connexion administrateur root directe autorisée
- **Volet** : Serveur
- **Procédure attendue** : connexion en tant qu'administrateur via compte nominatif + `sudo`, jamais de connexion root directe (ANSSI, guide d'hygiène informatique)
- **Observation** : `sshd_config` retourne `PermitRootLogin yes`
- **Preuve d'audit** : commande exécutée par l'auditrice le 17/09/26 15:36 sur ScalarX
- **Écart qualifié** : connexion SSH root directe possible, aucune traçabilité individuelle sur les actions d'administration, cible privilégiée des attaques automatisées

> Non qualifiable en l'état, lié à cet écart : la valeur effective de `PasswordAuthentication` et `PubkeyAuthentication` n'a pas été retournée par le `grep` du 17/09. Vérification à faire : `sshd -T | grep -i authentication`.
> Non qualifiable en l'état, lié à cet écart : le niveau de mise à jour du système (`sudo apt update` non exécutable faute de droits).

#### Écart 2 — Absence de sauvegarde planifiée détectée (niveau serveur)
- **Volet** : Serveur
- **Procédure attendue** : sauvegardes régulières et documentées des données critiques (ANSSI, guide d'hygiène informatique)
- **Observation** : `ls -la /etc/cron.d/` ne montre aucune tâche de sauvegarde sur ScalarX, uniquement de la maintenance système
- **Preuve d'audit** : commande exécutée le 17/09/26 16:13 sur ScalarX
- **Écart qualifié** : aucune sauvegarde visible via cron système. Voir aussi écart 24 (absence de sauvegarde au niveau des données AWS, un problème distinct mais de même nature)

#### Écart 3 — Fenêtre de rétention des logs limitée
- **Volet** : Serveur / SI
- **Procédure attendue** : journalisation permettant une investigation sur une durée raisonnable (ANSSI)
- **Observation** : rotation hebdomadaire avec 4 copies conservées, soit environ 4 semaines d'historique, sans centralisation externe
- **Preuve d'audit** : commandes exécutées le 17/09/26 16:16 sur ScalarX
- **Écart qualifié** : au-delà de 4 semaines, aucune investigation rétroactive n'est possible

#### Écart 4 — Absence de journalisation applicative centralisée
- **Volet** : Serveur / SI (transverse)
- **Procédure attendue** : journalisation applicative centralisée avec supervision et alerting, au-delà des logs système locaux (ISO 27001:2022, Annexe A, contrôles 8.15/8.16)
- **Observation** : aucun système de centralisation des logs applicatifs ni de supervision n'est en place ; seule la rotation locale des logs système de ScalarX existe (écart 3, fenêtre de 4 semaines, sans agrégation ni alerte)
- **Preuve d'audit** : témoignage équipe, 25/09 ; recoupé avec l'absence de centralisation constatée à l'écart 3
- **Écart qualifié** : aucune détection d'incident en temps réel n'est possible ; ce point aggrave directement la vraisemblance du chemin d'attaque retenu à l'atelier 4 du plan d'audit (J1) — une intrusion réussie ne déclencherait aucune alerte

> Conforme (aucun écart), constaté au passage : `fail2ban` actif et sans interruption depuis plus de 3 mois sur ScalarX. Contredit l'hypothèse initiale posée en J1 par témoignage.

---

### B. Serveur de staging

#### Écart 5 — Authentification SSH faible sur le staging
- **Volet** : Serveur
- **Procédure attendue** : authentification forte (clé SSH) sur tout accès administrateur distant (ANSSI/CNIL)
- **Observation** : connexion SSH au serveur de staging possible en mot de passe uniquement, pas de clé
- **Preuve d'audit** : constat direct de l'auditrice lors d'une connexion au serveur
- **Écart qualifié** : authentification faible, fragilise l'ensemble de la chaîne staging → preprod → prod

> Non qualifiable en l'état, lié à cet écart : statut fail2ban et pare-feu sur le staging, non vérifiés directement.

---

### C. Organisationnel — accès et comptes

#### Écart 6 — Comptes systèmes partagés par projet, pas de compte nominatif individuel
- **Volet** : Organisationnel
- **Procédure attendue** : comptes d'accès nominatifs, individuels par collaborateur (ANSSI ; ISO 27001:2022 Annexe A)
- **Observation** : environ une centaine de comptes système sur ScalarX, chacun correspondant à un projet et non à une personne
- **Preuve d'audit** : `awk` sur `/etc/passwd`, commande du 17/09/26 15:50
- **Écart qualifié** : architecture par conception sans compte individuel

#### Écart 7 — Traçabilité individuelle des connexions impossible
- **Volet** : Organisationnel
- **Procédure attendue** : traçabilité individuelle des connexions et actions (ANSSI)
- **Observation** : `last -a` montre plusieurs comptes-projets connectés depuis la même IP, parfois à quelques minutes d'intervalle
- **Preuve d'audit** : commande du 17/09/26 16:09
- **Écart qualifié** : découle directement de l'écart 6 — impossible d'identifier quelle personne physique a agi

#### Écart 8 — Pas de procédure de transfert de connaissances au départ d'un collaborateur
- **Volet** : Organisationnel — **Procédure attendue** : ISO 27001:2022, Annexe A 6.5 — **Observation** : aucune documentation transmise en cas de départ — **Preuve d'audit** : témoignage équipe — **Écart qualifié** : perte de connaissance et de continuité

#### Écart 9 — Pas de suppléant sur le rôle admin
- **Volet** : Organisationnel — **Procédure attendue** : ISO 27001:2022, Annexe A 5.3 — **Observation** : aucun suppléant identifié — **Preuve d'audit** : témoignage équipe — **Écart qualifié** : point unique de défaillance

#### Écart 10 — Cadence de communication insuffisante
- **Volet** : Organisationnel — **Procédure attendue** : ISO 27001:2022, clause 7.4 — **Observation** : pas de réunion récurrente — **Preuve d'audit** : témoignage équipe — **Écart qualifié** : risque de rupture de circulation d'information

---

### D. AWS — Amplify Gen 1

#### Écart 11 — Fin de support Amplify Gen 1 prochaine
- **Volet** : AWS / Hébergement
- **Procédure attendue** : plateforme d'hébergement maintenue et supportée (ANSSI, maintien en condition de sécurité)
- **Observation** : application weevus sur Amplify Gen 1 ; maintenance dès mai 2026, fin de vie le 1er mai 2027 ; migration Gen 2 non finalisée
- **Preuve d'audit** : capture console Amplify, 25/09
- **Écart qualifié** : dépendance sur une techno en fin de vie proche

> Conforme (aucun écart), constaté au passage : HTTPS/certificat géré automatiquement par Amplify ; historique de déploiement présent et traçable.

---

### E. AWS — Accès et permissions (IAM + CodeCommit)

#### Écart 12 — Absence de séparation des privilèges (IAM et dépôt de code)
- **Volet** : AWS / IAM
- **Procédure attendue** : principe du moindre privilège (ANSSI ; AWS Well-Architected)
- **Observation** : sur 8 utilisateurs IAM au total, 6 appartiennent au groupe "Groupe 1" (permissions administrateur) et disposent de fait d'un accès total au dépôt CodeCommit (permissions héritées des groupes IAM)
- **Preuve d'audit** : capture IAM Users/Groups (25/09) ; capture de la policy attachée à "Groupe 1"
- **Écart qualifié** : un compte développeur compromis équivaut à une compromission administrateur complète, code source inclus

> Non qualifiable en l'état, lié à cet écart : les 2 comptes restants appartiennent à un "Groupe 0" (dont un compte client) dont les permissions ne sont pas documentées. À vérifier : IAM > User groups > Groupe 0 > Permissions.

#### Écart 13 — Aucun MFA activé sur les comptes IAM
- **Volet** : AWS / IAM
- **Procédure attendue** : MFA activé sur tous les comptes IAM, en particulier les comptes à privilèges (ANSSI/CNIL)
- **Observation** : confirmé sur les 8 comptes IAM du compte, aucun n'a le MFA activé
- **Preuve d'audit** : IAM Users, colonne MFA, 25/09
- **Écart qualifié** : absence totale de MFA sur des comptes majoritairement administrateurs (écart 12) — un mot de passe seul suffit à compromettre le compte AWS entier

#### Écart 14 — Absence de protection de branche, notification informelle uniquement
- **Volet** : AWS / CodeCommit — Pipelines
- **Procédure attendue** : règles de protection de branche, notifications formalisées (OWASP DevSecOps Guideline)
- **Observation** : aucune règle de protection de branche configurée ; seul mécanisme de notification, un webhook Discord informel
- **Preuve d'audit** : capture page paramètres du dépôt (25/09) ; capture du webhook Discord
- **Écart qualifié** : push direct possible sur toute branche, y compris de production

> **Note aggravante** (déploiement automatisé, cf. section 1.D) : un push sur la branche `dev` déclenche automatiquement un build Amplify et rend l'application disponible en preprod, sans étape manuelle interposée. Combinée à l'absence de protection de branche (cet écart) et à l'absence de revue de code (écart 17), l'automatisation supprime le seul moment où une erreur ou un code malveillant pourrait être intercepté avant d'atteindre un environnement accessible — n'importe quel code poussé sur `dev` y arrive directement.

> **Portée confirmée à l'échelle de l'entreprise** : l'équipe confirme que l'absence de protection de branche constatée sur CodeCommit/weevus est représentative de la pratique généralisée sur l'ensemble des dépôts Gitea — "l'équipe a pris l'habitude de travailler comme ça". Ce n'est donc pas une exception propre à un projet, mais une pratique établie à l'échelle de l'entreprise, ce qui élargit la portée réelle de cet écart bien au-delà de weevus.

#### Écart 15 — Politique de mot de passe IAM faible sur des comptes à privilèges
- **Volet** : AWS / IAM
- **Procédure attendue** : politique de mot de passe renforcée sur les comptes à privilèges, au-delà du minimum AWS par défaut (ANSSI)
- **Observation** : la politique de mot de passe du compte AWS est celle par défaut — 8 caractères minimum, 3 types de caractères sur 4 exigés, aucune expiration forcée, aucune règle explicite de non-réutilisation visible
- **Preuve d'audit** : IAM > Account settings > Password policy, 25/09
- **Écart qualifié** : combinée à l'absence de MFA (écart 13), cette politique minimale laisse des comptes majoritairement administrateurs protégés par un seul facteur, lui-même faiblement contraint

#### Écart 16 — Rôles IAM inactifs non nettoyés
- **Volet** : AWS / IAM
- **Procédure attendue** : revue périodique et suppression des accès inutilisés (ANSSI, principe de réduction de la surface d'attaque)
- **Observation** : présence de rôles IAM dont la dernière activité remonte à plus de 1400 jours (environ 4 ans)
- **Preuve d'audit** : IAM > Roles, tri par dernière activité, 25/09
- **Écart qualifié** : des accès potentiellement obsolètes restent actifs et exploitables sans qu'aucune personne n'en ait probablement plus l'usage ni le suivi

#### Écart incohérence de permissions Lambda (rattaché à l'écart 12)
- **Observation** : les permissions accordées aux fonctions Lambda ne sont pas uniformément excessives : la fonction `CustomMessage` (déclencheur Cognito) est correctement restreinte à ses propres logs (`logs:CreateLogGroup/CreateLogStream/PutLogEvents`, ressource scopée à son propre log group), tandis que la fonction `PostAuthentication` dispose d'un accès `appsync:GraphQL` sur `types/Query/*` et `types/Mutation/*` — soit l'intégralité des opérations de l'API GraphQL, non justifiée par son rôle (vérifier ou modifier un utilisateur après connexion)
- **Preuve d'audit** : policies JSON des deux fonctions, IAM, 25/09
- **Écart qualifié** : l'octroi des permissions est incohérent d'une fonction à l'autre plutôt que systématiquement excessif — signe d'une configuration ponctuelle insuffisamment revue plutôt que d'un choix délibéré ; une fonction liée à l'authentification peut, si compromise, lire ou modifier n'importe quelle donnée exposée par l'API

---

### E-bis. Pipelines — qualité de code et intégration continue

*Ces trois écarts découlent directement du même constat que l'écart 14 (absence de garde-fou avant mise en production) ; ils sont traités séparément car chacun porte sa propre mesure corrective dans le plan d'action.*

#### Écart 17 — Absence de revue de code obligatoire avant fusion
- **Volet** : Pipelines
- **Procédure attendue** : revue de code systématique avant toute fusion sur une branche de production (OWASP Code Review Guide)
- **Observation** : aucune revue de code obligatoire n'est en place avant fusion sur les dépôts CodeCommit/Gitea ; la pratique est laissée à l'appréciation ponctuelle de chacun
- **Preuve d'audit** : témoignage équipe, 25/09 ; cohérent avec l'absence de protection de branche constatée à l'écart 14
- **Écart qualifié** : du code non revu peut atteindre la production sans aucun contrôle humain, ce qui aggrave directement l'écart 14 (déploiement automatisé sans garde-fou)

#### Écart 18 — Absence de couverture de tests minimale
- **Volet** : Pipelines
- **Procédure attendue** : suite de tests automatisée exécutée a minima avant mise en production (OWASP DevSecOps Guideline)
- **Observation** : aucune suite de tests automatisée identifiée sur les projets audités
- **Preuve d'audit** : témoignage équipe, 25/09
- **Écart qualifié** : absence de filet de sécurité automatisé avant déploiement, une régression fonctionnelle ou de sécurité peut atteindre la production sans détection

#### Écart 19 — Quasi-absence de pipeline CI/CD
- **Volet** : Pipelines
- **Procédure attendue** : pipeline CI/CD assurant a minima build, lint et tests avant déploiement (OWASP DevSecOps Guideline)
- **Observation** : quasi-absence de pipeline CI/CD sur l'ensemble des projets de l'entreprise ; seul le build Amplify déclenché par push existe (cf. note aggravante, écart 14), sans étape de contrôle intermédiaire
- **Preuve d'audit** : témoignage équipe, 25/09
- **Écart qualifié** : sans pipeline, aucune étape automatisée (scan de secrets, analyse de dépendances, tests) ne peut être insérée avant le déploiement — ce constat conditionne aussi les écarts 21 et 22 ci-dessous

---

### F. Gestion des secrets et des permissions applicatives (transverse)

#### Écart 20 — Absence de gestion centralisée des secrets
- **Volet** : Transverse (AWS + organisationnel)
- **Procédure attendue** : secrets jamais stockés en clair hors d'un système dédié (ANSSI, guide d'hygiène informatique)
- **Observation** : identifiants partagés en clair sur Discord (témoignage) ; variables d'environnement Amplify en clair, identiques sur toutes les branches, visibles par tous les contributeurs
- **Preuve d'audit** : témoignage équipe ; capture console Amplify > Environment variables, 25/09
- **Écart qualifié** : absence de tout système de gestion des secrets, secrets visibles par des comptes qui n'en ont pas nécessairement besoin (cf. écart 12)

#### Écart 21 — Absence de contrôle systématique de détection de secrets dans le code
- **Volet** : Pipelines (échelle entreprise)
- **Procédure attendue** : contrôle systématique, même basique (scan automatisé type git-secrets/trufflehog, ou a minima une checklist de revue formalisée), pour détecter des secrets commités par erreur avant qu'ils n'atteignent un dépôt (OWASP DevSecOps Guideline)
- **Observation** : aucun contrôle automatisé n'existe sur les dépôts de l'entreprise ; la seule pratique est une revue manuelle ponctuelle "si nécessaire", non systématique et non documentée
- **Preuve d'audit** : témoignage équipe, 25/09
- **Écart qualifié** : la détection d'un secret commité par erreur dépend entièrement du jugement humain ponctuel plutôt que d'un contrôle systématique ; rien ne garantit qu'un secret déjà présent dans l'historique d'un dépôt ait été repéré

#### Écart 22 — Absence d'analyse des vulnérabilités des dépendances
- **Volet** : Pipelines (échelle entreprise)
- **Procédure attendue** : analyse des dépendances logicielles pour vulnérabilités connues (CVE), même manuelle ou ponctuelle (OWASP DevSecOps Guideline)
- **Observation** : la quasi-absence de pipeline CI/CD sur les projets de l'entreprise (écart 19) empêche même une analyse automatisée des dépendances ; aucune analyse manuelle documentée n'existe non plus
- **Preuve d'audit** : témoignage équipe, 25/09
- **Écart qualifié** : des dépendances logicielles contenant des vulnérabilités connues peuvent être utilisées en production sans qu'aucune détection n'ait lieu, sur l'ensemble des projets de l'entreprise

---

### G. Conformité / localisation des données

#### Écart 23 — Répartition géographique incohérente des ressources AWS
- **Volet** : Conformité (RGPD)
- **Procédure attendue** : localisation des données dans l'UE pour les données à caractère personnel, ou base légale de transfert documentée (RGPD Art. 44-49)
- **Observation** : plusieurs éléments confirment qu'une partie significative de la pile applicative est hébergée hors UE, en Ohio (us-east-2), pendant que d'autres ressources restent en région Paris (eu-west-3) : (1) la table DynamoDB `users` est en Ohio ; (2) les fonctions Lambda déclencheurs Cognito (`PostAuthentication`, `CustomMessage`) et l'API AppSync associée sont également en Ohio ; (3) au moins une autre table (`genealogy`, données de relations familiales) a été vérifiée en Ohio. Ce n'est donc pas un cas isolé d'une seule table mal placée, mais une partition géographique cohérente entre deux régions pour des raisons qui restent à clarifier avec l'équipe technique
- **Preuve d'audit** : ARNs AppSync/Lambda (région us-east-2), console DynamoDB (région des tables), 17/09 et 25/09
- **Écart qualifié** : transfert de données à caractère personnel hors UE (dont des données de relations familiales) sans garantie identifiée à ce stade

#### Écart 24 — Absence de sauvegarde continue (PITR) sur les tables DynamoDB sensibles
- **Volet** : Conformité / Serveur (données AWS)
- **Procédure attendue** : sauvegardes régulières des données critiques, notamment celles à caractère personnel (ANSSI ; RGPD Art. 32, intégrité et disponibilité)
- **Observation** : le Point-in-Time Recovery (PITR) est désactivé sur la table `users` et sur la table `genealogy` (relations familiales), 0 sauvegarde existante sur ces deux tables
- **Preuve d'audit** : console DynamoDB, onglet Backups, 25/09
- **Écart qualifié** : en cas de suppression ou corruption accidentelle des données utilisateurs ou de généalogie familiale, aucune restauration n'est possible — risque de perte de données irréversible sur les données les plus sensibles de l'application

> Point d'attention, sans écart formel séparé : le chiffrement au repos est bien actif sur les tables vérifiées (`users`, `genealogy`), mais via une clé gérée par AWS ("AWS owned key") plutôt qu'une clé gérée par le client (KMS CMK). Cela reste conforme aux exigences de base, mais laisse VNWeb sans contrôle propre sur la rotation ou la révocation de cette clé pour des données aussi sensibles que les relations familiales.

---

### H. AWS — Cognito (côté utilisateurs finaux de weevus)

#### Écart 25 — MFA optionnel (non obligatoire) pour les utilisateurs finaux
- **Volet** : AWS / Cognito
- **Procédure attendue** : authentification renforcée recommandée pour une application traitant des données personnelles sensibles (photos, généalogie familiale) (ANSSI/CNIL)
- **Observation** : le MFA est configuré en mode "Optional" dans le pool d'utilisateurs Cognito, donc non imposé aux utilisateurs finaux de weevus
- **Preuve d'audit** : Cognito > Sign-in experience, 25/09 9:54
- **Écart qualifié** : les comptes utilisateurs finaux (familles) restent protégés par un seul facteur par défaut, alors que l'application héberge des données familiales sensibles

> Non testé / non applicable, constaté au passage : aucun fournisseur d'identité externe (Google, Facebook, etc.) n'est connecté au pool Cognito — pas de risque lié à un IdP tiers à ce stade. Aucun domaine Cognito (ni par défaut ni personnalisé) n'est configuré, ce qui suggère que l'authentification passe directement par le SDK Amplify plutôt que par la Hosted UI — le point du plan d'audit sur "domaine exposé publiquement" est donc sans objet ici.
> Non qualifiable en l'état : le point "verrouillage de compte après tentatives échouées" n'a pas pu être confirmé directement — la capture obtenue montre le paramètre de récupération de compte (self-service recovery par email/SMS), pas la politique de verrouillage adaptatif. À vérifier via Cognito > Advanced security / App client > si les fonctionnalités de protection contre les menaces (threat protection) sont activées.

---

### I. AWS — S3 (stockage photos)

*Bucket vérifié : `weevusproda04d877fe4864b39a4201d8fdb869c86fb3df-dev` (environnement DEV). Le bucket équivalent en environnement PROD n'a pas été vérifié séparément — voir limite en section 2.*

#### Écart 26 — Versioning désactivé sur le bucket de stockage des photos
- **Volet** : AWS / S3
- **Procédure attendue** : versioning activé sur les buckets contenant des données critiques ou irremplaçables, en protection contre une suppression ou un écrasement accidentel (ANSSI ; même logique que le PITR sur DynamoDB, écart 24)
- **Observation** : le "Bucket Versioning" est à l'état "Disabled" sur le bucket de stockage des photos weevus (environnement DEV)
- **Preuve d'audit** : S3 > bucket > onglet Properties > Bucket Versioning, 25/09
- **Écart qualifié** : en cas de suppression ou d'écrasement accidentel (ou malveillant) d'une photo, aucune version antérieure n'est récupérable — perte de données irréversible sur le contenu le plus personnel de l'application (photos de famille), cohérent avec le même type de risque déjà qualifié à l'écart 24 pour les données DynamoDB

> Conforme (aucun écart), constaté au passage : **Block public access** est activé en totalité sur le bucket ("Block *all* public access: On") ; aucune bucket policy n'accorde d'accès public ou trop large (vérification du 25/09) ; le chiffrement au repos est actif par défaut (SSE-S3, clés gérées par Amazon S3) — même réserve que pour DynamoDB (écart 24) sur l'usage d'une clé gérée par AWS plutôt qu'une clé client (KMS CMK), sans qu'il s'agisse d'un écart formel.
> Élément complémentaire, rattaché à l'écart 4 (absence de journalisation applicative centralisée) plutôt qu'à un écart séparé : le "Server access logging" est également désactivé sur ce bucket ("Disabled", 25/09) — aucun journal des accès (lecture/écriture/suppression d'objets) n'est produit, ce qui prive d'une source de preuve en cas d'incident sur les photos stockées.

---

### J. Réseau — exposition des domaines publics

#### Écart 27 — En-têtes de sécurité HTTP incomplets sur extranet.vnweb.fr (HSTS et CSP absents)
- **Volet** : Réseau / Application web
- **Procédure attendue** : sur un service web public exposé à l'ensemble d'Internet, en-tête `Strict-Transport-Security` (HSTS) pour empêcher tout downgrade vers HTTP après la redirection initiale, et en-tête `Content-Security-Policy` (CSP) comme défense principale contre l'injection/XSS (OWASP Secure Headers Project ; ANSSI)
- **Observation** : `extranet.vnweb.fr` résout et répond en HTTP 200/302 depuis l'ensemble des zones géographiques testées (Europe, Amériques, Asie, Océanie — aucune restriction géographique ou réseau constatée), avec redirection correcte `http://` → `https://` (`Location: https://extranet.vnweb.fr/`). Les en-têtes `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `X-Permitted-Cross-Domain-Policies` et `Permissions-Policy` sont bien présents. En revanche, aucun en-tête `Strict-Transport-Security` ni `Content-Security-Policy` n'est retourné. Les en-têtes `X-XSS-Protection` et `Expect-CT`, tous deux obsolètes et sans effet sur les navigateurs modernes, sont présents à leur place — signe d'une configuration mise en place à un moment donné puis non revue depuis
- **Preuve d'audit** : vérification multi-régions via check-host.net (DNS + HTTP, 14+ localisations dont Australie, Brésil, Inde, Hong Kong, Canada) le 25/09/26 ; `curl -I http://extranet.vnweb.fr` exécuté par l'auditrice le 25/09/26 15:39, en-têtes de réponse complets
- **Écart qualifié** : (1) sans HSTS, une première connexion (ou toute connexion après purge du cache navigateur) reste vulnérable à une attaque de type SSL-stripping avant que la redirection HTTPS n'intervienne ; (2) sans CSP, l'application ne dispose d'aucune défense en profondeur contre l'injection de scripts, alors que le service est confirmé accessible sans restriction depuis l'intégralité d'Internet

> Conforme (aucun écart), constaté au passage : `gitea.vnweb.fr` ne résout dans aucune zone testée en dehors du réseau interne VNWeb — cohérent avec du DNS split-horizon et le fait que l'outil de gestion de code source ne doit pas être exposé publiquement. Contraste utile avec l'écart 27 : VNWeb applique donc une restriction réseau là où c'est pertinent (Gitea), ce qui suggère que l'ouverture totale de l'extranet est un choix assumé (accès partenaires/clients) plutôt qu'un oubli — mais qui appelle alors un durcissement des en-têtes en conséquence.

---

## 2. Éléments du plan d'audit non encore testés ou non qualifiables

**Non qualifiables (preuve insuffisante, à compléter avant conclusion)** :
- Valeur effective de `PasswordAuthentication`/`PubkeyAuthentication` sur ScalarX (écart 1)
- Niveau de mise à jour réel du système ScalarX (écart 1)
- Statut fail2ban et pare-feu sur le serveur de staging (écart 5)
- Permissions des 2 comptes du "Groupe 0" IAM, dont un compte client (écart 12)
- Politique de verrouillage de compte après tentatives échouées, Cognito (écart 25)
- Pare-feu hôte sur ScalarX : `iptables` est installé (`/sbin/iptables`, `/usr/sbin/iptables`, liens vers `/etc/alternatives/iptables`, v1.8.7, backend `nf_tables`) ; `nft`, `ufw` et `firewalld` sont absents du système. `iptables -L` a été tenté directement (`/sbin/iptables -L`) et retourne `Could not fetch rule set generation id: Permission denied (you must be root)`, confirmant que le compte auditeur ne dispose pas des droits nécessaires pour lire les règles. Ni la présence ni l'absence d'un filtrage effectif ne peuvent être confirmées sans accès root ou confirmation d'un administrateur

**Non testés (prévus en J1, pas encore abordés)** :
- S3 — bucket PROD non vérifié séparément (seul le bucket DEV a été audité, écart 26 et conformes associés) ; à confirmer que la configuration est identique entre environnements
- Chiffrement au repos et PITR sur les tables autres que `users` et `genealogy` (échantillon de 2 sur 47 tables, jugé suffisant pour ce niveau d'audit entreprise)
- Présence effective de secrets déjà commités dans l'historique Gitea/CodeCommit (l'absence de tout contrôle de détection est désormais qualifiée en écart 21 ; reste la question factuelle de savoir si des secrets y figurent déjà, non vérifiable sans un scan de l'historique)
- Volet Réseau restant (VPN, segmentation interne, cartographie complète — le pare-feu hôte de ScalarX est traité ci-dessus ; l'exposition publique d'extranet.vnweb.fr et de gitea.vnweb.fr est désormais couverte, écart 27 et conforme associé)
- Sensibilisation réelle de l'équipe aux pratiques de base

**Conformes constatés au passage** (à conserver pour le dossier d'accréditation) :
- Root user : MFA activé
- Votre compte (Ecaterina) : pas de clé d'accès inactive depuis plus d'un an
- fail2ban actif sur ScalarX
- HTTPS/certificat et historique de déploiement Amplify
- Fonction `CustomMessage` : permissions correctement restreintes à ses propres logs
- Chiffrement au repos actif sur `users` et `genealogy` (avec la réserve sur le type de clé, cf. écart 24)
- Processus de déploiement weevus automatisé : un push sur `dev` déclenche un build Amplify et rend l'application disponible en preprod (répond au point "déploiement automatisé et auditable" du plan d'audit — voir toutefois la note aggravante rattachée à l'écart 14, ci-dessus, concernant l'absence de garde-fou avant ce déploiement)
- `gitea.vnweb.fr` non exposé publiquement (cf. écart 27)
- S3 (bucket DEV) : Block public access intégral, bucket policy sans accès public, chiffrement au repos actif (cf. écart 26)

---

## 3. Plan d'action (C4)

| Écart # | Mesure | Type | Justification | Responsable | Échéance | Preuve attendue |
|---|---|---|---|---|---|---|
| 1 | Désactiver `PermitRootLogin`, imposer un accès par compte nominatif + `sudo` | Corrective | ANSSI | À définir | À définir | `PermitRootLogin no` confirmé |
| 2 | Mettre en place une sauvegarde planifiée et vérifiée pour ScalarX | Corrective | ANSSI | À définir | À définir | Tâche de sauvegarde visible et testée |
| 3 | Étendre la fenêtre de rétention des logs ou centraliser | Préventive | ANSSI | À définir | À définir | Configuration modifiée |
| 4 | Mettre en place une journalisation applicative centralisée | Corrective + Préventive | ANSSI ; ISO 27001:2022 Annexe A 8.15/8.16 | À définir | À définir | Logs centralisés, alerte testée |
| 5 | Migrer le staging vers une authentification par clé SSH | Corrective | ANSSI/CNIL | À définir | À définir | `sshd_config` mis à jour |
| 6 | Étudier la faisabilité de comptes individuels (couvre aussi l'écart 7) | Préventive | ANSSI ; ISO 27001:2022 Annexe A | À définir | À définir | Solution documentée |
| 8 | Rédiger un document de transfert de connaissance par rôle/projet | Préventive | ISO 27001:2022, Annexe A 6.5 | À définir | À définir | Document existant et à jour |
| 9 | Désigner un suppléant sur le rôle admin | Corrective | ISO 27001:2022, Annexe A 5.3 | Responsable | À définir | Suppléant identifié et testé |
| 10 | Instaurer un point de synchronisation régulier | Préventive | ISO 27001:2022, clause 7.4 | À définir | À définir | Récurrence sur plusieurs semaines |
| 11 | Planifier la migration Amplify Gen 1 vers Gen 2 en sandbox | Préventive | ANSSI | À définir | Avant mai 2027 | Environnement sandbox validé |
| 12 | Revoir les groupes IAM pour le moindre privilège (impact CodeCommit inclus) | Corrective | ANSSI ; AWS Well-Architected | À définir | À définir | Nouvelle policy IAM testée |
| 13 | Activer le MFA sur tous les comptes IAM | Corrective | ANSSI/CNIL | À définir | À définir | MFA actif confirmé pour les 8 comptes |
| 14 | Configurer une règle de protection de branche | Corrective | OWASP DevSecOps Guideline | À définir | À définir | Règle active, testée |
| 15 | Renforcer la politique de mot de passe IAM (longueur minimale ≥ 12, réutilisation interdite) sur les comptes à privilèges | Corrective | ANSSI | À définir | À définir | Politique modifiée dans IAM > Account settings |
| 16 | Identifier et supprimer les rôles IAM inactifs depuis plus d'un an, après vérification qu'ils ne sont plus référencés | Corrective | ANSSI, réduction de la surface d'attaque | À définir | À définir | Liste des rôles supprimés, aucun impact constaté |
| 17 | Introduire une revue de code obligatoire avant fusion | Corrective + Préventive | OWASP Code Review Guide | À définir | À définir | Trace de revue sur les fusions |
| 18 | Introduire une couverture de tests minimale | Préventive | OWASP DevSecOps Guideline | À définir | À définir | Suite de tests exécutable |
| 19 | Mettre en place un pipeline CI/CD minimal | Préventive | OWASP DevSecOps Guideline | À définir | À définir | Pipeline fonctionnel |
| 20 | Mettre en place un gestionnaire de secrets, purger Discord et les variables Amplify en clair | Corrective + Préventive | ANSSI | À définir | À définir | Outil déployé, secrets migrés |
| 21 | Mettre en place un contrôle systématique de détection de secrets (scan automatisé type git-secrets/trufflehog, ou a minima une checklist de revue formalisée avant push) | Préventive | OWASP DevSecOps Guideline | À définir | À définir | Contrôle actif et documenté, testé sur un cas volontaire |
| 22 | Mettre en place une analyse, même ponctuelle/manuelle au départ, des dépendances logicielles pour vulnérabilités connues (CVE) sur les projets prioritaires | Préventive | OWASP DevSecOps Guideline | À définir | À définir | Premier rapport d'analyse produit sur au moins un projet |
| 23 | Clarifier et documenter la répartition géographique des ressources AWS (Cognito, AppSync, DynamoDB) ; migrer ou documenter une base légale pour ce qui reste hors UE | Corrective | RGPD Art. 44-49 | À définir | À définir | Cartographie des régions à jour, ressources migrées ou base légale documentée |
| 24 | Activer le Point-in-Time Recovery sur les tables `users` et `genealogy`, puis étendre aux autres tables sensibles | Corrective | ANSSI ; RGPD Art. 32 | À définir | À définir | PITR actif, test de restauration réalisé |
| 25 | Rendre le MFA obligatoire pour les utilisateurs finaux Cognito | Corrective | ANSSI/CNIL | À définir | À définir | MFA obligatoire confirmé sur le pool |
| 26 | Activer le Versioning sur le bucket S3 de stockage des photos (DEV et PROD) | Corrective | ANSSI | À définir | À définir | Bucket Versioning "Enabled" confirmé sur les deux environnements |
| 27 | Ajouter les en-têtes `Strict-Transport-Security` et `Content-Security-Policy` sur extranet.vnweb.fr, retirer les en-têtes obsolètes (`X-XSS-Protection`, `Expect-CT`) | Corrective | OWASP Secure Headers Project ; ANSSI | À définir | À définir | En-têtes confirmés via `curl -I https://extranet.vnweb.fr` |

*L'écart 7 ne porte pas de mesure propre : il découle de la mesure 6. L'incohérence de permissions Lambda (rattachée à l'écart 12) est couverte par la mesure 12. Les points non qualifiables (section 2) n'ont volontairement pas de mesure associée tant qu'ils ne sont pas confirmés.*

### Détail des références citées (nouvelles entrées uniquement)

**Mesure 23** — RGPD, Chapitre V (Art. 44 et suivants), déjà cité ; portée élargie suite aux nouvelles preuves Cognito/AppSync.

**Mesure 25** — ANSSI/CNIL, *Recommandations relatives à l'authentification multifacteur et aux mots de passe* (2021), déjà cité pour la mesure 13 ; s'applique également à l'authentification des utilisateurs finaux d'une application traitant des données familiales sensibles.

**Mesure 24** — RGPD, Art. 32 (Sécurité du traitement) : le responsable du traitement doit mettre en œuvre des mesures garantissant la capacité à restaurer la disponibilité et l'accès aux données à caractère personnel en cas d'incident.

**Mesures 16, 15** — ANSSI, *Guide d'hygiène informatique* : réduction de la surface d'exposition (suppression des accès inutilisés) et robustesse des mécanismes d'authentification, déjà cités.

*Le reste des références est inchangé, voir section 7.*

---

## 4. Arbitrage

### Critères et pondération (inchangés)

| Critère | Poids |
|---|---|
| Gravité | 30 |
| Dépendances | 20 |
| Moyens | 15 |
| Ressources | 15 |
| Organisation | 10 |
| Contraintes réglementaires | 10 |
| **Total** | **100** |

Score pondéré = Σ (note × poids) ÷ 100. Notes de 1 à 5 par critère, proposées par défaut, à ajuster selon votre jugement.

### Classement des mesures (score pondéré, mis à jour)

| Rang | Mesure | Gravité (30) | Dépend. (20) | Moyens (15) | Ressources (15) | Organisation (10) | Réglem. (10) | Score |
|---|---|---|---|---|---|---|---|---|
| 1 | 1 — PermitRootLogin (ScalarX) | 5 | 3 | 5 | 5 | 4 | 1 | **4,10** |
| 1 | 13 — MFA comptes IAM | 5 | 2 | 5 | 5 | 5 | 2 | **4,10** |
| 3 | 5 — Authentification par clé SSH (staging) | 5 | 2 | 5 | 5 | 5 | 1 | **4,00** |
| 4 | 4 — Journalisation / supervision applicative | 4 | 5 | 3 | 4 | 5 | 2 | **3,95** |
| 5 | 12 — Moindre privilège IAM/CodeCommit | 5 | 4 | 3 | 3 | 3 | 2 | **3,70** |
| 5 | 20 — Gestion centralisée des secrets | 5 | 2 | 4 | 4 | 4 | 2 | **3,70** |
| 7 | 24 — PITR tables sensibles | 4 | 1 | 5 | 5 | 4 | 2 | **3,50** |
| 7 | 26 — Versioning S3 (photos) | 4 | 1 | 5 | 5 | 4 | 2 | **3,50** |
| 9 | 25 — MFA obligatoire utilisateurs finaux | 4 | 2 | 4 | 4 | 3 | 3 | **3,40** |
| 10 | 14 — Protection de branche | 3 | 2 | 5 | 5 | 4 | 1 | **3,30** |
| 10 | 15 — Politique de mot de passe IAM | 3 | 2 | 5 | 5 | 4 | 1 | **3,30** |
| 10 | 21 — Détection systématique de secrets | 4 | 2 | 4 | 4 | 3 | 2 | **3,30** |
| 13 | 23 — Répartition géographique / RGPD | 4 | 1 | 2 | 3 | 3 | 5 | **2,95** |
| 14 | 19 — Pipeline CI/CD minimal | 3 | 3 | 2 | 3 | 5 | 1 | **2,85** |
| 14 | 6 — Étude comptes individuels (+ écart 7) | 3 | 2 | 4 | 3 | 3 | 2 | **2,85** |
| 16 | 27 — En-têtes HSTS/CSP (extranet) | 3 | 1 | 5 | 5 | 2 | 1 | **2,80** |
| 17 | 2 — Sauvegarde planifiée (ScalarX) | 4 | 1 | 3 | 3 | 2 | 1 | **2,60** |
| 17 | 3 — Rétention des logs (ScalarX) | 3 | 2 | 3 | 3 | 2 | 2 | **2,60** |
| 17 | 22 — Analyse des dépendances (CVE) | 3 | 2 | 3 | 3 | 3 | 1 | **2,60** |
| 20 | 10 — Cadence de communication | 2 | 1 | 5 | 4 | 3 | 1 | **2,55** |
| 20 | 11 — Migration Amplify Gen 1 → Gen 2 | 3 | 2 | 2 | 3 | 4 | 1 | **2,55** |
| 22 | 18 — Couverture de tests | 3 | 3 | 2 | 2 | 3 | 1 | **2,50** |
| 23 | 17 — Revue de code obligatoire | 3 | 1 | 4 | 3 | 2 | 1 | **2,45** |
| 24 | 8 — Document de transfert de connaissance | 2 | 1 | 4 | 4 | 3 | 1 | **2,40** |
| 25 | 9 — Suppléant sur le rôle admin | 3 | 1 | 4 | 2 | 1 | 1 | **2,20** |
| 26 | 16 — Nettoyage des rôles IAM inactifs | 2 | 1 | 4 | 3 | 2 | 1 | **2,15** |

**Point de vigilance** : les mesures 24 (PITR DynamoDB) et 26 (Versioning S3) obtiennent toutes deux un score élevé (3,50) alors qu'elles ne coûtent presque rien à mettre en œuvre (une case à cocher dans leur console respective) — ce sont d'excellents candidats pour un traitement rapide malgré leur classement au 7ᵉ rang, exactement le type de mesure "quick win" à prioriser dans la pratique même sans attendre son tour strict dans le classement.

---

## 5. Dossier de preuves pour l'accréditation (C5) — ébauche

**Norme cible proposée** : ANSSI, guide d'hygiène informatique (référentiel de pratiques — à confirmer avec l'enseignant si une norme certifiante type ISO 27001 est attendue à la place)

| Exigence (exemple) | Statut | Preuve | Renvoi plan d'action |
|---|---|---|---|
| Authentification forte sur les accès distants | Non couverte | Écarts 1, 5 | Mesures 1, 5 |
| Authentification multifacteur sur les comptes à privilèges | Non couverte | Écart 13 | Mesure 13 |
| Authentification multifacteur sur les comptes utilisateurs finaux | Non couverte | Écart 25 | Mesure 25 |
| Moindre privilège / séparation des accès | Non couverte | Écart 12 | Mesure 12 |
| Gestion sécurisée des secrets | Non couverte | Écart 20 | Mesure 20 |
| Sauvegarde et restauration des données | Non couverte | Écarts 2, 24, 26 | Mesures 2, 24, 26 |
| Supervision et détection d'incident | Non couverte | Écart 4 | Mesure 4 |
| Continuité sur les rôles à privilèges | Non couverte | Écart 9 | Mesure 9 |
| Authentification du compte root | **Couverte** | MFA actif (conforme, section 2) | — |

*Ébauche minimale, à étoffer une fois la norme cible confirmée et les points non qualifiables tranchés.*

---

## 6. Synthèse (Résultat, Preuve, Limite, Suite)

- **Résultat** : 27 écarts qualifiés à ce stade (serveur, AWS IAM/CodeCommit/Cognito/DynamoDB/S3, organisationnel, pipelines, conformité, exposition réseau — dont deux confirmés comme représentatifs de la pratique à l'échelle de l'entreprise plutôt que propres à weevus), plusieurs points conformes constatés au passage (dont le compte root, la non-exposition publique de gitea.vnweb.fr, et la configuration d'accès du bucket S3), 4 points non qualifiables en l'état, plan d'action rédigé pour l'ensemble, arbitrage chiffré sur 26 mesures. Numérotation revue pour suivre l'ordre du document (voir note en tête de rapport).
- **Preuve** : constats directs de l'auditrice pour les éléments techniques vérifiables (serveur ScalarX, console AWS, vérification multi-régions de l'exposition DNS/HTTP), témoignages d'équipe pour les éléments organisationnels et de gouvernance des pipelines.
- **Limite** : S3 vérifié uniquement sur le bucket DEV, le bucket PROD reste à confirmer ; chiffrement/PITR vérifiés sur seulement 2 tables DynamoDB sur 47 (échantillon jugé suffisant à ce niveau d'audit) ; le serveur des sites de voyance reste hors périmètre faute d'accès ; responsables et échéances du plan d'action restent à assigner avec l'équipe.
- **Suite** : lever les points non qualifiables restants, compléter S3, clarifier avec l'équipe technique la raison de la répartition Paris/Ohio, valider le plan d'action et les échéances.

---

## 7. Sources et références citées

| Source | Lien | Utilisé pour |
|---|---|---|
| ANSSI, *Guide d'hygiène informatique* | https://messervices.cyber.gouv.fr/documents-guides/guide_hygiene_informatique_anssi.pdf | Mesures 1, 2, 3, 4, 6, 11, 12, 15, 16, 20, 26 |
| ANSSI/CNIL, *Recommandations authentification multifacteur et mots de passe* (2021) | https://messervices.cyber.gouv.fr/documents-guides/anssi-guide-authentification_multifacteur_et_mots_de_passe.pdf | Mesures 5, 13, 25 |
| RGPD, Chapitre V (Art. 44 et suivants) | https://www.gdpr-expert.eu/article.html?id=44 | Mesure 23 |
| RGPD, Art. 32 (Sécurité du traitement) | https://www.gdpr-expert.eu/article.html?id=32 | Mesure 24 |
| OWASP, *Code Review Guide* | https://owasp.org/www-project-code-review-guide/ | Mesure 17 |
| OWASP, *Secure Code Review Cheat Sheet* | https://cheatsheetseries.owasp.org/cheatsheets/Secure_Code_Review_Cheat_Sheet.html | Mesures 17, 18 |
| OWASP, *DevSecOps Guideline* | https://owasp.org/www-project-developer-guide/release/operations/devsecops_guideline/ | Mesures 14, 19, 21, 22 |
| OWASP, *Secure Headers Project* | https://owasp.org/www-project-secure-headers/ | Mesure 27 |
| ISO 27001:2022, clause 7.4 (Communication) | https://advisera.com/iso27001/clause-7-4-communication/ | Mesure 10 |
| ISO 27001:2022, Annexe A, contrôle 6.5 | https://www.isms.online/iso-27001/annex-a-2022/6-5-responsibilities-after-termination-change-of-employment-2022/ | Mesure 8 |
| ISO 27001:2022, Annexe A, contrôle 5.3 | https://www.isms.online/iso-27001/annex-a-2022/5-3-segregation-of-duties-2022/ | Mesure 9 |
| ISO 27001:2022, Annexe A, contrôles 8.15/8.16 | https://www.isms.online/iso-27001/annex-a-2022/8-15-logging-2022/ | Mesure 4 |

**Note sur l'accès aux sources** : le texte intégral officiel de la norme ISO 27001:2022 est payant ; les liens ci-dessus renvoient vers des explications de référence tierces. Les sources ANSSI, RGPD et OWASP sont en accès libre.

---

*Document de travail, construit à partir des observations disponibles au 25/09/2026. Sections marquées "non testé" ou "non qualifiable" nécessitent une vérification ou une décision supplémentaire avant intégration au rapport final.*
