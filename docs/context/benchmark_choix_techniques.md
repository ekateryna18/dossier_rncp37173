# Benchmark des choix techniques, Centre Art & Danse

Ce document compare, choix par choix, la technologie réellement utilisée dans `api-dance` / `app-dance` / `cdn-app-dance` avec des alternatives sérieuses du marché, et motive pourquoi le choix en place tient ou ne tient pas face à ces alternatives. Chaque constat de configuration cité ici est vérifié dans le code, pas supposé. Niveau de preuve indiqué par choix (`Niv`) : L1, spécification officielle ou standard reconnu (RFC, OWASP ASVS) ; L2, documentation produit officielle ou recommandation d'un organisme de référence (OWASP Cheat Sheet Series) ; L3, consensus technique large sans texte normatif unique.

---

## 1. Framework backend : AdonisJS

**Choix en place** : AdonisJS 6, TypeScript, avec ORM (Lucid), authentification (`@adonisjs/auth`), autorisation (`@adonisjs/bouncer`), validation (VineJS) et limitation de débit (`@adonisjs/limiter`) intégrés au même framework.

| Option | Modules de sécurité intégrés | Maturité de l'écosystème Node | Courbe d'apprentissage pour une équipe TypeScript | Niv |
|---|---|---|---|---|
| AdonisJS 6 (retenu) | Auth, autorisation, validation, rate limiting fournis nativement, versionnés ensemble | Écosystème plus restreint que Express, mais complet pour les besoins d'un CRUD applicatif | Structure proche de Laravel/NestJS, familière pour une équipe déjà orientée TypeScript | L2 |
| Express (nu) | Aucun, chaque brique de sécurité est une dépendance tierce choisie et maintenue séparément | Écosystème le plus large du marché Node | Minimaliste, mais tout le socle sécurité est à assembler soi-même | L3 |
| NestJS | Modules de sécurité disponibles mais souvent via des paquets tiers (Passport, class-validator) | Écosystème large, orienté entreprise | Injection de dépendances et décorateurs, courbe plus raide pour une petite équipe | L2 |
| Fastify | Plugins de sécurité disponibles (helmet, rate-limit) mais à assembler | Écosystème solide, orienté performance brute | Minimaliste comme Express, assemblage manuel du socle sécurité | L3 |

**Recommandation** : AdonisJS reste le choix le plus cohérent pour une équipe de projet de trois personnes sans expert sécurité dédié en interne : avoir l'authentification, l'autorisation, la validation et le rate limiting du même éditeur, mis à jour ensemble, réduit le risque qu'une brique de sécurité tierce prenne du retard sans que personne ne s'en aperçoive. Express ou Fastify auraient demandé d'assembler et de maintenir ce socle à la main, un coût que la taille de l'équipe ne permet pas d'absorber durablement. Ce choix a un revers déjà visible dans ce projet : une partie du socle de sécurité (CORS restreint sur l'API HTTP) est bien configurée, mais une autre (CORS ouvert sur le canal socket.io, voir section 7) ne l'est pas, ce qui montre que le cadre fourni par le framework ne suffit pas seul, il doit être appliqué de façon uniforme sur toute la surface exposée.

---

## 2. ORM et accès aux données : Lucid

