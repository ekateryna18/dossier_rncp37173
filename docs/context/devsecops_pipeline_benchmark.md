# Benchmark outillage DevSecOps — pipeline CI/CD à valider

> Document de comparaison d'outils, à valider par l'utilisateur avant mise en œuvre. Il ne contient aucun fichier de configuration ni aucun code : c'est un choix d'outillage à trancher, pas une implémentation. CI/CD désigne ici l'intégration continue (vérifications automatiques à chaque changement) et le déploiement continu (mise en production automatisée une fois les vérifications passées). Périmètre et constats de départ : `docs/context/context_project.md` et `docs/context/plan_audit.md`.

## Pourquoi cet ordre

Chaque étape ci-dessous porte un numéro qui correspond à sa position réelle dans la chaîne d'exécution : une étape plus tôt dans la liste s'exécute avant celles qui suivent, sur chaque changement de code. L'idée directrice est d'arrêter un changement problématique au point le moins coûteux possible : d'abord les vérifications rapides et locales (syntaxe, types), puis les tests, puis l'analyse de sécurité du code, puis la construction et le déploiement, et enfin l'analyse dynamique une fois qu'une version tourne réellement quelque part.

**Précision sur « déjà en place ».** Pour certaines étapes (1 et 3 en particulier), l'outil existe déjà dans le code du projet — installé en dépendance, configuré, utilisable en local par un développeur qui lance la commande à la main. Mais sans pipeline, rien ne déclenche cette commande automatiquement à chaque changement. « Déjà en place » signifie donc : l'outil est prêt, pas qu'il tourne déjà en continu. Construire le pipeline consiste, pour ces étapes-là, à écrire le `job` qui appelle une commande déjà existante — pas à installer un nouvel outil. Les étapes qui n'ont pas cette mention (détection de secrets, SAST, SCA, DAST) demandent, elles, l'ajout d'un outil qui n'existe pas encore dans le projet.

## Étape 1 — Qualité de code (lint, format, typage)

Première porte, avant tout le reste : elle ne coûte que quelques secondes et arrête les erreurs les plus basiques avant qu'elles n'avancent plus loin dans la chaîne.

| Domaine | Outil déjà en place | Couverture actuelle | Niv |
|---|---|---|---|
| Lint JavaScript/TypeScript | ESLint | `app-dance` et `api-dance` : configuré. `cdn-app-dance` : aucun script de lint défini. | L1 |
| Formatage | Prettier | Présent dans `api-dance` (via `project_tech_stack.md`) ; à confirmer pour les deux autres sous-projets. | L1 |
| Vérification de types | `tsc --noEmit` | `api-dance` : en place. `app-dance` et `cdn-app-dance` : TypeScript présent en dépendance, vérification en continu à confirmer. | L1 |

Il n'y a pas de choix d'outil à faire ici : ESLint, Prettier et `tsc --noEmit` sont déjà les standards adoptés dans deux des trois sous-projets et n'ont pas de raison d'être remplacés. Le point réel à traiter est un trou de couverture, pas un choix technique : `cdn-app-dance` n'a aucun script de lint, ce qui laisse ce sous-projet hors de cette première porte tant qu'un script `eslint` n'y est pas ajouté avec une configuration alignée sur les deux autres sous-projets.

## Étape 2 — Détection de secrets

