# Constat 03 : secrets présents dans l'historique du dépôt (clé APP_KEY, clé Firebase)

| Élément | Valeur |
|---|---|
| Date | 13/11/2026 |
| Application | Centre Art & Danse, serveur applicatif (API) |
| Détecté par | Gitleaks, analyse de tout l'historique git, dans le pipeline d'intégration continue |
| Type | Secrets publiés dans le gestionnaire de versions (CWE-312 : stockage en clair d'informations sensibles ; CWE-540 : informations sensibles dans le code source) |
| Exploitabilité actuelle | Faible pour la clé APP_KEY, nulle pour la clé Firebase côté application ; dépend de l'accès au dépôt |
| Gravité | Moyenne : la clé APP_KEY est identique sur tous les environnements, production comprise |
| Statut | À corriger (révocation et renouvellement des secrets) |

## Contexte de la détection

Une étape Gitleaks a été ajoutée en tête du pipeline. Elle parcourt l'ensemble de l'historique du dépôt, et pas seulement l'état actuel des fichiers, à la recherche de secrets : clés, mots de passe, clés privées. Elle a signalé deux fuites, toutes deux datant de juillet 2025 et présentes sur les branches principales du dépôt distant.

## Description des fuites

| Fichier publié | Contenu sensible | État actuel |
|---|---|---|
| Fichier de configuration d'environnement de l'API | Clé APP_KEY de l'application ; mot de passe de la base de données locale | Retiré du dépôt, mais encore lisible dans l'historique |
| Fichier de compte de service Firebase | Clé privée du compte de service du projet Firebase de l'application | Retiré du dépôt, mais encore lisible dans l'historique |

Supprimer un fichier d'un dépôt git ne l'efface pas de l'historique : toute personne ayant accès au dépôt, ou à une copie ancienne de celui-ci, peut relire ces secrets.

## Analyse de l'impact

**Clé APP_KEY.** AdonisJS utilise cette clé pour chiffrer des données et signer les cookies de session. L'analyse du code montre que l'application n'utilise aujourd'hui ni le chiffrement de données ni les sessions par cookie : l'authentification repose sur des jetons d'accès enregistrés en base de données, qui ne dépendent pas de cette clé. L'impact direct est donc faible aujourd'hui.

Le risque vient de deux points :
- la clé publiée est identique sur tous les environnements, production comprise (confirmé le 13/11/2026) ;
- toute fonctionnalité future qui utiliserait le chiffrement ou les sessions reposerait sur une clé déjà connue.

**Mot de passe de la base de données.** Il concerne une base locale, accessible uniquement depuis la machine elle-même. Il doit cependant être changé s'il a été réutilisé sur un autre environnement.

**Clé Firebase.** Firebase avait été prévu pour les notifications, mais le service qui l'utilisait n'a pas été branché dans l'application. Il a été remplacé en octobre 2025 par les notifications web standard, puis le code Firebase a été retiré en décembre 2025. La clé ne sert donc plus à l'application, mais elle reste valide côté Google tant qu'elle n'est pas révoquée : une personne qui la récupère peut agir sur le projet Firebase avec les droits de ce compte de service.

## Décision sur l'historique

Réécrire l'historique pour effacer les secrets a été écarté :
- cela modifie toutes les versions publiées et oblige chaque développeur et chaque serveur (préproduction, production) à resynchroniser son dépôt ;
- cela ne protège pas contre les copies déjà faites.

La mesure efficace est de rendre les secrets inutilisables en les révoquant et en les renouvelant. Les deux fuites connues sont déclarées comme exceptions dans le fichier d'exceptions de Gitleaks, avec leur justification. Le pipeline continue ainsi de bloquer toute nouvelle fuite, sans bloquer sur ces fuites anciennes. Ces exceptions ne sont justifiées qu'une fois la révocation faite.

## Correction

1. **Clé Firebase** : révoquer la clé du compte de service dans la console Google Cloud, et supprimer le compte de service s'il n'a plus d'usage.
2. **Clé APP_KEY** : générer une clé distincte pour chaque environnement (développement, préproduction, production) avec la commande prévue par AdonisJS, puis redémarrer chaque service. Comme aucune donnée chiffrée ni session ne dépend de cette clé aujourd'hui, le changement est sans effet pour les utilisateurs : les jetons de connexion restent valides. Le changement en production se fait après accord de Vincent Nilles.
3. **Mot de passe de la base** : le changer sur tout environnement où il aurait été réutilisé.
4. **Prévention** : le fichier de configuration d'environnement est exclu du suivi git, et Gitleaks bloque désormais toute nouvelle fuite dès l'envoi du code.

## Enseignement

Un secret publié dans git reste exposé même après la suppression du fichier. La réponse adaptée est la révocation du secret, et non l'effacement de sa trace. Une clé par environnement limite aussi l'impact d'une fuite : la découverte de la clé de développement n'aurait alors donné aucun accès à la production.
