# Cahier des charges fonctionnel (CDCF), Centre Art & Danse

| | |
|---|---|
| **Projet** | Application de gestion de l'école de danse « Centre Art & Danse » |
| **Maître d'œuvre** | VNWeb, agence web (Antibes) |
| **Maître d'ouvrage** | Direction de l'école de danse |
| **Objet du document** | Décrire ce que l'application doit faire (exigences fonctionnelles) et les qualités qu'elle doit respecter (exigences non fonctionnelles, dont la sécurité), avec le moyen de vérifier chacune |
| **Document associé** | Le choix des composants techniques est traité dans le cahier des charges technique (CDCT), pas ici |

---

## 1. Contexte et enjeux

Centre Art & Danse est une application interne qui structure la communication d'une école de danse : publications par groupe ou globales, messagerie privée, suivi du parcours des enfants par leurs parents, gestion des comptes par la direction. Ce n'est pas un produit grand public, c'est un outil réservé au personnel et aux familles de l'école. C'est une application web installable sur téléphone (PWA), qui peut envoyer des notifications même fermée.

L'application est **déjà en production** et utilisée au quotidien. Le CDCF a donc deux fonctions :
- formaliser le périmètre fonctionnel existant, validé par le client ;
- fixer les évolutions et les exigences de sécurité à réaliser, qui n'ont pas fait l'objet d'un audit avant la mise en ligne.

**Enjeu principal.** La majorité des utilisateurs sont des enfants. Les élèves de 15 ans ou plus ont un compte propre. Les enfants de moins de 15 ans n'en ont pas : leur profil (identité, date de naissance, coordonnées) est créé et géré par le compte de leur parent. L'absence de compte propre ne supprime pas la donnée : la sécurité des informations des mineurs repose entièrement sur la sécurité du compte parent et de l'application.

## 2. Parties prenantes