**Choix en place** : Lucid (ORM natif d'AdonisJS), requêtes construites via le query builder plutôt qu'en SQL brut concaténé, migrations versionnées dans `database/migrations`.

| Option | Protection native contre l'injection SQL | Historique de schéma auditable | Coût de flexibilité (requêtes complexes) | Niv |
|---|---|---|---|---|
| Lucid (retenu) | Paramétrage automatique des requêtes du query builder | Migrations horodatées, rejouables, c'est la source utilisée pour reconstituer le schéma de ce projet | Requêtes très complexes parfois plus verbeuses qu'en SQL brut | L2 |
| SQL brut (requêtes manuelles) | Dépend entièrement de la discipline du développeur à paramétrer chaque requête | Aucun, le schéma vit uniquement dans la tête de l'équipe ou une documentation externe à maintenir à la main | Contrôle total, mais un seul oubli de paramétrage ouvre une injection SQL (OWASP Top 10, catégorie A03, Injection) | L1 |
| Prisma | Requêtes paramétrées automatiquement, schéma déclaratif dans un fichier unique | Migrations versionnées, générées à partir du schéma déclaratif | Moins intégré nativement à AdonisJS, demande une couche de compatibilité | L2 |
| TypeORM / Sequelize | Requêtes paramétrées automatiquement | Migrations versionnées | Mature mais historiquement moins actif sur AdonisJS que Lucid, qui est l'ORM de référence du framework | L3 |

**Recommandation** : Lucid reste le choix le plus défendable ici, pas seulement par confort d'intégration : le paramétrage automatique des requêtes ferme par construction la catégorie d'injection SQL la plus répandue (OWASP Top 10, A03), sur une application qui manipule des données de mineurs. Face au SQL brut, la différence n'est pas une question de style mais de surface d'erreur humaine possible à chaque requête écrite. Face à Prisma, TypeORM ou Sequelize, l'écart se joue sur l'intégration native à AdonisJS plutôt que sur la sécurité, les quatre ORM protégeant équivalemment contre l'injection par paramétrage.

---

## 3. Base de données : MySQL

**Choix en place** : MySQL, via le pilote `mysql2`, configuré dans `config/database.ts`. Une configuration PostgreSQL existe dans le même fichier mais reste désactivée.

| Option | Adéquation au modèle relationnel du projet | Coût opérationnel pour une équipe de projet de 3 personnes | Support des contraintes d'intégrité (clés étrangères en cascade) | Niv |
|---|---|---|---|---|
| MySQL (retenu) | Fort : comptes, groupes, superviseurs et posts sont fortement relationnels avec des clés étrangères en cascade | Faible, hébergement et outillage largement répandus | Complet, déjà utilisé sur les 17 tables du schéma actuel | L2 |
| PostgreSQL | Fort, équivalent à MySQL sur ce type de modèle, avec en plus des types avancés (JSONB indexé, contraintes `CHECK` plus riches) | Comparable à MySQL | Complet | L2 |
| MongoDB (document) | Faible : le modèle actuel repose sur des relations many-to-many strictes (superviseurs/enfants, utilisateurs/groupes), qui demanderaient une dénormalisation manuelle en base documentaire | Comparable | Pas de clé étrangère native, l'intégrité référentielle serait à recoder côté application | L3 |

**Recommandation** : MySQL est un choix cohérent pour ce modèle de données, mais pas parce qu'il serait supérieur à PostgreSQL dans l'absolu : les deux conviennent également bien à un schéma aussi relationnel. Le point qui mérite d'être tranché n'est pas MySQL contre PostgreSQL, c'est la configuration PostgreSQL laissée présente mais désactivée dans le code : soit elle documente une migration envisagée à retirer proprement si elle n'est plus d'actualité, soit elle doit rester à jour si une bascule reste possible. MongoDB, en revanche, n'aurait pas été un bon choix ici : le modèle de données du projet est relationnel par nature, pas documentaire.

---

## 4. Forme de l'application côté client : PWA (React + Ionic) et impact sécurité

**Choix en place** : `app-dance` est une PWA (Progressive Web App) installable, construite avec React 19 et Ionic React, plutôt qu'une application native ou un développement natif séparé par plateforme.

| Option | Distribution | Surface d'attaque spécifique | Contrôle de l'éditeur sur les mises à jour de sécurité | Niv |
|---|---|---|---|---|
| PWA (retenu) | Installation directe depuis le navigateur, sans passage par un store d'application | Le Service Worker et le cache applicatif deviennent une surface à sécuriser explicitement (données mises en cache localement, y compris potentiellement des informations sur des mineurs) | Mise à jour immédiate au prochain chargement, aucune validation d'un store tiers ne retarde un correctif de sécurité | L2 |
| Application native (Swift/Kotlin, une par plateforme) | Passage obligatoire par l'App Store / Google Play, avec leurs propres contrôles de sécurité à la soumission | Stockage natif sécurisé disponible (Keychain, Keystore), mais deux bases de code distinctes à sécuriser séparément | Délai de validation du store avant qu'un correctif de sécurité atteigne les utilisateurs | L2 |
| React Native / Flutter (cross-platform natif) | Passage par les stores, un seul code source pour les deux plateformes | Stockage natif sécurisé disponible comme en natif pur | Même délai de validation de store que le natif pur | L2 |

**Recommandation** : le choix de la PWA n'est pas neutre en sécurité, dans les deux sens. Il retire la dépendance à un store tiers pour distribuer un correctif urgent, un vrai atout pour une application qui manipule des données de mineurs et qui doit pouvoir être corrigée vite. En contrepartie, il déplace la responsabilité de sécuriser le stockage local vers le Service Worker et le cache du navigateur, un point que l'audit (section 6 du dossier) doit vérifier explicitement : ce que l'application met en cache côté client, et si des données personnelles d'enfants s'y retrouvent au-delà de ce qui est strictement nécessaire à l'usage hors ligne. Une application native aurait offert un stockage local chiffré fourni par l'OS (Keychain/Keystore), ce que le navigateur ne propose pas nativement au même niveau de garantie.

---

## 5. Hachage des mots de passe : scrypt

**Choix en place, vérifié dans `config/hash.ts`** :

```
default: 'scrypt'
scrypt: { cost: 16384, blockSize: 8, parallelization: 1, maxMemory: 33554432 }
```

Ce sont les paramètres par défaut fournis par AdonisJS pour le pilote scrypt, non retouchés pour ce projet.

| Option | Résistance au calcul massivement parallèle (GPU/ASIC) | Recommandation OWASP (Password Storage Cheat Sheet) | Statut sur ce projet | Niv |
|---|---|---|---|---|
| scrypt (retenu) | Élevée, algorithme à mémoire coûteuse conçu pour limiter le parallélisme matériel | Cité comme option acceptable, en dessous d'Argon2id dans l'ordre de préférence OWASP | En place, paramètres par défaut du framework, non audités ni ajustés au contexte de cette application | L2 |
| Argon2id | Élevée, vainqueur de la Password Hashing Competition (2015), paramétrable en mémoire, temps et parallélisme | Premier choix recommandé par OWASP pour le hachage de mot de passe | Non utilisé sur ce projet | L1 |
| bcrypt | Bonne, mais coût uniquement en temps CPU, pas en mémoire, donc plus sensible à l'accélération GPU qu'un algorithme à coût mémoire | Deuxième choix recommandé par OWASP, si Argon2id n'est pas disponible | Non utilisé sur ce projet | L2 |
| PBKDF2 | Plus faible que les trois précédents, coût purement CPU, sans composante mémoire | Dernier choix recommandé par OWASP, à réserver aux environnements contraints (ex. certification FIPS) | Non utilisé sur ce projet | L2 |

**Recommandation** : scrypt n'est pas un mauvais choix, il figure dans la liste des algorithmes acceptés par OWASP pour le stockage de mots de passe, et il est nettement préférable à un simple hachage rapide (MD5, SHA-256 seul, sans dérivation de clé lente). Mais deux points méritent d'être remontés dans l'audit de sécurité (section 6 du dossier) plutôt que d'être laissés tels quels : d'une part, Argon2id est la recommandation de premier rang d'OWASP et n'a pas été retenu ici sans qu'une raison documentée n'explique ce choix ; d'autre part, les paramètres actuels (`cost: 16384`, `maxMemory: 32 Mo`) sont ceux fournis par défaut par AdonisJS, pas des valeurs choisies après une analyse du contexte de menace de ce projet précis. Un paramètre de coût non revu n'est pas nécessairement insuffisant, mais il n'a pas non plus été validé comme suffisant.

---

## 6. Stockage des données sensibles hors mot de passe

**Constat vérifié dans le code** : la clé `APP_KEY` existe (`config/app.ts`), utilisée par le framework pour le chiffrement des cookies et la signature d'URL, mais aucun usage du module de chiffrement d'AdonisJS n'a été trouvé dans `app/` pour chiffrer une colonne applicative. Les colonnes contenant des données personnelles des mineurs (`birth_date`, `phone_number`, `address`, `postal_code`, `city`, `first_name`, `last_name`) sont stockées en clair dans la table `users`, protégées uniquement par le contrôle d'accès applicatif et par la sécurité de la base de données elle-même.

| Option | Protection si la base de données est copiée ou compromise directement | Complexité d'implémentation | Impact sur les requêtes (recherche, tri) | Niv |
|---|---|---|---|---|
| Colonnes en clair (situation actuelle) | Aucune, toute donnée est lisible dès l'accès à la base | Nulle, c'est l'état par défaut d'une colonne SQL classique | Aucun, recherche et tri natifs | L3 |
| Chiffrement au niveau champ (colonnes sensibles chiffrées avec la clé applicative) | Les colonnes chiffrées restent illisibles sans la clé applicative, même en cas de copie de la base seule | Moyenne, chiffrement/déchiffrement à chaque lecture/écriture, gestion de la clé à part de la base | Recherche et tri natifs perdus sur les colonnes chiffrées, à contourner par des colonnes dérivées ou un index séparé | L2 |
| Chiffrement au niveau disque uniquement (chiffrement du volume de la base) | Protège contre le vol physique du support, pas contre un accès applicatif ou un identifiant de base compromis | Faible, géré par l'hébergeur ou le système de fichiers, sans changement de code | Aucun impact sur les requêtes | L2 |

**Recommandation** : le chiffrement au niveau champ n'est pas systématiquement le bon choix pour toutes les colonnes, il ajoute de la complexité et casse la recherche native. Mais pour ce projet précis, où les données concernent en majorité des mineurs, il mérite d'être évalué au moins sur les colonnes les plus sensibles (date de naissance, adresse), en complément du chiffrement de disque qui protège un scénario différent (vol du support physique) et ne dispense pas d'un chiffrement applicatif si l'objectif est de résister aussi à une fuite d'identifiants de connexion à la base. Ce point est à traiter dans le plan de sécurisation (section 9 du dossier), pas comme une correction immédiate isolée : il a un impact sur le modèle de données et sur les requêtes existantes.

---

## 7. Communication en temps réel : socket.io retenu face à Transmit

**Constat déjà établi dans le dossier (section 5.2.6)** : socket.io est le seul canal temps réel réellement utilisé (messagerie privée), Transmit est installé mais son seul point d'usage n'est relié à aucune route. Point supplémentaire vérifié ici : le serveur socket.io est configuré avec `cors: { origin: '*' }` (`start/ws.ts`), alors que l'API HTTP principale, elle, restreint ses origines à une liste explicite (`config/cors.ts` : domaines de l'application et de préproduction uniquement). Le cadre de sécurité existe donc dans ce projet, il n'a simplement pas été appliqué de façon uniforme sur le canal temps réel.