S'exécute sur chaque `commit` (un enregistrement de changement dans l'historique du code) ou chaque `push` (l'envoi de ces changements vers le dépôt partagé), avant toute autre étape de sécurité. Objectif : empêcher qu'une clé d'API, un mot de passe ou un jeton se retrouve dans l'historique du code, y compris de façon transitoire.

| Outil | Licence / coût | Intégration CI | Portée | Niv |
|---|---|---|---|---|
| Gitleaks | Open source (MIT), gratuit | Action GitHub officielle, configuration en quelques lignes | Scan de l'historique complet ou du diff du push | L2 (documentation officielle du projet) |
| TruffleHog | Version open source gratuite ; offre Enterprise payante pour la vérification active des identifiants trouvés | Action GitHub disponible côté communauté et éditeur | Détection par motif plus vérification active des secrets trouvés (interroge le fournisseur pour confirmer qu'un secret est valide) en version payante | L2 (documentation éditeur) |
| Détection de secrets native GitHub (`secret scanning`) | Gratuite sur dépôt public ; sur dépôt privé, nécessite GitHub Advanced Security (licence additionnelle payante) | Native, aucune configuration de workflow nécessaire | Motifs de fournisseurs connus (partenaires GitHub), alertes directement dans l'onglet Sécurité du dépôt | L2 (documentation officielle GitHub) |
| detect-secrets (Yelp) | Open source (Apache 2.0), gratuit | Nécessite un script d'intégration manuel (pas d'action officielle) | Scan par motif avec fichier de référence (`baseline`) à maintenir | L3 (projet communautaire établi, sans benchmark chiffré public) |

Recommandation : Gitleaks en tête, pour un rapport coût/simplicité d'intégration adapté à une équipe de trois personnes — configuration minimale, action GitHub officielle, licence permissive. La détection native GitHub est un bon complément gratuit si le dépôt est public ; sur un dépôt privé sans licence GitHub Advanced Security, elle n'est pas disponible et Gitleaks reste alors la seule option gratuite de cette liste avec une intégration prête à l'emploi.

## Étape 3 — Tests automatisés (unitaires et intégration)

Une fois le code syntaxiquement correct et sans secret détecté, l'étape suivante vérifie que le comportement attendu est respecté. Traité par sous-projet, chacun ayant un point de départ différent.

| Sous-projet | Situation actuelle | Options | Niv |
|---|---|---|---|
| `api-dance` (backend) | Japa configuré (`@japa/runner`, `@japa/assert`, `@japa/api-client`, `@japa/plugin-adonisjs`), exécuté via `node ace test` ; aucun fichier de test réel constaté à ce jour | Japa (déjà en place) vs Vitest + Supertest (remplacement complet) | L1 (documentation AdonisJS) |
| `app-dance` (frontend) | Aucun framework de test configuré | Vitest vs Jest | L2 (documentation officielle des deux projets) |
| `cdn-app-dance` (service d'upload) | Aucun script de test défini (`test` en place-holder d'erreur) | Vitest + Supertest vs Jest + Supertest | L2 |

Pour `api-dance`, Japa est l'outil de test officiellement maintenu par l'éditeur d'AdonisJS et distribué avec le squelette du framework : il s'intègre nativement avec l'injection de dépendances et le cycle de vie HTTP d'AdonisJS, ce qu'un remplacement par Vitest devrait reconstruire à la main. Garder Japa est le choix cohérent ; le travail réel à faire n'est pas un changement d'outil mais l'écriture des tests eux-mêmes, absents à ce jour.

Pour `app-dance`, qui utilise Vite comme outil de build, Vitest partage la même configuration (`vite.config`) et le même moteur de transformation, ce qui réduit la duplication de configuration par rapport à Jest, qui nécessite une configuration de transformation séparée pour fonctionner avec Vite et les modules ES natifs. Vitest est le choix cohérent avec la stack déjà en place.

Pour `cdn-app-dance` (Express), aucun des deux runners n'a d'avantage structurel puisqu'il n'y a pas de dépendance à Vite ici ; le critère de cohérence d'équipe l'emporte. Recommandation : Vitest, pour rester sur un seul runner de test à travers les trois sous-projets côté JavaScript/TypeScript (`api-dance` gardant Japa comme cas particulier justifié), réduisant l'effort d'apprentissage à un seul outil supplémentaire plutôt que deux.

## Étape 4 — SAST (analyse statique de sécurité du code)

SAST : Static Application Security Testing, une analyse du code source à la recherche de motifs de vulnérabilité connus, sans exécuter l'application. S'exécute après les tests, sur le code jugé fonctionnellement correct.

| Outil | Licence / coût sur dépôt privé | Couverture JS/TypeScript | Intégration GitHub Actions | Niv |
|---|---|---|---|---|
| CodeQL | Gratuit sur dépôt public ; sur dépôt privé, inclus dans GitHub Advanced Security (licence additionnelle) | Bonne couverture JavaScript/TypeScript, requêtes maintenues par GitHub et la communauté de sécurité | Action officielle `github/codeql-action`, intégration directe à l'onglet Sécurité | L2 (documentation officielle GitHub) |
| Semgrep | Version CLI open source gratuite (règles communautaires) ; plateforme Semgrep AppSec Platform avec fonctionnalités additionnelles en offre payante | Bonne couverture JavaScript/TypeScript, règles orientées OWASP Top 10 disponibles en registre public | Action officielle `semgrep/semgrep-action`, configuration rapide | L2 (documentation officielle Semgrep) |
| SonarQube Community | Édition Community gratuite et auto-hébergée ; éditions payantes pour les fonctionnalités avancées (analyse de sécurité approfondie réservée aux éditions payantes dans certaines versions, à vérifier à l'installation) | Couverture large multi-langages, y compris JavaScript/TypeScript, avec un fort accent sur la dette technique en plus de la sécurité | Nécessite l'hébergement d'un serveur SonarQube (auto-hébergé ou SonarCloud, ce dernier étant un service géré séparé) — effort d'exploitation supérieur aux deux options précédentes | L2 (documentation officielle SonarSource) |

Recommandation : CodeQL et Semgrep en tête, à des rôles complémentaires plutôt qu'en substitution l'un de l'autre — CodeQL pour sa profondeur d'analyse de flux de données (utile sur un point comme le contrôle d'accès déjà identifié dans le plan d'audit) et son intégration native à l'onglet Sécurité GitHub, Semgrep pour la rapidité de mise en place et la personnalisation de règles ciblées (par exemple une règle dédiée pour repérer une configuration CORS permissive comme celle déjà relevée sur le canal `socket.io`). SonarQube Community demande un effort d'exploitation supplémentaire (hébergement d'un serveur) qui ne se justifie pas tant qu'aucune infrastructure de supervision n'existe déjà pour l'héberger et le maintenir.

## Étape 5 — SCA (analyse de composition logicielle / dépendances)

SCA : Software Composition Analysis, l'inventaire des bibliothèques tierces utilisées par le projet et la vérification qu'aucune ne porte de vulnérabilité connue et publiée. S'exécute en parallèle ou juste après le SAST, sur le même changement.

| Outil | Licence / coût | Portée | Intégration | Niv |
|---|---|---|---|---|
| Dependabot | Natif GitHub, gratuit sur dépôt public et privé | Alertes de vulnérabilité connues (base de données GitHub Advisory) et ouverture automatique de demandes de mise à jour des dépendances | Activation native dans les paramètres du dépôt, sans fichier de configuration obligatoire (un fichier `dependabot.yml` optionnel affine le comportement) | L2 (documentation officielle GitHub) |
| Snyk | Offre gratuite avec quota mensuel de scans limité ; offres payantes au-delà | Base de vulnérabilités propriétaire, souvent mise à jour plus vite que les bases publiques pour certains écosystèmes | Action GitHub officielle, tableau de bord dédié | L2 (documentation officielle Snyk, quotas à reconfirmer au moment de la mise en place) |
| `npm audit` | Inclus nativement avec `npm`, gratuit | Base de vulnérabilités `npm`/GitHub Advisory | Une seule ligne de commande dans un `job` de workflow, sans dépendance externe | L1 (documentation officielle npm) |
| OWASP Dependency-Check | Open source, gratuit | Base de données nationale des vulnérabilités (NVD), multi-langages | Nécessite un `job` dédié avec mise en cache de la base de données NVD pour rester performant en CI | L2 (projet OWASP, documentation officielle) |

Recommandation : Dependabot en tête, pour son intégration native gratuite sans configuration de workflow séparée et sa cohérence avec un dépôt déjà hébergé sur GitHub. `npm audit` en complément ponctuel dans le `job` de build (coût quasi nul) plutôt qu'en remplacement. Snyk et OWASP Dependency-Check restent des options valables si une couverture de vulnérabilités plus large ou multi-langages devient nécessaire, mais n'apportent pas d'avantage déterminant sur ce projet à ce stade.

## Étape 6 — Construction (build)

Position dans le pipeline seulement, sans comparaison d'outils : cette étape s'exécute une fois les portes précédentes (1 à 5) passées avec succès. Chaque sous-projet a déjà sa commande de construction : `vite build` pour `app-dance`, `node ace build` pour `api-dance`, `tsc` pour `cdn-app-dance`. Le pipeline se contente d'exécuter ces commandes existantes ; il n'y a pas de nouvel outil à choisir ici.

## Étape 7 — Déploiement vers un environnement (staging)

Position dans le pipeline seulement : une fois la construction réussie, le pipeline peut pousser le résultat vers l'environnement de staging (hébergé sur le serveur de VnWeb). L'outillage de déploiement lui-même est hors périmètre de ce benchmark. Un point de contexte à noter : l'absence de conteneurisation (aucun `Dockerfile` dans les trois sous-projets) élimine les options de déploiement standard basées sur une image de conteneur (registre d'images, orchestrateur) — un déploiement vers un serveur nu passe plutôt par une copie de fichiers et un redémarrage du processus Node (par exemple via SSH et un gestionnaire de processus comme PM2), une piste à creuser séparément si le sujet est repris.

## Étape 8 — DAST (analyse dynamique de sécurité)

DAST : Dynamic Application Security Testing, une analyse qui interroge l'application pendant qu'elle tourne réellement (contrairement au SAST qui lit le code sans l'exécuter). S'exécute après le déploiement en staging, contre cette cible.

| Outil | Licence / coût | Automatisation en CI | Portée | Niv |
|---|---|---|---|---|
| OWASP ZAP | Open source, gratuit, projet phare OWASP | Action GitHub officielle (`zaproxy/action-baseline` pour un scan rapide, `zaproxy/action-full-scan` pour une analyse plus complète) | Scan passif et actif des points d'entrée HTTP, règles orientées OWASP Top 10 | L2 (documentation officielle du projet OWASP ZAP) |
| Burp Suite Community | Gratuit | Pas d'automatisation en ligne de commande ni d'intégration CI officielle dans l'édition Community — usage manuel via interface graphique | Scan manuel, adapté à un test ponctuel plutôt qu'à une exécution automatique répétée | L2 (documentation officielle PortSwigger) |
| Burp Suite Professional | Payant, licence par utilisateur | Automatisation en ligne de commande disponible (Burp Suite Enterprise Edition va plus loin sur ce point, avec une licence encore supérieure) | Couverture large, moteur de scan reconnu dans la profession | L2 (documentation officielle PortSwigger) |
| Nikto | Open source, gratuit | Exécution en ligne de commande simple à intégrer dans un `job` | Analyse de serveur web plus généraliste, orientée configuration de serveur et fichiers exposés plutôt qu'analyse applicative en profondeur | L3 (projet communautaire de longue date, sans benchmark chiffré public) |

Recommandation : OWASP ZAP en tête, seul outil de cette liste combinant gratuité, automatisation native en CI via une action GitHub officielle, et une portée applicative directement alignée sur les vulnérabilités déjà visées par le reste du pipeline (OWASP Top 10). Burp Suite Community n'a pas de chemin d'automatisation en CI dans son édition gratuite, ce qui le rend adapté à un test ponctuel manuel mais pas à cette étape automatisée ; Burp Suite Professional reste une option à considérer plus tard si un test manuel approfondi complète le pipeline automatisé, mais représente un coût de licence à budgéter séparément.

## Étape 9 — Orchestrateur CI/CD (porte tout ce qui précède)

C'est l'outil qui exécute les huit étapes précédentes, dans l'ordre, à chaque changement de code.

| Outil | Coût sur dépôt privé | Intégration avec l'hébergement actuel | Niv |
|---|---|---|---|
| GitHub Actions | Quota de minutes gratuit mensuel selon le type de compte, au-delà facturé à la minute | Natif si le dépôt applicatif est hébergé sur GitHub — aucune plateforme supplémentaire à opérer | L1 (documentation officielle GitHub) |
| GitLab CI | Quota de minutes gratuit selon le type de compte GitLab, au-delà facturé | Nécessite un hébergement GitLab (ou une mise en miroir du dépôt vers GitLab), une plateforme supplémentaire si le dépôt principal reste ailleurs | L1 (documentation officielle GitLab) |
| Jenkins | Open source, gratuit | Nécessite d'installer et de maintenir un serveur Jenkins soi-même — effort d'exploitation le plus élevé de cette liste, alors qu'aucune infrastructure de supervision n'existe encore pour un service supplémentaire à surveiller | L2 (documentation officielle du projet) |
| CircleCI | Offre gratuite avec quota de crédits mensuels, au-delà facturé | Fonctionne avec un dépôt hébergé sur GitHub ou GitLab, sans remplacer l'hébergement | L2 (documentation officielle CircleCI) |

Le plan d'audit vérifie spécifiquement, dans son constat sur l'absence de pipeline, l'existence d'un dossier `.github/workflows` à la racine du dépôt applicatif — ce qui indique que la cible d'hébergement du code est déjà GitHub. Sur cette base, ce choix se resserre naturellement vers GitHub Actions : c'est la seule option native de cette liste par rapport à l'hébergement déjà en place, sans plateforme supplémentaire à opérer ni compte à créer ailleurs. Cette conclusion suppose que le dépôt applicatif est bien hébergé sur GitHub ; à confirmer directement auprès de l'équipe si un doute existe, puisque ce document n'a pas d'accès direct au dépôt de code de l'application (seul le dossier de diplôme est disponible dans cet espace de travail).

## Hors périmètre — scan de conteneur / image

Non applicable à ce stade : aucun `Dockerfile` n'existe dans les trois sous-projets (`app-dance`, `api-dance`, `cdn-app-dance`), donc aucune image de conteneur à scanner. Des outils comme Trivy ou Grype couvrent ce besoin et seraient à comparer séparément si une conteneurisation est adoptée plus tard — ce n'est pas un manque de ce benchmark, c'est une étape qui n'existe pas encore dans l'architecture de déploiement actuelle.

## Pipeline recommandé (à valider)

1. Qualité de code — ESLint + Prettier + `tsc --noEmit` (déjà en place ; combler le trou sur `cdn-app-dance`).
2. Détection de secrets — Gitleaks.
3. Tests automatisés — Japa pour `api-dance`, Vitest pour `app-dance` et `cdn-app-dance`.
4. SAST — CodeQL et Semgrep, en complément l'un de l'autre.
5. SCA — Dependabot, complété par `npm audit` dans le `job` de build.
6. Construction — commandes de build déjà existantes par sous-projet (`vite build`, `node ace build`, `tsc`).
7. Déploiement en staging — outillage hors périmètre de ce document, à traiter séparément.
8. DAST — OWASP ZAP (scan de référence, puis scan complet si le temps d'exécution le permet).
9. Orchestrateur — GitHub Actions, sous réserve de confirmation que le dépôt applicatif est bien hébergé sur GitHub.
