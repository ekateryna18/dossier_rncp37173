# Schéma de base de données — api-dance

Reconstitué à partir des 42 fichiers de migration réels (`api-dance/database/migrations/`), appliqués dans l'ordre chronologique — c'est la source de vérité, pas les modèles Lucid qui peuvent diverger. Base : MySQL (voir `config/database.ts`).

## users

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| email | string(254) | unique, nullable |
| password | string | NOT NULL |
| first_name | string | nullable |
| last_name | string | NOT NULL |
| status | string(20) | NOT NULL, défaut `'élève'` — porte le rôle (voir enum `UserStatus`) |
| is_admin | boolean | NOT NULL, défaut `false` |
| enabled | boolean | défaut `0` — compte actif/suspendu (« surpress ») |
| has_valid_email | boolean | NOT NULL, défaut `0` |
| is_created_by_admin | boolean | défaut `false` |
| phone_number | string | nullable |
| address | string | nullable |
| postal_code | string(5) | nullable |
| city | string | nullable |
| birth_date | date | (colonne créée en `timestamp`, convertie en `date` par une migration ultérieure) |
| gender | enum('male','female','unknown') | NOT NULL, défaut `'unknown'` |
| avatar_url | string | nullable |
| seen_modal | boolean | défaut `false` |
| created_at | timestamp | NOT NULL |
| updated_at | timestamp | nullable |

**Rôle (`status`)** — valeurs réelles de l'enum `UserStatus` (`api-dance/app/enums/user_status.ts`) : `professeur`, `admin`, `superviseur`, `élève`, `élève_superviseur`.

Note de cohérence : `is_admin` (booléen) et `status = 'admin'` représentent tous les deux l'adminship — un doublon déjà signalé dans `app_functionalities.md`, confirmé ici au niveau de la colonne.

## auth_access_tokens

