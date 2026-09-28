# Procédure de veille, VNWeb
### RNCP37173, M1.4, document associé à la politique de sécurité

*Document de travail du 21/09/2026. Il complète la PSSI (`Politique_Securite_VNWeb.md`) : il décrit comment VNWeb suit quatre familles d'évolutions qui peuvent rendre ses règles insuffisantes ou obsolètes sans que personne ne s'en aperçoive : les textes et référentiels (conformité), les failles de sécurité, les menaces et attaques, la fin de support des logiciels et les changements chez les fournisseurs. Il répond au principe PR-8 (amélioration continue) et prolonge la section 7 (référentiels applicables). Le nom du fichier est conservé, car des renvois y pointent ; l'ancien titre ne couvrait que la conformité.*

*Les statuts de l'information suivent ceux de la PSSI : **Confirmé**, **Déclaré** (dit par l'équipe, non vérifié), **À préciser** ou **À vérifier**. Les valeurs chiffrées sont des choix de VNWeb, marquées « valeur proposée » et soumises à la validation de la direction. La mention **[à vérifier]** signale un nom de source, d'outil ou une affirmation sur leur fonctionnement : elle est levée après vérification quand le fact-check du 21/09/2026 l'a confirmé au niveau L2, et conservée quand il ne l'a pas pu. Le résultat du fact-check est résumé en annexe.*

Références : ISO/IEC 27001:2022 A.5.6, A.5.7, A.5.22, A.5.31, A.5.36, A.8.8 (numéros concordants sur des sources tierces, texte de la norme non consulté [à vérifier]).

---

## 1. Objet et principe

La PSSI s'appuie sur des textes et des référentiels extérieurs (le RGPD, les guides de l'ANSSI, les recommandations de la CNIL, la norme ISO 27001, OWASP), et sur des composants techniques que VNWeb ne maîtrise pas entièrement : systèmes, logiciels, services d'hébergement. Tout cela évolue : nouvelle version d'un guide, décision de la CNIL, faille dans un logiciel, campagne d'attaque, fin de support d'une version. Une règle écrite en 2026 peut être dépassée en 2027 sans que rien ne le signale. Cette procédure organise la surveillance de ces évolutions.

**Quatre thèmes, un filtre.** Un thème de veille existe seulement s'il peut déclencher une action dans la PSSI : c'est le principe de proportionnalité (PR-6). Une information qui ne peut donner lieu à aucune action n'est pas surveillée, même si elle est intéressante.

**Trois temps séparés : collecter, trier, décider.**

- La **collecte** est mécanique : un lecteur de flux RSS, sans intelligence artificielle, relève les nouveautés d'une liste fermée de sources officielles (section 4). Un flux RSS est un fil de nouveautés publié par un site et lu par un logiciel. L'auditrice ajoute les sources à son lecteur.
- Le **tri et le résumé** sont faits par l'intelligence artificielle (IA), à partir des seuls éléments du flux : elle ne cherche rien d'elle-même.
- La **décision** est humaine : le référent sécurité lit, qualifie et propose, la direction décide (section 6).

Ce partage déplace le risque. L'IA n'a plus l'occasion d'oublier de chercher ou de s'appuyer sur un site secondaire, mais ce qui n'est pas dans la liste de sources n'est pas vu : le choix des sources devient le point sensible du dispositif (section 4).

- Aucune règle de la PSSI n'est modifiée parce qu'un fichier de veille le suggère. Toute modification passe par le chemin de décision écrit du §8.3 de la PSSI (proposition, réponse de la direction sous 30 jours, ou fiche de dérogation).
- Une exception de délai : pour une faille critique, un correctif de sécurité relève déjà de la règle R-34 et ne demande pas de décision préalable ; un chemin court est prévu (6.5).

**Ce que cette procédure promet, et ce qu'elle ne promet pas.** Elle ne garantit pas qu'aucune évolution n'échappe à la surveillance : un dispositif de veille ne peut pas l'affirmer, et une source oubliée ou un flux incomplet créent des angles morts. Elle garantit ceci : chaque élément qui figure dans une source surveillée est lu dans un délai connu, classé, tracé dans un registre, et donne lieu à une décision écrite quand il touche une règle ; et un raté, quand il est constaté, est noté et corrigé (4.5). C'est ce qui rend l'actualité de la PSSI vérifiable.

---

## 2. Les quatre thèmes

| Thème | Volet | Ce qu'il surveille | Ce qu'il déclenche dans la PSSI | Cadence de lecture (valeur proposée) |
|---|---|---|---|---|
| **1. Conformité** (existe déjà, automatisé sur Claude : **Déclaré**) | Organisationnel | RGPD, CNIL, ANSSI, ISO 27001, NIS2 et DORA | Modification de règle, dérogation (§8.3, règle R-05) | Hebdomadaire |
| **2. Failles de sécurité** (nouveau) | Technique | Failles des systèmes, des frameworks, des dépendances et des services AWS | Correctif (règle R-34) | Immédiate si faille critique, sinon hebdomadaire |
| **3. Menaces et attaques** (nouveau) | Organisationnel et technique | Hameçonnage, rançongiciels visant les PME et les agences web, fraude par usurpation (VO-05) | Contenu de la formation (section 21) ; point d'attention pour la gestion d'incident (section 17) | Mensuelle ; annuelle pour le *Panorama de la cybermenace* de l'ANSSI |
| **4. Fin de support et fournisseurs** (nouveau) | Technique | Versions qui n'auront plus de correctifs (système, PHP, environnements d'exécution AWS) ; incidents et changements chez AWS, Discord, Gitea et les hébergeurs | Migration planifiée (inventaire, règle R-33) ; revue de l'exigence envers un prestataire (section 13) | Mensuelle |

**Ce qui n'est pas surveillé, et pourquoi.**

- Les journaux et alertes internes : c'est de la supervision (section 12), pas de la veille.
- Les exigences des clients : la question est posée à chaque nouveau contrat (ISO 27001:2022 A.5.31), elle n'est pas suivie en continu.
- Les tendances générales du secteur : aucune règle de la PSSI n'en découle.

### 2.1 Thème 1, conformité : référentiels et poids

Tous les référentiels n'ont pas le même poids. Le RGPD est une obligation légale et prévaut sur la PSSI en cas de divergence (PSSI §7) ; les guides de l'ANSSI et de la CNIL sont des recommandations ; ISO 27001 et OWASP sont des références de sécurité que VNWeb choisit de suivre. Le poids détermine l'urgence de la réaction.

