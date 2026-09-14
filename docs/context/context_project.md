# Contexte projet — Centre Art & Danse

> Document de référence, état des lieux (« AS-IS ») du projet « Centre Art & Danse » : périmètre fonctionnel, stack technique, architecture applicative, état de l'infrastructure et des environnements, et constats et risques immédiats. Il sert de socle factuel unique à l'évaluation de sécurité du dossier RNCP37173.

## 1. Description du projet

### 1.1 Objet de l'application

« Centre Art & Danse » (nom court affiché sur l'appareil : « CAD ») est une application de gestion d'une école de danse. Le dirigeant de l'école est administrateur de l'application : il ajoute les professeurs, les parents et les élèves. L'application sert avant tout à structurer la communication interne de l'école (remplacer les messages envoyés un par un à chaque famille par des publications ciblées) et à donner aux parents un suivi du parcours de leurs enfants.

C'est une PWA (Progressive Web App, une application web qui peut s'installer sur un téléphone comme une application mobile classique, sans passer par un store d'application). Elle peut envoyer des notifications push (des messages que l'utilisateur reçoit même quand l'application n'est pas ouverte).

### 1.2 Rôles

| Rôle | Description |
|---|---|
| `PROFESSOR` | Crée des posts, s'auto-assigne ou se retire d'un groupe. |
| `ADMIN` | Tous les droits, plus la page d'administration ; peut supprimer des posts. |
| `SUPERVISOR` | Parent d'un ou plusieurs élèves. |
| `STUDENT` | Rôle le plus limité. Modifie ses cours et groupes suivis. À partir de 15 ans, peut créer son propre compte ; en dessous de 15 ans, c'est un parent qui crée le compte de son enfant. |
| `STUDENT_SUPERVISOR` | Cumul de `SUPERVISOR` et `STUDENT` (un élève qui est aussi parent d'un autre élève). |

Il n'existe pas d'autre combinaison de rôles que `STUDENT_SUPERVISOR` : un professeur ne peut pas être également superviseur dans le modèle actuel.

Point de vigilance fonctionnel confirmé et assumé par l'équipe : l'auto-assignation à un groupe (côté professeur) et la modification des groupes suivis (côté élève) se font en libre-service, sans validation d'un tiers (professeur, admin ou parent). Aucun contrôle métier n'empêche donc un élève de rejoindre un groupe qui ne le concerne pas.

### 1.3 Cycle de vie d'un compte

1. **Création** — auto-inscription (élève de 15 ans ou plus), ou création directe par un parent (enfant de moins de 15 ans) ou par l'admin. Un compte créé directement par un parent ou par l'admin est activé (`enabled = true`) sans validation email ni validation admin. Un compte auto-inscrit démarre avec `hasValidEmail = false` et `enabled = false`.
2. **Validation de l'email** — lien envoyé par email, valide une heure. Passé ce délai, seul l'admin peut renvoyer un nouveau lien.
3. **Validation par l'admin** — une fois l'email confirmé, le compte reste désactivé jusqu'à validation manuelle par l'admin. Tout compte auto-inscrit passe par cette double étape, sans exception, y compris un élève de 15 ans ou plus.
4. **Suspension** — l'admin repasse le compte à désactivé. Les données restent en base ; le compte n'apparaît plus comme actif (exemple : exclu des conversations tant qu'il est désactivé).
5. **Suppression ou bannissement** — deux libellés pour la même opération réelle : suppression définitive de la ligne utilisateur en base.

### 1.4 Fonctionnalités par page

**Accueil** — tous : voir les posts (de groupe et globaux), liker un post ; `ADMIN`/`PROFESSOR` : créer un post ; `ADMIN` : supprimer un post.

**Groupes** — tous : voir la liste des groupes, des professeurs, des élèves ; `ADMIN`/`PROFESSOR` : créer un post dans un groupe, s'auto-assigner ou se retirer d'un groupe ; tous : liker un post de groupe.

**Messagerie** — tous : envoyer un message à n'importe qui, en chat 1-à-1 exclusivement (pas de chat de groupe ; la communication de groupe passe par les posts).

**Profil** — tous : voir et modifier ses informations personnelles ; `STUDENT_SUPERVISOR`/`SUPERVISOR` : lister ses enfants, modifier les informations d'un enfant, retirer un enfant de l'application.

**Connexion** — créer un compte (si 15 ans ou plus), mot de passe oublié, connexion par email et mot de passe.

### 1.5 Groupes et cours

Un « groupe » est l'entité nommée (exemple : « Jazz moderne »). Un « cours » est une classe concrète rattachée à ce groupe, définie par un nom de groupe, un niveau et une photo. Plusieurs cours partageant le même nom de groupe sont ainsi regroupés sous ce groupe. Il n'existe pas de liste fixe de catégories : l'admin crée librement les catégories et niveaux selon les besoins de l'école.

### 1.6 Posts

Un post peut être rattaché à un groupe (visible par les membres de ce groupe) ou être global (visible par tous, page Accueil). Interactions disponibles : le like, et les commentaires. La fonctionnalité de commentaires est bien active des deux côtés (backend et frontend) — une hypothèse initiale la disant retirée du frontend a été vérifiée et infirmée par lecture directe du code.

### 1.7 Invitation de superviseur

Fonctionnalité réelle et voulue : un parent peut inviter par email une autre personne à devenir superviseur d'un de ses enfants (cas d'usage type : parents séparés, tuteur). Si l'email correspond à un compte existant, une notification push est envoyée. L'invitation n'est effective que si la personne invitée l'accepte explicitement — ce n'est pas un rattachement automatique.

Un écart de contrôle d'accès est identifié sur cette fonctionnalité : la route qui traite l'invitation vérifie seulement le rôle de l'appelant, pas son lien réel avec l'enfant ciblé. En connaissant l'identifiant d'un enfant, un compte ayant un rôle éligible peut donc inviter un tiers à devenir superviseur de cet enfant, même sans lien avec lui. C'est le constat de sécurité le plus prioritaire identifié à ce jour (catégorie OWASP A01 : Broken Access Control), les données concernées étant celles de mineurs.

### 1.8 Notifications push

| Déclencheur | Destinataires |
|---|---|
| Nouveau post global | Tous les utilisateurs sauf l'auteur |
| Nouveau post de groupe | Membres du groupe sauf l'auteur |
| Nouveau message 1-à-1 | Le destinataire du message |
| Validation d'email par un utilisateur | Tous les admins |
| Invitation à superviser un enfant | La personne invitée, si son compte existe déjà |

### 1.9 Espace admin

- **Utilisateurs** — statistiques, liste des utilisateurs (recherche), suppression (= bannissement), création directe d'un compte par l'admin.
- **Validation** — recherche, validation d'un compte, bannissement ou suppression.
- **Emails non vérifiés** — recherche, liste des emails non confirmés, renvoi de l'email de validation (nouveau lien, valide une heure).
- **Cours** — liste de tous les cours (recherche), modification, suppression, création d'un cours (nom de groupe, niveau, photo).

### 1.10 Écarts fonctionnels connus (non bloquants)

- Le champ booléen `isAdmin` et le champ `status` (qui porte aussi la valeur `ADMIN`) représentent tous les deux l'adminship dans le modèle actuel — une redondance à garder en tête (voir aussi section 2.3).
- Aucune validation métier n'encadre l'auto-assignation aux groupes (élèves comme professeurs) — choix assumé, pas une lacune de documentation.

## 2. Stack technique

Le dépôt regroupe trois sous-projets indépendants, chacun avec son propre `package.json` : `app-dance` (frontend), `api-dance` (backend) et `cdn-app-dance` (service d'upload et de diffusion de fichiers).

### 2.1 app-dance (frontend)

| Domaine | Détail |
|---|---|
| Nature | PWA (Progressive Web App) installable, nommée « Centre Art & Danse » (manifeste) |
| Framework UI | React 19 (`react` / `react-dom` ^19.1.0) |
| Composants mobiles/UI | Ionic React (`@ionic/react` ^8.6.3) |
| Outil de build/dev | Vite ^6.3.5, plugin `@vitejs/plugin-react` |
| Serveur de dev | `vite`, port 5173, rechargement à chaud (HMR) en WebSocket |
| Build de production | `vite build` |
| Framework CSS | Bootstrap 5 (^5.3.7) et `react-bootstrap` (^2.10.10), avec `bootstrap-icons` |
| Préprocesseur CSS | Sass (^1.89.2) |
| Animations | Framer Motion (^12.23.6) |
| Composants de données | AG Grid (`ag-grid-react` + `@ag-grid-community/locale`, ^34.1.2) |
| Composants divers | `react-select`, `react-tooltip`, `react-hot-toast` |
| Routage | `react-router-dom` ^6.30.1 |
| Gestion d'état | Zustand ^5.0.6 |
| Temps réel côté client | `socket.io-client` ^4.8.1 (connexion au serveur socket.io de `api-dance`) |
| Stockage local mobile | `@react-native-async-storage/async-storage` ^1.24.0 |
| Services tiers | Firebase (^11.10.0) — authentification et/ou services Google ; configuration précise à documenter séparément |
| Internationalisation | `i18next` + `react-i18next` (^25.5.2 / ^15.7.3) |
| Dates relatives | `javascript-time-ago` + `react-time-ago` |
| Lint | ESLint, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh` |
| Tests automatisés | Aucun script de test défini dans `package.json` à ce stade |
| Variables d'environnement | Préfixe `VITE_` pour exposer une variable au code client au moment du build ; le `.env` du sous-projet est exclu du suivi Git |
| Conteneurisation | Aucune (pas de Dockerfile) |

### 2.2 api-dance (backend)

| Domaine | Détail |
|---|---|
| Runtime | Node.js, module ESM (`"type": "module"`) |
| Langage | TypeScript, vérifié par `tsc --noEmit` |
| Framework | AdonisJS 6 (`@adonisjs/core` ^6.19.1) |
| Build | `node ace build` |
| Serveur de dev | `node ace serve --hmr` |
| ORM | Lucid (`@adonisjs/lucid` ^21.8.1) |
| Base de données active | MySQL (client `mysql2`), configurée dans `config/database.ts` |
| Configuration alternative présente mais inactive | PostgreSQL (`client: pg`), commentée dans le même fichier ; le paquet `pg` reste en dépendance sans être utilisé |
| Migrations | Gérées par Lucid, chemin `database/migrations` |
| Authentification | `@adonisjs/auth` ^9.5.1, jetons d'accès opaques stockés en base (table `auth_access_tokens`) |
| Autorisation | `@adonisjs/bouncer` ^3.1.6 (politiques d'accès) |
| CORS | `@adonisjs/cors` ^2.2.1, configuré dans `config/cors.ts` |
| Limitation de débit | `@adonisjs/limiter` ^2.4.0 |
| Validation d'entrée | VineJS (`@vinejs/vine` ^3.0.1) |
| Assets statiques | `@adonisjs/static` ^1.1.1 |
| Temps réel natif AdonisJS | `@adonisjs/transmit` ^2.0.2 (Server-Sent Events) — installé mais non relié à une route active (code mort) |
| WebSockets | `socket.io` ^4.8.1 — canal temps réel réellement utilisé, porte la messagerie privée |
| Notifications push | `web-push` ^3.6.7 |
| Email | `@adonisjs/mail` ^9.2.2, transport `nodemailer` ^7.0.11 |
| Documentation API | `@foadonis/openapi` ^0.4.1, `swagger-jsdoc` ^6.2.8 |
| Journalisation | `pino-pretty`, `pino-roll` (rotation de logs) |
| Hachage de mot de passe | scrypt, paramètres par défaut d'AdonisJS (`cost: 16384, blockSize: 8, parallelization: 1, maxMemory: 33554432`), non ajustés au contexte du projet |
| Tests automatisés | Japa (`@japa/runner`, `@japa/assert`, `@japa/api-client`, `@japa/plugin-adonisjs`), exécuté via `node ace test` — présent en outillage mais aucun fichier de test réel constaté (voir section 4) |
| Lint / formatage / typage | ESLint, Prettier, `tsc --noEmit` |
| Variables d'environnement | Sensibles, exclues du suivi Git (`.env` dans `.gitignore`) |
| Conteneurisation | Aucune (pas de Dockerfile) |

### 2.3 cdn-app-dance (API d'upload et de diffusion de fichiers)

| Domaine | Détail |
|---|---|
| Nom interne | `cdn.cad.server02`, dédié à `app.centreartetdanse.com` |
| Runtime | Node.js, module ESM |
| Langage | TypeScript ^5.9.2, compilé via `tsc` (cible ES2020) |
| Framework serveur | Express 5 (^5.1.0) |
| Point d'entrée | `app.ts` (source dans `src/`, sortie compilée dans `dist/`) |
| Upload de fichiers | Multer ^2.0.2 |
| CORS | `cors` ^2.8.5 — configuration précise (liste blanche ou wildcard) à vérifier avant mise en production |
| Configuration d'environnement | `dotenv` ^17.2.1 |
| Tests automatisés | Aucun (script `test` non défini, place-holder d'erreur) |
| Lint | Aucun script de lint défini, à la différence de `api-dance` et `app-dance` |
| Authentification | Aucun mécanisme repéré au niveau de la stack déclarée — point à clarifier avant de considérer les routes d'upload comme protégées |
| Conteneurisation | Aucune (pas de Dockerfile) |
| Variables d'environnement | Le `.env` du sous-projet est exclu du suivi Git |

### 2.4 Base de données — schéma (MySQL, 42 migrations)

Reconstitué à partir des migrations réelles de `api-dance/database/migrations` (source de vérité, plutôt que les modèles Lucid qui peuvent diverger). Tables principales :

- **users** — compte utilisateur, porte le rôle via la colonne `status` (`professeur`, `admin`, `superviseur`, `élève`, `élève_superviseur`), un booléen `is_admin` redondant avec `status = 'admin'`, un booléen `enabled` (compte actif/suspendu), un booléen `has_valid_email`, et les données personnelles en clair (`first_name`, `last_name`, `phone_number`, `address`, `postal_code`, `city`, `birth_date`).
- **auth_access_tokens** — table standard `@adonisjs/auth`, jetons d'API liés à `users` (CASCADE).
- **rate_limits** — table du package `@adonisjs/limiter`.
- **tokens** — jetons applicatifs (validation email, réinitialisation de mot de passe).
- **supervisor_users** — relation parent-enfant (`supervisor_id`, `user_id`), contrainte d'unicité par paire.
- **supervisor_invitations** — invitation d'un tiers à devenir superviseur d'un enfant, contrainte d'unicité par paire (email, enfant).
- **groups** — le groupe au sens large (nom, image, niveau).
- **groups_names** — catalogue de suggestion de noms de groupes, sans clé étrangère stricte vers `groups`.
- **groups_users** — liaison utilisateur-groupe, colonne `type` distinguant suivi (élève, `FOLLOW = 1`) et assignation (professeur, `SUBSCRIBE = 2`).
- **posts** — contenu, auteur, `group_id` nullable (`NULL` = post global).
- **medias** — fichiers attachés à un post.
- **likes** — un like par utilisateur et par post (contrainte d'unicité).
- **comments** — contenu, auteur, post, `parent_id` auto-référencé (fil de discussion), suppression douce via `is_deleted`. Table pleinement active (voir section 1.6).
- **conversations** — conversation 1-à-1 entre deux utilisateurs (contrainte d'unicité sur la paire).
- **conversation_config** — état de masquage par utilisateur et par conversation (l'état « masqué » vit ici, pas sur la conversation entière).
- **messages** — contenu, expéditeur, conversation, indicateur de lecture.
- **subscriptions** — abonnements aux notifications push (endpoint, clés Web Push).

`users` est le pivot central du modèle : auto-relation via `supervisor_users`, lié à `groups` via `groups_users`, auteur de `posts`, `comments`, `likes` et `messages`. Le modèle est fortement relationnel, avec des clés étrangères en cascade sur la quasi-totalité des relations.

## 3. Architecture applicative

### 3.1 Assemblage des trois sous-projets

- **`app-dance`** (client) consomme l'API HTTP exposée par `api-dance` et se connecte à son serveur `socket.io` pour la messagerie privée en temps réel. Elle s'appuie sur `cdn-app-dance` pour l'upload et la diffusion des fichiers (photos).
- **`api-dance`** (backend) porte la logique métier, l'accès aux données (MySQL via Lucid) et l'authentification pour l'ensemble de la plateforme.
- **`cdn-app-dance`** est un service indépendant, dédié au dépôt et à la diffusion de fichiers pour `app.centreartetdanse.com`, isolé du reste de la logique métier.

Chaque sous-projet a sa propre base de code et son propre cycle de dépendances ; aucun des trois n'est conteneurisé (pas de Dockerfile) et aucun n'a de pipeline CI/CD configuré (voir section 4).

## 4. État actuel de l'infrastructure et des environnements

Cette section décrit l'état réel du cycle de déploiement et d'exploitation au moment de la rédaction de ce document, tel que constaté, pour servir de socle factuel à l'audit.

### 4.1 Absence de pipeline CI/CD

Aucun pipeline d'intégration continue ni de déploiement continu n'est en place à ce jour, sur aucun des trois sous-projets : aucun dossier `.github/workflows` n'existe à la racine du dépôt. Les conséquences pratiques :

- Aucun test automatisé, aucune vérification de type, aucun lint et aucun audit de dépendances ne s'exécutent automatiquement avant la mise en production d'un changement.
- Aucun outil d'analyse statique de sécurité (SAST) ni de détection de secrets ne tourne en continu — les outils recommandés (CodeQL, Semgrep, Gitleaks, Dependabot) ne sont pas encore déployés ; ils sont proposés comme prochaine étape.
- Chaque mise en production repose sur une intervention manuelle.

### 4.2 Environnements et hébergement

Trois environnements distincts sont utilisés, chacun hébergé sur une infrastructure séparée :

| Environnement | Hébergement |
|---|---|
| Staging | Serveur de VnWeb |
| Préproduction (préprod) | Serveur externe, distinct de VnWeb |
| Production | Autre serveur externe, distinct du serveur de préproduction |

Aucun des trois environnements ne partage donc la même infrastructure. Le service `cdn-app-dance` est lui aussi hébergé et maintenu par VNWeb, et une liste CORS distingue les domaines de l'application et de préproduction dans `config/cors.ts` — ce qui est cohérent avec l'existence d'un environnement de préproduction séparé décrit ici.

### 4.3 Mode de déploiement de l'application

L'application backend (`api-dance`) est en Node.js et tourne comme un processus Node exécuté directement sur le serveur cible. Aucune conteneurisation ni orchestration n'est en place : ni Docker (déjà constaté sous-projet par sous-projet : aucun Dockerfile sur les trois sous-projets), ni système d'orchestration de conteneurs (Kubernetes ou équivalent). Le déploiement est donc « à plat » sur le serveur, c'est-à-dire que le code s'exécute directement sur la machine hôte, sans couche d'isolation ni de reproductibilité d'environnement fournie par un conteneur.

### 4.4 Supervision et détection

Aucun outil de supervision (monitoring) n'est en place, quel que soit l'environnement. Aucun SIEM (Security Information and Event Management, un outil qui centralise et met en corrélation les événements de sécurité provenant de plusieurs sources pour détecter une activité suspecte) n'est déployé. Il n'existe donc actuellement aucun mécanisme de détection automatique d'incident, ni côté disponibilité (panne, ralentissement) ni côté sécurité (activité anormale, tentative d'intrusion).

### 4.5 Synthèse de maturité

La maturité DevSecOps du projet est, à ce stade, très faible : peu de choses sont faites côté sécurisation du cycle de déploiement lui-même (au-delà des constats de sécurité applicative déjà relevés par ailleurs). Cet état n'est pas une anomalie cachée : il est cohérent avec le profil du projet (équipe de trois personnes, sans expert sécurité dédié en interne) et avec les priorités déjà fixées, qui placent la mise en place d'un pipeline CI/CD minimal en position 6 sur 12, comme « condition de fiabilité de tous les correctifs suivants ».

## 5. Équipe et contexte client

### 5.1 Composition de l'équipe actuelle

L'équipe actuelle compte deux personnes : le développeur auteur de ce dossier et un développeur front-end, tous deux en charge de la suite du projet. La version initiale de l'application a été mise en place par un développeur back-end qui a depuis quitté le projet et n'y contribue plus. Cette rotation d'équipe a une conséquence concrète pour l'audit : une partie des choix de conception antérieurs n'a pas de documentation laissée par son auteur d'origine et doit être reconstituée par lecture directe du code plutôt que confirmée avec la personne à l'origine du choix.

### 5.2 Client et gouvernance fonctionnelle

Le projet est réalisé pour un client externe. La validation des fonctionnalités et l'expression des besoins (demandes d'évolution ou de correction) reviennent au client, et non à une équipe produit interne. Les choix fonctionnels décrits dans ce document (rôles, cycle de vie d'un compte, fonctionnalités par page) reflètent donc des décisions validées par le client plutôt que des arbitrages internes à l'équipe technique.

## 6. Constats et risques immédiats

Cette synthèse reste factuelle et ne cherche pas à redoubler inutilement les constats déjà posés ailleurs dans le présent document.

- **Absence de CI/CD → absence de porte de qualité automatique.** Aucun test, aucun scan de sécurité (SAST, audit de dépendances, détection de secrets) ne s'exécute avant qu'un changement n'atteigne un environnement. Ce point est un risque structurel : il conditionne la fiabilité de tous les correctifs de sécurité identifiés par ailleurs. Concrètement, un correctif de sécurité (par exemple une restriction CORS sur le canal socket.io) ne bénéficierait lui-même d'aucune vérification automatique avant sa propre mise en production tant que ce point reste ouvert.
- **Trois environnements sur trois infrastructures distinctes, sans automatisation du déploiement → risque de dérive de configuration.** Le staging (serveur VnWeb), la préproduction et la production (deux autres serveurs externes distincts) sont déployés manuellement, chacun sur sa propre machine. Sans pipeline pour garantir qu'un même artefact est déployé de façon identique sur les trois environnements, le risque n'est pas seulement une divergence de version de code, mais aussi une divergence de configuration (variables d'environnement, dépendances installées, paramètres réseau) entre les environnements — un risque qui complique en particulier la détection d'un problème en préproduction qui ne se reproduirait pas de la même façon en production, ou l'inverse.
- **Déploiement Node.js à plat, sans conteneurisation → reproductibilité et isolation limitées.** L'absence de conteneurisation, déjà constatée sous-projet par sous-projet (aucun Dockerfile sur les trois sous-projets), signifie que l'environnement d'exécution de chaque serveur dépend de son état propre (versions de Node.js, paquets système installés) plutôt que d'une définition reproductible versionnée avec le code.
- **Absence de monitoring et de SIEM → absence de détection d'incident.** Ni panne ni activité suspecte ne déclenchent aujourd'hui d'alerte automatique. L'analyse dynamique (DAST) elle-même reste reportée à la disponibilité d'un environnement de test : même une fois mise en place, il s'agirait d'un contrôle ponctuel, pas d'une supervision continue. L'absence de SIEM signifie aussi qu'une tentative d'exploitation des vulnérabilités déjà identifiées (par exemple une tentative d'invitation de superviseur non autorisée, ou une tentative d'upload malveillant sur `cdn-app-dance`) ne laisserait aujourd'hui aucune trace exploitable en temps réel.
- **Priorisation.** Les correctifs de contrôle d'accès touchant les mineurs restent la priorité la plus haute, la mise en place d'un pipeline CI/CD minimal reste une condition structurelle pour fiabiliser les correctifs suivants, et l'absence de monitoring et de SIEM vient s'ajouter à cette liste, à traiter en aval de ces correctifs prioritaires.
