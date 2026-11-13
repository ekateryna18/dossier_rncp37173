# Constat 01 : mot de passe provisoire écrit en dur pour les comptes créés par l'administrateur

| Élément | Valeur |
|---|---|
| Date | 13/11/2026 |
| Application | Centre Art & Danse, serveur applicatif (API) |
| Détecté par | Semgrep, jeu de règles nodejsscan, dans le pipeline d'intégration continue |
| Type | Faiblesse : identifiant par défaut écrit en dur (CWE-798, CWE-1392) |
| Exploitabilité actuelle | Nulle (vérifiée par test) |
| Gravité | Faible aujourd'hui, élevée si la protection actuelle disparaît |
| Statut | À corriger |

## Contexte de la détection

Le pipeline utilisait jusqu'ici Semgrep avec une sélection automatique de règles, qui ne signalait aucun problème. Lors du renforcement du pipeline, cette sélection a été remplacée par des jeux de règles fixes, dont celui de l'outil njsscan, spécialisé dans les applications Node.js. Ce jeu contient des règles de recherche de secrets écrits en dur. Il a signalé deux alertes bloquantes sur la même ligne du code : un mot de passe en clair et un nom d'utilisateur en clair.

Le code concerné existait déjà : c'est l'élargissement des règles qui l'a fait apparaître, et non une modification récente.

## Description

Quand l'administrateur crée un compte depuis l'espace d'administration, la personne n'a pas encore choisi son mot de passe. La base de données exigeant une valeur, le code attribue à chaque compte créé de cette façon le même mot de passe provisoire, de deux caractères, écrit en dur dans le code. Le compte est créé activé ; il devient utilisable dès que la personne valide son adresse courriel.

Le modèle utilisateur chiffre automatiquement le mot de passe à l'enregistrement : ce mot de passe provisoire devient donc un vrai mot de passe, identique pour tous ces comptes et connu de toute personne ayant accès au code.

L'alerte sur le nom d'utilisateur est un faux positif : aucun nom d'utilisateur n'est écrit en dur, la règle a réagi au même motif que l'alerte sur le mot de passe.

## Analyse et vérification

L'hypothèse d'attaque était la suivante : après la validation de l'adresse, se connecter au compte avec le mot de passe provisoire.

Test réalisé le 13/11/2026 en environnement de développement :
1. création d'un compte depuis l'espace d'administration ;
2. validation de l'adresse par le lien reçu ;
3. tentative de connexion avec le mot de passe provisoire.

Résultat : connexion refusée avec l'erreur de longueur minimale. Le serveur exige au moins 8 caractères pour tout mot de passe saisi à la connexion, et ce contrôle intervient avant la comparaison du mot de passe. Il est fait côté serveur et ne peut pas être contourné depuis l'interface. La connexion est le seul endroit de l'application qui vérifie un mot de passe.

Une fuite de la base de données ne donnerait pas non plus une valeur commune : l'algorithme de chiffrement (scrypt) ajoute un élément aléatoire propre à chaque compte, chaque empreinte est donc différente.

**Conclusion : la faiblesse est réelle, mais elle n'est pas exploitable aujourd'hui.**

## Pourquoi la corriger quand même

La protection repose sur une seule règle, sans lien avec la création de compte : la longueur minimale à la connexion. Elle tombe si :
- cette règle est assouplie ou retirée ;
- un nouveau moyen de connexion est ajouté sans ce contrôle (autre application, outil d'administration) ;
- ce code est réutilisé dans un projet aux règles différentes.

Dans ces cas, tout compte créé par l'administrateur et dont le mot de passe n'a pas été changé s'ouvrirait avec le mot de passe provisoire.

## Défaut fonctionnel associé

Le test a mis en évidence un second problème : après la validation de son adresse, la personne ne reçoit aucun courriel lui permettant de choisir son mot de passe. Elle doit passer d'elle-même par « Mot de passe oublié ». Ce courriel est envoyé automatiquement pour les comptes rattachés à un parent, mais pas pour les comptes créés par l'administrateur sans parent. Le comportement attendu décrit dans le cahier des charges fonctionnel n'est donc pas respecté.

## Correction proposée

1. Remplacer le mot de passe provisoire fixe par une valeur aléatoire et longue, différente pour chaque compte et inconnue de tous : le compte n'a alors aucun mot de passe devinable, quelles que soient les règles futures.
2. Envoyer automatiquement le lien de choix du mot de passe après la validation de l'adresse pour les comptes créés par l'administrateur.

En attendant la correction, l'alerte peut être déclarée comme exception justifiée sur la ligne concernée, afin que le pipeline ne bloque pas le travail et que la décision reste visible dans le code.

## Enseignement

Une alerte d'outil d'analyse se vérifie par un test avant d'être classée. Ici, l'alerte était fondée sur le principe, mais le risque était bloqué par une autre protection. La bonne réponse est une correction de consolidation, et non une correction d'urgence.
