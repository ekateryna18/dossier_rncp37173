# Plan d'Audit de Sécurité : Centre Art & Danse

Document autonome, rédigé par Axel (agent DevSecOps du dépôt). Il sert de base
de travail à l'audit de sécurité des trois sous-projets et sera résumé plus
tard dans la section 6 du dossier RNCP37173. Toute mention "[à vérifier]"
signale un point non tranché par lecture directe du code : à traiter avant de
considérer l'audit complet.

Date de rédaction : 2026-07-24.

## 1. Périmètre de l'audit

### 1.1 Sous-projets couverts

| Sous-projet | Stack | Rôle | Exposition |
|---|---|---|---|
| `api-dance` | AdonisJS 6 / TypeScript, Lucid ORM, MySQL | API backend, authentification, données métier | Publique (API HTTP + WebSocket) |
| `app-dance` | React 19 / Ionic, Vite, PWA installable | Interface utilisateur (parents, professeurs, élèves) | Client, servi publiquement |
| `cdn-app-dance` | Express 5, Multer | Upload et diffusion de fichiers (photos, vidéos) | Publique (aucune authentification constatée, voir 3.6) |

### 1.2 Couches auditées

1. Code applicatif : contrôleurs, validateurs, requêtes ORM, dans les trois sous-projets.
2. Dépendances tierces : les trois `package.json` (versions figées, vulnérabilités connues).
3. Secrets et configuration sensible : présence et exclusion Git des fichiers `.env`, gestion de `APP_KEY`.
4. Configuration réseau : CORS (HTTP et WebSocket), en-têtes de sécurité.
5. Contrôle d'accès : vérification de rôle, vérification de propriété (le lien entre un utilisateur et l'enfant qu'il supervise).
6. Stockage des données : colonnes en base, chiffrement ou absence de chiffrement des données personnelles.
7. Analyse dynamique : reportée à la disponibilité d'un environnement de test (voir section 6).

### 1.3 Priorité aux parcours touchant les mineurs

Le cœur de gravité de cette mission de sécurisation est la protection des
données des élèves mineurs (moins de 15 ans), qui n'ont pas de compte propre
et sont gérés via le compte de leur parent superviseur (table `users`,
relation `supervisors`/`supervisees` dans `api-dance/app/models/user.ts`,
lignes 124 à 140). Deux raisons concrètes placent ces parcours en tête de
liste, toutes deux vérifiées par lecture directe du code (détail en
section 3) :

- Le lien parent-enfant peut être modifié par un tiers qui connaît seulement
  l'identifiant de l'enfant (constat 3.1).
- Les colonnes personnelles de ces mineurs (date de naissance, téléphone,
  adresse, code postal, ville, nom, prénom) sont stockées en clair dans la
  table `users` (constat 3.4).

Tout ce qui touche à la relation superviseur/enfant, à l'inscription d'un
enfant, et au stockage de ses données, passe donc en priorité haute par
défaut dans ce plan.

## 2. Méthodologie

Six catégories d'audit, menées dans cet ordre de dépendance logique (une
catégorie amont peut invalider un contrôle en aval) :

1. **Analyse statique du code (SAST)** : recherche de vulnérabilités dans le
   code source sans exécution, sur les trois sous-projets.
2. **Audit des dépendances** : recherche de CVE connues sur les paquets
   npm figés dans chaque `package.json`.
3. **Détection de secrets** : recherche de clés, tokens ou mots de passe
   qui auraient pu être committés par erreur, malgré l'exclusion Git des
   `.env` (voir 3.9).
4. **Revue manuelle de configuration** : CORS, gestion de session,
   validation d'entrée, limitation de débit (rate limiting), en s'appuyant
   sur la lecture directe des fichiers de configuration.
5. **Revue de contrôle d'accès** : pour chaque route sensible, vérifier que
   le contrôle porte sur la propriété de la ressource et pas seulement sur
   le rôle de l'appelant.
6. **Analyse dynamique (DAST)** : contre un environnement de test réel, une
   fois disponible. Un DAST contre rien ne produit rien de valide.

### Référentiel de fond

Le référentiel utilisé pour juger la solidité d'un contrôle est
**l'OWASP ASVS** (Application Security Verification Standard). C'est le
même référentiel que celui utilisé pour juger la preuve du bloc 3 du
RNCP37173, ce qui garde une grille de lecture cohérente entre l'implémentation
et l'évaluation. En complément, **l'OWASP Top 10:2021** sert de nomenclature
courte pour classer chaque constat. Correspondance retenue pour les constats
de ce plan :

