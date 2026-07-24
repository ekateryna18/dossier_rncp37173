# Dossier de Présentation, RNCP37173

## Experte en Sécurité des Développements Informatiques, Niveau 7

**Candidate :** Ecaterina Munteanu **École :** AcadéNice, Nice **Année :** [à préciser] **Projet :** Centre Art & Danse : Sécurisation d'une application interne de gestion d'école de danse, déjà en production

---

## 1. Présentation, Profil, entreprise et lien avec le projet

### 1.1 Profil personnel

Je m'appelle Ecaterina Munteanu, je suis originaire de Roumanie et je prépare actuellement le titre RNCP37173, Experte en Sécurité des Développements Informatiques, Niveau 7, à AcadéNice à Nice.

Avant d'intégrer cette formation, j'ai obtenu une Licence en Informatique à l'Université Babeș-Bolyai de Cluj-Napoca, en Roumanie, une université classée parmi les premières du pays dans le domaine des sciences et de l'informatique. Cette formation m'a apporté une base solide et transversale en informatique : algorithmique, bases de données, développement logiciel, réseaux. Elle a été un socle indispensable, mais volontairement généraliste.

En regardant les postes que j'aurais aimé occuper, j'ai remarqué que tous exigeaient un master. C'est ce qui m'a décidée à poursuivre ma formation avec un master de niveau 7, spécialisé en sécurité des développements. Ce domaine me semblait à la fois crucial et insuffisamment maîtrisé dans les équipes de développement, et c'est vers lui que j'ai dirigé ma spécialisation.

Avant cette formation, j'ai réalisé deux stages en entreprise :

- **Stage 1, Développeur C# (2023), Accenture Industry X, Cluj-Napoca :** Migration d'une interface utilisateur de Windows Forms vers WPF, refactoring et amélioration de la qualité du code. Stack : C#, Windows Forms, WPF, POO, UDP, Apache Log4Net.
- **Stage 2, Développeur SAP ABAP (2024), MSG Systems, Cluj-Napoca :** Réalisation d'un projet client de gestion des réservations hôtelières en équipe Agile, conception, coordination, tests fonctionnels. Stack : SAP ABAP 7.4, ABAP Objects, RAP, Jira.

Ces deux expériences, bien que courtes, m'ont montré concrètement que la sécurité doit être intégrée dès la conception, pas ajoutée après coup.

### 1.2 Le choix d'AcadéNice

Intégrer AcadéNice n'a pas été un choix par défaut, mais une décision motivée par une conviction personnelle forte : je n'apprends véritablement que lorsque je peux pratiquer.

Lors de ma visite de découverte de l'école, c'est Ludovic, le directeur, qui m'a accueillie. La première chose qu'il m'a dite, et qui m'a immédiatement convaincue, est que l'enseignement chez AcadéNice repose sur la pratique avant la théorie. Ce positionnement pédagogique correspond exactement à la manière dont j'assimile le mieux les connaissances : en confrontant la théorie à la réalité, en expérimentant, en commettant des erreurs et en en tirant les leçons.

Ce que j'ai observé tout au long de cette année a pleinement confirmé ce premier ressenti. Chaque module abordé, qu'il s'agisse du RGPD (Règlement Général sur la Protection des Données), de la cybersécurité, de UX/UI ou encore du No-Code, a systématiquement été accompagné d'une mise en pratique concrète, soit lors des hackathons organisés par l'école, soit lors des séances de travaux pratiques. Ces formats d'apprentissage ont rendu les concepts abstraits accessibles et mémorisables, parce qu'ils étaient ancrés dans une expérience réelle, avec de vraies contraintes et de vrais résultats à produire.

C'est cette cohérence entre le discours pédagogique et sa mise en œuvre effective qui m'a décidée à rejoindre AcadéNice. Je ne regrette pas ce choix.

### 1.3 Contexte de l'alternance, VNWeb, Antibes

Je réalise mon alternance au sein de l'entreprise VNWeb, une agence web spécialisée dans le développement informatique, implantée à Antibes. VNWeb est une structure à taille humaine : cinq personnes au total, dont quatre développeurs et le dirigeant. Cette dimension réduite m'a permis d'avoir rapidement des responsabilités concrètes et de contribuer directement aux projets clients.

Mon maître d'apprentissage est Vincent Nilles.

