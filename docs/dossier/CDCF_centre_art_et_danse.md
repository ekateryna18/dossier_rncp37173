# Cahier des Charges Fonctionnel (CDCF) : Centre Art & Danse

> **Objet** : application web et mobile installable (PWA) de communication et de gestion de l'école de danse « Centre Art & Danse », destinée à la direction, aux professeurs, aux parents et aux élèves.
> **Version** : 1.0
> **Date** : 13/11/2026
> **Rédigé par** : Ecaterina Munteanu, développeuse, VNWeb

---

## 1. Présentation du projet

### 1.1 Contexte
« Centre Art & Danse » est une école de danse. Son dirigeant, également administrateur de l'application, ajoute et gère les professeurs, les parents et les élèves. L'application est développée et maintenue par l'agence VNWeb (Antibes) pour le compte de la direction de l'école.

L'application est en production et utilisée au quotidien. Ce document décrit son comportement attendu et les évolutions prévues. Elle se compose d'une interface installable sur téléphone, d'un serveur applicatif et d'un service de dépôt et de diffusion des fichiers.

### 1.2 Problématique
Avant l'application, l'école communiquait en écrivant à chaque famille individuellement. L'application répond à trois difficultés :
- diffuser une information à un groupe de danse ou à toute l'école sans écrire à chacun ;
- permettre aux parents de suivre les cours de leurs enfants ;
- centraliser la gestion des comptes (professeurs, parents, élèves) par la direction.

### 1.3 Objectifs
- **Métier** : centraliser la communication interne de l'école et la gestion de ses utilisateurs.
- **Utilisateur** : permettre aux professeurs de publier une seule fois pour un groupe, et aux parents de suivre les groupes et cours de leurs enfants.
- **Opérationnel** : donner à la direction la maîtrise des comptes (validation, suspension, suppression) et des cours.
- **Protection** : protéger les données des familles, en grande majorité celles de mineurs.

### 1.4 Périmètre
**Inclus**
- Comptes, rôles et cycle de vie des comptes.
- Supervision des enfants par leurs parents, invitation d'un second superviseur.
- Groupes et cours.
- Publications, mentions « j'aime », commentaires.
- Messagerie privée entre deux personnes.
- Notifications push et courriels automatiques.
- Dépôt et diffusion de photos et de vidéos.
- Espace d'administration.

**Hors périmètre**
- Messagerie de groupe (la communication de groupe passe par les publications).
- Paiement, facturation, planning des cours, gestion de la présence.
- Compte autonome pour un enfant de moins de 15 ans (son profil est géré par un parent).
- Accès sans compte : seules les pages de connexion, d'inscription et de mot de passe oublié sont publiques.

### 1.5 Définitions
| Terme | Définition |
|---|---|
| PWA | Application web installable sur un téléphone comme une application, sans passer par une boutique d'applications |
| Notification push | Message reçu sur l'appareil même lorsque l'application est fermée |
| Superviseur | Parent ou tuteur lié à un ou plusieurs élèves |
| Groupe | Ensemble de cours portant le même nom (exemple : « Jazz moderne ») |
| Cours | Classe rattachée à un groupe, définie par un nom de groupe, un niveau et une photo |
| Publication | Message publié par un professeur ou l'administrateur, pour toute l'école ou pour un groupe |
| Suspension | Désactivation d'un compte sans suppression de ses données |
| RGPD | Règlement général sur la protection des données |
| WCAG | Règles internationales d'accessibilité des contenus web |

## 2. Utilisateurs et parties prenantes

### 2.1 Rôles
La majorité des utilisateurs sont des enfants : les élèves de 15 ans ou plus ont un compte propre, ceux de moins de 15 ans sont gérés par un parent.

| Rôle | Description | Droits principaux |
|---|---|---|
| Administrateur | Dirigeant de l'école | Tous les droits : espace d'administration, création et suppression de publications |
| Professeur | Enseignant | Publier, s'assigner à un groupe ou s'en retirer, commenter, messagerie |
| Superviseur | Parent ou tuteur d'un ou plusieurs élèves | Gérer ses enfants, inviter un second superviseur, consulter, aimer, commenter, messagerie |
| Élève | Élève de l'école, compte propre à partir de 15 ans | Choisir ses groupes, consulter, aimer, commenter, messagerie |
| Élève-superviseur | Élève qui est aussi parent d'un autre élève | Droits cumulés de l'élève et du superviseur |

Un professeur ne peut pas être également superviseur. Le détail des droits figure dans la matrice en annexe.

### 2.2 Parties prenantes
| Partie prenante | Responsabilité |
|---|---|
| Direction de l'école | Commanditaire et référent métier : exprime les besoins, valide les fonctionnalités et la recette |
| Vincent Nilles, dirigeant de VNWeb | Maître d'apprentissage : valide les orientations techniques et les priorités |
| Ecaterina Munteanu, VNWeb | Développeuse en charge du projet |
| Professeurs, parents, élèves | Utilisateurs finaux |