| Acteur | Rôle dans le projet |
|---|---|
| Direction de l'école de danse | Commanditaire et cliente. Exprime les besoins, valide les fonctionnalités et les évolutions. Administratrice de l'application. |
| Dirigeant de VNWeb (maître d'apprentissage) | Valide les orientations techniques et les priorités. |
| Développeur front-end VNWeb | Développe et fait évoluer l'interface de l'application. |
| Alternante, développeuse et responsable de la sécurité applicative | Développe, corrige, conduit l'audit, la sécurisation et la documentation (dossier RNCP37173). |
| Professeurs, parents, élèves | Utilisateurs finaux. Attendent une application utilisable au quotidien, sans exposition de leurs données ni de celles de leurs enfants. |

## 3. Utilisateurs et rôles

| Rôle | Description |
|---|---|
| Administrateur | Dirigeant de l'école. Tous les droits, accès à l'espace d'administration, suppression des publications. |
| Professeur | Crée des publications, s'assigne à un groupe ou s'en retire. |
| Superviseur | Parent d'un ou plusieurs élèves. Gère le profil de ses enfants. |
| Élève | Rôle le plus limité. Modifie ses cours et groupes suivis. Crée son compte à partir de 15 ans. En dessous, un parent crée le compte. |
| Élève-superviseur | Cumul d'élève et de superviseur : un élève qui est aussi parent d'un autre élève. |

Aucune autre combinaison de rôles n'existe : un professeur ne peut pas être également superviseur dans le modèle actuel.

### Matrice des droits

| Fonction | Admin | Professeur | Superviseur | Élève | Élève-superviseur |
|---|:-:|:-:|:-:|:-:|:-:|
| Consulter les publications (globales et de ses groupes) | ● | ● | ● | ● | ● |
| Aimer une publication | ● | ● | ● | ● | ● |
| Créer une publication | ● | ● | | | |
| Supprimer une publication | ● | | | | |
| S'assigner à un groupe ou s'en retirer | ● | ● | | | |
| Modifier les cours et groupes qu'il suit | | | | ● | ● |
| Écrire en messagerie privée | ● | ● | ● | ● | ● |
| Gérer le profil de ses enfants | | | ● | | ● |
| Inviter un tiers à superviser un de ses enfants | | | ● | | ● |
| Accéder à l'espace d'administration | ● | | | | |

## 4. Objectifs et critères de succès

**Objectifs**
1. Conserver le périmètre fonctionnel actuel, validé et utilisé en production.
2. Corriger les écarts de contrôle d'accès touchant les données des mineurs.
3. Mettre en place des garde-fous durables : vérification automatique avant mise en production, protection des comptes, détection d'incident.
4. Documenter la conformité au RGPD pour les données des mineurs.

**Critères de succès**
- Aucune vulnérabilité critique ou haute en production sur les parcours traitant des données de mineurs.
- Registre des traitements et base légale documentés pour les données des mineurs et des comptes gérés par un superviseur.
- Une vérification automatique des librairies (failles connues) avant chaque mise en production.
- Les cibles chiffrées (disponibilité, temps de réponse, couverture de tests) sont à définir avec VNWeb (voir section 12).

## 5. Périmètre

**Dans le périmètre**
- Comptes, rôles et cycle de vie des comptes.
- Supervision des enfants par les parents.
- Groupes et cours, publications, messagerie privée, notifications, médias.
- Espace d'administration.
- Exigences de sécurité et de conformité sur l'ensemble de ces fonctions.

**Hors périmètre**
- Messagerie de groupe : la communication de groupe passe par les publications.
- Paiement, facturation, planning de cours, présence : non couverts par l'application actuelle.
- Réécriture de l'application : le chantier porte sur l'existant.

## 6. Exigences fonctionnelles

**Priorités (MoSCoW)** : Must = indispensable, Should = important, Could = souhaitable.
**Statut** : « En production » = fonction existante à conserver ; « À réaliser » = évolution ou correction à produire ; « À vérifier » = à confirmer par l'audit ; « En cours » = exigence applicable pendant tout le chantier.

### 6.1 Comptes et accès

| ID | Exigence | Rôles | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|---|
| EF-01 | Une personne de 15 ans ou plus peut créer elle-même son compte. | Élève | Must | En production | Le compte créé est inactif tant que l'email et l'administrateur ne l'ont pas validé. |
| EF-02 | Un parent peut créer le compte d'un enfant de moins de 15 ans. L'administrateur peut créer tout compte. Un compte ainsi créé est actif immédiatement. | Superviseur, Admin | Must | En production | Le compte est utilisable sans validation d'email ni validation administrateur. |
| EF-03 | Un compte auto-inscrit doit valider son adresse email par un lien valable une heure. Passé ce délai, seul l'administrateur peut renvoyer un lien. | Tous, Admin | Must | En production | Un lien expiré est refusé ; l'utilisateur ne peut pas en régénérer un seul ; l'administrateur peut en renvoyer un. |
| EF-04 | Tout compte auto-inscrit doit être validé manuellement par l'administrateur après confirmation de l'email, sans exception. | Admin | Must | En production | Un compte à email confirmé mais non validé ne peut pas se connecter. |
| EF-05 | L'utilisateur se connecte par email et mot de passe et dispose d'une procédure de mot de passe oublié. | Tous | Must | En production | Connexion réussie avec des identifiants valides, refusée sinon ; un lien de réinitialisation est envoyé à l'adresse du compte. |
| EF-06 | Le système gère cinq rôles (administrateur, professeur, superviseur, élève, élève-superviseur) avec les droits de la matrice de la section 3. | Tous | Must | En production | Chaque fonction de la matrice est refusée aux rôles non autorisés. |
| EF-07 | Chaque utilisateur consulte et modifie ses informations personnelles. | Tous | Must | En production | Les modifications sont enregistrées et visibles à la reconnexion. |
| EF-08 | L'administrateur peut suspendre un compte. Les données sont conservées ; le compte n'apparaît plus comme actif et est exclu des conversations. | Admin | Must | En production | Un compte suspendu ne peut plus se connecter ni recevoir de message. |
| EF-09 | L'administrateur peut supprimer définitivement un compte (« suppression » et « bannissement » désignent la même opération). | Admin | Must | En production | Le compte et ses données associées n'existent plus après l'opération. |

### 6.2 Supervision des enfants

| ID | Exigence | Rôles | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|---|
| EF-10 | Un superviseur peut lister ses enfants, modifier leurs informations et retirer un enfant de l'application. | Superviseur, Élève-superviseur | Must | En production | Un superviseur ne voit et ne modifie que ses propres enfants. |
| EF-11 | Un superviseur peut inviter par email un tiers (autre parent, tuteur) à superviser un de ses enfants. L'invitation n'est effective qu'après acceptation explicite de la personne invitée. | Superviseur, Élève-superviseur | Must | En production | Sans acceptation, aucun lien de supervision n'est créé. Si l'email correspond à un compte, une notification est envoyée. |
| EF-12 | Seul un superviseur déjà lié à un enfant peut envoyer une invitation de supervision pour cet enfant. | Superviseur, Élève-superviseur | Must | À réaliser | Une invitation pour un enfant sans lien avec l'appelant est refusée, y compris pour un professeur ou le superviseur d'un autre enfant. |

### 6.3 Groupes et cours

| ID | Exigence | Rôles | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|---|
| EF-13 | L'administrateur crée, modifie et supprime les cours. Un cours est défini par un nom de groupe, un niveau et une photo ; les catégories et niveaux sont libres. | Admin | Must | En production | Plusieurs cours de même nom de groupe apparaissent sous le même groupe. |
| EF-14 | Tout utilisateur consulte la liste des groupes, des professeurs et des élèves. | Tous | Must | En production | Les trois listes s'affichent pour chaque rôle. |
| EF-15 | Un professeur s'assigne à un groupe ou s'en retire. Un élève modifie ses cours et groupes suivis. | Professeur, Élève | Must | En production | La modification est immédiate et visible sur le profil. |
| EF-16 | L'auto-assignation aux groupes est encadrée : la règle à appliquer (validation par un tiers ou accès libre assumé) est arrêtée avec le client. | Admin, Professeur, Élève | Should | À réaliser | La règle retenue est documentée et vérifiée : un élève ne peut pas rejoindre un groupe qui ne le concerne pas si la règle l'interdit. |

### 6.4 Publications

| ID | Exigence | Rôles | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|---|
| EF-17 | Un administrateur ou un professeur crée une publication, rattachée à un groupe (visible de ses membres) ou globale (visible de tous). | Admin, Professeur | Must | En production | Une publication de groupe n'est pas visible hors du groupe. |
| EF-18 | Tout utilisateur consulte les publications globales et celles de ses groupes. | Tous | Must | En production | Les publications s'affichent sur la page d'accueil et dans la page du groupe. |
| EF-19 | Tout utilisateur peut aimer une publication, une seule fois. | Tous | Should | En production | Un second « j'aime » du même utilisateur n'est pas compté. |
| EF-20 | Les utilisateurs peuvent commenter une publication. Un commentaire ne peut être supprimé que par son auteur ou l'administrateur. | Tous | Should | En production | La suppression par un tiers non administrateur est refusée. |
| EF-21 | L'administrateur peut supprimer une publication. | Admin | Must | En production | La publication disparaît pour tous les utilisateurs. |

### 6.5 Messagerie

| ID | Exigence | Rôles | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|---|
| EF-22 | Un utilisateur échange des messages privés en un-à-un, en temps réel, avec tout utilisateur actif. Il n'existe pas de conversation de groupe. L'utilisateur peut masquer une conversation et voir l'indicateur de saisie et de lecture. | Tous | Must | En production | Le destinataire reçoit le message en temps réel ; une conversation masquée ne l'est que pour l'utilisateur qui l'a masquée. |

### 6.6 Notifications

| ID | Exigence | Rôles | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|---|
| EF-23 | Le système envoie une notification aux utilisateurs qui l'ont acceptée, selon les déclencheurs ci-dessous. | Tous | Should | En production | Chaque déclencheur produit une notification chez le destinataire attendu, et pas chez l'auteur de l'action. |

| Déclencheur | Destinataires |
|---|---|
| Nouvelle publication globale | Tous les utilisateurs sauf l'auteur |
| Nouvelle publication de groupe | Membres du groupe sauf l'auteur |
| Nouveau message privé | Le destinataire du message |
| Validation d'un email par un utilisateur | Tous les administrateurs |
| Invitation à superviser un enfant | La personne invitée, si son compte existe |

### 6.7 Médias

| ID | Exigence | Rôles | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|---|
| EF-24 | Le système permet de déposer et de diffuser des fichiers (photos de cours, avatars, médias attachés aux publications) via un service de fichiers dédié. | Tous | Must | En production | Un fichier déposé est consultable depuis l'application ; un type de fichier non prévu est refusé. |

### 6.8 Administration

| ID | Exigence | Rôles | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|---|
| EF-25 | L'espace d'administration comprend quatre écrans : utilisateurs (statistiques, recherche, création, suppression), validation des comptes, emails non vérifiés (liste, renvoi du lien), cours (liste, modification, suppression, création). | Admin | Must | En production | Chaque écran est accessible à l'administrateur et refusé aux autres rôles. |

## 7. Exigences non fonctionnelles

### 7.1 Sécurité et protection des données

| ID | Exigence | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|
| ENF-01 | **Confidentialité des données des mineurs.** Les données d'un enfant ne sont accessibles qu'au compte superviseur qui lui est associé (et à l'administrateur). Le contrôle d'accès est vérifié sur tous les points d'accès manipulant des données d'enfants. | Must | À réaliser | L'audit de contrôle d'accès ne relève aucun accès à un enfant par un compte sans lien avec lui. |
| ENF-02 | **Protection des comptes.** La double authentification est obligatoire pour les comptes administrateur et superviseur, et proposée aux autres utilisateurs. | Should | À réaliser | Une connexion avec mot de passe seul est refusée sur un compte administrateur ; 100 % des comptes à privilèges ont la double authentification. |
| ENF-03 | **Protection contre les tentatives répétées.** La connexion et la réinitialisation du mot de passe sont limitées en nombre d'essais. | Should | À réaliser | Au-delà du seuil fixé, les essais suivants sont refusés temporairement. |
| ENF-04 | **Chiffrement des échanges.** Toutes les communications entre l'utilisateur et l'application passent par HTTPS. | Must | À vérifier | L'audit confirme l'absence d'accès non chiffré. |
| ENF-05 | **Canal de messagerie restreint.** Seuls les domaines connus de l'application peuvent se connecter au canal de messagerie en temps réel. | Must | À réaliser | Une connexion depuis un site tiers est refusée. |
| ENF-06 | **Sécurité du service de fichiers.** Seuls les utilisateurs authentifiés peuvent déposer un fichier, et l'extension du fichier est contrôlée par liste blanche avant tout traitement sur le serveur. | Must | À réaliser | Un dépôt sans authentification est refusé ; un nom de fichier construit pour injecter une commande est neutralisé. |
| ENF-07 | **Vérification des failles connues.** Les librairies utilisées sont analysées automatiquement avant chaque mise en production, et une faille grave bloque la livraison. | Must | À réaliser | Zéro faille critique ouverte et un rapport d'analyse daté par projet. |
| ENF-08 | **Traçabilité et détection.** Les connexions et les actions d'administration sensibles (validation, suspension, suppression de compte) sont journalisées hors du serveur qui les produit, et une alerte prévient le référent sécurité en cas d'anomalie. | Should | À réaliser | Les journaux sont consultables hors du serveur ; une alerte de test est reçue par le référent. |
| ENF-09 | **Conformité RGPD.** Un registre des traitements est tenu, une base légale explicite est documentée pour les données des mineurs et des comptes gérés par un superviseur, et la suppression définitive d'un compte est possible. | Must | À réaliser | Registre et base légale rédigés et validés par la direction. |
| ENF-10 | **Cohérence des droits administrateur.** Une seule information porte le statut d'administrateur d'un compte, pour éviter une incohérence exploitable dans les contrôles d'accès. | Could | À réaliser | Un compte ne peut pas être à la fois administrateur et non administrateur selon l'écran. |

### 7.2 Disponibilité, qualité et usage

| ID | Exigence | Priorité | Statut | Critère d'acceptation |
|---|---|---|---|---|
| ENF-11 | **Disponibilité.** Aucune interruption de service non planifiée pendant les interventions de sécurisation ; toute intervention à risque est validée au préalable par le dirigeant de VNWeb. | Must | En cours | Aucune interruption relevée sur la période d'intervention. |
| ENF-12 | **Absence de régression.** Chaque correction est vérifiée sur le parcours concerné avant mise en production. | Must | En cours | Un compte rendu de vérification par correction. |
| ENF-13 | **Utilisabilité.** L'application est installable comme une application mobile et utilisable sur smartphone et navigateurs courants. | Must | En production | Installation et usage vérifiés sur les appareils courants des familles. |

## 8. Contraintes

- **Technique** : la pile technologique existante est conservée. Le chantier porte sur l'existant, sans réécriture. Le détail est dans le CDCT.
- **Production** : l'application étant en service, toute correction se fait sans interruption ni perte de données.
- **Organisationnelle** : équipe de deux personnes (un développeur front-end et l'alternante), sans expert en sécurité dédié ; la sécurisation s'insère dans le rythme de l'alternance (quatre jours en entreprise, un jour en formation par semaine). Les priorités sont validées par le dirigeant de VNWeb. Le développeur back-end d'origine a quitté le projet : une partie des choix de conception n'est pas documentée et se reconstitue par lecture du code.
- **Réglementaire** : conformité au RGPD, avec une attention particulière aux données des mineurs et aux élèves de moins de 15 ans dont le profil est géré par un parent.
- **Calendrier** : dossier RNCP37173 à finaliser pour juin 2027.

## 9. Données traitées

| Catégorie | Contenu | Sensibilité |
|---|---|---|
| Identité et coordonnées | Nom, prénom, date de naissance, email, téléphone, adresse, ville, code postal | Élevée, dont mineurs |
| Liens familiaux | Rattachement parent-enfant, invitations de supervision | Élevée |
| Appartenance | Groupes et cours suivis ou animés | Moyenne |
| Contenus | Publications, commentaires, mentions « j'aime » | Moyenne |
| Messages privés | Contenu des conversations en un-à-un | Élevée |
| Médias | Photos de cours, avatars, fichiers attachés | Moyenne à élevée |
| Techniques | Jetons de connexion, abonnements aux notifications | Élevée |

## 10. Modalités de recette

- Chaque exigence est vérifiée par le critère d'acceptation de sa ligne, par un essai manuel documenté tant qu'aucun test automatisé n'existe.
- Les exigences « À réaliser » sont recettées avant mise en production, sur l'environnement de préproduction.
- La direction de l'école valide les exigences fonctionnelles, le dirigeant de VNWeb valide les exigences de sécurité et leur calendrier.

## 11. Traçabilité avec le dossier

| Besoin du dossier (section 4) | Exigences du CDCF |
|---|---|
| BF-01 Comptes et rôles | EF-01, EF-02, EF-05, EF-06, EF-07, EF-08, EF-09 |
| BF-02 Cycle de validation d'un compte | EF-03, EF-04 |
| BF-03 Gestion des enfants | EF-10 |
| BF-04 Invitation de superviseur | EF-11, EF-12 |
| BF-05 Groupes et cours | EF-13, EF-14 |
| BF-06 Auto-assignation | EF-15, EF-16 |
| BF-07 Publications | EF-17, EF-18, EF-21 |
| BF-08 Like | EF-19 (les commentaires, EF-20, ne figurent pas dans la liste du dossier) |
| BF-09 Messagerie | EF-22 |
| BF-10 Notifications | EF-23 |
| BF-11 Médias | EF-24 |
| BF-12 Espace admin | EF-25 |

| Besoin non fonctionnel ou user story du dossier | Exigences du CDCF |
|---|---|
| BNF-01 Confidentialité des mineurs, US-02, US-07 | ENF-01, EF-12 |
| BNF-02 Disponibilité | ENF-11 |
| BNF-03 Non-régression | ENF-12 |
| BNF-04 Traçabilité des actions admin | ENF-08 |
| BNF-05 RGPD, US-03 | ENF-09 |
| BNF-06 Sécurité des échanges | ENF-04 |
| US-01 Pipeline et portes de sécurité | ENF-07 |
| US-04 Auto-assignation | EF-16 |
| US-05 Extension de fichier | ENF-06 |
| US-06 Champs administrateur | ENF-10 |
| US-08 Origines de la messagerie | ENF-05 |

## 12. Informations à compléter avec VNWeb et le client

Ces points ne figurent dans aucun document du projet et ne sont pas fixés ici arbitrairement :

- Nombre d'utilisateurs et volume de données (volumétrie).
- Cibles chiffrées : disponibilité, temps de réponse, couverture de tests.
- Durée de conservation des données et règle de suppression des comptes inactifs.
- Règle retenue pour l'auto-assignation aux groupes (EF-16).
- Seuil de tentatives de connexion avant blocage (ENF-03).
- Budget et calendrier de réalisation des exigences « À réaliser ».
- Coordonnées et nom de la direction cliente.
- Niveau d'accessibilité visé.