| Constat | Catégorie OWASP Top 10:2021 |
|---|---|
| 3.1 Contrôle d'accès superviseur/enfant | A01 : Broken Access Control |
| 3.2 CORS wildcard sur le canal socket.io | A05 : Security Misconfiguration |
| 3.3 Paramètres scrypt par défaut | A02 : Cryptographic Failures |
| 3.4 Données de mineurs en clair | A02 : Cryptographic Failures |
| 3.5 Code mort `channels_controller.ts` | A05 : Security Misconfiguration (surface inutile) |
| 3.6 Absence d'authentification sur `cdn-app-dance` | A01 : Broken Access Control |
| 3.7 Absence de tests et de CI/CD | Hors nomenclature Top 10, mais condition de A06 (composants non vérifiés) |
| 3.8 Fonctionnalité commentaires (correction du constat initial) | Non applicable, voir détail |
| 3.9 Redondance `isAdmin` / `status` | A04 : Insecure Design |
| 3.10 Auto-assignation aux groupes sans validation métier | A01 : Broken Access Control |
| 3.11 Risque d'injection via l'extension de fichier non filtrée (`cdn-app-dance`) | A03 : Injection |
| 3.12 Absence de limitation de débit sur la réinitialisation de mot de passe | A07 : Identification and Authentication Failures |
| 3.13 Absence de `@adonisjs/shield` (CSRF) | A05 : Security Misconfiguration |
| 3.14 `.gitignore` de `cdn-app-dance` sans exclusion `.env` | A05 : Security Misconfiguration |

## 3. Constats déjà identifiés à la lecture du code

Chaque constat ci-dessous a été relu directement dans le fichier cité avant
rédaction de ce plan.

### 3.1 Contrôle d'accès incomplet sur l'ajout de superviseur

**Fichier** : `api-dance/app/controllers/auth_controller.ts`, méthode
`addSupervisor`, lignes 560 à 612 (route `POST /add-supervisor`, déclarée
dans `api-dance/app/routes/auth.ts` ligne 10).

**Risque** : la méthode vérifie seulement que l'appelant a le statut
professeur, superviseur ou élève-superviseur (lignes 562 à 567), puis
recherche l'enfant par le `childId` fourni dans le corps de requête
(ligne 571). Aucune vérification que l'appelant est déjà superviseur de cet
enfant précis. Un utilisateur authentifié avec un statut valide peut donc
inviter un tiers de son choix à devenir superviseur d'un enfant qui n'est pas
le sien, en connaissant seulement l'identifiant de cet enfant.

**Priorité** : Haute. Ce parcours touche directement la protection des
mineurs : c'est le point d'entrée qui décide qui peut superviser un enfant.

### 3.2 CORS incohérent sur le canal WebSocket

**Fichier** : `api-dance/start/ws.ts`, lignes 6 à 11.

**Risque** : le serveur socket.io est initialisé avec `cors: { origin: '*' }`,
alors que la configuration CORS de l'API HTTP principale
(`api-dance/config/cors.ts`, lignes 11 à 17) restreint explicitement les
origines à une liste de cinq domaines nommés. Le canal socket.io, qui porte
la messagerie privée (`ChatService.handleConnection`, ligne 13), accepte donc
des connexions depuis n'importe quelle origine, contrairement au reste de
l'API.

**Priorité** : Haute. Incohérence directe entre deux couches de la même
application, sur un canal qui porte des échanges privés.

### 3.3 Paramètres de hachage de mot de passe non ajustés

**Fichiers** : `api-dance/config/hash.ts` (lignes 3 à 14) et
`api-dance/app/models/user.ts` (ligne 19, `withAuthFinder(() => hash.use('scrypt'), ...)`).