## 3. Fonctionnalités

### 3.1 Comptes et accès
| ID | Fonctionnalité | Comportement attendu | Acteurs |
|---|---|---|---|
| F-01 | Inscription | La personne saisit son adresse courriel, son nom, son prénom, sa date de naissance, un mot de passe et son type de compte. Le compte est créé inactif et un courriel de vérification est envoyé. Refus si moins de 15 ans, si l'adresse est déjà utilisée ou si les deux mots de passe diffèrent. | Élève de 15 ans ou plus, adulte |
| F-02 | Validation de l'adresse courriel | La personne ouvre le lien reçu (valable une heure) ; l'adresse est validée et les administrateurs sont notifiés. Un lien expiré est refusé ; un nouveau lien peut être demandé s'il n'en existe pas déjà un valable. | Tous |
| F-03 | Connexion et déconnexion | Connexion par adresse et mot de passe ; la session dure un jour. Refus si les identifiants sont faux, si le compte est inactif ou si l'adresse n'est pas validée. Trois échecs en une minute bloquent les tentatives pendant vingt minutes. | Tous |
| F-04 | Mot de passe oublié | Un lien de réinitialisation valable une heure est envoyé à l'adresse du compte. La personne choisit un nouveau mot de passe et peut déconnecter ses autres appareils. La réponse est identique que l'adresse existe ou non. | Tous |
| F-05 | Profil | Consultation et modification des informations personnelles et de la photo. Un changement d'adresse courriel n'est effectif qu'après clic sur un lien envoyé à la nouvelle adresse. | Tous |

### 3.2 Supervision des enfants
| ID | Fonctionnalité | Comportement attendu | Acteurs |
|---|---|---|---|
| F-06 | Gestion des enfants | Le superviseur ajoute un enfant (nom, prénom, date de naissance, adresse facultative, groupes), modifie ses informations et ses groupes, ou le retire. Un enfant sans adresse est actif immédiatement ; avec une adresse, il doit la valider. Le retrait supprime définitivement le compte de l'enfant, après confirmation. Seul un superviseur lié à l'enfant peut agir sur lui. | Superviseur, élève-superviseur |
| F-07 | Invitation d'un second superviseur | Le superviseur saisit une ou plusieurs adresses. La personne invitée reçoit une notification push si elle a un compte, un courriel sinon. Le lien de supervision n'est créé que si elle accepte depuis son profil ; un refus supprime l'invitation. | Superviseur, élève-superviseur, personne invitée |

### 3.3 Groupes et cours
| ID | Fonctionnalité | Comportement attendu | Acteurs |
|---|---|---|---|
| F-08 | Consultation des groupes | Liste des groupes, des professeurs et des élèves ; page de détail d'un groupe avec ses publications. | Tous |
| F-09 | Rejoindre ou quitter un groupe | Un professeur s'assigne à un groupe ou s'en retire ; un élève choisit les groupes qu'il suit. L'action est immédiate, sans validation d'un tiers. | Professeur, administrateur, élève, superviseur pour ses enfants |

### 3.4 Publications
| ID | Fonctionnalité | Comportement attendu | Acteurs |
|---|---|---|---|
| F-10 | Publier | Publication d'un texte et de dix photos ou vidéos au plus, pour toute l'école ou pour un groupe. Les destinataires sont notifiés. Refus si le rôle n'est pas autorisé ou si un fichier dépasse les limites. | Administrateur, professeur |
| F-11 | Fil et mentions « j'aime » | Le fil affiche les publications générales et celles des groupes de l'utilisateur. Un clic sur « j'aime » ajoute la mention ; un second clic la retire. | Tous |
| F-12 | Commentaires | Ajout de commentaires et de réponses sous une publication. Sur une publication de groupe, seuls les membres du groupe et les administrateurs commentent. Un commentaire supprimé par son auteur ou l'administrateur est masqué. | Tous |
| F-13 | Suppression d'une publication | L'auteur ou l'administrateur supprime une publication après confirmation ; elle disparaît pour tous. | Administrateur, auteur |

### 3.5 Messagerie et notifications
| ID | Fonctionnalité | Comportement attendu | Acteurs |
|---|---|---|---|
| F-14 | Messagerie privée | Conversations entre deux personnes, en temps réel, avec indication de saisie et de lecture. Le destinataire reçoit une notification. Chacun peut masquer une conversation pour lui seul. Un compte suspendu n'apparaît plus dans la liste des destinataires. | Tous |
| F-15 | Notifications push | L'utilisateur autorise les notifications sur son appareil et les reçoit pour les événements de la section 9, sauf pour ses propres actions. | Tous |

