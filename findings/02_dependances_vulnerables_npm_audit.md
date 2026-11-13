# Constat 02 : dépendances vulnérables nécessitant une migration du framework

| Élément | Valeur |
|---|---|
| Date | 13/11/2026 |
| Application | Centre Art & Danse : serveur applicatif (API), interface, service de fichiers |
| Détecté par | npm audit, ajouté au pipeline d'intégration continue |
| Type | Composants tiers comportant des vulnérabilités connues (OWASP Top 10 : composants vulnérables et obsolètes) |
| Gravité | Élevée (4 vulnérabilités élevées restantes sur l'API) |
| Statut | Correction partielle faite sur l'API ; migration du framework à planifier |

## Contexte de la détection

Une étape d'audit des dépendances a été ajoutée au pipeline. Elle signale les bibliothèques tierces utilisées par l'application qui font l'objet d'une vulnérabilité publiée.

## Premier état de l'API et correction sans rupture

| Gravité | Avant correction | Après `npm audit fix` |
|---|---|---|
| Critique | 2 | 0 |
| Élevée | 24 | 4 |
| Moyenne | 21 | 25 |
| Faible | 3 | 2 |
| **Total** | **50** | **31** |

La commande `npm audit fix` met à jour les dépendances en restant dans les versions autorisées par le projet, sans changement de version majeure. Elle a supprimé les deux vulnérabilités critiques (dont celles de la lecture des requêtes reçues par l'API et de la détermination de l'adresse du client) et vingt vulnérabilités élevées. Cette mise à jour ne modifie que le fichier de verrouillage des versions ; elle doit être testée (connexion, envoi de courriels, messagerie) avant d'être déployée en préproduction.

## Vulnérabilités élevées restantes sur l'API

| Bibliothèque | Rôle dans l'application | Vulnérabilités | Correction possible |
|---|---|---|---|
| nodemailer (version 7) | Envoi des courriels | Injection de commandes dans le dialogue avec le serveur de courriels ; envoi possible vers un domaine non prévu | Version 10, accessible uniquement via le module de courriels du framework en version 10 |
| Module de courriels d'AdonisJS (version 9) | Envoi des courriels | Hérite des vulnérabilités de nodemailer | Version 10, qui exige AdonisJS 7 |
| Module d'accès à la base d'AdonisJS, Lucid (version 21) | Accès à la base de données | Hérité du cœur du framework | Version 22, qui exige AdonisJS 7 |
| Faker (version 9) | Génération de données de test | Exécution de code possible via une fonction de génération | Fourni avec Lucid ; utilisé uniquement pour les données de test |

Le cœur du framework (AdonisJS 6) présente en outre une vulnérabilité moyenne de redirection ouverte dans son serveur web.

## Pourquoi une correction forcée n'est pas possible

L'outil propose de forcer la correction avec `npm audit fix --force`. Cette option autorise les changements de version majeure, et donc les ruptures de compatibilité. Dans ce projet, elle ne réalise pas la migration attendue : elle propose notamment de remplacer le module d'accès à la base, actuellement en version 21.8, par la version 8.5, c'est-à-dire de revenir plusieurs années en arrière. L'API ne démarrerait plus.

La seule correction réelle des quatre vulnérabilités restantes est la **migration de l'API d'AdonisJS 6 vers AdonisJS 7**, avec les versions correspondantes de ses modules (courriels, accès à la base). Une migration de version majeure a un impact sur le code existant : adaptation des appels qui ont changé, vérification complète des parcours, recette en préproduction. C'est un chantier à planifier, et non une mise à jour de routine.

## Décision prise pour le pipeline

L'audit des dépendances est rendu **non bloquant** dans le pipeline : il s'exécute à chaque modification et son rapport reste visible, mais il n'empêche pas le travail de continuer tant que la migration n'est pas planifiée. Il redeviendra bloquant une fois la migration réalisée.

## État des deux autres composants (non corrigés à ce jour)

| Composant | Critique | Élevée | Correction |
|---|---|---|---|
| Service de fichiers | 1 | 3 | Toutes corrigeables sans changement de version majeure, dont la bibliothèque qui reçoit les fichiers déposés |
| Interface | 3 | 19 | Toutes corrigeables sans changement de version majeure, dont l'outil de construction et le routage des pages |

Le service de fichiers est le composant le plus exposé de l'application (dépôt de fichiers depuis l'extérieur) : sa mise à jour est la plus utile à court terme.

## Actions

1. Tester puis déployer la mise à jour sans rupture de l'API.
2. Appliquer la même mise à jour sans rupture au service de fichiers, puis à l'interface.
3. Planifier la migration de l'API vers AdonisJS 7, avec validation de Vincent Nilles et recette en préproduction.
4. Rendre l'audit bloquant une fois la migration faite.
5. Mettre en place un outil de mise à jour automatique des dépendances compatible avec Gitea (Renovate), pour éviter que les retards de mise à jour ne s'accumulent.