| Option | Origine autorisée | Cohérence avec la politique CORS déjà en place sur l'API HTTP | Niv |
|---|---|---|---|
| Configuration actuelle du canal socket.io | `*`, n'importe quelle origine | Incohérente avec la liste blanche déjà utilisée sur l'API HTTP | L3 |
| Alignement sur la même liste blanche que `config/cors.ts` | Domaines de l'application et de préproduction uniquement | Cohérente, un seul standard de configuration CORS pour toute la plateforme | L2 |

**Recommandation** : restreindre l'origine du serveur socket.io à la même liste que l'API HTTP (déjà proposé comme US-08 dans le backlog du dossier, section 4.6). Ce n'est pas un arbitrage entre deux approches équivalentes, la configuration actuelle est une incohérence par rapport à un standard déjà appliqué ailleurs dans le même projet, pas un choix technique à débattre.

---

## 8. Authentification : jetons d'accès AdonisJS plutôt que JWT ou sessions serveur

**Choix en place** : `@adonisjs/auth`, jetons d'accès opaques stockés en base (table `auth_access_tokens`), plutôt que des JWT auto-portants ou des sessions serveur classiques.

| Option | Révocation immédiate d'un accès compromis | Charge portée par le serveur | Donnée exposée si le jeton est intercepté | Niv |
|---|---|---|---|---|
| Jetons d'accès en base (retenu) | Immédiate, il suffit de supprimer la ligne correspondante en base | Une vérification en base à chaque requête authentifiée | Le jeton seul ne révèle aucune information sur l'utilisateur | L2 |
| JWT auto-porté (sans vérification en base) | Impossible avant expiration naturelle du jeton, sauf liste de révocation supplémentaire à maintenir | Aucune vérification en base nécessaire, le jeton se suffit à lui-même | Le contenu du jeton (payload) est lisible par quiconque l'intercepte, sauf chiffrement supplémentaire | L1 |
| Session serveur classique (cookie + état en mémoire ou Redis) | Immédiate | Un état de session à maintenir et partager si plusieurs instances du serveur tournent en parallèle | Le cookie de session seul ne révèle aucune information | L2 |