### 3.6 Administration et médias
| ID | Fonctionnalité | Comportement attendu | Acteurs |
|---|---|---|---|
| F-16 | Gestion des utilisateurs | Statistiques, recherche, création, modification, suspension et suppression des comptes. Un compte créé par l'administrateur reçoit un courriel de vérification. La suppression efface définitivement le compte et ses données ; les enfants d'un parent supprimé sont conservés et l'administrateur est averti de ceux restés sans superviseur. | Administrateur |
| F-17 | Validation des inscriptions | L'administrateur valide ou refuse les comptes inscrits dont l'adresse est validée, et renvoie un lien aux adresses non vérifiées. Un compte refusé est supprimé. | Administrateur |
| F-18 | Gestion des cours | Création, modification et suppression des cours (nom de groupe, niveau, photo). Les cours de même nom sont regroupés sous un même groupe. | Administrateur |
| F-19 | Dépôt de médias | Dépôt de photos de cours, de photos de profil et de médias de publications, dans les formats et tailles de l'exigence Médias (ENF-12). Les photos sont converties et les vidéos compressées pour la diffusion. | Utilisateurs connectés |

## 4. Parcours utilisateurs et navigation

### 4.1 Parcours principaux
**Inscription d'un élève de 15 ans ou plus** (élève) : inscription → ouverture du lien reçu par courriel → validation du compte par l'administrateur → connexion et arrivée sur l'accueil. Si le lien a expiré, l'élève en demande un nouveau.

**Création du compte d'un enfant** (superviseur) : profil, section enfants → saisie des informations et des groupes → l'enfant apparaît dans la liste du parent. Si l'enfant a une adresse, il reçoit un courriel pour la valider puis choisir son mot de passe.

**Invitation d'un second superviseur** (superviseur, puis personne invitée) : saisie de l'adresse de la personne → notification ou courriel → la personne se connecte ou crée son compte → elle accepte l'invitation depuis son profil et voit l'enfant dans sa liste.

**Publication dans un groupe** (professeur) : assignation au groupe si nécessaire → rédaction de la publication et ajout de médias → les membres du groupe sont notifiés et voient la publication.

**Échange de messages** (tout utilisateur) : messagerie → choix d'un destinataire → envoi → le destinataire reçoit le message et une notification. Une conversation masquée réapparaît à l'arrivée d'un nouveau message.

### 4.2 Navigation
Cinq entrées principales : Accueil (fil des publications), Groupes, Messagerie, Profil et, pour l'administrateur seulement, l'espace d'administration (Utilisateurs, Validation, Courriels non vérifiés, Cours). La page de connexion, qui regroupe connexion, inscription et mot de passe oublié, est la seule accessible sans compte. La recherche est disponible dans l'espace d'administration.

### 4.3 États particuliers
- Liste vide : un message explicite remplace la liste (exemple : « Soyez le premier à commenter »).
- Chargement : un indicateur s'affiche pendant l'attente.
- Succès et erreur : un message de confirmation ou d'erreur compréhensible, en français, s'affiche après chaque action.
- Absence de connexion : l'utilisateur est informé que l'action n'a pas abouti.
- Session expirée : au bout d'un jour, l'utilisateur est redirigé vers la page de connexion.

## 5. Interface utilisateur

### 5.1 Écrans
| Écran | Objectif | Utilisateurs |
|---|---|---|
| Connexion | Connexion, inscription, mot de passe oublié, validation d'un lien | Personnes non connectées |
| Nouveau mot de passe | Choisir un mot de passe à partir d'un lien reçu | Tous |
| Accueil | Fil des publications générales et de ses groupes | Tous |
| Groupes | Groupes, professeurs, élèves, détail et publications d'un groupe | Tous |
| Messagerie | Conversations privées | Tous |
| Profil | Informations personnelles, enfants, invitations, notifications | Tous (enfants : superviseurs) |
| Administration : utilisateurs | Statistiques, recherche, création, modification, suspension, suppression | Administrateur |
| Administration : validation | Valider ou refuser les comptes en attente | Administrateur |
| Administration : courriels non vérifiés | Lister les adresses non vérifiées et renvoyer un lien | Administrateur |
| Administration : cours | Créer, modifier, supprimer les cours | Administrateur |

### 5.2 Comportements et identité visuelle
- Formulaires : contrôle des champs avant envoi ; confirmation du mot de passe à l'inscription ; confirmation demandée avant toute suppression.
- Messages : en français ; les erreurs renvoyées par le serveur sont traduites en messages compréhensibles.
- Adaptation : application conçue d'abord pour le téléphone, en plein écran et en orientation portrait ; utilisable sur tablette et ordinateur.
- Accessibilité : voir l'exigence Accessibilité (ENF-06).
- Identité visuelle : nom court « CAD », logo horizontal de l'école dans l'application et dans les courriels, présentation cohérente d'un écran à l'autre.

## 6. Règles de gestion