**Risque** : le hachage utilise scrypt avec les paramètres par défaut
d'AdonisJS (`cost: 16384, blockSize: 8, parallelization: 1,
maxMemory: 33554432`), non ajustés au contexte du projet ni comparés à la
capacité réelle du serveur de production. Argon2id, premier choix recommandé
par l'OWASP Password Storage Cheat Sheet, n'est pas utilisé.

**Priorité** : Moyenne. Le risque ne se matérialise qu'en cas de fuite de la
base ; il reste réel puisque les comptes concernés incluent des parents
gérant des données de mineurs.

### 3.4 Données personnelles de mineurs stockées en clair

**Fichier** : `api-dance/app/models/user.ts`, colonnes `birthDate` (ligne
111), `phoneNumber` (ligne 67), `address` (ligne 71), `postalCode` (ligne 85),
`city` (ligne 78), `firstName` (ligne 52), `lastName` (ligne 56).

**Risque** : ces colonnes sont déclarées comme des colonnes simples
(`@column()`), sans transformation de chiffrement au niveau champ. La clé
`APP_KEY` existe (`api-dance/config/app.ts`, ligne 13) et sert au chiffrement
des cookies et à la signature d'URL (`http.cookie`, lignes 32 à 39), mais
aucun usage de cette clé pour chiffrer ces colonnes n'a été trouvé dans le
modèle. Ces colonnes concernent aussi bien les comptes adultes que les fiches
d'enfants créées par `addChild` (`auth_controller.ts`, ligne 311).

**Priorité** : Haute, spécifiquement pour les colonnes qui portent des
informations d'enfants mineurs.

### 3.5 Code mort : contrôleur de canaux non routé

**Fichiers** : `api-dance/app/controllers/channels_controller.ts` (utilise
`@adonisjs/transmit`) et `api-dance/config/transmit.ts`.

**Risque** : une recherche du mot "channel" dans tout le dossier
`api-dance/app/routes/` ne retourne aucun résultat. Ce contrôleur n'est
relié à aucune route et constitue un reliquat du gabarit de démarrage
AdonisJS Transmit. Une surface de code non routée n'est pas un risque actif
en soi, mais élargit la surface auditée pour rien et peut être réactivée par
erreur lors d'un futur ajout de route.

**Priorité** : Basse. Nettoyage recommandé plutôt que correctif de sécurité.

### 3.6 Absence de mécanisme d'authentification sur `cdn-app-dance`

**Fichier** : `cdn-app-dance/src/app.ts`, l'intégralité du fichier (fichier
unique portant toutes les routes du service).

**Risque** : confirmé par lecture complète du code source de
`cdn-app-dance` (`src/app.ts`, `src/utils/env.ts`, `src/utils/sanitizer.ts`,
`src/utils/str.ts`, `src/utils/string.ts`, `src/DTO/CleanMediaDTO.ts`,
c'est-à-dire l'ensemble du service). Aucune vérification d'identité,
aucun token, aucune session n'encadre les routes `/upload/group`,
`/upload/user`, `/upload/post` ni `/medias/clean`. Un contrôle par origine et
par adresse IP existait (lignes 24 à 41) mais est entièrement commenté, donc
inactif. Seule protection active : une liste blanche CORS (lignes 19 et
44 à 48), qui filtre le navigateur appelant mais pas un appel direct
serveur-à-serveur ou en ligne de commande. La limite de taille de fichier
acceptée est en outre très large (`limits: { fileSize: 5000 * 1024 * 1024 }`,
ligne 18, soit environ 5 giga-octets par fichier), ce qui aggrave l'exposition
en l'absence de contrôle d'identité.

**Priorité** : Haute. Service exposé publiquement, sans aucun contrôle
d'accès actif, capable de stocker des fichiers volumineux.

### 3.7 Absence de tests automatisés et de pipeline CI/CD

**Constat** : aucun fichier `.spec.ts` ou `.test.ts` hors `node_modules` n'a
été trouvé dans les trois sous-projets. Aucun dossier `.github/workflows` à
la racine du dépôt.

**Risque** : aucune porte de qualité automatique ne bloque une régression ou
une vulnérabilité avant mise en production. Chaque correctif de ce plan
devra être vérifié manuellement tant que ce point reste ouvert.

**Priorité** : Haute, en tant que condition structurelle : ce point
conditionne la fiabilité de tous les correctifs futurs.

### 3.8 Correction du constat initial sur la fonctionnalité commentaires

**Constat initial (à corriger)** : il avait été noté que le contrôleur et
les routes de commentaires existaient encore côté backend alors que la
fonctionnalité aurait été retirée de l'interface `app-dance`.

**Vérification faite** : `api-dance/app/routes/comments.ts` route bien
`POST /comment/add` et `POST /comment/delete/:commentId` vers
`comment_controller.ts`, qui applique une vérification de capacité
(`bouncer.denies(canCommentPost, post)`, ligne 22) et une vérification de
propriété avant suppression (`comment.author.id !== user.id && !user.isAdmin`,
ligne 52). Côté frontend, une recherche dans `app-dance/src` montre que les
composants `AddComment.tsx` et `CommentPost.tsx`
(`app-dance/src/features/wall/components/WallPost/`) sont bien importés et
rendus par `WallPost.tsx` (lignes 3, 10, 245 et 264), lui-même utilisé dans
`app-dance/src/pages/actuPage.tsx` et
`app-dance/src/features/groups/pages/GroupDetailPage.tsx`.

**Conclusion** : le constat initial est **infirmé**. La fonctionnalité de
commentaires est active des deux côtés, pas retirée du frontend. Elle reste
néanmoins dans le périmètre de l'audit du contrôle d'accès (constat déjà
couvert par le bouncer `canCommentPost`, à revalider en section 5).

**Priorité** : Information, pas un correctif à planifier en soi.

### 3.9 Redondance entre `isAdmin` et `status`

**Fichiers** : `api-dance/app/models/user.ts` (colonne `isAdmin`, lignes 94 à
96), `api-dance/app/enums/user_status.ts` (valeur `ADMIN = 'admin'`, ligne 3),
`api-dance/app/middleware/admin_middleware.ts` (vérifie `user.isAdmin`,
ligne 17, pas `status`).

**Risque** : deux champs portent une notion d'administration qui peut
diverger : un compte pourrait avoir `status = 'admin'` sans `isAdmin = true`,
ou l'inverse, selon la façon dont il a été créé ou modifié. Le contrôle
d'accès admin ne s'appuie que sur `isAdmin` (`admin_middleware.ts`), donc le
risque immédiat est limité, mais la double source de vérité complique toute
future revue de droits et peut induire une incohérence d'affichage ou de
logique métier ailleurs dans le code.

**Priorité** : Moyenne. Risque de conception plutôt que vulnérabilité
immédiate exploitable.

### 3.10 Auto-assignation aux groupes sans validation métier

**Fichier** : `api-dance/app/controllers/group_controller.ts`, méthode
`join`, lignes 33 à 86 (route `POST /group/join`).

**Risque** : tout utilisateur authentifié peut s'attacher lui-même à
n'importe quel `groupId` existant, sans vérification de son rôle (élève,
professeur, superviseur) ni d'une règle métier d'éligibilité au groupe
ciblé. C'est un choix assumé par l'équipe selon le contexte donné, mais il
reste à garder en audit : un élève pourrait s'auto-assigner à un groupe de
niveau ou un cours qui ne le concerne pas.

**Priorité** : Basse à Moyenne, à trancher avec l'équipe produit avant de
considérer ce point comme un défaut plutôt qu'un choix fonctionnel.

### 3.11 Risque d'injection via l'extension de fichier non filtrée dans `cdn-app-dance`

**Constat nouveau, trouvé lors de la vérification du constat 3.6.**

**Fichier** : `cdn-app-dance/src/app.ts`, route `POST /upload/post`
(lignes 129 à 256) et `src/utils/sanitizer.ts`.

**Risque** : `sanitizeFilename` (lignes 3 à 16 de `sanitizer.ts`) nettoie
bien la partie nom du fichier, mais conserve l'extension telle quelle
(`return \`${safeName}${ext.toLowerCase()}\``, ligne 15). Dans la route
`/upload/post`, l'extension utilisée pour construire les chemins passés à
`exec()` (ffmpeg) vient de `p.extname(file.originalname)`
(ligne 200), donc du nom de fichier fourni par l'appelant, et n'est filtrée
par aucune liste blanche d'extensions autorisées. Cette valeur est ensuite
interpolée directement dans une chaîne de commande shell
(`exec(\`ffmpeg -i "${targetPath}" ... "${compressedPath}"\`)`, lignes 213,
217 à 219, 229 à 234). Un nom de fichier construit pour placer une
substitution de commande shell juste avant l'extension (par exemple un nom
se terminant par une séquence interprétable par le shell) atteindrait cette
interpolation sans être filtré par `sanitizeFilename`, qui ne s'applique
qu'au nom, pas à l'extension. Ce point est à confirmer par un test
d'injection dédié (voir DAST/SAST section 4) avant d'être traité comme
exploité, mais le chemin de données est vérifié dans le code tel quel.

