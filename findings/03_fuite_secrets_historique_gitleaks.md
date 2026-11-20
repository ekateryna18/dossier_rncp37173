# Constat 03 : secrets présents dans l'historique du dépôt (clé APP_KEY, clé Firebase)

| Élément | Valeur |
|---|---|
| Date | 13/11/2026 |
| Application | Centre Art & Danse, serveur applicatif (API) |
| Détecté par | Gitleaks, analyse de tout l'historique git, dans le pipeline d'intégration continue |
| Type | Secrets publiés dans le gestionnaire de versions (CWE-312 : stockage en clair d'informations sensibles ; CWE-540 : informations sensibles dans le code source) |
| Exposition | Limitée : lecture réservée aux personnes disposant d'un compte sur le serveur Gitea de l'entreprise ou d'une copie du dépôt |
| Exploitabilité actuelle | Nulle pour la clé APP_KEY (aucune fonction de l'API ne l'utilise) ; possible pour la clé Firebase tant qu'elle n'est pas révoquée |
| Gravité | Faible |
| Statut | À corriger par hygiène (révocation et renouvellement des secrets) |

## Contexte de la détection

Une étape Gitleaks a été ajoutée en tête du pipeline. Elle parcourt l'ensemble de l'historique du dépôt, et pas seulement l'état actuel des fichiers, à la recherche de secrets : clés, mots de passe, clés privées. Elle a signalé deux fuites, toutes deux datant de juillet 2025 et présentes sur les branches principales du dépôt.

## Description des fuites

| Fichier publié | Contenu sensible | État actuel |
|---|---|---|
| Fichier de configuration d'environnement de l'API | Clé APP_KEY de l'application ; mot de passe de la base de données locale | Retiré du dépôt, mais encore lisible dans l'historique |
| Fichier de compte de service Firebase | Clé privée du compte de service du projet Firebase de l'application | Retiré du dépôt, mais encore lisible dans l'historique |

Supprimer un fichier d'un dépôt git ne l'efface pas de l'historique : toute personne pouvant lire le dépôt, ou disposant d'une copie ancienne de celui-ci, peut relire ces secrets.

## Qui peut lire ces secrets

Le dépôt est hébergé sur le serveur Gitea de l'entreprise et sa lecture exige un compte. Les personnes concernées sont donc peu nombreuses : l'équipe de développement actuelle et passée. Deux points restent à vérifier pour confirmer cette exposition limitée :
- **la visibilité du dépôt** : le serveur Gitea répond sur une adresse publique ; le dépôt doit être configuré en privé pour que seuls les comptes autorisés y accèdent ;
- **les accès des anciens membres de l'équipe** : un ancien développeur peut encore disposer d'un compte actif ou d'une copie locale du dépôt. Son compte doit être désactivé ou ses droits retirés sur le dépôt.

Une fuite ne vient pas seulement d'une attaque extérieure : une copie du dépôt sur un poste perdu ou volé, ou un accès resté ouvert après un départ, suffit à exposer l'historique.

## Analyse de l'impact

**Clé APP_KEY.** Dans AdonisJS, cette clé sert à quatre usages : chiffrer des données, signer les cookies, signer les sessions de connexion par cookie et signer des liens temporaires. Avec la clé, un attaquant pourrait déchiffrer ces données, fabriquer un faux cookie de session ou créer un faux lien signé.

L'analyse du code de l'API montre qu'aucun de ces quatre usages n'existe : pas de configuration de session, pas de chiffrement de données, pas de lien signé, pas de cookie signé. La connexion repose sur des jetons d'accès enregistrés en base de données, qui ne dépendent pas de cette clé. **Aujourd'hui, la clé ne protège rien : la connaître ne donne aucun accès.**

Deux raisons justifient néanmoins son renouvellement :
- **les fonctions futures** : un lien de téléchargement signé ou une session ajoutés plus tard reposeraient sur une clé déjà connue, sans que personne s'en souvienne ;
- **une clé unique pour tous les environnements** : la même clé est utilisée en développement, en préproduction et en production (confirmé le 13/11/2026). Quiconque a vu la clé de développement connaît aussi celle de la production, ce qui contredit le principe d'un secret propre à chaque environnement.

**Mot de passe de la base de données.** Il concerne une base locale, accessible uniquement depuis la machine elle-même. Il doit être changé s'il a été réutilisé sur un autre environnement.

**Clé Firebase.** Firebase avait été prévu pour les notifications, mais le service qui l'utilisait n'a pas été branché dans l'application. Il a été remplacé en octobre 2025 par les notifications web standard, puis le code Firebase a été retiré en décembre 2025. La clé ne sert donc plus à l'application. En revanche, contrairement à la clé APP_KEY, elle peut être utilisée directement depuis internet contre le projet Firebase chez Google, avec les droits du compte de service, tant qu'elle n'est pas révoquée. C'est le point le plus sérieux de ce constat, malgré l'exposition limitée du dépôt.

## Évaluation du risque

| Facteur | Évaluation |
|---|---|
| Qui peut lire les secrets | Personnes disposant d'un compte Gitea ou d'une copie du dépôt : peu nombreuses |
| Ce que permet la clé APP_KEY aujourd'hui | Aucun accès : aucune fonction de l'API ne l'utilise |
| Ce que permet la clé Firebase | Un accès au projet Firebase, si elle est encore active |
| Risque futur | Une fonction ajoutée plus tard reposerait sur une clé APP_KEY connue |
| Coût de la correction | Très faible : une commande par environnement, sans effet pour les utilisateurs |

**Gravité retenue : faible.** Il ne s'agit pas d'une urgence, mais d'une mesure d'hygiène à appliquer parce qu'elle ne coûte presque rien. La révocation de la clé Firebase est à traiter en premier.

## Décision sur l'historique

Réécrire l'historique pour effacer les secrets a été écarté :
- cela modifie toutes les versions publiées et oblige chaque développeur et chaque serveur (préproduction, production) à resynchroniser son dépôt ;
- cela ne protège pas contre les copies déjà faites.

La mesure efficace est de rendre les secrets inutilisables en les révoquant et en les renouvelant. Les deux fuites connues sont déclarées comme exceptions dans le fichier d'exceptions de Gitleaks, avec leur justification. Le pipeline continue ainsi de bloquer toute nouvelle fuite, sans bloquer sur ces fuites anciennes. Ces exceptions ne sont justifiées qu'une fois la révocation faite.

## Correction

1. **Clé Firebase** : révoquer la clé du compte de service dans la console Google Cloud, et supprimer le compte de service s'il n'a plus d'usage.
2. **Clé APP_KEY** : générer une clé distincte pour chaque environnement (développement, préproduction, production) avec la commande prévue par AdonisJS, puis redémarrer chaque service. Aucune donnée chiffrée ni session ne dépendant de cette clé, le changement est sans effet pour les utilisateurs : les jetons de connexion restent valides. Le changement en production se fait après accord de Vincent Nilles.
3. **Mot de passe de la base** : le changer sur tout environnement où il aurait été réutilisé.
4. **Accès au dépôt** : vérifier que le dépôt est privé dans Gitea et retirer les droits des anciens membres de l'équipe.
5. **Prévention** : le fichier de configuration d'environnement est exclu du suivi git, et Gitleaks bloque désormais toute nouvelle fuite dès l'envoi du code.

## Enseignement

Une alerte de fuite se qualifie en deux temps : qui peut lire le secret, et ce que le secret permet réellement. Ici, l'exposition est limitée et la clé APP_KEY ne protège aucune fonction aujourd'hui : le risque est faible. La réponse adaptée reste la révocation des secrets plutôt que l'effacement de leur trace, et une clé par environnement pour qu'une fuite en développement ne donne aucun accès à la production.