| ID | Règle | Fonctionnalité concernée |
|---|---|---|
| RG-01 | Une adresse courriel ne peut être associée qu'à un seul compte | Inscription (F-01), Gestion des enfants (F-06), Gestion des utilisateurs (F-16) |
| RG-02 | Un compte inscrit par la personne elle-même est créé inactif, avec une adresse non validée | Inscription (F-01) |
| RG-03 | L'inscription autonome exige 15 ans révolus, contrôlés par le système | Inscription (F-01) |
| RG-04 | Le mot de passe contient au moins 8 caractères | Inscription (F-01), Mot de passe oublié (F-04) |
| RG-05 | Un lien de validation ou de réinitialisation est valable une heure | Validation de l'adresse (F-02), Mot de passe oublié (F-04), Validation des inscriptions (F-17) |
| RG-06 | Un nouveau lien n'est pas envoyé tant qu'un lien valable existe | Validation de l'adresse (F-02), Validation des inscriptions (F-17) |
| RG-07 | Un compte inscrit par la personne elle-même ne peut se connecter qu'après validation de son adresse puis validation par l'administrateur | Validation des inscriptions (F-17), Connexion (F-03) |
| RG-08 | La connexion exige un compte actif et une adresse validée | Connexion (F-03) |
| RG-09 | Trois échecs de connexion en une minute bloquent les tentatives pendant vingt minutes, pour une même adresse courriel depuis une même adresse réseau | Connexion (F-03) |
| RG-10 | Une photo de profil ou de cours ne dépasse pas 2 Mo et est au format jpg, png ou webp | Profil (F-05), Gestion des cours (F-18), Dépôt de médias (F-19) |
| RG-11 | Un changement d'adresse courriel est effectif après clic sur le lien envoyé à la nouvelle adresse | Profil (F-05) |
| RG-12 | Un enfant de moins de 15 ans n'a pas de compte autonome : il est créé et géré par un superviseur | Gestion des enfants (F-06) |
| RG-13 | Un enfant créé sans adresse courriel est actif immédiatement ; avec une adresse, il doit la valider | Gestion des enfants (F-06) |
| RG-14 | Seul un superviseur lié à un enfant peut le modifier ou le retirer ; le retrait supprime définitivement son compte | Gestion des enfants (F-06) |
| RG-15 | Une invitation de supervision n'a d'effet qu'après acceptation ; une adresse ne peut être invitée qu'une fois par enfant | Invitation d'un second superviseur (F-07) |
| RG-16 | Seul un superviseur lié à l'enfant peut inviter, et seul un enfant de rôle élève peut recevoir un second superviseur | Invitation d'un second superviseur (F-07) |
| RG-17 | Un utilisateur voit les publications générales et celles de ses groupes | Consultation des groupes (F-08), Fil (F-11) |
| RG-18 | Rejoindre ou quitter un groupe ne demande la validation de personne | Rejoindre ou quitter un groupe (F-09) |
| RG-19 | Une publication contient dix médias au plus, aux formats jpg, jpeg, png, webp, webm, mov, ogg ou mp4 | Publier (F-10), Dépôt de médias (F-19) |
| RG-20 | Seuls l'administrateur et les professeurs publient | Publier (F-10) |
| RG-21 | Un utilisateur aime une publication au plus une fois ; un nouveau clic retire sa mention | Fil et mentions « j'aime » (F-11) |
| RG-22 | Sur une publication de groupe, seuls les membres du groupe et les administrateurs commentent ; un commentaire n'est supprimé que par son auteur ou l'administrateur, et il est alors masqué | Commentaires (F-12) |
| RG-23 | Seuls l'administrateur et l'auteur d'une publication peuvent la supprimer | Suppression d'une publication (F-13) |
| RG-24 | Une conversation réunit exactement deux personnes, qui n'ont qu'une conversation entre elles | Messagerie privée (F-14) |
| RG-25 | Le masquage d'une conversation ne concerne que la personne qui la masque | Messagerie privée (F-14) |
| RG-26 | L'auteur d'une action ne reçoit pas la notification qu'elle déclenche | Notifications push (F-15) |
| RG-27 | Un compte suspendu conserve ses données mais ne peut plus se connecter et n'apparaît plus dans la messagerie | Gestion des utilisateurs (F-16) |
| RG-28 | Supprimer et bannir désignent la même opération : la suppression définitive du compte et de ses données | Gestion des utilisateurs (F-16) |
| RG-29 | Les cours de même nom de groupe sont regroupés sous un même groupe ; noms et niveaux sont libres | Gestion des cours (F-18) |

## 7. Exigences non fonctionnelles