Table standard du package `@adonisjs/auth` (tokens d'API).

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| tokenable_id | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| type | string | NOT NULL |
| name | string | nullable |
| hash | string | NOT NULL |
| abilities | text | NOT NULL |
| created_at | timestamp | |
| updated_at | timestamp | |
| last_used_at | timestamp | nullable |
| expires_at | timestamp | nullable |

## rate_limits

Table du package `@adonisjs/limiter`.

| Colonne | Type | Contraintes |
|---|---|---|
| key | string(255) | PK |
| points | integer(9) | NOT NULL, défaut `0` |
| expire | bigint unsigned | |

## tokens

Tokens applicatifs (validation email, réinitialisation de mot de passe — voir `Token.generatePasswordResetToken`).

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| user_id | integer unsigned | FK → `users.id` (CASCADE) |
| type | string | NOT NULL (ex. `VALIDATE_EMAIL`) |
| token | string(64) | NOT NULL |
| expires_at | timestamp (tz) | |
| payload | json | nullable |
| created_at | timestamp | |
| updated_at | timestamp | |

## supervisor_users

Relation parent ↔ enfant.

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| supervisor_id | integer unsigned | FK → `users.id` (CASCADE) |
| user_id | integer unsigned | FK → `users.id` (CASCADE) — l'enfant supervisé |
| created_at | timestamp | |
| updated_at | timestamp | |

Contrainte : `unique(supervisor_id, user_id)` — un superviseur ne peut pas être lié deux fois au même enfant.

## supervisor_invitations

Invitation d'un tiers à devenir superviseur d'un enfant (voir `app_functionalities.md`, fonctionnalité repérée dans le code).

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| email | string | NOT NULL |
| child_id | integer unsigned | FK → `users.id` (CASCADE) |
| created_at | timestamp (tz) | |
| updated_at | timestamp (tz) | |

Contrainte : `unique(email, child_id)` — une même adresse ne peut pas être invitée deux fois pour le même enfant.

## groups

Le groupe au sens large (ex. « Jazz moderne »).

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| name | string | NOT NULL |
| image_url | string | NOT NULL |
| level | string | nullable |
| created_at | timestamp | |
| updated_at | timestamp | |

## groups_names

Catalogue des noms de groupes déjà utilisés — sert de liste de suggestion à l'admin (endpoint `getNames`), ce n'est pas une clé étrangère stricte depuis `groups.name`.

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| name | string | NOT NULL, défaut `''`, unique |

## groups_users

Table de liaison utilisateur ↔ groupe.

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| user_id | integer unsigned | FK → `users.id` (CASCADE) |
| group_id | integer unsigned | FK → `groups.id` (CASCADE) |
| type | integer | NOT NULL, défaut `1` (voir enum `SubscribeType`) |

Contrainte : `unique(user_id, group_id)`. **`type`** — `SubscribeType.FOLLOW = 1` (un élève qui suit le groupe) ou `SubscribeType.SUBSCRIBE = 2` (un professeur assigné au groupe) : la même table couvre les deux sens d'appartenance décrits dans `app_functionalities.md`, distingués par cette seule colonne.

## posts

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| content | text | nullable (a été rendu nullable après création — probablement pour autoriser un post uniquement composé de médias) |
| author_id | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| group_id | integer unsigned | nullable, FK → `groups.id` (CASCADE) — `NULL` = post global, valeur renseignée = post de groupe |
| created_at | timestamp | |
| updated_at | timestamp | |

## medias

Fichiers attachés à un post.

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| url | string | NOT NULL |
| post_id | integer unsigned | FK → `posts.id` (CASCADE) |
| type | string | NOT NULL |
| frame | string | nullable, défaut `null` |
| created_at | timestamp | |
| updated_at | timestamp | |

## likes

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| user_id | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| post_id | integer unsigned | NOT NULL, FK → `posts.id` (CASCADE) |
| created_at | timestamp | |

Contrainte : `unique(user_id, post_id)` — un like par utilisateur et par post.

## comments

Table encore présente côté base (voir la remarque « écart » dans `app_functionalities.md` : fonctionnalité retirée de l'interface mais route/modèle encore présents).

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| content | text | NOT NULL |
| author_id | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| post_id | integer unsigned | NOT NULL, FK → `posts.id` (CASCADE) |
| parent_id | integer unsigned | nullable, FK → `comments.id` (CASCADE) — auto-référence, structure de réponses en fil |
| is_deleted | boolean | NOT NULL, défaut `false` — suppression douce |
| created_at | timestamp | |
| updated_at | timestamp | |

## conversations

Conversation 1-à-1.

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| user_id_a | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| user_id_b | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| created_at | timestamp (tz) | défaut `now()` |
| updated_at | timestamp (tz) | défaut `now()` |

Contrainte : `unique(user_id_a, user_id_b)`. Note : une colonne `is_hidden` avait été ajoutée ici puis retirée par la migration suivante — l'état « masqué » vit finalement dans `conversation_config`, par utilisateur plutôt que par conversation entière.

## conversation_config

État propre à chaque participant d'une conversation (masquage individuel).

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| user_id | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| conversation_id | integer unsigned | NOT NULL, FK → `conversations.id` (CASCADE) |
| is_hidden | boolean | défaut `false` |
| created_at | timestamp (tz) | |
| updated_at | timestamp (tz) | |

Contrainte : `unique(user_id, conversation_id)`.

## messages

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| conversation_id | integer unsigned | NOT NULL, FK → `conversations.id` (CASCADE) |
| sender_id | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| value | text | NOT NULL |
| read | boolean | défaut `false` |
| created_at | timestamp (tz) | défaut `now()` |
| updated_at | timestamp (tz) | défaut `now()` |

## subscriptions

Abonnements aux notifications push (Web Push API).

| Colonne | Type | Contraintes |
|---|---|---|
| id | integer | PK, auto-incrément |
| user_id | integer unsigned | NOT NULL, FK → `users.id` (CASCADE) |
| endpoint | text | NOT NULL, unique |
| p256dh | text | NOT NULL |
| token | text | NOT NULL |
| expiration_time | timestamp | nullable |
| created_at | timestamp | |
| updated_at | timestamp | |

## Relations — vue d'ensemble

- `users` est le pivot central : auto-relation via `supervisor_users` (parent ↔ enfant), lié à `groups` via `groups_users` (rôle porté par `type`), auteur de `posts`/`comments`/`likes`/`messages`.
- `posts` → `medias` (1-N), `posts` → `comments` (1-N, avec fil via `parent_id`), `posts` → `likes` (1-N).
- `conversations` relie deux `users` ; `messages` en dépend ; `conversation_config` porte l'état par utilisateur (masquage) sans dupliquer la conversation elle-même.
- `groups_names` est indépendante de `groups` en base (pas de clé étrangère) — c'est un simple catalogue consulté pour la saisie, pas une relation stricte.

## Écart à noter

La table `comments` et sa colonne `is_deleted` (suppression douce déjà prévue) confirment que le modèle de données du commentaire a été construit avec soin puis mis de côté côté interface — l'ensemble reste cohérent et réactivable, ce n'est pas du code laissé à moitié fini.
