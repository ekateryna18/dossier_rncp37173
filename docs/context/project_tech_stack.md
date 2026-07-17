# Stack technique du projet

Ce document décrit la stack technique réelle des 3 sous-projets du dépôt : `app-dance` (frontend), `api-dance` (backend) et `cdn-app-dance` (API d'upload/CDN). Chacun est indépendant, avec son propre `package.json`.

---

## 1. app-dance (frontend)

### Vue d'ensemble

`app-dance` est l'application cliente du projet — une PWA (Progressive Web App) construite avec React et Ionic, nommée « Centre Art & Danse » (voir `manifest.webmanifest`). Elle consomme l'API exposée par `api-dance` et s'appuie sur `cdn-app-dance` pour l'upload/la diffusion de fichiers (photos).

### Runtime et outillage de build

| Élément | Valeur |
|---|---|
| Framework UI | React 19 (`react` / `react-dom` ^19.1.0) |
| Composants mobiles/UI | Ionic React (`@ionic/react` ^8.6.3) |
| Outil de build/dev | Vite ^6.3.5, plugin `@vitejs/plugin-react` |
| Serveur de dev | `vite` (port 5173, HMR configuré en WebSocket sur `192.168.0.12:5173`, hôte autorisé `cad.server02`) |
| Build de production | `vite build` |
| Prévisualisation | `vite preview` (`preview` sur le port par défaut, `prod` sur le port 5174) |

### Style et UI

- **Framework CSS** : Bootstrap 5 (`bootstrap` ^5.3.7) et `react-bootstrap` ^2.10.10, avec `bootstrap-icons` et `react-bootstrap-icons`.
- **Préprocesseur** : Sass (`sass` ^1.89.2), configuré dans `vite.config.js` (avertissements de dépréciation Sass volontairement réduits sur `color-functions`, `import`, `global-builtin`).
- **Animations** : Framer Motion (`framer-motion` ^12.23.6).
- **Composants de données** : AG Grid (`ag-grid-react` + `@ag-grid-community/locale` ^34.1.2) pour les tableaux/grilles.
- **Composants divers** : `react-select` (listes déroulantes), `react-tooltip` (infobulles), `react-hot-toast` (notifications visuelles).

### Routage et état

- **Routage** : `react-router-dom` ^6.30.1.
- **Gestion d'état** : Zustand ^5.0.6.

### Communication temps réel et backend

- **WebSockets côté client** : `socket.io-client` ^4.8.1 — se connecte au serveur `socket.io` exposé par `api-dance`.
- **Stockage local mobile** : `@react-native-async-storage/async-storage` ^1.24.0.

### Services tiers

- **Firebase** : `firebase` ^11.10.0 (authentification et/ou services Google — la configuration précise du projet Firebase et de ses règles de sécurité est à documenter séparément, elle n'est pas déductible du seul `package.json`).

### Internationalisation et dates

- **i18n** : `i18next` + `react-i18next` (^25.5.2 / ^15.7.3).
- **Dates relatives** : `javascript-time-ago` + `react-time-ago`.

### PWA

- **Manifeste** : `manifest.json` et `manifest.webmanifest` — application installable, nom court « CAD », mode d'affichage `standalone`, orientation portrait.

### Tests et qualité

- **Lint** : ESLint (`eslint.config.js`), avec `eslint-plugin-react-hooks` et `eslint-plugin-react-refresh`, commande `npm run lint`.
- **Tests automatisés** : aucun script de test n'est défini dans `package.json` à ce stade.

### Variables d'environnement

Le préfixe `VITE_` expose une variable au code client au moment du build — aucune valeur secrète ne doit y être placée (seules des clés publiques, ex. configuration Firebase publique protégée par ses propres règles de sécurité côté service, doivent l'être). Le fichier `.env` du sous-projet est exclu du suivi Git.

### État constaté (au moment de la rédaction de ce document)

- Aucun pipeline CI/CD n'est configuré pour ce sous-projet.
- Aucune conteneurisation (pas de Dockerfile).
- Aucun test automatisé n'est présent à ce stade.

---

## 2. api-dance (backend)

### Vue d'ensemble

`api-dance` est l'API backend du projet. C'est une application AdonisJS 6 en TypeScript, responsable de la logique métier, de l'accès aux données et de l'authentification pour l'ensemble de la plateforme (app-dance en consomme les routes, cdn-app-dance lui est complémentaire pour l'upload de fichiers).

### Runtime et framework

| Élément | Valeur |
|---|---|
| Runtime | Node.js (module ESM, `"type": "module"`) |
| Langage | TypeScript (compilation via `@adonisjs/tsconfig`, vérification par `tsc --noEmit`) |
| Framework | AdonisJS 6 (`@adonisjs/core` ^6.19.1) |
| Compilateur/bundler dev | `@adonisjs/assembler` (rechargement à chaud via `hot-hook`, watch sur `app/controllers` et `app/middleware`) |
| Build | `node ace build` |
| Serveur de dev | `node ace serve --hmr` |

### Base de données et persistance

- **ORM** : Lucid (`@adonisjs/lucid` ^21.8.1).
- **Connexion active** : MySQL (client `mysql2`), configurée dans `config/database.ts`. Une configuration Postgres (`client: pg`) existe dans le même fichier mais est désactivée (commentée) — le paquet `pg` reste présent en dépendance sans être la connexion utilisée.
- **Migrations** : gérées par Lucid, chemin `database/migrations`.

### Authentification et sécurité applicative

- **Authentification** : `@adonisjs/auth` ^9.5.1.
- **Autorisation** : `@adonisjs/bouncer` ^3.1.6 (politiques d'accès, voir `app/policies` et `app/abilities`).
- **CORS** : `@adonisjs/cors` ^2.2.1, configuré dans `config/cors.ts`.
- **Limitation de débit** : `@adonisjs/limiter` ^2.4.0.
- **Validation d'entrée** : VineJS (`@vinejs/vine` ^3.0.1), providers enregistrés dans `adonisrc.ts` (`vinejs_provider`).
- **Assets statiques** : `@adonisjs/static` ^1.1.1.

Le détail des contrôles de sécurité effectivement en place (activation réelle du CSRF/shield, configuration précise du rate limiting, revue CORS) relève de l'agent dédié `@axel`, pas de ce document.

### Temps réel et notifications

- **Diffusion temps réel native AdonisJS** : `@adonisjs/transmit` ^2.0.2 (Server-Sent Events).
- **WebSockets** : `socket.io` ^4.8.1, en complément côté serveur (le client correspondant est utilisé par `app-dance`).
- **Notifications push web** : `web-push` ^3.6.7.
- **E-mail** : `@adonisjs/mail` ^9.2.2 avec transport `nodemailer` ^7.0.11.

### Documentation API et outillage divers

- **OpenAPI/Swagger** : `@foadonis/openapi` ^0.4.1 et `swagger-jsdoc` ^6.2.8 pour la documentation de l'API.
- **Dates** : `luxon` ^3.7.2.
- **Templates serveur** : `edge.js` ^6.4.0 (moteur de templates AdonisJS).
- **Utilitaires** : `camelcase-keys` (normalisation de clés), `node-fetch` (requêtes HTTP sortantes), `open-graph-scraper` (extraction de métadonnées Open Graph), `reflect-metadata` (décorateurs TypeScript).
- **Journalisation** : `pino-pretty` et `pino-roll` (rotation de logs).

### Tests et qualité

- **Framework de test** : Japa (`@japa/runner`, `@japa/assert`, `@japa/api-client`, `@japa/plugin-adonisjs`) — exécuté via `node ace test`.
- **Lint** : ESLint avec `@adonisjs/eslint-config`, commande `npm run lint`.
- **Formatage** : Prettier avec `@adonisjs/prettier-config`, commande `npm run format`.
- **Vérification de types** : `tsc --noEmit`, commande `npm run typecheck`.

### Structure des imports

Le projet utilise les imports mappés Node.js (champ `imports` du `package.json`, préfixes `#controllers`, `#models`, `#services`, `#middleware`, `#validators`, `#policies`, `#abilities`, `#events`, `#listeners`, `#mails`, `#providers`, `#database`, `#start`, `#tests`, `#config`) plutôt que des chemins relatifs profonds.

### État constaté (au moment de la rédaction de ce document)

- Aucun pipeline CI/CD n'est configuré pour ce sous-projet.
- Aucune conteneurisation (pas de Dockerfile).
- Les variables d'environnement sensibles sont exclues du suivi Git (`.env` dans `.gitignore`).

---

## 3. cdn-app-dance (API d'upload/CDN)

### Vue d'ensemble

`cdn-app-dance` (nom interne `cdn.cad.server02`) est le service dédié au dépôt et à la diffusion de fichiers (notamment les photos) pour `app.centreartetdanse.com`. C'est un service indépendant des deux autres sous-projets, avec sa propre base de code TypeScript compilée.

### Runtime et framework

| Élément | Valeur |
|---|---|
| Runtime | Node.js (module ESM, `"type": "module"`) |
| Langage | TypeScript ^5.9.2, compilé via `tsc` (cible `ES2020`, résolution `NodeNext`) |
| Framework serveur | Express 5 (`express` ^5.1.0) |
| Point d'entrée | `app.ts` (source dans `src/`, sortie compilée dans `dist/`) |

### Upload et gestion de fichiers

- **Middleware d'upload** : Multer ^2.0.2 (`multer`) — gère la réception de fichiers multipart.
- **CORS** : `cors` ^2.8.5 — la configuration précise (liste blanche d'origines ou wildcard) est à vérifier avant mise en production, ce n'est pas déductible du seul `package.json`.
- **Configuration d'environnement** : `dotenv` ^17.2.1.

### Build et exécution

| Script | Commande |
|---|---|
| `build` | `npx tsc` |
| `execute` | `npx tsc && node dist/app.js` |
| `test` | non défini (`echo "Error: no test specified" && exit 1`) |

### Tests et qualité

- Aucun test automatisé n'est présent à ce stade.
- Aucun script de lint n'est défini dans `package.json` pour ce sous-projet, contrairement à `api-dance` et `app-dance`.

### Points d'attention structurels

- Ce service, par sa fonction (réception de fichiers uploadés), concentre des risques spécifiques (validation du type réel de fichier, taille, chemin de stockage) qui relèvent de l'agent dédié `@axel` pour l'audit et le durcissement — ce document décrit la stack, pas la posture de sécurité.
- Aucun mécanisme d'authentification n'a été repéré sur ce service au niveau de la stack déclarée (`package.json`) — un point à clarifier avant de considérer les routes d'upload comme protégées.

### État constaté (au moment de la rédaction de ce document)

- Aucun pipeline CI/CD n'est configuré pour ce sous-projet.
- Aucune conteneurisation (pas de Dockerfile).
- Le fichier `.env` du sous-projet est exclu du suivi Git.

---

## Agents de référence pour ce dépôt

- `@dev` (Amelia) — implémentation, code review.
- `@architect` (Winston) — décisions d'architecture et de conception.
- `@axel` — sécurité applicative (SAST/DAST, CI/CD, durcissement de configuration).
- `@jury-rncp37173` (Cassandre) — validation de ces sous-projets comme preuve pour le dossier RNCP37173.