| ID | Catégorie | Exigence | Critère mesurable |
|---|---|---|---|
| ENF-01 | Sécurité : confidentialité des mineurs | Les données d'un enfant ne sont accessibles qu'aux superviseurs liés à cet enfant et à l'administrateur, pour toute consultation ou modification | Aucun accès aux données d'un enfant par un compte sans lien avec lui lors d'un test de contrôle d'accès |
| ENF-02 | Sécurité : authentification | Mot de passe de 8 caractères minimum ; session d'un jour ; double authentification pour les administrateurs et les superviseurs | 100 % des comptes administrateur protégés par double authentification |
| ENF-03 | Sécurité : tentatives répétées | Limitation à 3 échecs de connexion par minute, suivie d'un blocage de 20 minutes | Au quatrième essai pendant le blocage, refus |
| ENF-04 | Sécurité : réinitialisation | Limitation du nombre de demandes de réinitialisation du mot de passe et de renvoi de lien | Au-delà de 3 demandes par heure pour une même adresse, refus |
| ENF-05 | Protection des données personnelles | Conformité au RGPD : registre des traitements, base légale pour les mineurs et les comptes gérés par un parent, droit à l'effacement | Registre et base légale rédigés et validés par la direction |
| ENF-06 | Accessibilité | Respect du niveau WCAG 2.1 AA | Aucune anomalie bloquante de niveau A et AA sur les parcours principaux |
| ENF-07 | Performance | Pages rapides à charger sur un réseau mobile courant | 95 % des pages chargées en moins de 3 secondes sur l'accueil, les groupes et la messagerie |
| ENF-08 | Disponibilité | Service disponible hors maintenance planifiée ; maintenance annoncée aux utilisateurs | 99 % de disponibilité mensuelle |
| ENF-09 | Compatibilité | Navigateurs récents sur Android, iOS et ordinateur ; installation sur l'écran d'accueil du téléphone | Parcours principaux vérifiés sur les deux dernières versions de Chrome, Safari et Firefox |
| ENF-10 | Fiabilité | Confirmation avant toute suppression ; message compréhensible en cas d'erreur ; la suppression d'un élément entraîne celle des données qui en dépendent | Aucune donnée orpheline ni perte non voulue lors d'une suppression |
| ENF-11 | Évolutivité | Accueillir la croissance du nombre d'utilisateurs sans refonte | Fonctionnement normal avec le double de la volumétrie actuelle |
| ENF-12 | Médias | Photos de profil et de cours : 2 Mo au plus (jpg, png, webp). Médias de publication : photos 5 Mo au plus, vidéos 100 Mo au plus, dix fichiers au plus. Dépôt réservé aux utilisateurs connectés | Fichier hors limite ou dépôt sans authentification refusé |
| ENF-13 | Chiffrement des échanges | Toutes les communications passent par HTTPS | Aucun accès non chiffré en production |
| ENF-14 | Traçabilité | Les connexions et les actions d'administration sensibles (validation, suspension, suppression) sont journalisées hors du serveur applicatif | Journaux consultables hors du serveur |
| ENF-15 | Qualité | Chaque modification est vérifiée sur le parcours concerné avant mise en production ; un contrôle automatique de qualité et de sécurité s'exécute à chaque modification du code | Compte rendu par modification ; contrôle automatique réussi |

## 8. Données

### 8.1 Données traitées
| Donnée | Format | Obligatoire | Sensibilité |
|---|---|---|---|
| Adresse courriel | Texte (254 caractères) | Oui (facultative pour un enfant) | Élevée |
| Mot de passe | Texte, stocké sous forme chiffrée irréversible | Oui | Élevée |
| Nom, prénom | Texte (255 caractères) | Nom : oui ; prénom : non | Élevée (mineurs) |
| Date de naissance | Date | Oui à l'inscription | Élevée (mineurs) |
| Genre | Homme, femme, non précisé | Non | Moyenne |
| Téléphone, adresse postale, code postal, ville | Texte ; code postal à 5 caractères | Non | Élevée |
| Rôle et état du compte | Valeurs fixes (actif, adresse validée) | Oui | Moyenne |
| Liens parent-enfant et invitations | Identifiants | Oui pour un enfant | Élevée |
| Groupes et cours | Nom, niveau, image | Non | Moyenne |
| Publications, commentaires, mentions « j'aime » | Texte, médias | Non | Moyenne |
| Messages privés | Texte | Non | Élevée |
| Photos et vidéos | jpg, png, webp, mp4, etc. | Non | Moyenne à élevée |
| Données techniques | Jetons de session et de validation, abonnements aux notifications | Oui | Élevée |

### 8.2 Règles de validation
- Format : adresse courriel valide, code postal de 5 caractères, date de naissance valide, mot de passe d'au moins 8 caractères.
- Cohérence : 15 ans minimum pour l'inscription autonome ; chaque enfant est lié à au moins un superviseur.
- Unicité : une adresse par compte, un lien par couple parent-enfant, une mention « j'aime » par utilisateur et par publication, une invitation par adresse et par enfant, une conversation par couple d'utilisateurs.

### 8.3 Accès, conservation et suppression
- Accès : chaque utilisateur accède à ses données, le superviseur à celles de ses enfants, l'administrateur à l'ensemble. Les messages privés ne sont lisibles que par leurs deux participants.
- Conservation : les données sont conservées tant que le compte est actif, puis supprimées ou anonymisées 3 ans après le départ de l'élève.
- Rectification et suppression : par l'utilisateur ou son superviseur ; suppression définitive par l'administrateur ou, pour un enfant, par son superviseur. Une demande d'export des données est traitée par l'administrateur.
- Sauvegarde : sauvegarde quotidienne de la base de données ; actions sensibles journalisées (exigence Traçabilité, ENF-14).