**Priorité** : Haute. Exécution de commande shell sur le serveur d'upload à
partir d'une valeur en partie contrôlée par l'appelant.

### 3.12 Absence de limitation de débit sur la réinitialisation de mot de passe

**Fichier** : `api-dance/app/controllers/password_reset_controller.ts`.

**Risque** : contrairement à la route de connexion, qui utilise
`getLoginLimiter()` (`auth_controller.ts`, ligne 158, avec le paquet
`@adonisjs/limiter` présent dans `api-dance/package.json`), aucune recherche
du mot "limiter" ne retourne de résultat dans
`password_reset_controller.ts`. La route d'envoi de lien de réinitialisation
n'a donc pas de limite de fréquence constatée, ce qui ouvre un risque
d'énumération d'emails ou de sollicitation abusive du service d'envoi
d'email.

**Priorité** : Moyenne.

### 3.13 Absence de `@adonisjs/shield` (protection CSRF)

**Fichier** : `api-dance/package.json` (aucune entrée `@adonisjs/shield`
trouvée par recherche), pas de fichier `config/shield.ts`.

**Contexte** : l'authentification de l'API repose sur un guard par jeton
(`api-dance/config/auth.ts`, `tokensGuard`, ligne 6), pas sur une session par
cookie pour les routes protégées, ce qui réduit fortement la surface CSRF
classique. Le fichier `api-dance/config/app.ts` configure malgré tout un
cookie HTTP (lignes 32 à 39, `sameSite: 'lax'`), dont l'usage réel reste
`[à vérifier]` avant de conclure que le CSRF est hors sujet sur ce dépôt.

