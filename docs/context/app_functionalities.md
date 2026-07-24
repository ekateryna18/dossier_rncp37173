# Contexte fonctionnel — Centre Art & Danse

Version consolidée de `docs/context/app_functionalities.txt`, complétée par l'interview du 2026-07-17 et vérifiée contre le code réel de `api-dance` là où c'était possible (comportement des notifications, cycle de vie d'un compte). Les points encore ouverts sont listés en fin de document plutôt que devinés.

## Vue d'ensemble

Application de gestion d'une école de danse (« Centre Art & Danse »). Le dirigeant de l'école est administrateur de l'app, il ajoute professeurs, parents et élèves. L'app structure la communication interne : posts par groupe ou globaux, messagerie privée, suivi du parcours des enfants par les parents. C'est une PWA : le site s'installe comme une app mobile et peut envoyer des notifications push.

## Rôles

| Rôle | Description |
|---|---|
| `PROFESSOR` | Peut créer des posts, s'auto-assigner ou se retirer d'un groupe. |
| `ADMIN` | Tous les droits + page admin, peut supprimer des posts. |
| `SUPERVISOR` | Parent d'un ou plusieurs élèves. |
| `STUDENT` | Le rôle le plus limité. Peut modifier ses cours/groupes suivis. À partir de 15 ans, peut créer son propre compte ; en dessous, c'est un parent qui crée le compte de son enfant. |
| `STUDENT_SUPERVISOR` | Cumul `SUPERVISOR` + `STUDENT` (un élève qui est aussi parent d'un autre élève). |

Il n'existe pas d'autre combinaison de rôles que `STUDENT_SUPERVISOR` — un professeur ne peut pas être également superviseur dans le modèle actuel, par exemple.

Un point de vigilance fonctionnel confirmé : l'auto-assignation à un groupe (côté professeur) et la modification des groupes suivis (côté élève) se font en libre-service, sans validation d'un tiers (professeur, admin ou parent). C'est un choix assumé du fonctionnement actuel, pas un oubli à corriger dans ce document — mais c'est un point à garder en tête pour tout audit de sécurité ou de cohérence des données (aucun contrôle métier n'empêche un élève de rejoindre un groupe qui ne le concerne pas).

## Cycle de vie d'un compte

Confirmé par le code (`api-dance/app/models/user.ts`, `admin_controller.ts`) :

1. **Création** : soit auto-inscription (élève de 15 ans ou plus), soit création directe par un parent (pour un enfant de moins de 15 ans) ou par l'admin. Les comptes créés directement par un parent ou par l'admin sont activés (`enabled = true`) sans passer par la validation email/admin. Un compte auto-inscrit démarre avec `hasValidEmail = false` et `enabled = false`.
2. **Validation de l'email** : un lien envoyé par email, valide une heure. Passé ce délai, l'utilisateur ne peut pas le régénérer lui-même — seul l'admin peut renvoyer un nouveau lien (action « resend email to validate account »).
3. **Validation par l'admin** : une fois l'email confirmé (`hasValidEmail = true`), le compte reste `enabled = false` jusqu'à validation manuelle par l'admin (écran « Validation », action « validate account »). Tout compte auto-inscrit passe par cette double étape, sans exception, y compris un élève de 15 ans ou plus.
4. **Suspension (« surpress »)** : l'admin repasse `enabled` à `false` sur un compte existant. Les données restent en base, mais le compte n'apparaît plus comme actif (ex. exclu des conversations tant qu'il est désactivé).
5. **Suppression / bannissement** : les actions « delete user » et « ban account » désignent la même opération réelle — suppression définitive de la ligne utilisateur en base. Ce sont deux libellés pour la même action, pas deux mécanismes distincts.

## Fonctionnalités par page

### Accueil

- Tous : voir les posts (de groupe et globaux), liker un post.
- `ADMIN`, `PROFESSOR` : créer un post.
- `ADMIN` : supprimer un post.

### Groupes

- Tous : voir la liste des groupes, des professeurs, des élèves.
- `ADMIN`, `PROFESSOR` : créer un post dans un groupe, s'auto-assigner/se retirer d'un groupe.
- Tous : liker un post fait dans un groupe.

### Messagerie

- Tous : envoyer un message à n'importe qui — chat 1-à-1 exclusivement (pas de chat de groupe ; la communication de groupe passe par les posts, pas par la messagerie).

### Profil

- Tous : voir/modifier ses informations personnelles.
- `STUDENT_SUPERVISOR`, `SUPERVISOR` : lister ses enfants, modifier les informations d'un enfant, retirer un enfant de l'app.

### Connexion

- Créer un compte (si 15 ans ou plus).
- Mot de passe oublié.
- Connexion par email + mot de passe.

## Groupes et cours

Précision apportée par l'interview : un « groupe » est l'entité nommée (ex. « Jazz moderne ») ; un « cours » (`cours`) est une classe concrète rattachée à ce groupe, définie par un nom de groupe, un niveau, et une photo. Plusieurs cours partageant le même nom de groupe sont ainsi regroupés sous ce groupe. Il n'existe pas de liste fixe de catégories : c'est l'admin qui crée librement les catégories/niveaux selon les besoins de l'école, il n'y a pas de nomenclature figée à respecter en amont.

## Posts

- Un post peut être rattaché à un groupe (visible par les membres de ce groupe) ou être global (visible par tous, page Accueil).
- Seule interaction disponible actuellement : le like.
- Correction apportée lors du plan d'audit de sécurité (`docs/context/plan_audit.md`, constat 3.8) : contrairement à ce qui était noté ici auparavant, la fonctionnalité de commentaires est bien active des deux côtés. Le backend (`comment_controller.ts`, `routes/comments.ts`) applique une vérification de capacité avant ajout et une vérification de propriété avant suppression, et le frontend affiche réellement les composants de commentaires (`AddComment.tsx`, `CommentPost.tsx`) dans `WallPost.tsx`, utilisé sur la page d'accueil et sur la page de détail d'un groupe.

## Invitation de superviseur

Fonctionnalité réelle et voulue, confirmée par l'utilisatrice après relecture du code (absente des notes fonctionnelles de départ, simple oubli de documentation). Le code de `api-dance` (`auth_controller.ts`) montre un mécanisme distinct de l'ajout direct d'un enfant : un parent peut inviter par email une autre personne à devenir superviseur d'un de ses enfants (cas d'usage type : parents séparés, tuteur). Si l'email correspond à un compte existant, une notification push « Nouvelle invitation » est envoyée. L'invitation n'est effective que si la personne invitée l'accepte explicitement (`handleSupervision`, `payload.accepted`) : ce n'est pas un rattachement automatique.

**Écart de contrôle d'accès identifié dans le code** : la route `addSupervisor` (`auth_controller.ts`, ligne 560) vérifie seulement le rôle de l'appelant (professeur, superviseur, élève-superviseur), pas son lien réel avec l'enfant ciblé. Aucune vérification ne confirme que l'appelant est déjà superviseur de l'enfant pour lequel il envoie l'invitation. En connaissant l'identifiant d'un enfant, un compte ayant l'un de ces rôles peut donc inviter un tiers à devenir superviseur de cet enfant, même sans lien avec lui. Point à traiter en priorité lors de l'audit (section 6 du dossier), les données concernées étant celles de mineurs.

## Notifications push

Les déclencheurs réels, retrouvés dans le code (`web_push_service.ts` et ses appelants), plutôt que devinés :

| Déclencheur | Destinataires | Source |
|---|---|---|
| Nouveau post global | Tous les utilisateurs sauf l'auteur | `post_controller.ts` |
| Nouveau post de groupe | Membres du groupe sauf l'auteur | `post_controller.ts` |
| Nouveau message 1-à-1 | Le destinataire du message | `chat_service.ts` |
| Validation d'email par un utilisateur | Tous les admins (« Nouvelle inscription ») | `user_email_validated_listener.ts` |
| Invitation à superviser un enfant | La personne invitée, si son compte existe déjà | `auth_controller.ts` |

## Espace admin

### Utilisateurs

Statistiques, liste des utilisateurs (recherche), suppression (= bannissement, voir cycle de vie ci-dessus), création directe d'un compte par l'admin.

### Validation

Recherche, validation d'un compte (`enabled = true`), bannissement/suppression.

### Emails non vérifiés

Recherche, liste des emails non confirmés, renvoi de l'email de validation (nouveau lien, valide une heure).

### Cours

Liste de tous les cours (recherche), modification, suppression, création d'un cours (nom de groupe + niveau + photo).

## Écarts et points encore ouverts

- Le champ `isAdmin` (booléen) et le champ `status` (qui porte aussi la valeur `ADMIN`) représentent tous les deux l'adminship dans le modèle actuel — une redondance à garder en tête, pas un point bloquant pour ce document.
- Aucune validation métier n'encadre l'auto-assignation aux groupes (élèves comme professeurs) — confirmé comme choix assumé, pas une lacune de ce document.