### 8.4 Modèle de données
Le modèle conceptuel des données figure en annexe.

## 9. Notifications

| Événement | Destinataire | Canal | Contenu |
|---|---|---|---|
| Nouvelle publication générale | Tous sauf l'auteur | Push | Annonce de la publication |
| Nouvelle publication de groupe | Membres du groupe sauf l'auteur | Push | Annonce de la publication du groupe |
| Nouveau message privé | Destinataire | Push | Annonce du message |
| Adresse validée par un nouvel inscrit | Administrateurs | Push | « Nouvelle inscription » |
| Comptes en attente de validation depuis plus d'une heure | Administrateurs | Push | Rappel de validation |
| Invitation de supervision (compte existant) | Personne invitée | Push | « Nouvelle invitation » |
| Inscription ou création d'un compte avec adresse | Titulaire de l'adresse | Courriel « Vérification du compte » | Lien de validation valable une heure |
| Mot de passe oublié ; adresse validée par un compte rattaché à un superviseur | Titulaire du compte | Courriel « Réinitialisation du mot de passe » | Lien de réinitialisation valable une heure |
| Changement d'adresse courriel | Nouvelle adresse | Courriel « Validation du nouvel email » | Lien de confirmation |
| Invitation de supervision (adresse sans compte) | Personne invitée | Courriel « Invitation à vous connecter » | Lien vers l'application |

- Les courriels sont envoyés depuis l'adresse noreply@centreartetdanse.com. Les courriels de validation et de réinitialisation ne peuvent pas être désactivés.
- L'utilisateur active ou désactive les notifications push depuis son appareil et l'application.
- Une notification est envoyée par événement, sans regroupement.
- Un échec d'envoi ne bloque pas l'action d'origine ; il est journalisé.

## 10. Services externes

| Service | Objectif | Données échangées | En cas d'indisponibilité |
|---|---|---|---|
| Serveur d'envoi de courriels | Courriels automatiques | Adresse, contenu, lien | Les courriels ne partent pas ; le reste de l'application fonctionne |
| Services de notification des navigateurs (Google, Apple, Mozilla) | Notifications push | Abonnement, contenu court | Les notifications ne partent pas ; le reste de l'application fonctionne |
| Service de fichiers de VNWeb | Dépôt et diffusion des photos et vidéos | Fichiers, adresses de diffusion | Les médias ne s'affichent pas et ne peuvent pas être déposés |
| Lecteurs vidéo externes | Lecture des vidéos liées dans une publication | Adresse de la vidéo | La vidéo liée ne s'affiche pas |
| Sites liés dans une publication | Aperçu d'un lien | Adresse du lien | Le lien s'affiche sans aperçu |

## 11. Recette

### 11.1 Organisation
La recette se déroule sur l'environnement de préproduction avant toute mise en production. Le dirigeant de l'école valide les fonctionnalités ; Vincent Nilles valide les priorités techniques. Chaque essai est documenté.

### 11.2 Tests d'acceptation
| ID | Fonctionnalité | Situation et action | Résultat attendu |
|---|---|---|---|
| T-01 | Inscription (F-01) | Une personne de 16 ans avec une adresse inconnue s'inscrit | Compte inactif créé, courriel de vérification envoyé |
| T-02 | Inscription (F-01) | Une personne de 14 ans tente de s'inscrire | Inscription refusée |
| T-03 | Validation de l'adresse (F-02) | Ouverture d'un lien reçu il y a plus d'une heure | Lien refusé |
| T-04 | Validation des inscriptions (F-17) | L'administrateur valide un compte à adresse validée, puis la personne se connecte | Connexion réussie |
| T-05 | Connexion (F-03) | Trois échecs puis une tentative avec le bon mot de passe dans les vingt minutes | Tentative refusée |
| T-06 | Mot de passe oublié (F-04) | Demande d'un lien, ouverture, saisie d'un nouveau mot de passe | Mot de passe modifié |
| T-07 | Gestion des enfants (F-06) | Un superviseur tente de modifier ou de retirer l'enfant d'un autre parent | Action refusée |
| T-08 | Gestion des enfants (F-06) | Un superviseur retire son enfant | Compte de l'enfant supprimé |
| T-09 | Invitation d'un second superviseur (F-07) | Un superviseur lié à l'enfant invite une adresse sans compte | Courriel d'invitation envoyé |
| T-10 | Invitation d'un second superviseur (F-07) | Une personne non liée à l'enfant tente d'inviter pour lui | Invitation refusée |
| T-11 | Invitation d'un second superviseur (F-07) | La personne invitée refuse | Aucun lien de supervision créé |
| T-12 | Rejoindre ou quitter un groupe (F-09) | Un élève rejoint un groupe | Groupe visible sur son profil |
| T-13 | Publier (F-10) | Un professeur publie dans un groupe | Seuls les membres du groupe voient la publication |
| T-14 | Commentaires (F-12) | Un membre du groupe commente une publication du groupe | Commentaire affiché |
| T-15 | Commentaires (F-12) | Un utilisateur non administrateur tente de supprimer le commentaire d'un autre | Action refusée |
| T-16 | Messagerie privée (F-14) | Deux utilisateurs actifs échangent un message | Réception en temps réel |
| T-17 | Gestion des utilisateurs (F-16) | Un compte suspendu tente de se connecter | Connexion refusée |
| T-18 | Gestion des utilisateurs (F-16) | L'administrateur supprime un parent qui a un enfant | L'enfant est conservé, l'administrateur est averti |
| T-19 | Dépôt de médias (F-19) | Envoi d'une photo de profil de 3 Mo | Fichier refusé |
| T-20 | Notifications push (F-15) | Publication générale par un professeur | Les autres utilisateurs sont notifiés, pas l'auteur |