**Recommandation** : les jetons d'accès stockés en base sont le choix le plus adapté ici, précisément parce qu'ils permettent une révocation immédiate, un point non négociable pour un compte lié à un enfant en cas de compromission (perte de téléphone d'un parent, par exemple). Un JWT auto-porté aurait été plus léger pour le serveur, mais au prix de ne pas pouvoir couper l'accès d'un jeton volé avant son expiration, sauf à reconstruire une liste de révocation, ce qui revient à recréer la vérification en base que ce choix évite justement.

---

## 9. Upload et diffusion de fichiers : service dédié (`cdn-app-dance`) plutôt qu'un stockage objet cloud

**Choix en place** : un service Express/Multer séparé (`cdn-app-dance`), hébergé en interne, plutôt qu'un service de stockage objet géré (S3, Cloudinary, Google Cloud Storage) ou un stockage intégré directement à `api-dance`.

| Option | Isolation de la surface d'attaque liée à l'upload | Contrôle d'accès natif au service | Coût d'exploitation | Niv |
|---|---|---|---|---|
| Service dédié interne (retenu) | Bonne, séparé du reste de la logique métier | Aucun mécanisme d'authentification repéré sur ce service à ce jour (déjà noté en section 5.2.5 du dossier) | Hébergement et maintenance à la charge de VNWeb | L3 |
| Stockage objet cloud géré (S3 et équivalents) | Bonne, le fournisseur gère l'isolation et le durcissement de la brique de stockage | Contrôle d'accès fin natif (politiques par bucket, URL signées à durée limitée) | Facturation à l'usage, pas d'hébergement à maintenir soi-même | L2 |
| Stockage intégré à `api-dance` | Faible, une faille sur l'upload s'exécute dans le même processus que le reste de la logique métier et de l'accès à la base de données | Hérite du contrôle d'accès déjà en place sur `api-dance` | Aucun service supplémentaire à héberger | L3 |

**Recommandation** : séparer le service d'upload du reste de l'API reste une bonne décision d'isolation, elle est confirmée par les bonnes pratiques de réduction de surface d'attaque. Mais le choix d'un service interne plutôt qu'un stockage objet cloud managé laisse à la charge de VNWeb la responsabilité de sécuriser lui-même ce service, ce qui n'est pas encore fait puisqu'aucune authentification n'y a été repérée. Un service cloud managé aurait fourni ce contrôle d'accès nativement (URL signées, politiques de bucket), au prix d'une dépendance à un fournisseur tiers et d'une facturation à l'usage. Ce n'est pas un point à trancher immédiatement dans ce document, mais un arbitrage à documenter dans le plan de sécurisation (section 9 du dossier) : durcir le service actuel, ou migrer vers un stockage géré.

---

## Sources citées

OWASP Top 10 (catégorie A03:2021, Injection) · OWASP Password Storage Cheat Sheet (Password Hashing Competition, Argon2id/bcrypt/scrypt/PBKDF2) · OWASP ASVS (contrôle d'accès et gestion de session) · Documentation officielle AdonisJS (`config/hash.ts`, `@adonisjs/auth`, `@adonisjs/cors`) · Code source vérifié de ce dépôt (`api-dance/config/*.ts`, `api-dance/app/models/user.ts`, `api-dance/start/ws.ts`).