**Priorité** : Basse, sous réserve de la vérification ci-dessus.

### 3.14 `.gitignore` de `cdn-app-dance` sans exclusion de `.env`

**Fichier** : `cdn-app-dance/.gitignore` (contenu : `/dist/`, `/uploads/`,
`/node_modules/`, aucune ligne `.env`), à comparer avec le `.gitignore`
racine et ceux de `api-dance` et `app-dance`, qui excluent bien `.env` et
`.env.*`.

**Risque** : aucun fichier `.env` n'est actuellement suivi par Git dans
`cdn-app-dance` (vérifié par `git ls-files`), donc aucune fuite actuelle.
Le risque est latent : si un `.env` est introduit plus tard dans ce
sous-projet sans corriger d'abord le `.gitignore`, il serait committé par
défaut.

**Priorité** : Basse aujourd'hui, corrective avant tout ajout de secret dans
ce sous-projet.

## 4. Outils par catégorie

| Catégorie | Outil retenu | Justification |
|---|---|---|
| SAST principal | CodeQL | Intégration native GitHub Actions, gratuit sur dépôt public ou avec licence GitHub Advanced Security, analyse de flux de données profonde pour JavaScript/TypeScript, pertinent pour tracer le chemin de la donnée du constat 3.11. |
| SAST complémentaire | Semgrep | Règles personnalisables, utile pour cibler des motifs spécifiques à Lucid ORM ou VineJS, exécution locale rapide avant push. |
| Audit de dépendances | Dependabot (natif GitHub) + `npm audit --audit-level=high` en porte CI | Zéro infrastructure à maintenir, alerte automatique sur nouvelle CVE, cohérent avec les trois `package.json` indépendants. |
| Détection de secrets | Gitleaks (CI) + GitHub Secret Scanning natif | Double filet : un scan explicite versionné dans le pipeline, un filet natif côté plateforme, pertinent vu la lacune constatée en 3.14. |
| Revue de configuration manuelle | Lecture directe + checklist OWASP ASVS par chapitre (V4 Contrôle d'accès, V6 Cryptographie stockée, V13 API) | Pas d'outil automatique fiable pour juger si un contrôle d'accès porte sur la bonne propriété métier (constat 3.1) : la lecture humaine reste nécessaire. |
| DAST | OWASP ZAP (Baseline Scan puis Full Scan) | Voir benchmark section 5. |

## 5. Benchmark d'outils

### 5.1 Choix de l'outil SAST

| Option | Intégration CI native GitHub | Couverture JS/TS | Coût | Niveau de preuve |
|---|---|---|---|---|
| CodeQL | Native (`github/codeql-action`) | Analyse de flux de données inter-procédurale | Gratuit sur ce type de dépôt avec GitHub Advanced Security | L2 (documentation produit officielle GitHub) |
| Semgrep | Action GitHub tierce, configuration simple | Analyse par motifs, règles communautaires et personnalisées | Gratuit en usage standard (CLI open source) | L2 (documentation produit officielle) |
| ESLint + plugins sécurité (`eslint-plugin-security`) | Déjà présent si ESLint est configuré dans le projet | Détection de motifs à risque limités (pas d'analyse de flux) | Gratuit | L3 (consensus technique large, pas un standard formel) |

**Recommandation** : CodeQL en outil principal, Semgrep en complément ciblé.
CodeQL est le seul des trois à tracer un flux de données de bout en bout
(pertinent pour confirmer ou infirmer le constat 3.11), et s'intègre sans
service tiers dans GitHub Actions. Semgrep comble sa limite principale :
des règles rapides à écrire pour des motifs propres à ce dépôt (usage de
`.rawQuery()`, absence de validateur VineJS en tête de contrôleur).
ESLint + plugin sécurité reste utile en filet local avant commit, mais ne
remplace pas une analyse de flux.

### 5.2 Choix de l'outil DAST

| Option | Coût | Scriptable en CI | Couverture attendue sur ce périmètre | Niveau de preuve |
|---|---|---|---|---|
| OWASP ZAP (Baseline puis Full Scan) | Gratuit | Oui (`zaproxy/action-baseline`, `zaproxy/action-full-scan`) | Bonne sur les endpoints HTTP classiques ; ne couvre pas nativement le canal socket.io (constat 3.2) | L2 (projet officiel OWASP, recommandation directe de la fondation) |
| Nikto | Gratuit | Oui, mais moins maintenu pour les API modernes | Orienté serveur web générique, peu adapté à une API JSON/TypeScript | L3 (consensus technique large, outil plus ancien) |
| Burp Suite (édition communautaire) | Gratuit en usage manuel, payant pour l'automatisation CI | Non automatisable en CI dans l'édition gratuite | Bonne en usage manuel ponctuel, mais pas dans un pipeline continu | L2 (documentation produit officielle) |

**Recommandation** : OWASP ZAP. C'est le seul des trois à combiner gratuité
et automatisation CI réelle, ce qui correspond à la contrainte de ce projet
(pas de budget outillage, équipe de trois personnes). Sa limite connue, la
non-couverture native du canal socket.io, doit être compensée par un test
manuel ciblé sur la messagerie (constat 3.2) : point à inscrire dans les
prochaines étapes.

## 6. Prochaines étapes concrètes

Ordre de priorité, du plus urgent au moins urgent :

1. Corriger le contrôle d'accès de `addSupervisor` (constat 3.1) : vérifier
   que l'appelant est déjà superviseur de l'enfant ciblé avant d'envoyer une
   invitation. Correctif le plus direct sur la protection des mineurs.
2. Confirmer ou infirmer le risque d'injection de `cdn-app-dance`
   (constat 3.11) par un test dédié (nom de fichier construit pour la
   preuve), puis filtrer l'extension par liste blanche avant tout usage dans
   `exec()`.
3. Mettre en place une authentification minimale sur `cdn-app-dance`
   (constat 3.6), au moins un jeton partagé entre les services internes, en
   priorité sur les routes d'upload.
4. Aligner la configuration CORS du canal socket.io sur celle de l'API HTTP
   (constat 3.2).
5. Chiffrer au niveau champ les colonnes personnelles des mineurs
   (constat 3.4), avec l'aide de `APP_KEY` déjà disponible.
6. Mettre en place le pipeline CI/CD minimal (lint, vérification de types,
   `npm audit`, CodeQL) sur les trois sous-projets (constat 3.7), condition
   de fiabilité de tous les correctifs suivants.
7. Ajouter Gitleaks en CI et corriger le `.gitignore` de `cdn-app-dance`
   (constat 3.14).
8. Ajouter une limitation de débit sur la réinitialisation de mot de passe
   (constat 3.12).
9. Revoir les paramètres scrypt ou migrer vers Argon2id (constat 3.3),
   après mesure de la capacité réelle du serveur de production.
10. Trancher avec l'équipe produit le statut du constat 3.10
    (auto-assignation aux groupes) : défaut à corriger ou choix
    fonctionnel à documenter.
11. Nettoyer le code mort (`channels_controller.ts`, `config/transmit.ts`,
    constat 3.5) et clarifier la redondance `isAdmin`/`status`
    (constat 3.9).
12. Une fois un environnement de test disponible, lancer le premier
    Baseline Scan OWASP ZAP contre `api-dance`, puis un test manuel ciblé
    sur le canal socket.io.