### 11.3 Conditions de validation
- Tous les tests des fonctionnalités Must réussis, ou écart accepté par écrit par le dirigeant de l'école.
- Aucune anomalie bloquante acceptée : perte de données, accès aux données d'un tiers, impossibilité de se connecter.
- Un compte rendu de recette daté est produit pour chaque modification importante.

## 12. Priorisation et évolutions

### 12.1 Priorités (méthode MoSCoW)
| Priorité | Fonctionnalités |
|---|---|
| Must have (indispensable) | Comptes et accès, supervision des enfants, groupes, publication, fil, suppression d'une publication, messagerie, administration, dépôt de médias |
| Should have (important) | Mentions « j'aime », notifications push |
| Could have (souhaitable) | Commentaires |
| Won't have (exclu) | Messagerie de groupe, paiement, planning des cours, gestion de la présence |

### 12.2 Évolutions prévues
| ID | Évolution | Exigence ou fonctionnalité liée | Priorité |
|---|---|---|---|
| EV-01 | Contrôler le lien superviseur-enfant sur toutes les actions concernant un enfant | Confidentialité des mineurs (ENF-01), Gestion des enfants (F-06), Invitation d'un second superviseur (F-07) | Must |
| EV-02 | Contrôler l'âge de 15 ans et le type de compte au niveau du serveur lors de l'inscription | Inscription (F-01) | Must |
| EV-03 | Réserver le dépôt de fichiers aux utilisateurs connectés et contrôler formats et tailles | Médias (ENF-12), Dépôt de médias (F-19) | Must |
| EV-04 | Limiter l'accès à la messagerie en temps réel aux seules pages de l'application | Messagerie privée (F-14) | Must |
| EV-05 | Rédiger le registre des traitements et la base légale RGPD | Protection des données personnelles (ENF-05) | Must |
| EV-06 | Mettre en place la double authentification pour les administrateurs et les superviseurs | Authentification (ENF-02) | Should |
| EV-07 | Limiter les demandes de réinitialisation et de renvoi de lien | Réinitialisation (ENF-04) | Should |
| EV-08 | Avertir l'administrateur des enfants restés sans superviseur | Gestion des utilisateurs (F-16) | Should |
| EV-09 | Permettre une validation par un professeur ou l'administrateur pour rejoindre un groupe | Rejoindre ou quitter un groupe (F-09) | Could |
| EV-10 | Retirer les composants logiciels inutilisés et le doublon d'information sur le rôle administrateur | Fiabilité (ENF-10) | Could |

### 12.3 Jalons
| Jalon | Responsable | Échéance |
|---|---|---|
| Validation du CDCF | Direction de l'école, Vincent Nilles | Avant toute évolution |
| Évolutions Must (EV-01 à EV-05) | Ecaterina Munteanu | En priorité |
| Évolutions Should et Could (EV-06 à EV-10) | Ecaterina Munteanu | Après les évolutions Must |
| Recette | Ecaterina Munteanu, validation par la direction | Avant chaque mise en production |

## 13. Contraintes, hypothèses et risques

### 13.1 Contraintes
- Budget : pas de budget dédié ; les évolutions sont réalisées dans le cadre de la maintenance de l'application.
- Délais : pas d'échéance imposée par le client ; les évolutions suivent l'ordre de priorité.
- Réglementation : RGPD, avec une attention particulière aux données des mineurs et aux enfants de moins de 15 ans gérés par un parent.
- Organisation : une seule développeuse en charge du projet ; priorités validées par Vincent Nilles.
- Technique : technologies existantes conservées, sans réécriture ; toute modification est réalisée sans interruption de service ni perte de données.

