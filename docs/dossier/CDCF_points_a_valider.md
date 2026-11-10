# Points à valider : CDCF Centre Art & Danse (version 1.0, 06/11/2026)

Document de travail, à ne pas joindre au dossier. Il liste tout ce qui n'est pas encore tranché derrière le CDCF CDCF_centre_art_et_danse.md. Le CDCF est rédigé au propre : il présente directement les valeurs proposées ci-dessous comme exigences. Si un point est refusé, il faut corriger la section du CDCF indiquée.

Validateurs : **VN** = Vincent Nilles (dirigeant de VNWeb, maître d'apprentissage) ; **DIR** = direction de l'école de danse ; **EM** = Ecaterina Munteanu (vérification dans le code ou rédaction) ; **VN+DIR** = les deux.
Statuts : Ouvert / À vérifier / Résolu.

## A. Décisions fonctionnelles

| ID | Section CDCF | Question | Proposition actuelle | Validateur | Statut |
|---|---|---|---|---|---|
| PAV-01 | 2.3, 16 | Nom de la direction de l'école, du référent métier et du product owner ? | « [à compléter] » ; référent métier = dirigeant de l'école | DIR | Ouvert |
| PAV-02 | FCT-011, RG-018, A-08 | L'inscription libre à un groupe (sans validation, un élève peut rejoindre n'importe quel groupe, et le type d'appartenance est choisi par le client) reste-t-elle la règle ? | Documentée comme règle actuelle ; validation par un tiers proposée en priorité Could | DIR (et VN) | Ouvert |
| PAV-03 | FCT-014, RG-022, MOD-005 | Les commentaires sont-ils voulus ? Vérification du code : ils sont actifs dans l'interface (accueil et page de groupe). Sur la page d'un groupe, seuls professeurs assignés et administrateurs voient la zone de saisie, alors que le serveur autorise tout membre. Quelle intention ? | Conserver et classer Could, en attendant la décision | VN | Ouvert |
| PAV-17 | 12, 12.2 | Les améliorations A-01 à A-10 (initiatives de l'autrice) sont-elles retenues et dans quel ordre ? | Priorités indicatives du tableau 12.2 | VN | Ouvert |
| PAV-20 | 1.2 | Quelle était la situation de l'école avant l'application (outils, difficultés) ? | Texte générique : messages individuels aux familles | DIR | Ouvert |
| PAV-36 | FCT-018, 13.4 | Quand un parent est supprimé, ses enfants n'ont plus de superviseur (le code ne supprime que le lien). Que faire : interdire la suppression, rattacher à un autre superviseur, supprimer l'enfant ? | Règle à définir ; test TEST-018 décrit le comportement actuel | DIR et VN | Ouvert |
| PAV-37 | PAR-005, RG-025 | Quand A masque une conversation puis B écrit, la conversation réapparaît-elle chez A ? | Non constaté dans le code lu | EM | À vérifier |
| PAV-35 | FCT-017, 9 | Que faire en cas d'échec d'envoi d'une notification push ou d'un courriel (nouvelle tentative, alerte) ? | Aucun traitement constaté ; pas de règle proposée | VN | Ouvert |

## B. Exigences non fonctionnelles proposées (ni validées, ni implémentées sauf mention)

| ID | Section CDCF | Question | Proposition actuelle | Validateur | Statut |
|---|---|---|---|---|---|
| PAV-04 | ENF-002, A-06 | Double authentification obligatoire pour administrateurs et superviseurs ? | Proposée, non implémentée | VN+DIR | Ouvert |
| PAV-05 | ENF-004, A-07 | Limitation des demandes de réinitialisation du mot de passe et du renvoi de lien ; seuil ? (la connexion est déjà limitée : 3 échecs par minute, blocage de 20 minutes) | Proposée, seuil à définir | VN | Ouvert |
| PAV-06 | ENF-007 | Cible de performance | 95 % des pages en moins de 3 secondes sur réseau mobile courant | VN+DIR | Ouvert (proposition) |
| PAV-07 | ENF-008 | Cible de disponibilité | 99 % mensuel hors maintenance planifiée | VN+DIR | Ouvert (proposition) |
| PAV-08 | ENF-006 | Niveau d'accessibilité visé | WCAG 2.1 AA, besoin du dossier RNCP ; engagement client inconnu | VN | Ouvert (proposition) |
| PAV-09 | ENF-009 | Navigateurs et appareils pris en charge | Deux dernières versions majeures de Chrome, Safari, Firefox ; Android, iOS, ordinateur | VN | Ouvert (proposition) |
| PAV-10 | ENF-014 | Journalisation hors serveur des connexions et actions d'administration, avec alerte | Proposée, non implémentée | VN | Ouvert |
| PAV-11 | 8.3 | Durée de conservation des données après le départ d'un élève et règle sur les comptes inactifs | Compte actif conservé ; suppression ou anonymisation après 3 ans d'inactivité | DIR | Ouvert (proposition) |
| PAV-12 | 2.1, ENF-011 | Nombre d'utilisateurs et volume de données | À définir ; hypothèse : quelques centaines | DIR | Ouvert |
| PAV-13 | ENF-005, A-05 | Registre des traitements, base légale (mineurs, comptes gérés par un parent), désignation éventuelle d'un délégué à la protection des données | À rédiger par l'autrice ; décisions de gestion à la direction | DIR et VN | Ouvert |
| PAV-14 | ENF-012 | Limites de taille et de type pour les médias | Constaté : 2 Mo pour les images de profil et de cours (jpg, png, webp) ; 500 Mo par fichier de publication côté application et 5 000 Mo côté service de fichiers, dix fichiers au plus. Proposé : photos 5 Mo, vidéos 100 Mo | VN | Ouvert |
| PAV-15 | 8.3 | Politique de sauvegarde et de restauration des données | Non documentée | VN | Ouvert |
| PAV-38 | ENF-013 | HTTPS sur tous les environnements publics ? | À vérifier sur la production | EM | À vérifier |
| PAV-18 | 13.2 | Hébergement exact de chacun des quatre environnements (serveur d'essai personnel, développement, préproduction, production) ; état du pipeline | Dépôt sur le Gitea de VNWeb, chaîne de contrôle automatique présente (style du code et analyse Semgrep) ; hébergement à confirmer | VN | À vérifier |

## C. Écarts constatés dans le code (à confirmer et à décider)

| ID | Section CDCF | Constat | Proposition actuelle | Validateur | Statut |
|---|---|---|---|---|---|
| PAV-16 | 10 | Firebase est une dépendance de l'interface, sans usage trouvé dans le code source de l'interface | Hypothèse : inutilisée, à retirer ; non listée comme intégration | EM (recherche) puis VN | À vérifier |
| PAV-30 | FCT-001, A-02 | L'âge de 15 ans n'est contrôlé que dans l'interface, pas par le serveur ; le type de compte n'est pas contrôlé côté serveur | Contrôle côté serveur proposé | VN | Ouvert |
| PAV-31 | FCT-002, 15.4 | Le renvoi du lien de vérification est possible par l'utilisateur sans être connecté, pas seulement par l'administrateur | À décider : conserver (confort utilisateur) ou réserver à l'administrateur | VN | Ouvert |
| PAV-32 | FCT-006 | Le changement d'adresse courriel existe côté serveur ; est-il accessible dans l'interface ? | Documenté comme fonctionnalité du profil | EM | À vérifier |
| PAV-33 | FCT-007, FCT-008, ENF-001, A-01 | La modification d'un enfant ne contrôle pas le lien superviseur-enfant, et l'invitation d'un superviseur ne contrôle que le rôle de la personne qui invite ; un professeur peut inviter | Contrôle du lien proposé ; le rôle Professeur est-il voulu comme inviteur ? | VN | Ouvert |
| PAV-39 | FCT-007 | À l'ajout d'un enfant, ses groupes sont enregistrés avec le type « assignation » (valeur 2, celle des professeurs) au lieu de « suivi » ; conséquence sur les droits de commentaire de l'enfant | À confirmer avec l'intention d'origine | EM puis VN | À vérifier |
| PAV-40 | A-04 | Le canal de messagerie en temps réel accepte les connexions de n'importe quel site | Restriction proposée | VN | Ouvert |
| PAV-41 | INT-003, A-03 | Service de fichiers : aucune authentification constatée pour le dépôt de fichiers, contrôle de provenance des requêtes désactivé, limite de 5 000 Mo, extension reprise du nom du fichier envoyé | Authentification et limites proposées | VN | Ouvert |
| PAV-42 | NOT-006 | Des commandes d'entretien existent (rappel des comptes à valider, suppression des comptes non vérifiés, nettoyage des jetons, courriel « ressaisir son adresse ») ; aucune planification (tâche planifiée) n'a été trouvée dans les dépôts | Savoir lesquelles tournent réellement sur la production | EM puis VN | À vérifier |
| PAV-43 | 9 | Le modèle de courriel invitant à ressaisir son adresse, et sa commande d'envoi, sont-ils encore utilisés ? | Non listés comme notification active | EM | À vérifier |

## D. Livrables de contenu à fournir

| ID | Section CDCF | Élément | Validateur | Statut |
|---|---|---|---|---|
| PAV-21 | 4.1 | Relecture des parcours PAR-001 à PAR-005 (déduits du code) | DIR | Ouvert |
| PAV-22 | 4.2, 4.3, 5.2, 5.3 | Schéma de navigation, états particuliers (vide, chargement, erreur, hors connexion), charte graphique, maquettes | DIR et EM | Ouvert |

## E. Points résolus pendant la rédaction

| ID | Question | Résolution |
|---|---|---|
| PAV-34 | Un second « j'aime » est-il compté ? | Résolu : le second clic retire le « j'aime » |
| Équipe | Taille de l'équipe | Une personne |
| Rôle visiteur | Faut-il un rôle visiteur ? | Non |
| Périmètre produit | Produit ou chantier de sécurité ? | Produit ; correctifs en amélioration A-xx |
| MVP | Bloc MVP ? | Supprimé ; MoSCoW de l'existant en section 12 |

## F. Valeurs écrites directement dans le CDCF, à faire confirmer

Ces éléments n'avaient pas de valeur dans les sources. Le CDCF les présente comme exigences ; à confirmer ou corriger.

| ID | Section CDCF | Valeur écrite dans le CDCF | Validateur | Statut |
|---|---|---|---|---|
| PAV-44 | ENF-004 | Au-delà de 3 demandes de réinitialisation par heure pour une même adresse, refus | VN | Ouvert |
| PAV-45 | 8.3 | Sauvegarde quotidienne de la base de données (à vérifier sur l'existant) | VN | À vérifier |
| PAV-46 | FCT-018, 13.4 | À la suppression d'un parent, l'administrateur est averti des enfants restés sans superviseur | VN+DIR | Ouvert |
| PAV-47 | PAR-005 | Une conversation masquée réapparaît quand un nouveau message arrive (à vérifier sur l'existant) | EM | À vérifier |
| PAV-48 | FCT-017, 9 | Un échec d'envoi de notification ne bloque pas l'action et est journalisé | EM | À vérifier |
| PAV-49 | ENF-011 | Fonctionnement normal avec le double de la volumétrie actuelle | VN | Ouvert |
| PAV-50 | 10 | En cas de panne d'un service externe, seule la fonction concernée est indisponible | EM | À vérifier |
| PAV-51 | 4.3, 5.2, 5.3 | États particuliers (vide, chargement, erreur, hors connexion) et identité visuelle décrits de façon générique, sans maquette | DIR | Ouvert |
| PAV-52 | FCT-014, 12 | Commentaires classés Could have, décrits comme une fonctionnalité normale (actifs dans l'interface) | VN | Ouvert |
| PAV-53 | 15.3, FCT-008 | Le professeur ne peut pas inviter de superviseur (le code l'autorise aujourd'hui) | VN | Ouvert |
| PAV-54 | 2.3, 16 | Nom du représentant de la direction laissé vide dans le tableau de validation, à remplir à la signature | DIR | Ouvert |