Le rythme de l'alternance est le suivant : quatre jours en entreprise, un jour à l'école par semaine. Ce rythme très ancré en entreprise m'a offert une immersion professionnelle intensive, propice à l'acquisition rapide de compétences opérationnelles.

Mes missions principales chez VNWeb :

- Développement de nouvelles fonctionnalités applicatives pour les projets clients de l'agence
- Intégration et configuration de services tiers selon les projets (authentification, stockage de fichiers, envoi d'emails transactionnels)
- Application rigoureuse des conventions de codage et des bonnes pratiques de développement définies en interne

Parmi les projets de l'agence, celui qui structure le présent dossier est Centre Art & Danse : une application interne de gestion d'école de danse, composée d'un backend (`api-dance`), d'une application cliente (`app-dance`) et d'un service dédié à l'upload de fichiers (`cdn-app-dance`), déjà en production au moment où j'en ai pris en charge la dimension sécurité.

Mon rôle chez VNWeb consiste à implémenter les fonctionnalités demandées par le dirigeant, dans le respect des conventions internes. C'est dans ce cadre opérationnel que j'applique concrètement les notions de sécurité étudiées en formation : validation des entrées, contrôle des accès, bonnes pratiques de codage.

---

## 2. Présentation du Projet, Contexte

### 2.1 Contexte général

Centre Art & Danse est une application interne développée par VNWeb pour la gestion d'une école de danse : gestion des comptes (admin, professeurs, parents, élèves), communication par groupes et messagerie privée, suivi des groupes et des cours. Ce n'est pas un produit grand public : c'est un outil métier réservé au personnel et aux familles de l'école.

L'application a été mise en production avec un niveau de sécurité minimal : au moment où j'en ai pris en charge la dimension sécurité, aucun pipeline CI/CD, aucune analyse statique ou dynamique de sécurité, et aucune politique de sécurité formalisée n'accompagnaient sa mise en ligne. C'est ce constat qui a fait de la sécurisation de cette application ma mission au sein de VNWeb, au-delà du développement de fonctionnalités.

L'enjeu est renforcé par la nature des données traitées. La majorité des utilisateurs sont des enfants : les élèves de 15 ans ou plus disposent de leur propre compte, mais les mineurs de moins de 15 ans n'ont pas de compte autonome : leur profil (identité, date de naissance, coordonnées) est créé et géré par le compte de leur parent superviseur. L'absence de compte propre ne signifie pas absence de données : les informations personnelles des mineurs existent bien dans le système, et leur exposition en cas d'incident repose entièrement sur la sécurité du compte parent et de l'application elle-même. Une application déjà en production, traitant ce type de données sensibles, avec un niveau de sécurité minimal au départ, représente une fragilité réelle en cas d'attaque ou de fuite de données, pas un risque théorique.

### 2.2 Parties prenantes

| Acteur | Rôle | Périmètre |
|---|---|---|
| Vincent Nilles | Maître d'apprentissage, dirigeant VNWeb | Validation des orientations techniques et des priorités |
| Ecaterina Munteanu | Développeuse fullstack, alternante | Sécurité applicative sur `api-dance`, `app-dance`, `cdn-app-dance` ; dossier RNCP |
| [autres développeurs VNWeb impliqués sur ce projet] | [à préciser] | [à préciser] |

| Acteur externe | Rôle | Attentes |
|---|---|---|
| [Direction de l'école de danse] | Commanditaire, cliente de VNWeb | Application fiable, conforme sur les données des familles |
| Professeurs, parents, élèves | Utilisateurs finaux | Application utilisable au quotidien, sans exposition de leurs données ou de celles de leurs enfants |
| AcadéNice | Organisme de formation et évaluateur | Couverture des compétences du référentiel RNCP37173 |

### 2.3 Objectifs du projet

**Scope fonctionnel acquis** : le périmètre fonctionnel est développé et utilisé en production : gestion des comptes et des rôles (admin, professeur, superviseur, élève), gestion des groupes et des cours, publication de posts (de groupe ou globaux) avec système de likes, messagerie privée 1-à-1, notifications push, espace d'administration complet (utilisateurs, validation des comptes, gestion des cours).

**Objectifs restants** (constat détaillé dans le plan d'audit et de sécurisation, sections 6 à 9 de ce dossier) :

- Mettre en place un pipeline CI/CD avec des portes de sécurité bloquantes : aucun n'existe à ce jour.
- Auditer le contrôle d'accès sur les endpoints manipulant des données utilisateur, en particulier ceux liés aux enfants supervisés.
- Mettre en place l'analyse statique de sécurité (SAST), l'audit des dépendances et la détection de secrets.
- Conduire une analyse RGPD dédiée aux données des mineurs et documenter la base légale de leur traitement.
- Clarifier et encadrer l'auto-assignation aux groupes, actuellement possible sans validation d'un tiers.
- Statuer sur la surface encore exposée par les routes de commentaires non utilisées côté interface.

**Critères de succès** :

- Zéro vulnérabilité critique ou haute en production sur les parcours traitant des données de mineurs : seuil non négociable, sans compensation possible.
- Conformité RGPD documentée : registre des traitements et base légale explicite pour les données des mineurs et des comptes gérés par un superviseur.
- Pipeline CI/CD opérationnel avec au minimum lint, audit de dépendances et analyse statique bloquants.
- [Cibles chiffrées (couverture de tests, disponibilité, performance) à définir avec VNWeb avant la version finale du dossier, non fixées arbitrairement dans ce brouillon.]

### 2.4 État actuel du projet

L'application est en production active, utilisée au quotidien par l'école. À la différence d'un projet encore en développement où la sécurité peut être intégrée avant toute exposition réelle, la situation ici est inverse : l'application expose déjà des données de familles réelles, sans qu'un audit de sécurité formel ait précédé sa mise en ligne. Cela oriente la démarche vers une logique corrective autant que préventive : identifier les failles existantes, les corriger en priorité sur les parcours les plus sensibles (comptes, données des mineurs, messagerie), puis mettre en place les garde-fous durables (CI/CD sécurisé, politique de sécurité formalisée) pour que la sécurité cesse d'être, comme le formule la fiche RNCP37173 elle-même, un sujet traité en fin de projet.

[à rédiger]

---

## 3. Analyse Stratégique de l'Entreprise, PESTEL & SWOT

Cette analyse porte sur VNWeb, l'agence web d'Antibes au sein de laquelle j'effectue mon alternance, et non sur le projet applicatif lui-même. Elle vise à situer l'entreprise dans son environnement macro-économique et concurrentiel.

### 3.1 PESTEL, VNWeb, agence web (Antibes / Sophia Antipolis)

**Politique**

La directive européenne NIS2 (UE 2022/2555), en cours de transposition en France, élargit les obligations de cybersécurité à un périmètre croissant de PME (Petites et Moyennes Entreprises) et ETI (Entreprises de Taille Intermédiaire), analyses de risques obligatoires, notification d'incident sous 24h, référent sécurité désigné. Plusieurs milliers de structures françaises entrent désormais dans ce périmètre, ce qui génère une demande de mise en conformité pour les clients de l'agence. La montée en puissance du discours sur la souveraineté numérique pousse par ailleurs le secteur public et les grands donneurs d'ordre vers des solutions hébergées et opérées en France, un argument que VNWeb peut valoriser en qualité d'agence de proximité. Enfin, la Région Sud, la Communauté d'Agglomération Sophia Antipolis (CASA) et le Département des Alpes-Maritimes investissent activement dans l'écosystème numérique local, le pôle Alpha inauguré en janvier 2026 (8 500 m², 62 start-up hébergées) en est l'illustration, avec la cybersécurité identifiée comme filière prioritaire.

**Économique**

Le marché du numérique français retrouve une dynamique de croissance après une phase de ralentissement, avec une accélération attendue dans le secteur public (+1,9 % en 2026 selon Numeum). Sophia Antipolis représente à elle seule 6 Md€ de chiffre d'affaires, 2 500 entreprises et 43 000 emplois, un bassin économique dense mais très concurrentiel pour le recrutement de développeurs. Du côté de la demande, le baromètre France Num 2025 indique que 40 % des TPE/PME (Très Petites, Petites et Moyennes Entreprises) perçoivent le numérique comme un levier direct d'augmentation du chiffre d'affaires, et 78 % y ont recours pour externaliser des fonctions, des signaux favorables à la demande de prestations web. La tension RH sectorielle reste structurelle : plus de 68 700 offres d'emploi numériques resteraient non pourvues chaque année en France pour 61 450 personnes formées.

**Socioculturel**

La confiance dans l'IA progresse mais reste minoritaire chez les Français (46 %), tandis que l'usage explose (48 % de la population, +28 points en deux ans), créant une demande d'intégration IA croissante mais prudente côté clients. La sobriété numérique et l'éthique des données deviennent des critères de confiance pour les utilisateurs finaux, un argument différenciant potentiel pour une petite agence transparente. Localement, Sophia Antipolis cultive une identité de fertilisation croisée entre recherche, entreprises et grandes écoles (SKEMA, Polytech, Eurecom), ce qui facilite le réseau et l'accès à des profils stagiaires et alternants.

**Technologique**

L'IA générative est devenue un moteur de productivité dans le secteur : gains estimés à 12,5 % en 2025, attendus à 17 % en 2026 selon Numeum, avec 40 % des acteurs constatant un impact positif sur leurs marges. Le SaaS structure désormais 77 % des nouveaux projets (contre 53 % en 2021), la demande se déplace vers des architectures cloud et abonnement plutôt que des projets one-shot. Le mobile-first reste incontournable, plus de 70 % du trafic web s'effectuant sur smartphone. L'accessibilité numérique est en retard généralisé : la moitié des éditeurs ne maîtrise pas encore les référentiels RGAA (Référentiel Général d'Amélioration de l'Accessibilité)/WCAG 2.2, alors que ces normes deviennent des obligations légales croissantes.

**Écologique**

L'éco-conception web, sobriété numérique, hébergement bas-carbone, optimisation des ressources, devient un critère de différenciation dans plusieurs analyses de marché 2026, sans être encore un standard imposé. Le pôle Alpha à Sophia Antipolis met en avant des bâtiments à très faible empreinte énergétique (géothermie, photovoltaïque), signe que le territoire valorise ce positionnement et que les donneurs d'ordre publics locaux commencent à l'intégrer dans leurs critères de sélection.

**Légal**

L'empilement réglementaire s'accélère : NIS2, Cyber Resilience Act, RGAA/WCAG 2.2, facturation électronique obligatoire (grandes entreprises et ETI dès septembre 2026, généralisation aux PME en 2027). Cette complexité croissante des projets web crée simultanément une demande de mise en conformité à valoriser. Sur la cybersécurité, le panorama ANSSI (Agence Nationale de la Sécurité des Systèmes d'Information) 2026 confirme que 48 % des victimes de rançongiciels en France sont des PME/TPE/ETI, et que 74 % d'entre elles restent sous le niveau de sécurité "Essentiel" défini par l'ANSSI. Seulement 9 % des TPE/PME se déclarent assujetties ou en cours de préparation à NIS2, et 64 % ignorent simplement son existence, un large marché d'accompagnement encore inexploité.

### 3.2 SWOT, VNWeb, agence web (Antibes / Sophia Antipolis)

**Forces**

VNWeb est implantée à Antibes, à proximité immédiate de Sophia Antipolis, première technopole d'Europe (2 500 entreprises, 43 000 emplois, 6 Md€ de CA). Cette proximité géographique est un avantage réel face aux grandes agences parisiennes : plusieurs analyses 2026 identifient la relation de confiance directe et la réactivité locale comme des critères de choix déterminants pour les PME. La taille humaine de l'agence (5 personnes) renforce cette dynamique : adaptation au niveau de technicité du client, interlocuteur unique, communication fluide sans couche de management intermédiaire. La polyvalence de l'équipe est notable, frontend, fullstack/admin, développeur IA-natif (maîtrise avancée des outils IA de développement : GitHub Copilot, génération de code, revue assistée) et une compétence cybersécurité en construction interne via alternance, une combinaison rare pour une structure de cette taille, alors que le marché souffre d'un déficit structurel de profils en sécurité.

**Faiblesses**

La taille de cinq personnes est aussi la principale vulnérabilité opérationnelle : capacité de charge limitée, fragilité en cas d'absence ou de départ d'un collaborateur, difficulté à répondre à plusieurs projets importants en parallèle. La compétence cybersécurité est encore en phase de formation (alternance), c'est un actif en devenir plutôt qu'un service immédiatement monétisable. L'agence ne dispose vraisemblablement pas des certifications et labels reconnus que d'autres agences mettent en avant commercialement (Google Partner, labels sectoriels). Enfin, l'empilement réglementaire (NIS2, RGAA, facturation électronique) exige une veille et une mise à jour de compétences constante, coûteuse en temps pour une équipe restreinte sans ressource dédiée à la veille.

**Opportunités**

La demande de mise en conformité cybersécurité chez les TPE/PME est quasiment inexploitée : 74 % restent sous le niveau "Essentiel" ANSSI, 64 % ignorent NIS2. Un large marché local n'est pas encore adressé, la compétence interne en cybersécurité peut se transformer en offre commerciale (audit, sensibilisation, mise en conformité packagée avec les prestations de développement). L'écosystème local est en expansion active : ouverture du pôle Alpha, réseau French Tech Côte d'Azur, Sophia Club Entreprises, autant d'occasions de visibilité et de partenariats avec des PME locales. La montée en puissance de l'IA générative représente un avantage de productivité actif dans l'équipe, alors que 40 % des acteurs du secteur constatent un impact positif sur leurs marges. L'accessibilité numérique (RGAA/WCAG 2.2) est une niche encore sous-exploitée : la moitié du marché ne la maîtrise pas alors qu'elle devient une obligation légale croissante.

**Menaces**

La concurrence s'exerce à plusieurs niveaux : agences locales de taille similaire, grandes structures nationales qui rassurent davantage les grands comptes, et plateformes ou agences spécialisées low-cost qui tirent les prix vers le bas sur les projets simples. En tant qu'agence détenant des accès clients (hébergements, identifiants, données de production), VNWeb est elle-même une cible potentielle de la même vague d'attaques qui touche le tissu PME, phishing et piratage de comptes en tête des menaces 2025-2026. La tension sur les talents tech dans un bassin d'emploi où coexistent 2 500 entreprises et des grands groupes internationaux (Amadeus, Thales, IBM, Toyota R&D) rend le recrutement et la fidélisation de développeurs supplémentaires difficile et coûteux. Enfin, l'instabilité macroéconomique et politique pointée par Numeum peut freiner les décisions d'investissement numérique des clients PME.

### 3.3 Synthèse stratégique

Le positionnement le plus cohérent qui émerge de cette analyse est celui d'une agence web de proximité à taille humaine, sur la Côte d'Azur, avec une carte différenciante rare : la cybersécurité intégrée au développement, à un moment où le marché des PME françaises est massivement sous-protégé et sous-informé sur le sujet. C'est un angle que peu d'agences web de cette taille peuvent revendiquer de façon crédible, et il correspond précisément à la composition d'équipe actuelle.

Les leviers les plus actionnables à court terme sont la valorisation de l'audit et de la sensibilisation cybersécurité comme service additionnel, en s'appuyant sur les référentiels gratuits ANSSI et cybermalveillance.gouv.fr pour construire une offre crédible sans investissement lourd, le rapprochement du réseau French Tech Côte d'Azur et du pôle Alpha pour la visibilité et les mises en relation avec des PME locales, et la capitalisation sur la productivité IA de l'équipe comme avantage de délai et de coût face à des structures plus lourdes, tout en maintenant une exigence de qualité et de sécurité du code produit.

*Sources : ANSSI, Panorama de la cybermenace 2025 (mars 2026) · Numeum, Bilan 2025 et perspectives 2026 · France Num, Baromètre 2025 · Cybermalveillance.gouv.fr, Baromètre TPE/PME 2025 · CASA / pôle Alpha, actualités janvier–avril 2026*

---

## 4. Cadrage du Projet, Besoins, Contraintes & Gestion de Projet

### 4.1 Besoins fonctionnels

| ID | Besoin fonctionnel | Module |
|---|---|---|
| BF-01 | Gestion des comptes et des rôles (admin, professeur, superviseur, élève) | api-dance (auth, admin) |
| BF-02 | Cycle de validation d'un compte (validation email puis validation admin) | api-dance (auth) |
| BF-03 | Gestion des enfants par un compte superviseur (création, modification, retrait) | api-dance (users) |
| BF-04 | Invitation d'un tiers comme superviseur d'un enfant | api-dance (auth) |
| BF-05 | Gestion des groupes et des cours | api-dance (groups) |
| BF-06 | Auto-assignation ou retrait d'un groupe (professeur, élève) | api-dance (groups) |
| BF-07 | Publication de posts (de groupe ou globaux) | api-dance (posts), app-dance |
| BF-08 | Like sur un post | api-dance (posts) |
| BF-09 | Messagerie privée en un-à-un | api-dance (chat), app-dance |
| BF-10 | Notifications push (nouveau post, nouveau message, validation d'email, invitation de superviseur) | api-dance (web push), app-dance |
| BF-11 | Upload et diffusion des médias (photos de cours, avatars) | cdn-app-dance |
| BF-12 | Espace admin (utilisateurs, validation des comptes, emails non vérifiés, cours) | api-dance, app-dance |

Ce tableau reflète le périmètre fonctionnel réellement développé et vérifié dans le code, pas un périmètre cible.

### 4.2 Besoins non-fonctionnels

| ID | Besoin non-fonctionnel | Critère mesurable |
|---|---|---|
| BNF-01 | Confidentialité des données des mineurs | Aucune donnée personnelle d'un enfant accessible sans authentification valide du compte superviseur associé, vérifié lors de l'audit (section 6) |
| BNF-02 | Disponibilité en production | Aucune interruption de service non planifiée pendant les interventions de sécurisation, validation préalable de Vincent Nilles pour toute intervention à risque |
| BNF-03 | Absence de régression fonctionnelle | Chaque correction de sécurité validée sur le parcours concerné avant mise en production |
| BNF-04 | Traçabilité des actions administrateur sensibles | Journalisation des actions de validation, suspension et suppression de compte, portée exacte définie en section 6 |
| BNF-05 | Conformité RGPD sur les données des mineurs | Registre des traitements et base légale documentés, section 10 |
| BNF-06 | Sécurité des échanges réseau | Chiffrement HTTPS de bout en bout, à vérifier lors de l'audit (section 6) |
| BNF-07 | [Cibles chiffrées complémentaires (couverture de tests, temps de réponse) à définir avec VNWeb, non fixées arbitrairement dans ce brouillon.] | |

### 4.3 Contraintes

**Techniques** : la stack existante est imposée (AdonisJS/MySQL/Lucid pour `api-dance`, React/Ionic/Firebase pour `app-dance`, Express/Multer pour `cdn-app-dance`) ; le chantier de sécurisation porte sur l'existant, ce n'est pas une réécriture. L'application étant déjà en production, toute correction doit se faire sans interruption de service ni perte de données.

**Organisationnelles** : l'équipe VNWeb est réduite à cinq personnes, dont quatre développeurs ; la sécurisation n'est pas un chantier à temps plein dédié, elle s'insère dans le rythme de l'alternance (quatre jours en entreprise, un jour à l'école par semaine). Les priorités sont validées par Vincent Nilles, maître d'apprentissage et dirigeant de VNWeb.

**Réglementaires** : conformité RGPD requise, avec une attention particulière portée aux données des mineurs, notamment les élèves de moins de 15 ans dont le profil est géré par le compte de leur parent superviseur.

**Temporelles** : dossier RNCP37173 à finaliser pour juin 2027.

### 4.4 Problématiques identifiées & Solutions retenues

**P1, comment sécuriser une application déjà exposée en production, avec des données réelles de familles, sans qu'aucun audit n'ait précédé sa mise en ligne ?**
Solution retenue : une démarche corrective priorisée par criticité (section 6), en traitant en premier les parcours touchant les données des mineurs (comptes, messagerie), avant de mettre en place les garde-fous durables (pipeline CI/CD, politique de sécurité formalisée).

**P2, comment intégrer une pratique de sécurité durable dans une équipe de cinq personnes, sans expert cybersécurité dédié en interne ?**
Solution retenue : confier la mission de sécurisation à l'alternante en parallèle de sa formation RNCP37173, avec une mise en place progressive d'outils automatisés (analyse statique, audit de dépendances, détection de secrets) plutôt qu'un contrôle manuel récurrent qui ne serait pas soutenable dans le temps.

**P3, comment garantir qu'aucune régression fonctionnelle n'accompagne les corrections de sécurité, alors qu'aucune suite de tests automatisés n'existe sur le projet ?**
Constat : à ce jour, aucun test automatisé n'a été écrit sur `api-dance`, `app-dance` ou `cdn-app-dance`. La vérification de non-régression repose uniquement sur des essais manuels avant mise en production.
Solution retenue : maintenir la validation manuelle ciblée sur le parcours modifié à chaque correction de sécurité, et positionner la construction d'une suite de tests automatisés comme un chantier à part entière du plan de sécurisation (section 9), et non comme un prérequis bloquant les corrections urgentes.

**P4, comment traiter les écarts découverts pendant l'analyse du code (routes de commentaires encore actives côté backend malgré leur retrait de l'interface, redondance entre `isAdmin` et `status`, absence de vérification du lien superviseur-enfant sur l'invitation d'un tiers) sans bloquer l'avancement du chantier ?**
Solution retenue : chaque écart est documenté et traité comme point d'audit à part entière (sections 6 et 9), plutôt que corrigé au fil de l'eau sans traçabilité.

### 4.5 Gestion de projet, WBS et suivi Kanban

Le chantier de sécurisation est découpé en lots qui reprennent la structure même de ce dossier, chaque lot correspondant à une section du plan de travail :

| Lot | Contenu | Section du dossier |
|---|---|---|
| Lot 1 | Cadrage, recueil des besoins et des contraintes | Section 4 |
| Lot 2 | Audit de sécurité de l'existant | Section 6 |
| Lot 3 | Analyse des risques (AMDEC, EBIOS RM) | Sections 7 et 8 |
| Lot 4 | Plan de migration et sécurisation, pipeline DevSecOps | Section 9 |
| Lot 5 | Politique de sécurité formalisée | Section 10 |
| Lot 6 | Plan de reprise d'activité et continuité de service | Section 11 |
| Lot 7 | Bilan critique et bilan de compétences | Sections 12 et 13 |

Le code est hébergé sur Gitea. Le suivi des tâches du projet se fait sur l'extranet interne de VNWeb, l'outil de gestion de projet hébergé par l'agence ; Notion reste un usage personnel de prise de notes, en dehors de ce circuit de suivi partagé avec l'équipe.

### 4.6 Extrait du backlog

| ID | User Story | Priorité (MoSCoW) | Statut |
|---|---|---|---|
| US-01 | En tant qu'admin, je veux que le dépôt dispose d'un pipeline CI/CD avec des portes de sécurité bloquantes, afin qu'aucune vulnérabilité connue ne soit déployée en production. | Must | À faire |
| US-02 | En tant que responsable sécurité, je veux auditer le contrôle d'accès sur les endpoints manipulant des données d'enfants supervisés, afin de vérifier qu'un utilisateur ne peut pas accéder aux données d'un enfant qui n'est pas le sien. | Must | À faire |
| US-03 | En tant que VNWeb, je veux une analyse RGPD dédiée aux données des mineurs, afin de documenter la base légale de leur traitement. | Must | À faire |
| US-04 | En tant qu'admin, je veux que l'auto-assignation aux groupes soit clarifiée, afin d'éviter qu'un élève rejoigne un groupe qui ne le concerne pas sans aucun contrôle. | Should | À faire |
| US-05 | En tant que responsable sécurité, je veux statuer sur les routes de commentaires encore actives côté backend mais retirées de l'interface, afin de réduire la surface d'attaque exposée inutilement. | Should | À faire |
| US-06 | En tant que développeuse, je veux résoudre la redondance entre `isAdmin` et `status` pour l'adminship, afin d'éviter une incohérence exploitable dans les contrôles d'accès. | Could | À faire |
| US-07 | En tant que responsable sécurité, je veux qu'une invitation de superviseur ne puisse être envoyée que par un utilisateur déjà superviseur de l'enfant concerné, afin qu'un professeur ou un superviseur d'un autre enfant ne puisse pas donner accès aux données d'un mineur qui ne le concerne pas. | Must | À faire |

Cet extrait reprend les objectifs restants déjà identifiés en section 2.3 et les écarts relevés dans l'analyse du code, reformulés en user stories.

---

## 5. Architecture Technique, Benchmark & CDCT

### 5.1 Architecture de l'application

[à rédiger : schéma d'ensemble et rôle de chaque composant (api-dance, app-dance, cdn-app-dance, base de données, services tiers).]

### 5.2 CDCT, Choix Des Composants Technologiques

#### 5.2.1 Framework backend

[à rédiger : pourquoi AdonisJS pour api-dance]

#### 5.2.2 Base de données et ORM

[à rédiger : pourquoi MySQL/Lucid]

#### 5.2.3 Framework frontend

[à rédiger : pourquoi React/Ionic pour app-dance]

#### 5.2.4 Service d'authentification / services tiers

[à rédiger]

#### 5.2.5 Service de stockage et diffusion de fichiers

[à rédiger : pourquoi un service CDN dédié (cdn-app-dance) plutôt qu'un stockage intégré à api-dance]

#### 5.2.6 [Autre décision structurante à documenter]

[à rédiger]

---

## 6. Plan d'Audit de Sécurité

### 6.1 Périmètre de l'audit

[à rédiger]

### 6.2 Méthodologie

[à rédiger]

### 6.3 Findings identifiés

[à rédiger : F-01, F-02... avec criticité, catégorie OWASP, constat, recommandation, statut]

### 6.4 Synthèse et priorisation

[à rédiger]

---

## 7. AMDEC, Analyse des Modes de Défaillance, Effets et Criticité

### Méthode et barème

[à rédiger]

### Analyse par composant

[à rédiger : AM-01, AM-02...]

### Synthèse AMDEC

[à rédiger]

---

## 8. EBIOS RM, Analyse des Menaces Intentionnelles

### Atelier 1 : Cadrage et socle de sécurité

[à rédiger : valeurs métier, biens supports, événements redoutés]

### Atelier 2 : Sources de risques et objectifs visés

[à rédiger]

### Atelier 3 : Scénarios stratégiques

[à rédiger]

### Atelier 4 : Scénarios opérationnels

[à rédiger]

### Atelier 5 : Traitement des risques

[à rédiger]

---

## 9. Plan de migration et sécurisation

### Sans coût, à engager immédiatement

[à rédiger]

### Faible coût, coût récurrent limité, valeur élevée

[à rédiger]

### Investissement structurant, impact architectural durable

[à rédiger]

### Synthèse

[à rédiger]

### Pipeline DevSecOps, sécurité automatisée à chaque phase du cycle

[à rédiger]

---

## 10. Politique de sécurité

### 10.1 Objectifs de la politique

[à rédiger]

### 10.2 Périmètre

[à rédiger]

### 10.3 Règles de conception et de codage

[à rédiger]

### 10.4 Gouvernance

[à rédiger]

### 10.5 Responsabilités

[à rédiger]

### 10.6 Processus de validation

[à rédiger]

---

## 11. Plan de Reprise d'Activité (PRA) et Continuité de Service

### 11.1 Cadre méthodologique

[à rédiger]

### 11.2 Services critiques et objectifs de reprise

[à rédiger]

### 11.3 Scénarios de sinistre et procédures de reprise

[à rédiger]

### 11.4 Articulation avec l'AMDEC

[à rédiger]

### 11.5 Dispositif de surveillance et déclenchement

[à rédiger]

---

## 12. Bilan critique, Ce que j'aurais fait différemment

### 12.1 [Décision technique à challenger]

[à rédiger]

### 12.2 [Décision de conformité ou d'architecture à challenger]

[à rédiger]

### 12.3 [Décision de service tiers à challenger]

[à rédiger]

---

## 13. Bilan de l'Année, Compétences Acquises et Axes de Progression

### 13.1 Compétences techniques acquises

[à rédiger, en ancrant sur le projet réel : audit et sécurisation de api-dance (AdonisJS, MySQL/Lucid), app-dance (React/Ionic, Firebase) et cdn-app-dance (Express, Multer). S'appuyer sur le travail effectivement mené avec l'agent `@axel` (SAST/DAST, CI/CD) et `@jury-rncp37173` (auto-évaluation bloc par bloc) comme preuve concrète plutôt que sur une généralité.]

### 13.2 Compétences organisationnelles et transverses

[à rédiger : expériences réelles de collaboration (hackathons, équipe VNWeb) ; ne pas réutiliser telles quelles les mentions d'un projet ou d'une équipe qui ne correspondent pas à la réalité de cette alternance.]

### 13.3 Application des compétences acquises : [projet personnel, si applicable]

[à rédiger si un projet personnel distinct existe ; sinon, retirer cette sous-section plutôt que la laisser vide dans la version finale.]

### 13.4 Axes de progression

[à rédiger : axes réels et personnels, pas une généralité recopiée.]