### 13.2 Dépendances et hypothèses
- Le projet dépend de la direction de l'école pour la validation des fonctionnalités et de la recette, et de Vincent Nilles pour les priorités.
- Quatre environnements sont utilisés : serveur d'essai, développement, préproduction et production. Le code est hébergé sur le serveur Gitea de VNWeb, avec un contrôle automatique à chaque modification.
- La direction valide les évolutions avant leur mise en œuvre.
- La volumétrie reste celle d'une école de danse, de l'ordre de quelques centaines d'utilisateurs.

### 13.3 Risques
| Risque | Impact | Probabilité | Réponse |
|---|---|---|---|
| Accès aux données d'un enfant par un compte sans lien avec lui | Élevé | Moyenne | Contrôle du lien superviseur-enfant (EV-01) |
| Dépôt de fichiers par une personne non connectée ou de taille excessive | Élevé | Moyenne | Authentification et limites du dépôt (EV-03) |
| Accès à la messagerie en temps réel depuis un site extérieur | Élevé | Moyenne | Restriction d'accès (EV-04) |
| Inscription d'un enfant de moins de 15 ans en contournant l'interface | Moyen | Moyenne | Contrôle de l'âge par le serveur (EV-02) |
| Dépendance à une seule personne | Élevé | Élevée | Documentation, tests, contrôle automatique |
| Incident non détecté | Moyen | Moyenne | Journalisation des actions sensibles (ENF-14) |
| Enfant resté sans superviseur après suppression d'un parent | Moyen | Moyenne | Alerte de l'administrateur (EV-08) |

## 14. Annexes

### 14.1 Modèle conceptuel des données

```mermaid
erDiagram
    USER ||--o{ SUPERVISOR_LINK : "est superviseur"
    USER ||--o{ SUPERVISOR_LINK : "est enfant supervise"
    USER ||--o{ SUPERVISOR_INVITATION : "enfant concerne"
    USER ||--o{ GROUP_MEMBERSHIP : "appartient"
    GROUP ||--o{ GROUP_MEMBERSHIP : "regroupe"
    USER ||--o{ POST : "ecrit"
    GROUP |o--o{ POST : "cible"
    POST ||--o{ MEDIA : "contient"
    POST ||--o{ LIKE : "recoit"
    USER ||--o{ LIKE : "donne"
    POST ||--o{ COMMENT : "recoit"
    USER ||--o{ COMMENT : "ecrit"
    COMMENT |o--o{ COMMENT : "reponse"
    USER ||--o{ CONVERSATION : "participe (deux)"
    CONVERSATION ||--o{ MESSAGE : "contient"
    USER ||--o{ MESSAGE : "envoie"
    CONVERSATION ||--o{ CONVERSATION_CONFIG : "etat par participant"
    USER ||--o{ PUSH_SUBSCRIPTION : "abonne"
    USER ||--o{ TOKEN : "possede"

    USER {
        int id
        string email
        string role
        bool actif
        bool email_valide
        date date_naissance
    }
    GROUP {
        int id
        string nom
        string niveau
        string image
    }
    GROUP_MEMBERSHIP {
        int type "suivi ou assignation"
    }
    POST {
        int id
        text contenu
    }
    COMMENT {
        int id
        text contenu
        bool supprime
    }
    MESSAGE {
        int id
        text valeur
        bool lu
    }
```

### 14.2 Matrice des droits

| Fonction | Administrateur | Professeur | Superviseur | Élève | Élève-superviseur |
|---|:-:|:-:|:-:|:-:|:-:|
| Consulter les publications générales et de ses groupes | Oui | Oui | Oui | Oui | Oui |
| Aimer une publication | Oui | Oui | Oui | Oui | Oui |
| Commenter (publication de groupe : membres seulement) | Oui | Oui | Oui | Oui | Oui |
| Créer une publication | Oui | Oui | Non | Non | Non |
| Supprimer une publication | Oui | La sienne | Non | Non | Non |
| S'assigner à un groupe ou s'en retirer | Oui | Oui | Non | Non | Non |
| Choisir les groupes suivis | Non | Non | Pour ses enfants | Oui | Oui |
| Messagerie privée | Oui | Oui | Oui | Oui | Oui |
| Gérer ses enfants | Non | Non | Oui | Non | Oui |
| Inviter un second superviseur | Non | Non | Oui | Non | Oui |
| Accéder à l'espace d'administration | Oui | Non | Non | Non | Non |

Valeurs techniques des rôles : Administrateur = ADMIN, Professeur = PROFESSOR, Superviseur = SUPERVISOR, Élève = STUDENT, Élève-superviseur = STUDENT_SUPERVISOR.

### 14.3 Documents de référence
- Plan d'audit de sécurité.
- Plan d'action de sécurisation.
- Audit de conformité RGPD.

## 15. Validation du document

| Partie prenante | Nom | Rôle | Décision | Date |
|---|---|---|---|---|
| Direction de l'école | | Commanditaire et référent métier | | |
| VNWeb | Vincent Nilles | Maître d'apprentissage, validation des priorités techniques | | |
| VNWeb | Ecaterina Munteanu | Rédactrice | Rédigé | 13/11/2026 |