| Référentiel | Poids | Sections de la PSSI qui en dépendent | Ce qu'on guette |
|---|---|---|---|
| RGPD (notamment chapitre V, articles 28, 30, 33), et les décisions, guides et sanctions de la CNIL et du Comité européen de la protection des données | Obligation légale | 13 (prestataires), 15 (données et conformité), 17 (incident, à venir) | Nouvelle décision ou doctrine de la CNIL, évolution du cadre applicable aux transferts hors Union européenne (il conditionne la règle R-57 : table `users` de weevus en Ohio), évolution des délais et formes de notification d'une fuite |
| Directive NIS2 et règlement DORA (règlement (UE) 2022/2554) | À établir : applicabilité à VNWeb non déterminée (PSSI §7) | Aucune à ce jour | NIS2 : état de la transposition en droit français. DORA : règlement, donc sans transposition [à vérifier : conséquence générale du droit de l'Union, source non ouverte], champ d'application (entités financières et leurs prestataires de services informatiques). Réévaluer avec la direction si VNWeb, ou ses clients, entrent dans le champ |
| ANSSI, *Guide d'hygiène informatique* (42 mesures, édition de 2017 ; version exacte à confirmer en ouvrant le PDF [à vérifier]) | Recommandation | 9 à 15 | Nouvelle version du guide, nouvelles mesures, mesures retirées |
| ANSSI, *Recommandations relatives à l'authentification multifacteur et aux mots de passe* (8 octobre 2021) ; CNIL, recommandation sur les mots de passe et autres secrets partagés (délibération n° 2022-100 du 21 juillet 2022) ; recommandation de la CNIL sur l'authentification multifacteur (date et numéro à vérifier) | Recommandation | 9 (accès), 10 (secrets) | Révision de ces textes, notamment sur la longueur des mots de passe et les facteurs recommandés |
| ISO 27001:2022, Annexe A | Norme volontaire, grille de comparaison sans visée de certification | Références citées en tête des sections 8 à 16 | Amendement, nouvelle édition, changement de numérotation ou de contenu d'un contrôle cité |
| OWASP, *Code Review Guide* et *DevSecOps Guideline* | Recommandation suivie volontairement ; projets OWASP de type Lab (*Code Review Guide*) et Incubator (*DevSecOps Guideline*), non finalisés | 14 (développement et livraison) | Nouvelle version des guides |

### 2.2 Les sources officielles par thème

Ces noms forment la liste de départ de la liste blanche (4.1). L'existence d'un flux RSS n'est pas établie pour chaque source : elle est indiquée ci-dessous quand elle a été constatée, et à vérifier source par source sinon (VEI-I, VEI-O). Les adresses ouvertes lors du fact-check figurent en annexe.

**Thème 1, conformité**

- ANSSI ; CNIL ; Comité européen de la protection des données.
- EUR-Lex, portail officiel du droit de l'Union européenne : il propose des flux RSS prédéfinis (Journal officiel séries L et C, législation, jurisprudence) ; la liste actuelle est à revérifier sur le site, la seule copie lue date de 2020 [à vérifier].
- Légifrance, service public de la diffusion du droit : données ouvertes et interface de programmation (API) ; existence d'un flux RSS non établie. Repli (4.1) : consultation mensuelle par le référent, ou lecture par l'API.
- ISO, publications relatives à la norme 27001 : existence d'un flux RSS incertaine [à vérifier] (le site n'a pas pu être ouvert par l'outil du fact-check, erreur 403). Test à faire à la main dans un navigateur : ouvrir la fiche de la norme 27001 sur le site de l'ISO et chercher un lien RSS ou une alerte d'état. Repli : consultation manuelle mensuelle, ou alerte de l'organisme national de normalisation AFNOR [à vérifier].
- OWASP (projets *Code Review Guide* et *DevSecOps Guideline*).

**Thème 2, failles de sécurité**

- CERT-FR, rubriques « Avis de sécurité » et « Alertes de sécurité », chacune avec son flux RSS.
- Catalogue des failles connues comme déjà exploitées de la CISA, l'agence américaine de cybersécurité (nom anglais : *Known Exploited Vulnerabilities*) : téléchargeable en CSV et en JSON ; pas de flux RSS constaté. Repli (4.1) : lecture du fichier JSON par un petit programme sans IA, ou consultation manuelle.
- AWS Security Bulletins (bulletins de sécurité d'AWS), avec flux RSS.
- Bulletins de sécurité de la distribution installée sur les serveurs : la distribution exacte est **À préciser** (le dossier cite l'outil `apt`, sans nommer la distribution).
- Avis des éditeurs des composants inventoriés : pour Gitea, voir le thème 4 ; pour les frameworks et bibliothèques des projets : **À préciser**, car leur liste n'est pas connue (5.1).
- Rapports des analyseurs de dépendances (5.3) : source interne, qui n'est pas un flux public.

**Thème 3, menaces et attaques**

- Cybermalveillance.gouv.fr, dispositif national d'assistance aux victimes de cybermalveillance, avec flux RSS (actualités, alertes, fiches réflexes). L'hameçonnage y est traité ; pour les rançongiciels, le terme n'a pas été retrouvé sur la page ouverte [à vérifier].
- CERT-FR, rubrique « Rapports Menaces et incidents », avec son flux dédié.
- ANSSI, *Panorama de la cybermenace* : publication **annuelle** (édition 2025 parue le 11 mars 2026, référence CERTFR-2026-CTI-002). Elle est rangée en veille annuelle, à côté de la lecture mensuelle des autres sources.

**Thème 4, fin de support et fournisseurs**

- AWS : calendrier de dépréciation des environnements d'exécution de Lambda (page « Lambda runtimes » de la documentation AWS), dates prévisionnelles susceptibles de changer ; préavis d'au moins 180 jours envoyés par courriel au contact principal du compte, dans l'AWS Health Dashboard et dans Trusted Advisor : ce n'est pas un flux RSS. Avis d'incident AWS : AWS Health Dashboard, à confirmer [à vérifier]. Action à décider : mettre le contact principal du compte AWS sur une boîte partagée lue par le référent (VEI-P).
- Calendrier de fin de vie de la distribution installée : **À préciser** (distribution non nommée).
- PHP : calendrier des versions prises en charge (page « Supported Versions » de php.net, à consulter à la main : ce n'est pas un flux RSS). Point actionnable, pris sur cette page : PHP 8.2 sort du support de sécurité le 31 décembre 2026, soit dans un peu plus de trois mois à la date de ce document, PHP 8.3 le 31 décembre 2027, PHP 8.4 le 31 décembre 2028. Le dossier ne dit pas quelles versions de PHP tournent : à relever (VEI-G, VEI-Q).
- Gitea : notes de version du blog Gitea, dont chaque publication liste les failles corrigées. Flux RSS non établi : repli à prévoir (4.1).
- Discord : page d'état de Discord, avec flux RSS et Atom.
- Les deux prestataires d'hébergement : leurs avis arrivent probablement par courriel plutôt que par un flux RSS ; canal **À préciser**.

---

## 3. Comment la veille fonctionne aujourd'hui

### 3.1 Thème 1, la collecte automatisée actuelle : **Déclaré**

L'auditrice a mis en place, sur Claude, des **tâches automatisées** qui s'exécutent **chaque matin**, une par thème de veille. Un de ces thèmes est la **conformité** : chaque matin, l'agent produit un fichier `.md` qui rassemble les dernières nouveautés en matière de conformité, notamment celles de l'ANSSI, du RGPD (CNIL), de l'ISO 27001 et des autres référentiels du tableau de la section 2.1.

Ce dispositif est un premier niveau de veille, mis en place avant cette procédure. Elle le complète par la partie qui manquait : la lecture, le tri, la trace et la décision. Les sections 4 à 6 décrivent la cible, qui change la manière de collecter (lecteur de flux à la place d'une recherche par l'agent).

Éléments à établir pour que le dispositif soit vérifiable :

| Élément | Statut | Action |
|---|---|---|
| Heure d'exécution et durée de conservation des fichiers | À préciser | Consigner l'heure ; conserver les fichiers au moins 12 mois (valeur proposée) |
| Emplacement où les fichiers sont déposés, et qui y a accès | À préciser | Un dossier partagé, lisible par le référent et son suppléant |
| Sources que l'agent interroge (sites officiels ou moteur de recherche généraliste) | À préciser | Remplacer par la liste blanche de la section 4.1 |
| Liste des autres thèmes de veille et de leurs tâches | À préciser | La lister (VEI-F) |
| Personne qui reçoit et lit le fichier chaque matin | À préciser | Le référent sécurité, et à défaut son suppléant (section 7) |

### 3.2 Thèmes 2 à 4 : statut **À préciser**

L'auditrice n'a pas indiqué lesquels des thèmes 2, 3 et 4 sont déjà couverts par une tâche automatisée, ni sous quelle forme. Tant que ce point n'est pas éclairci (VEI-F), les thèmes 2 à 4 sont décrits comme une cible et non comme une pratique constatée. De même, l'état du lecteur de flux (installé ou non, sources déjà ajoutées) est **À préciser**.

### 3.3 Les limites de l'automatisation

Le fichier de tri est produit par un modèle d'intelligence artificielle. Trois limites en découlent, et la procédure les traite :

1. **Il peut se tromper** : oublier une nouveauté, résumer de travers, confondre deux versions d'un texte, se tromper de date, ou s'appuyer sur un site secondaire plutôt que sur le texte officiel. Règle : **un fichier de veille est un signal, pas une preuve.** Tout élément retenu est vérifié dans le texte officiel (page de l'ANSSI, de la CNIL, texte du règlement, publication de l'ISO, avis de l'éditeur) avant qu'une décision s'y appuie. La section 4 réduit ce risque (sources fermées, lien et date obligatoires, contrôle de complétude) sans le supprimer.
2. **Il peut s'arrêter sans bruit** : une tâche qui échoue ne produit pas de fichier, et personne ne s'en rend compte. Règle : le référent contrôle la présence des fichiers lors de sa lecture hebdomadaire ; l'absence de fichier pendant plus de 3 jours (valeur proposée) est signalée et corrigée.
3. **Il ne juge pas l'effet sur VNWeb** : l'agent recense ce qui a changé, pas ce que cela change pour une équipe de quatre développeurs, une centaine de projets et deux prestataires d'hébergement. Ce jugement est humain (section 6).

---

## 4. Fiabiliser la veille

*Statut : dispositif cible, décidé ; sa mise en place n'est pas confirmée (À préciser). Ce qui suit dit comment il doit fonctionner.*

### 4.1 Une liste blanche de sources, sans moteur de recherche généraliste

La collecte ne lit que des sources officielles, nommées à l'avance (2.2). Aucune information n'est cherchée par un moteur de recherche généraliste ni relevée sur un site secondaire. Pour chaque source, une fiche consigne :

| Source | Thème | Flux RSS | Date de vérification | Ajoutée le, par qui, motif |
|---|---|---|---|---|
| | | Existence à vérifier | | |

Deux règles complètent la liste :

- Ajouter une source à la liste se fait sur proposition du référent, avec le motif et la date consignés ; une source qui n'y figure pas n'est pas lue.
- Une source sans flux RSS n'est pas abandonnée : un repli est choisi source par source (lettre d'information de la source, lecture par interface de programmation, lecture d'un fichier par un petit programme sans IA, ou consultation manuelle mensuelle par le référent) et inscrit dans la fiche.

État de départ des sources pour lesquelles aucun flux RSS n'est établi :

| Source | Repli prévu |
|---|---|
| Légifrance | Consultation mensuelle par le référent, ou lecture par l'API |
| Catalogue des failles exploitées de la CISA | Lecture du fichier JSON par un petit programme sans IA, ou consultation manuelle |
| ISO | Test manuel du site (2.2) ; à défaut, consultation manuelle mensuelle, ou alerte AFNOR [à vérifier] |
| PHP, page « Supported Versions » | Consultation manuelle |
| Gitea, notes de version du blog | À prévoir |
| Lambda, calendrier de dépréciation des environnements d'exécution | Courriel au contact principal du compte AWS, AWS Health Dashboard, Trusted Advisor (2.2) |
| Prestataires d'hébergement | Courriel ; canal à préciser |

### 4.2 Forme du fichier de tri

L'IA ne reçoit que les éléments du flux. Le fichier qu'elle produit respecte quatre consignes :

- **Une ligne par élément**, avec le lien vers la source et la date de publication, obligatoires.
- **Un élément sans lien est rejeté** : il n'est ni retenu ni résumé.
- **« Rien de nouveau »** est une réponse valide : quand le flux ne contient aucun élément sur la période, le fichier l'écrit, plutôt que de meubler avec des informations tirées d'ailleurs.
- L'IA n'ajoute aucune information extérieure au flux.

### 4.3 Contrôle de complétude, sans IA

Un petit programme (script), sans IA, compare le **nombre d'éléments du flux** au **nombre de lignes du fichier**, source par source. Il vérifie aussi que chaque ligne porte un lien et une date. Un écart signale au référent que l'IA a omis ou rejeté des éléments ; le fichier est alors refait ou complété avant lecture. Le résultat du contrôle est conservé avec le fichier. Ce contrôle prouve que rien n'a été perdu entre le flux et le fichier ; il ne prouve pas que le flux contient tout ce que la source a publié (4.4). Il ne compare que les éléments présents dans le flux au moment de la lecture : la fenêtre du flux (nombre d'éléments conservés) est relevée pour chaque source. Test à faire avant de figer cette règle : lire un flux réel deux jours de suite et compter les éléments (VEI-S).

### 4.4 Test mensuel de couverture

Chaque mois, le référent prend les **cinq dernières publications d'une source officielle** (une source différente chaque mois, valeur proposée) et vérifie leur présence dans les fichiers du mois. Le résultat, par exemple 5 sur 5, est un indicateur (section 8). Une publication absente est inscrite au journal des ratés (4.5) avec sa cause probable : flux incomplet, élément perdu par l'IA, source mal classée.

### 4.5 Journal des ratés

Quand une nouveauté est trouvée ailleurs que dans les fichiers de veille (par un développeur, par le test mensuel, par un client), elle est notée, puis la source ou la consigne est ajustée.

| Date | Nouveauté trouvée | Où elle a été trouvée | Pourquoi elle manquait (source hors liste, flux incomplet, omission de l'IA, consigne) | Ajustement | Date de l'ajustement |
|---|---|---|---|---|---|
| | | | | | |

### 4.6 Ce que cela ne garantit pas

Une source absente de la liste reste invisible ; un flux peut être publié avec retard ou rester incomplet ; une faille peut être exploitée avant d'être publiée. La liste blanche, le contrôle de complétude, le test de couverture et le journal des ratés réduisent l'écart et le rendent mesurable. Ils ne le suppriment pas.

---

## 5. Veille des failles de sécurité, fin de support, fournisseurs et menaces

*Statut : dispositif cible, décidé ; sa mise en place n'est pas confirmée (À préciser).*

### 5.1 Prérequis : un inventaire de ce qu'on utilise

Une alerte sur une faille n'a de sens que rapprochée de ce que VNWeb utilise réellement. Or l'inventaire actuel est incomplet.

- La règle **R-33** couvre les serveurs, les services et les logiciels installés (nom, rôle, environnement, hébergeur, responsable, version du système, date de dernière mise à jour).
- Elle ne couvre **pas les frameworks et bibliothèques** des environ cent projets hébergés. **C'est un trou :** une alerte sur une bibliothèque ne peut pas être rapprochée d'un projet, on lit l'alerte sans savoir si l'on est concerné.

Les technologies des projets sont **À préciser** : le dossier ne cite que PHP, sans version ni framework. Elles ne sont pas déduites ici.

| Élément à inventorier | Statut |
|---|---|
| Langages et versions des projets hébergés | À préciser (seul PHP est cité) |
| Frameworks et bibliothèques par projet | À préciser |
| Gestionnaires de paquets utilisés par projet | À préciser |
| Services AWS de weevus : Amplify, Cognito, Lambda, DynamoDB, S3 | Déclaré (J1) ; environnements d'exécution de Lambda : À préciser |
| Distribution et version du système des serveurs | À préciser |

Les analyseurs de dépendances lisent les dépendances exactes verrouillées de chaque projet (fichiers de verrouillage, par exemple `composer.lock` pour un projet PHP qui utilise Composer, ce qui reste à préciser). Leur rapport liste les failles trouvées, pas l'inventaire complet ; l'inventaire se construit à partir des fichiers de verrouillage eux-mêmes [à vérifier avec l'outil retenu]. La règle qui étend R-33 aux dépendances applicatives n'est pas écrite ici : elle sera écrite à la rédaction de la section 19 de la PSSI (VEI-N).

### 5.2 Les quatre couches surveillées

| Couche | Ce qu'on surveille | Comment | Qui | Ce que ça déclenche |
|---|---|---|---|---|
| **Serveurs** : vnw2, serveurs locaux (dont Gitea et l'Extranet) ; le serveur des sites de voyance suit le même principe par son prestataire | Mises à jour du système et des logiciels installés | Règle R-34 ; bulletins de sécurité de la distribution. Sur les serveurs externes, le prestataire applique et VNWeb demande par écrit (cahier d'exigences, règle R-43) | Référent (serveurs locaux) ; prestataires (serveurs externes) | Mise à jour selon les délais de 5.3 |
| **Dépendances des projets** | Failles connues dans les bibliothèques et frameworks de chaque projet | Analyseur de dépendances dans la livraison (règles R-53 et R-54) et outil de mises à jour automatiques ; outils à choisir selon les langages, à confirmer (VEI-H) | Développeurs, pour leurs propres projets | Correctif du projet, selon 5.3 |
| **Cloud (weevus)** | Failles et fins de support des services AWS utilisés | AWS Security Bulletins et avis de dépréciation d'AWS (2.2) | Référent, en lien avec le développeur de weevus | Correctif ou migration planifiée (thème 4) |
| **Alertes officielles** | Failles annoncées pour tout produit inventorié | Flux CERT-FR (avis et alertes) et catalogue CISA des failles déjà exploitées (fichier JSON lu par un petit programme, pas de flux RSS constaté), rapprochés de l'inventaire | Référent | Repérage puis chemin de 6.5 si critique |

Outils cités comme candidats, **à choisir selon les langages réellement utilisés** (VEI-H) :

- **Composer, pour PHP** : la commande `composer audit` interroge les avis de sécurité de Packagist ; l'option `--locked` la fait porter sur le fichier de verrouillage.
- **OSV-Scanner** : analyseur multi-langages, PHP inclus via `composer.lock`.
- **Renovate**, pour les mises à jour automatiques : pris en charge avec Gitea (fusion automatique à partir de Gitea 1.24 ; la version de Gitea de VNWeb est à relever, VEI-Q). Avec AWS CodeCommit, la prise en charge est expérimentale, son développement est gelé, et il n'y a ni fusion automatique ni affectation de relecteurs. CodeCommit est de nouveau en disponibilité générale chez AWS depuis le 24 novembre 2025.

La règle R-54 prévoit déjà « l'outil d'audit du gestionnaire de paquets utilisé ».

### 5.3 Priorité et délais

**Priorité.** Une faille déjà exploitée dans la nature passe avant une faille dont le score de gravité théorique est élevé mais qui n'est pas exploitée : le risque réel est plus grand quand des attaquants s'en servent déjà. Cette priorité est choisie par VNWeb ; elle est cohérente avec la consigne de la CISA (directive BOD 22-01) de traiter en priorité les failles connues comme exploitées. La CISA recommande d'utiliser son catalogue comme critère d'entrée, à combiner avec une évaluation du risque, pas comme seul critère. Le catalogue et les alertes du CERT-FR servent à savoir si une faille est exploitée. Les délais de la CISA valent pour les agences fédérales américaines ; ceux de VNWeb, ci-dessous, sont des valeurs proposées par VNWeb. Statut actuel de la directive BOD 22-01 : [à vérifier] (VEI-R).

**Délais proposés**, en cohérence avec la règle R-34 (mises à jour mensuelles, et sans attendre en cas de faille critique annoncée) :

| Situation | Délai de correction (valeur proposée) |
|---|---|
| Faille critique, déjà exploitée | Sous 72 heures |
| Faille critique, non exploitée à ce jour | Sous 7 jours |
| Autre faille | Dans le cycle mensuel de la règle R-34 |

Le seuil qui rend une faille « critique » suit la classification de la source (avis officiel, éditeur) ; il reste à préciser (VEI-L). La règle R-54 reprend ces délais depuis l'harmonisation du 21/09/2026 (7 jours pour une faille critique dans une dépendance, 72 heures si elle est déjà exploitée). Reste à les répercuter sur la règle R-34 (serveurs locaux) à la rédaction de la section 19 (VEI-K). Pour les serveurs externes, ces délais sont à demander aux prestataires : le cahier d'exigences (R-43) reprend R-34 mais pas encore ces délais.

### 5.4 Le rôle de l'IA pour les failles

L'IA produit un **résumé hebdomadaire lisible** pour le référent, à partir des rapports des analyseurs de dépendances et des éléments des flux (les rapports listent les failles trouvées, 5.1). Elle ne **détecte pas** les failles : la détection est faite par les analyseurs et les sources officielles. Le résumé est un signal ; la vérification se fait à la source (3.3).

### 5.5 Volet fin de support et fournisseurs (thème 4)

Ce volet surveille ce qui cessera d'être corrigé et ce qui change chez les fournisseurs : versions du système, de PHP, environnements d'exécution AWS ; incidents et changements chez AWS, Discord, Gitea et les hébergeurs. Lecture mensuelle (valeur proposée). Deux suites possibles :

- **Migration planifiée** : l'élément est rapproché de l'inventaire (règle R-33) et une échéance est inscrite au registre. Un exemple déjà connu, traité avant la mise en place de cette procédure : Amplify Gen 1 est en mode maintenance depuis le 1er mai 2026 (correctifs critiques et de sécurité seulement) et arrive en fin de vie le 1er mai 2027 (annonce dans le dépôt officiel aws-amplify/amplify-cli, ticket n° 14881) [la page de documentation Amplify consultée ne donne pas la date : à relire directement, VEI-R]. À la date de ce document (21/09/2026), Gen 1 est donc déjà en maintenance. Ces éléments sont cohérents avec VT-27 et la règle R-59 (statut Déclaré), qui ne sont pas modifiées.
  - Deuxième exemple actionnable, PHP : la version 8.2 sort du support de sécurité le 31 décembre 2026, la 8.3 le 31 décembre 2027, la 8.4 le 31 décembre 2028 (page « Supported Versions » de php.net). Les versions de PHP en service chez VNWeb ne sont pas connues : à relever (VEI-G, VEI-Q), puis à rapprocher de ces dates dans l'inventaire (règle R-33).
  - Autres dates de fin de support (distribution, environnements d'exécution de Lambda) : elles sont à relever aux sources de la section 2.2 ; aucune n'est indiquée ici.
- **Revue de l'exigence envers un prestataire** (section 13) : un changement chez un hébergeur ou un incident qu'il signale conduit à relire le cahier d'exigences et le contrat (règles R-43 et R-45). Un changement chez Discord est examiné au regard de la règle R-17 (aucun secret dans Discord).

### 5.6 Volet menaces (thème 3) : le condensé mensuel

Ce volet suit les hameçonnages, les rançongiciels visant les PME et les agences web, et la fraude par usurpation (VO-05, règle R-22). Chaque mois, le référent rédige un **condensé** à partir du tri de l'IA : quelques éléments retenus, chacun avec son lien et sa date. Le condensé est envoyé à toute l'équipe (cinq minutes de lecture) et sert à deux choses :

- alimenter le **contenu de la formation** (section 21) ;
- signaler un **point d'attention pour la gestion d'incident** (section 17), par exemple un mode opératoire qui change la manière de reconnaître un incident.

Un élément de ce thème ne modifie pas une règle par lui-même : s'il conduit à en proposer une, il suit le chemin du §8.3 (6.4).

---

## 6. Traitement : de la lecture à la décision

### 6.1 Le rythme

| Fréquence | Activité | Qui | Ce qui en reste |
|---|---|---|---|
| En continu | Collecte des nouveautés des sources de la liste blanche | Lecteur de flux (sans IA) | Éléments des flux |
| Chaque jour ouvré pour les thèmes 1 et 2 (valeur proposée) | Tri et résumé par l'IA ; contrôle de complétude (4.3) | IA ; script sans IA | Fichier `.md` daté, archivé, avec le résultat du contrôle |
| Chaque semaine (valeur proposée) | Lecture des fichiers des thèmes 1 et 2, tri selon les niveaux de 6.2, contrôle de la présence des fichiers, lecture du résumé hebdomadaire des failles | Référent (suppléant en son absence) | Éléments N1 et N2 inscrits au registre (6.3) |
| Dès repérage d'une faille critique | Chemin court (6.5) | Référent, développeurs, direction | Ligne au registre, correctif daté |
| Chaque mois (valeur proposée) | Tri des thèmes 3 et 4 ; condensé mensuel des menaces ; test de couverture (4.4) | IA ; référent | Condensé diffusé ; éléments du thème 4 au registre ; résultat du test |
| Sous 30 jours après inscription d'un élément N2 (valeur proposée) | Vérification dans le texte officiel, repérage des règles touchées, proposition écrite à la direction | Référent, direction | Fiche instruite ; décision selon le §8.3 |
| Chaque année, à la parution (édition 2025 parue le 11 mars 2026) | Lecture du *Panorama de la cybermenace* de l'ANSSI (thème 3), rapprochée du contenu de la formation (section 21) | Référent | Ligne au registre et, si besoin, mise à jour du condensé |
| Chaque trimestre | Point de veille dans le compte rendu de sécurité (règle R-03) : éléments repérés, en cours, décidés ; synthèse trimestrielle ; revue de l'inventaire (règle R-33) | Référent | Une ligne « veille » dans le compte rendu daté |
| Chaque année | Revue complète de la section 7 de la PSSI : version en vigueur de chaque référentiel, comparaison avec celle citée dans la PSSI, revue de la PSSI (principe PR-8) | Référent, direction | Section 7 de la PSSI à jour, date de la vérification consignée |
| Hors cycle | Nouveau texte majeur, incident, nouveau type de client ou de données, changement de prestataire : revue immédiate des sections concernées | Référent, direction | Registre et PSSI mis à jour |

Les cadences sont un choix de proportion (principe PR-6) : lire un fichier chaque matin serait une charge que quatre développeurs ne tiendraient pas, et il finirait par ne plus être lu. Le fichier quotidien des thèmes 1 et 2 existe pour que la lecture hebdomadaire ne soit pas le seul moment où une faille critique peut être vue ; le moyen de prévenir le référent sans attendre est à préciser (VEI-L).

### 6.2 Les niveaux de tri

| Niveau | Définition | Traitement |
|---|---|---|
| **N0, sans objet** | Information générale, hors du périmètre de VNWeb, ou déjà connue | Aucune action ; le fichier reste archivé |
| **N1, à surveiller** | Texte en projet, consultation en cours, évolution annoncée, sans effet immédiat | Inscrit au registre avec une date de réexamen |
| **N2, impact possible sur la PSSI** | Nouvelle obligation, nouvelle version d'un référentiel de la section 7 de la PSSI, décision ou doctrine qui touche une règle ; faille touchant un composant de l'inventaire ; fin de support ou changement de fournisseur touchant un élément de l'inventaire ; menace qui conduit à modifier la formation ou un point d'attention d'incident | Inscrit au registre, vérifié à la source, instruit et proposé à la direction (6.4) |
| **Urgent** | Échéance légale proche, évolution qui rend une règle actuelle contraire à la loi, ou faille critique déjà exploitée qui touche un composant utilisé | Le référent prévient la direction le jour même, sans attendre la lecture hebdomadaire suivante ; chemin court pour une faille (6.5) |

En cas de doute entre deux niveaux, on retient le plus élevé.

### 6.3 Le registre de veille

Le registre est le document qui prouve que la veille est suivie et pas seulement produite. Une ligne par élément N1 ou N2. Le référent le tient ; le suppléant y a le même accès.

| Réf | Thème | Date de repérage | Source officielle vérifiée | Résumé en une phrase | Référentiel ou composant | Niveau | Règles ou sections touchées | Décision | Responsable | Échéance ou date de réexamen |
|---|---|---|---|---|---|---|---|---|---|---|
| VEI-01 | | | | | | | | | | |

Le champ « Thème » prend la valeur 1, 2, 3 ou 4 (section 2). Le champ « Décision » prend l'une de ces valeurs : *aucune action* (avec le motif), *à instruire*, *PSSI modifiée* (avec la référence de la version), *dérogation signée* (règle R-05), *à réexaminer à la date indiquée*. Trois valeurs s'y ajoutent pour les thèmes techniques et de menace : *correctif appliqué* (avec la date, thème 2), *migration planifiée* ou *exigence prestataire à revoir* (avec l'échéance, thème 4), *intégré au condensé du mois* (thème 3). Un élément ne se clôt qu'avec une de ces valeurs, ce qui évite qu'un signal reste sans suite.

### 6.4 Ce que devient un élément N2

1. **Vérifier** dans la source officielle : que l'information existe, à quelle date elle s'applique, à qui elle s'adresse. Ni le fichier de tri ni le résumé de l'IA ne sont cités comme source : le lien de l'élément mène au texte officiel.
2. **Repérer les règles touchées** dans la PSSI (numéro de section, identifiant R-xx) et, si besoin, la charte informatique et les contrats avec les prestataires.
3. **Proposer par écrit** à la direction : ce qui change, la mesure envisagée, le risque traité, l'effort estimé. La direction répond sous 30 jours (règle R-04).
4. **Appliquer la décision** : modifier la PSSI (la version et la date de modification sont consignées et l'équipe est informée), ou signer une dérogation datée (règle R-05).
5. **Clore la ligne du registre** avec la décision et sa date.

### 6.5 Le chemin court pour une faille critique

Une faille critique qui touche un composant utilisé ne passe pas par la proposition écrite sous 30 jours : le correctif de sécurité applique déjà la règle R-34.

1. **Repérer** : l'élément est classé Urgent dans le fichier quotidien, ou signalé par un développeur.
2. **Vérifier à la source** (avis du CERT-FR, éditeur, catalogue CISA) que la faille existe, quels produits et quelles versions elle touche, et si elle est déjà exploitée ; **rapprocher de l'inventaire** pour savoir quels serveurs ou quels projets sont concernés.
3. **Prévenir** le même jour la direction, le développeur du projet touché et, pour un serveur externe, le prestataire.
4. **Corriger** dans le délai de 5.3 (72 heures si la faille est exploitée, 7 jours sinon, valeurs proposées). Si le délai ne peut pas être tenu, une mesure d'atténuation est appliquée et consignée, et le report fait l'objet d'une dérogation datée (règle R-05).
5. **Consigner** au registre : date de publication de la faille, date du correctif, personne. Ces deux dates alimentent l'indicateur de délai (section 8).
6. Si une exploitation de la faille sur les systèmes de VNWeb est suspectée, ouvrir la gestion d'incident (section 17 de la PSSI, à venir).

---

## 7. Rôles

D : décide. R : réalise. C : est consulté. I : est informé. Un tiret : non concerné. Mêmes lettres que le §8.2 de la PSSI.

| Activité | Direction | Référent | Suppléant | Développeurs |
|---|---|---|---|---|
| Contrôler que la veille se fait (fichiers présents, registre à jour) | I | R | R en son absence | - |
| Lire et trier les thèmes 1 et 2, tenir le registre | I | R | R en son absence | C |
| Surveiller les dépendances de ses propres projets (alertes de l'analyseur) et appliquer les correctifs | I | C (priorité) | - | R |
| Signaler une information vue hors du dispositif | - | I | I | R |
| Lire le condensé mensuel des menaces (thème 3), qui sert la formation | I | R (rédige) | - | I (5 minutes de lecture) |
| Décider des suites d'un élément qui touche une règle | D | R (propose) | - | C |
| Prendre connaissance de la synthèse trimestrielle | I | R (rend compte) | - | I |

**Principe.** Le référent pilote la veille et répond devant la direction. Les développeurs surveillent et corrigent les composants de leurs propres projets. Tout le monde reçoit le condensé mensuel. Une responsabilité unique par activité évite que chacun pense qu'un autre a vu. Le suppléant tient les fonctions du référent en son absence, avec les mêmes accès (principe PR-5 : la veille ne repose pas sur une seule personne).

**Charge de temps du référent :** environ une heure par semaine en moyenne (valeur proposée), à valider par la direction, qui alloue le temps et les moyens (§8.1 de la PSSI) ; elle est à mesurer sur les premiers mois, puis à ajuster (VEI-J).

**Maintenance du dispositif.** L'auditrice, à l'origine du dispositif automatisé, maintient les tâches (consignes données à l'agent, sources, planification) tant que cette maintenance n'est pas confiée au référent : à décider (VEI-C).

---

## 8. Indicateurs et preuves

| Indicateur | Comment on le mesure | Cible (valeur proposée) |
|---|---|---|
| Fichiers de veille produits | Nombre de fichiers présents sur le nombre de jours ouvrés, par thème | Pas d'absence de plus de 3 jours consécutifs |
| Lecture hebdomadaire | Semaines lues et inscrites au registre sur les semaines écoulées | 100 % |
| Éléments N2 traités dans le délai | Éléments N2 ayant reçu une décision sous 30 jours, sur le total | 100 % |
| Éléments du registre sans décision au-delà de leur échéance | Comptage à chaque revue trimestrielle | Zéro |
| Fraîcheur des référentiels | Date de la dernière vérification annuelle de chacun des référentiels de la section 2.1 | Moins de 12 mois |
| Couverture mensuelle des sources | Publications retrouvées dans les fichiers du mois sur les cinq vérifiées (4.4) | 5 sur 5 ; tout écart inscrit au journal des ratés |
| Délai de correction des failles critiques | Durée entre la publication de la faille et le correctif appliqué, par faille | 72 heures si exploitée, 7 jours sinon (5.3) |
| Part des projets couverts par un analyseur de dépendances | Projets actifs analysés sur le nombre de projets actifs | Progressive, 100 % à la vague 3 de la PSSI (18.4, règle R-54) |
| Fraîcheur de l'inventaire | Âge de la dernière revue de l'inventaire (règle R-33) | Moins de 3 mois (revue trimestrielle de R-33) |

Les preuves à conserver sont : les fichiers de veille archivés avec le résultat du contrôle de complétude, le registre, le journal des ratés, les condensés mensuels, les résultats du test de couverture, les comptes rendus trimestriels et le journal des versions de la PSSI.

---

## 9. Points restant à établir

VEI-A à VEI-F viennent de la version précédente ; ils sont reformulés quand les décisions de cette version les ont modifiés. VEI-G à VEI-S sont nouveaux (VEI-O à VEI-S viennent du fact-check du 21/09/2026).

| Réf | Point | Pourquoi |
|---|---|---|
| VEI-A | Valider la liste blanche de la section 2.2 (sources officielles par thème) et l'ajouter au lecteur de flux ; pour chaque source, la fiche de 4.1 est renseignée. L'indication de la source et de sa date pour chaque nouveauté n'est plus un point ouvert : elle est une consigne obligatoire (4.2) | Limiter le risque d'un résumé appuyé sur un site secondaire (3.3) |
| VEI-B | Mettre en place le contrôle automatique de la présence du fichier du jour et le contrôle de complétude (4.3), ou à défaut un rappel daté dans l'agenda du référent | Une tâche qui s'arrête ne doit pas passer inaperçue (3.3) |
| VEI-C | Décider qui maintient les tâches automatisées, le lecteur de flux et le script de contrôle quand l'auditrice n'est plus disponible | Éviter que le dispositif repose sur une seule personne (PR-5) |
| VEI-D | Évaluer avec la direction l'applicabilité de NIS2 et de DORA à VNWeb et à ses clients | Décision restée en suspens en section 7 de la PSSI |
| VEI-E | Rattacher cette procédure aux sections 19 (veille : conformité, failles de sécurité, menaces, fin de support), 20 (vérification de la conformité), 21 (formation), 17 (gestion d'incident) et 23 (suivi de la politique et revue annuelle) de la PSSI, et à la règle R-03 | Faire de la veille l'une des entrées de la revue annuelle |
| VEI-F | Dresser la liste des tâches automatisées existantes sur Claude, et indiquer pour chacune si elle couvre l'un des thèmes 2 à 4 ou si elle sort du filtre de la section 1 (dans ce cas, elle n'est pas rattachée à la PSSI) | Savoir quels thèmes sont déjà automatisés (3.2) |
| VEI-G | Établir les technologies des projets : langages et versions, frameworks, gestionnaires de paquets (le dossier ne cite que PHP) ; distribution et version des serveurs | Prérequis de la veille des failles (5.1) |
| VEI-H | Choisir l'analyseur de dépendances et l'outil de mises à jour automatiques selon les langages, et vérifier ce que chaque outil fait réellement | Les candidats de 5.2 sont documentés, mais leur adéquation aux langages réels de VNWeb n'est pas établie, et l'inventaire issu des fichiers de verrouillage reste à vérifier avec l'outil retenu (5.1) |
| VEI-I | Vérifier, source par source, l'existence d'un flux RSS et choisir un repli pour les sources qui n'en ont pas (ISO en particulier) | L'existence d'un flux n'est pas garantie ; l'état constaté par le fact-check est en 2.2 et 4.1 (VEI-O pour les tests manuels) |
| VEI-J | Faire valider par la direction la charge de temps du référent (environ une heure par semaine) et la mesurer sur les premiers mois | §8.1 de la PSSI : la direction alloue le temps et les moyens |
| VEI-K | Harmoniser les délais de 5.3 (72 heures, 7 jours) avec la règle R-34 (serveurs locaux : au moins chaque mois, sans attendre en cas de faille critique annoncée) et avec le cahier d'exigences aux prestataires (R-43). La règle R-54 est déjà alignée (21/09/2026) | Éviter deux délais différents pour la même faille |
| VEI-L | Définir le seuil « critique », la notion de faille « exploitée » et le moyen de prévenir le référent sans attendre la lecture hebdomadaire (notification du lecteur de flux, courriel, ou fichier quotidien) | Sans cela, « immédiate » n'a pas de contenu (6.1) |
| VEI-M | Décrire la chaîne technique : passage des éléments du lecteur de flux à l'IA, emplacement des fichiers, exécution du script de complétude, qui le maintient | Rendre le dispositif reproductible et vérifiable (3.1, 4.3) |
| VEI-N | Écrire, à la rédaction de la section 19 de la PSSI, une règle sur l'inventaire des dépendances applicatives, en extension de R-33. Aucune règle n'est créée par ce document | R-33 ne couvre pas les frameworks et bibliothèques des projets (5.1) |
| VEI-O | Tester à la main l'existence d'un flux RSS pour ISO (fiche de la norme 27001 sur le site de l'ISO), Légifrance et EUR-Lex (liste actuelle des flux prédéfinis), et arrêter le repli de chacune | L'outil du fact-check n'a pas pu ouvrir le site de l'ISO, Légifrance n'a pas de flux établi, et la seule copie lue de la liste des flux d'EUR-Lex date de 2020 (2.2, 4.1) |
| VEI-P | Décider de mettre le contact principal du compte AWS sur une boîte partagée lue par le référent | Les préavis de dépréciation des environnements d'exécution de Lambda partent par courriel vers ce contact (2.2) |
| VEI-Q | Relever les versions de PHP et de Gitea en service chez VNWeb (complète VEI-G) | Comparer aux dates de fin de support de PHP (5.5) et à la prise en charge de Renovate avec Gitea (5.2) |
| VEI-R | Relire directement à la source les points lourds que le fact-check n'a lus que par résumé : date de fin de vie d'Amplify Gen 1, statut de DORA comme règlement sans transposition, statut actuel de la directive BOD 22-01 de la CISA | Points qui fondent des délais ou des échéances (5.3, 5.5, 2.1) |
| VEI-S | Lire un flux réel deux jours de suite et compter les éléments, pour connaître la fenêtre du flux de chaque source | Le contrôle de complétude ne compare que ce qui est dans le flux au moment de la lecture (4.3) |

---

## Annexe. Vérification des sources par fact-check, 21/09/2026

Ce tableau ne reprend que les adresses ouvertes par le fact-check pendant la session du 21/09/2026. La colonne « Niveau » reprend le niveau de preuve retenu : L2 correspond à une documentation officielle ou à une page officielle de l'organisme concerné ; « à relire directement » signale un point à relire à la source (VEI-R). Le rapport de fact-check ne détaille pas le niveau affirmation par affirmation : la rédaction n'a pas pu le reconstituer plus finement.

| Source | Ce qui est confirmé | Niveau | Adresse ouverte |
|---|---|---|---|
| CERT-FR | Rubriques « Avis de sécurité », « Alertes de sécurité » et « Rapports Menaces et incidents », chacune avec son flux RSS | L2 | https://www.cert.ssi.gouv.fr/ (flux relevés sur la page : /feed/, /alerte/feed/, /avis/feed/, /cti/feed/) |
| CISA, catalogue des failles exploitées | Téléchargeable en CSV et en JSON ; pas de flux RSS constaté | L2 | https://www.cisa.gov/known-exploited-vulnerabilities-catalog |
| CISA, directive BOD 22-01 | Consigne de traiter en priorité les failles connues comme exploitées ; catalogue à utiliser comme critère d'entrée, à combiner avec une évaluation du risque ; délais valables pour les agences fédérales américaines ; statut actuel à vérifier | L2, à relire directement | https://www.cisa.gov/news-events/directives/bod-22-01-reducing-significant-risk-known-exploited-vulnerabilities |
| AWS Security Bulletins | Bulletins de sécurité d'AWS, avec flux RSS | L2 | https://aws.amazon.com/security/security-bulletins/ |
| AWS Lambda, environnements d'exécution | Calendrier de dépréciation à dates prévisionnelles ; préavis d'au moins 180 jours par courriel au contact principal du compte, dans l'AWS Health Dashboard et dans Trusted Advisor ; pas un flux RSS | L2 | https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html |
| Amplify Gen 1, ticket du dépôt officiel | Mode maintenance depuis le 1er mai 2026, fin de vie le 1er mai 2027 ; la page de documentation Amplify consultée ne donne pas la date | L2 pour l'annonce ; date à relire directement | https://github.com/aws-amplify/amplify-cli/issues/14881 |
| Cybermalveillance.gouv.fr | Dispositif national d'assistance aux victimes ; flux RSS (actualités, alertes, fiches réflexes) ; hameçonnage confirmé ; « rançongiciels » non confirmé sur la page ouverte | L2 ; rançongiciels : non confirmé | https://www.cybermalveillance.gouv.fr/ et https://www.cybermalveillance.gouv.fr/tous-les-fils-d-infos |
| ANSSI, *Panorama de la cybermenace* | Publication annuelle ; édition 2025 parue le 11 mars 2026, référence CERTFR-2026-CTI-002 | L2 | https://cyber.gouv.fr/nous-connaitre/publications/panoramas-de-la-cybermenace/panorama-de-la-cybermenace-2025/ |
| PHP, versions prises en charge | Fin du support de sécurité : 8.2 le 31/12/2026, 8.3 le 31/12/2027, 8.4 le 31/12/2028 ; consultation manuelle | L2 | https://www.php.net/supported-versions.php |
| Gitea, blog | Chaque publication de version liste les failles corrigées ; flux RSS non établi | L2 | https://blog.gitea.com/release-of-1.27.0/ |
| Discord, page d'état | Flux RSS et Atom | L2 | https://discordstatus.com/ |
| Composer | `composer audit` interroge les avis de sécurité de Packagist ; option `--locked` | L2 | https://getcomposer.org/doc/03-cli.md#audit |
| OSV-Scanner | Analyseur multi-langages, PHP inclus via `composer.lock` | L2 | https://google.github.io/osv-scanner/ |
| Renovate, Gitea et CodeCommit | Gitea pris en charge (fusion automatique à partir de Gitea 1.24) ; CodeCommit en prise en charge expérimentale, développement gelé, sans fusion automatique ni affectation de relecteurs | L2 | https://docs.renovatebot.com/modules/platform/gitea/ et https://docs.renovatebot.com/modules/platform/codecommit/ |
| DORA, page de la Commission européenne | Règlement (UE) 2022/2554 ; la partie « sans transposition » reste à vérifier | L2, à relire directement | https://finance.ec.europa.eu/regulation-and-supervision/financial-services-legislation/implementing-and-delegated-acts/digital-operational-resilience-regulation_en |
| CNIL, RGPD | Page de la CNIL sur le règlement européen sur la protection des données | L2 | https://www.cnil.fr/fr/reglement-europeen-protection-donnees |
| CNIL, mots de passe | Recommandation sur les mots de passe et autres secrets partagés, délibération n° 2022-100 du 21 juillet 2022 | L2 | https://www.cnil.fr/fr/mots-de-passe-une-nouvelle-recommandation-pour-maitriser-sa-securite |
| ANSSI, authentification multifacteur | Recommandations relatives à l'authentification multifacteur et aux mots de passe, 8 octobre 2021 | L2 | https://messervices.cyber.gouv.fr/guides/recommandations-relatives-lauthentification-multifacteur-et-aux-mots-de-passe |
| ANSSI, *Guide d'hygiène informatique* | 42 mesures, édition de 2017 ; version exacte à confirmer en ouvrant le PDF | L2, version à vérifier | https://messervices.cyber.gouv.fr/guides/guide-dhygiene-informatique |
| OWASP | *Code Review Guide* : projet de type Lab ; *DevSecOps Guideline* : projet de type Incubator ; non finalisés | L2 | https://owasp.org/www-project-code-review-guide/ et https://owasp.org/www-project-devsecops-guideline/ |

**Sources non ouvertes ou non lisibles.**

- Site de l'ISO : erreur 403, non ouvert par l'outil (existence d'un flux RSS non établie).
- Pages actuelles d'EUR-Lex : chargées vides ; la seule copie lue de la liste des flux date de 2020.
- Norme ISO/IEC 27001:2022 : payante, non consultée ; les numéros de contrôles cités viennent de sites tiers concordants (niveau L4). Le seuil L1 exigé pour une référence de conformité n'est pas atteint.

**Réserve de méthode.** Le fact-check a été fait avec un outil de lecture qui résume les pages ouvertes. Les points lourds (date de fin de vie d'Amplify Gen 1, DORA, directive BOD 22-01 de la CISA) sont à relire directement à la source (VEI-R).

**Confiance globale annoncée par le fact-check :** 10 affirmations sur 15 au niveau L2 ou plus (67 %), avant les corrections de cette version.

---

*Document de travail. La partie du dispositif actuel (section 3.1) est déclarative : les éléments « À préciser » sont à confirmer avant de présenter ce document comme une preuve de veille appliquée. Les thèmes 2 à 4 et le dispositif de fiabilisation (sections 4 et 5) décrivent une cible décidée, dont la mise en place et les noms de sources marqués [à vérifier] restent à confirmer.*
