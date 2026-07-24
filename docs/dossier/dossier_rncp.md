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

L'équipe affectée au projet Centre Art & Danse, dans le cadre de ma mission de sécurisation, est plus restreinte que l'agence entière : trois personnes, un développeur frontend, un développeur backend, et moi-même en tant qu'alternante en charge de la sécurité.

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
| Développeur frontend, VNWeb | Développement des interfaces des projets clients | [nom à préciser] |
| Développeur backend, VNWeb | Développement backend des projets clients | [nom à préciser] |

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
- Filtrer par liste blanche l'extension de fichier utilisée dans les commandes exécutées par `cdn-app-dance`, un point remonté par le plan d'audit comme risque d'injection.

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

| ID | Besoin fonctionnel | Composant concerné |
|---|---|---|
| BF-01 | Gestion des comptes et des rôles (admin, professeur, superviseur, élève) | Backend |
| BF-02 | Cycle de validation d'un compte (validation email puis validation admin) | Backend |
| BF-03 | Gestion des enfants par un compte superviseur (création, modification, retrait) | Backend |
| BF-04 | Invitation d'un tiers comme superviseur d'un enfant | Backend |
| BF-05 | Gestion des groupes et des cours | Backend |
| BF-06 | Auto-assignation ou retrait d'un groupe (professeur, élève) | Backend |
| BF-07 | Publication de posts (de groupe ou globaux) | Backend et interface utilisateur |
| BF-08 | Like sur un post | Backend |
| BF-09 | Messagerie privée en un-à-un | Backend et interface utilisateur |
| BF-10 | Notifications push (nouveau post, nouveau message, validation d'email, invitation de superviseur) | Backend et interface utilisateur |
| BF-11 | Upload et diffusion des médias (photos de cours, avatars) | Service de fichiers dédié |
| BF-12 | Espace admin (utilisateurs, validation des comptes, emails non vérifiés, cours) | Backend et interface utilisateur |

Ce tableau reflète le périmètre fonctionnel réellement développé et utilisé en production, pas un périmètre cible. Le backend, l'interface utilisateur et le service de fichiers dédié sont les trois composants applicatifs présentés en détail à la section 5.1.

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

**Techniques** : la stack existante est imposée (AdonisJS/MySQL/Lucid pour `api-dance`, React/Ionic pour `app-dance`, Express/Multer pour `cdn-app-dance`) ; le chantier de sécurisation porte sur l'existant, ce n'est pas une réécriture. L'application étant déjà en production, toute correction doit se faire sans interruption de service ni perte de données.

**Organisationnelles** : VNWeb compte cinq personnes au total, mais l'équipe affectée au projet Centre Art & Danse est réduite à trois personnes, un développeur frontend, un développeur backend et l'alternante en charge de la sécurité ; la sécurisation n'est pas un chantier à temps plein dédié, elle s'insère dans le rythme de l'alternance (quatre jours en entreprise, un jour à l'école par semaine). Les priorités sont validées par Vincent Nilles, maître d'apprentissage et dirigeant de VNWeb.

**Réglementaires** : conformité RGPD requise, avec une attention particulière portée aux données des mineurs, notamment les élèves de moins de 15 ans dont le profil est géré par le compte de leur parent superviseur.

**Temporelles** : dossier RNCP37173 à finaliser pour juin 2027.

### 4.4 Problématiques identifiées & Solutions retenues

**P1, comment sécuriser une application déjà exposée en production, avec des données réelles de familles, sans qu'aucun audit n'ait précédé sa mise en ligne ?**
Solution retenue : une démarche corrective priorisée par criticité (section 6), en traitant en premier les parcours touchant les données des mineurs (comptes, messagerie), avant de mettre en place les garde-fous durables (pipeline CI/CD, politique de sécurité formalisée).

**P2, comment intégrer une pratique de sécurité durable dans une équipe de projet de trois personnes, sans expert cybersécurité dédié en interne ?**
Solution retenue : confier la mission de sécurisation à l'alternante en parallèle de sa formation RNCP37173, avec une mise en place progressive d'outils automatisés (analyse statique, audit de dépendances, détection de secrets) plutôt qu'un contrôle manuel récurrent qui ne serait pas soutenable dans le temps.

**P3, comment garantir qu'aucune régression fonctionnelle n'accompagne les corrections de sécurité, alors qu'aucune suite de tests automatisés n'existe sur le projet ?**
Constat : à ce jour, aucun test automatisé n'a été écrit sur le backend, l'interface utilisateur ou le service de fichiers. La vérification de non-régression repose uniquement sur des essais manuels avant mise en production.
Solution retenue : maintenir la validation manuelle ciblée sur le parcours modifié à chaque correction de sécurité, et positionner la construction d'une suite de tests automatisés comme un chantier à part entière du plan de sécurisation (section 9), et non comme un prérequis bloquant les corrections urgentes.

**P4, comment traiter les écarts découverts pendant l'analyse du code (deux champs distincts qui représentent tous les deux le statut d'administrateur d'un compte, sans garantie de rester synchronisés entre eux, absence de vérification du lien superviseur-enfant sur l'invitation d'un tiers, risque d'injection de commande via le nom d'un fichier envoyé au service de fichiers) sans bloquer l'avancement du chantier ?**
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
| US-05 | En tant que responsable sécurité, je veux que l'extension d'un fichier envoyé au service de fichiers soit filtrée par liste blanche avant d'être utilisée dans un traitement exécuté sur le serveur (conversion ou compression de médias), afin d'empêcher une injection de commande via un nom de fichier construit à cet effet. | Must | À faire |
| US-06 | En tant que développeuse, je veux unifier les deux champs distincts qui représentent aujourd'hui le statut d'administrateur d'un compte, afin d'éviter une incohérence exploitable dans les contrôles d'accès. | Could | À faire |
| US-07 | En tant que responsable sécurité, je veux qu'une invitation de superviseur ne puisse être envoyée que par un utilisateur déjà superviseur de l'enfant concerné, afin qu'un professeur ou un superviseur d'un autre enfant ne puisse pas donner accès aux données d'un mineur qui ne le concerne pas. | Must | À faire |
| US-08 | En tant que responsable sécurité, je veux restreindre les origines autorisées à se connecter au canal de messagerie en temps réel, actuellement ouvert à n'importe quel site, à une liste de domaines connus, afin qu'un site tiers ne puisse pas s'y connecter. | Must | À faire |
| US-09 | En tant que développeuse, je veux retirer le code de démonstration hérité de l'outil de diffusion en temps réel qui n'a finalement pas été retenu pour la messagerie, afin de réduire la surface d'attaque et la charge de maintenance à du code réellement actif. | Could | À faire |

Cet extrait reprend les objectifs restants déjà identifiés en section 2.3, les écarts relevés dans l'analyse du code de la section 4.4, et les constats faits lors de la rédaction de l'architecture (section 5), reformulés en user stories.

---

## 5. Architecture Technique, Benchmark & CDCT

### 5.1 Architecture de l'application

L'application est découpée en trois sous-projets indépendants, chacun avec sa propre base de code et son propre cycle de déploiement, plus une base de données partagée et des services tiers.

```
                     (navigateur, installé en PWA)
                            app-dance
                    /            |            \
        API REST + temps réel    |      upload / diffusion
       (HTTP, SSE, WebSocket)    |          de fichiers
                /                |                \
           api-dance             |          cdn-app-dance
                |                |
            MySQL          notifications push
        (via Lucid ORM)    (web-push, vers les
                            navigateurs abonnés)
```

**Rôle de chaque composant :**

| Composant | Rôle | Technologie principale | Expose / Consomme |
|---|---|---|---|
| `app-dance` | Client web installable (PWA) | React 19, Ionic React, Vite | Consomme l'API REST et le temps réel de `api-dance` ; dépose et récupère les fichiers via `cdn-app-dance` |
| `api-dance` | Backend métier, source de vérité des données | AdonisJS 6, Lucid, MySQL | Expose l'API REST, le temps réel (Server-Sent Events via Transmit, WebSocket via socket.io), gère l'authentification et l'envoi des notifications push |
| `cdn-app-dance` | Service dédié au dépôt et à la diffusion de fichiers | Express 5, Multer | Reçoit les fichiers uploadés (photos de cours, avatars) et les sert ensuite en lecture |
| Base de données | Persistance de l'ensemble des données de l'application | MySQL, accédée uniquement par le backend | Stocke comptes, rôles, groupes, posts, messages, jetons de connexion, abonnements aux notifications |
| Service de notifications navigateur | Notifications push | Géré depuis le backend | Envoie les notifications aux navigateurs des utilisateurs qui y ont consenti |

Le service de fichiers est volontairement isolé des deux autres composants : il ne partage ni code ni base de données avec le backend, ce qui limite ce qu'un incident sur ce service pourrait exposer au reste de la plateforme.

### 5.2 CDCT, Choix Des Composants Technologiques

La stack décrite ci-dessous existait déjà avant le début de ma mission de sécurisation. Cette section documente ce qui justifie techniquement ces choix aujourd'hui, ce n'est pas un compte-rendu de la décision d'origine chez VNWeb.

#### 5.2.1 Framework backend

Le backend repose sur AdonisJS 6, un framework Node.js/TypeScript complet plutôt qu'un assemblage de briques indépendantes (type Express seul). Ce choix apporte, de série, plusieurs éléments directement utiles à un projet manipulant des données de mineurs : un ORM intégré (Lucid), un module d'authentification natif, un module d'autorisation par politiques d'accès, une validation stricte des données entrantes et un mécanisme de limitation du nombre de requêtes. Ces briques viennent du même écosystème et suivent le même cycle de mises à jour, ce qui réduit le risque d'incohérence de version entre des dépendances de sécurité choisies séparément.

#### 5.2.2 Base de données et ORM

Le choix de MySQL, accédé via l'ORM Lucid, correspond à un domaine fortement relationnel : comptes liés à des rôles, superviseurs liés à des enfants, utilisateurs liés à des groupes, posts liés à des médias et des likes. Les suppressions en cascade, présentes sur la quasi-totalité des relations, garantissent qu'un enfant supprimé n'y laisse pas de données orphelines ailleurs dans la base. L'historique des évolutions du schéma est versionné et horodaté, ce qui a permis de reconstituer avec exactitude le schéma de données de ce dossier à partir de 42 étapes d'évolution successives de la base. Une configuration pour une autre base de données (PostgreSQL) existe dans le code mais elle est désactivée : seule la connexion MySQL est active en pratique.

#### 5.2.3 Framework frontend

L'interface utilisateur utilise React 19 avec Ionic React plutôt qu'un développement natif séparé pour mobile et web. Ionic apporte des composants d'interface proches d'une application mobile native tout en restant une PWA installable depuis le navigateur, un choix cohérent avec le fait que l'application doit être utilisée aussi bien par des parents que par des professeurs sur des appareils variés, sans passer par un store d'application. L'outil de développement utilisé (Vite) assure un temps de build et de rechargement à chaud réduit par rapport à des outils plus anciens, ce qui compte pour la vitesse d'itération d'une équipe de deux développeurs.

#### 5.2.4 Service d'authentification / services tiers

L'authentification principale repose sur le module d'authentification natif du framework backend, avec des jetons d'accès stockés côté serveur, dans une table dédiée de la base de données. Aucun service d'authentification tiers n'est réellement utilisé par l'application : l'intégralité du cycle de connexion, de validation de compte et de gestion des jetons est portée par le backend lui-même.

#### 5.2.5 Service de stockage et diffusion de fichiers

Le service de fichiers est un service séparé, dédié à la réception et à la diffusion des fichiers, plutôt qu'un stockage intégré directement au backend. Séparer ce service isole la surface d'attaque propre à la réception de fichiers (validation de type, de taille, chemin de stockage) du reste de la logique métier : une faille sur l'envoi de fichiers ne donne pas un accès direct à la base de données ou aux jetons d'authentification, qui restent uniquement du ressort du backend. Ce choix a cependant un revers déjà identifié : aucun mécanisme d'authentification n'a été repéré sur ce service, un point qui devra être traité en priorité lors de l'audit (section 6).

#### 5.2.6 Coexistence de deux mécanismes temps réel

Le backend déclare deux mécanismes de communication en temps réel distincts, hérités de deux approches différentes disponibles dans son écosystème. La lecture du code montre qu'un seul des deux est réellement utilisé : il porte la messagerie privée en un-à-un (indicateur de saisie, envoi de message, marquage comme lu), et c'est celui-là que consomme l'interface utilisateur. Le second est installé et configuré, mais son seul point d'usage dans le code est un exemple de canal de démonstration qui n'est relié à aucune route accessible de l'application : c'est du code mort, un reliquat du gabarit de démarrage fourni par cet outil, pas une fonctionnalité active.

En creusant ce point, un autre constat mérite d'être noté ici : le canal temps réel réellement utilisé pour la messagerie est configuré pour accepter des connexions depuis n'importe quelle origine, une autorisation ouverte plutôt que restreinte à une liste de domaines connus. Sur un canal qui transporte des messages privés, c'est un point à vérifier en priorité lors de l'audit (section 6).

### 5.3 Benchmark détaillé des choix technologiques

Cette section compare, choix par choix, la technologie réellement utilisée dans les trois composants applicatifs (backend, interface utilisateur, service de fichiers) avec des alternatives sérieuses du marché, et motive pourquoi le choix en place tient ou ne tient pas face à ces alternatives. Chaque constat cité a été vérifié directement dans le code au moment de la rédaction, pas supposé. Niveau de preuve indiqué par choix (`Niv`) : L1, spécification officielle ou standard reconnu ; L2, documentation produit officielle ou recommandation d'un organisme de référence reconnu en sécurité (l'OWASP, Open Web Application Security Project, fondation de référence en sécurité applicative) ; L3, consensus technique large sans texte normatif unique.

#### 5.3.1 Framework backend : AdonisJS

**Choix en place** : AdonisJS 6, un framework backend complet en TypeScript, avec un ORM intégré (Lucid), un module d'authentification natif, un module d'autorisation par politiques d'accès, une validation stricte des données entrantes et une limitation du nombre de requêtes, tous intégrés au même framework.

| Option | Modules de sécurité intégrés | Maturité de l'écosystème Node | Courbe d'apprentissage pour une équipe TypeScript | Niv |
|---|---|---|---|---|
| AdonisJS 6 (retenu) | Auth, autorisation, validation, rate limiting fournis nativement, versionnés ensemble | Écosystème plus restreint que Express, mais complet pour les besoins d'un CRUD applicatif | Structure proche de Laravel/NestJS, familière pour une équipe déjà orientée TypeScript | L2 |
| Express (nu) | Aucun, chaque brique de sécurité est une dépendance tierce choisie et maintenue séparément | Écosystème le plus large du marché Node | Minimaliste, mais tout le socle sécurité est à assembler soi-même | L3 |
| NestJS | Modules de sécurité disponibles mais souvent via des paquets tiers (Passport, class-validator) | Écosystème large, orienté entreprise | Injection de dépendances et décorateurs, courbe plus raide pour une petite équipe | L2 |
| Fastify | Plugins de sécurité disponibles (helmet, rate-limit) mais à assembler | Écosystème solide, orienté performance brute | Minimaliste comme Express, assemblage manuel du socle sécurité | L3 |

**Recommandation** : AdonisJS reste le choix le plus cohérent pour une équipe de projet de trois personnes sans expert sécurité dédié en interne : avoir l'authentification, l'autorisation, la validation et le rate limiting du même éditeur, mis à jour ensemble, réduit le risque qu'une brique de sécurité tierce prenne du retard sans que personne ne s'en aperçoive. Express ou Fastify auraient demandé d'assembler et de maintenir ce socle à la main, un coût que la taille de l'équipe ne permet pas d'absorber durablement. Ce choix a un revers déjà visible dans ce projet : une partie du socle de sécurité (CORS restreint sur l'API HTTP) est bien configurée, mais une autre (CORS ouvert sur le canal socket.io, voir 5.3.7) ne l'est pas, ce qui montre que le cadre fourni par le framework ne suffit pas seul, il doit être appliqué de façon uniforme sur toute la surface exposée.

#### 5.3.2 ORM et accès aux données : Lucid

**Choix en place** : Lucid, l'ORM natif d'AdonisJS, avec des requêtes construites via un générateur de requêtes structuré plutôt qu'en SQL brut concaténé à la main, et un historique versionné des évolutions du schéma de la base.

| Option | Protection native contre l'injection SQL | Historique de schéma auditable | Coût de flexibilité (requêtes complexes) | Niv |
|---|---|---|---|---|
| Lucid (retenu) | Paramétrage automatique des requêtes du query builder | Migrations horodatées, rejouables, c'est la source utilisée pour reconstituer le schéma de ce projet | Requêtes très complexes parfois plus verbeuses qu'en SQL brut | L2 |
| SQL brut (requêtes manuelles) | Dépend entièrement de la discipline du développeur à paramétrer chaque requête | Aucun, le schéma vit uniquement dans la tête de l'équipe ou une documentation externe à maintenir à la main | Contrôle total, mais un seul oubli de paramétrage ouvre une injection SQL (OWASP Top 10, catégorie A03, Injection) | L1 |
| Prisma | Requêtes paramétrées automatiquement, schéma déclaratif dans un fichier unique | Migrations versionnées, générées à partir du schéma déclaratif | Moins intégré nativement à AdonisJS, demande une couche de compatibilité | L2 |
| TypeORM / Sequelize | Requêtes paramétrées automatiquement | Migrations versionnées | Mature mais historiquement moins actif sur AdonisJS que Lucid, qui est l'ORM de référence du framework | L3 |

**Recommandation** : Lucid reste le choix le plus défendable ici, pas seulement par confort d'intégration : le paramétrage automatique des requêtes ferme par construction la catégorie d'injection SQL la plus répandue (OWASP Top 10, A03), sur une application qui manipule des données de mineurs. Face au SQL brut, la différence n'est pas une question de style mais de surface d'erreur humaine possible à chaque requête écrite. Face à Prisma, TypeORM ou Sequelize, l'écart se joue sur l'intégration native à AdonisJS plutôt que sur la sécurité, les quatre ORM protégeant équivalemment contre l'injection par paramétrage.

#### 5.3.3 Base de données : MySQL

**Choix en place** : MySQL est la base de données réellement connectée en production. Une configuration pour une autre base de données (PostgreSQL) existe dans le code mais reste désactivée.

| Option | Adéquation au modèle relationnel du projet | Coût opérationnel pour une équipe de projet de 3 personnes | Support des contraintes d'intégrité (suppression en cascade) | Niv |
|---|---|---|---|---|
| MySQL (retenu) | Fort : comptes, groupes, superviseurs et posts sont fortement relationnels avec des suppressions en cascade | Faible, hébergement et outillage largement répandus | Complet, déjà utilisé sur l'ensemble des tables du schéma actuel | L2 |
| PostgreSQL | Fort, équivalent à MySQL sur ce type de modèle, avec en plus des types de données avancés et des règles de validation plus riches au niveau de la base | Comparable à MySQL | Complet | L2 |
| MongoDB (document) | Faible : le modèle actuel repose sur des relations many-to-many strictes (superviseurs/enfants, utilisateurs/groupes), qui demanderaient une dénormalisation manuelle en base documentaire | Comparable | Pas de clé étrangère native, l'intégrité référentielle serait à recoder côté application | L3 |

**Recommandation** : MySQL est un choix cohérent pour ce modèle de données, mais pas parce qu'il serait supérieur à PostgreSQL dans l'absolu : les deux conviennent également bien à un schéma aussi relationnel. Le point qui mérite d'être tranché n'est pas MySQL contre PostgreSQL, c'est la configuration PostgreSQL laissée présente mais désactivée dans le code : soit elle documente une migration envisagée à retirer proprement si elle n'est plus d'actualité, soit elle doit rester à jour si une bascule reste possible. MongoDB, en revanche, n'aurait pas été un bon choix ici : le modèle de données du projet est relationnel par nature, pas documentaire.

#### 5.3.4 Forme de l'application côté client : PWA (React + Ionic) et impact sécurité

**Choix en place** : l'interface utilisateur est une PWA (Progressive Web App) installable, construite avec React 19 et Ionic React, plutôt qu'une application native ou un développement natif séparé par plateforme.

| Option | Distribution | Surface d'attaque spécifique | Contrôle de l'éditeur sur les mises à jour de sécurité | Niv |
|---|---|---|---|---|
| PWA (retenu) | Installation directe depuis le navigateur, sans passage par un store d'application | Le Service Worker et le cache applicatif deviennent une surface à sécuriser explicitement (données mises en cache localement, y compris potentiellement des informations sur des mineurs) | Mise à jour immédiate au prochain chargement, aucune validation d'un store tiers ne retarde un correctif de sécurité | L2 |
| Application native (Swift/Kotlin, une par plateforme) | Passage obligatoire par l'App Store / Google Play, avec leurs propres contrôles de sécurité à la soumission | Stockage natif sécurisé disponible (Keychain, Keystore), mais deux bases de code distinctes à sécuriser séparément | Délai de validation du store avant qu'un correctif de sécurité atteigne les utilisateurs | L2 |
| React Native / Flutter (cross-platform natif) | Passage par les stores, un seul code source pour les deux plateformes | Stockage natif sécurisé disponible comme en natif pur | Même délai de validation de store que le natif pur | L2 |

**Recommandation** : le choix de la PWA n'est pas neutre en sécurité, dans les deux sens. Il retire la dépendance à un store tiers pour distribuer un correctif urgent, un vrai atout pour une application qui manipule des données de mineurs et qui doit pouvoir être corrigée vite. En contrepartie, il déplace la responsabilité de sécuriser le stockage local vers le Service Worker et le cache du navigateur, un point que l'audit (section 6) doit vérifier explicitement : ce que l'application met en cache côté client, et si des données personnelles d'enfants s'y retrouvent au-delà de ce qui est strictement nécessaire à l'usage hors ligne. Une application native aurait offert un stockage local chiffré fourni par l'OS (Keychain/Keystore), ce que le navigateur ne propose pas nativement au même niveau de garantie.

#### 5.3.5 Hachage des mots de passe : scrypt

**Choix en place** : les mots de passe sont hachés avec l'algorithme scrypt, configuré avec les paramètres fournis par défaut par le framework (un coût de calcul et une empreinte mémoire définis par défaut), non retouchés pour ce projet précis.

| Option | Résistance au calcul massivement parallèle (GPU/ASIC) | Recommandation OWASP (Password Storage Cheat Sheet) | Statut sur ce projet | Niv |
|---|---|---|---|---|
| scrypt (retenu) | Élevée, algorithme à mémoire coûteuse conçu pour limiter le parallélisme matériel | Cité comme option acceptable, en dessous d'Argon2id dans l'ordre de préférence OWASP | En place, paramètres par défaut du framework, non audités ni ajustés au contexte de cette application | L2 |
| Argon2id | Élevée, vainqueur de la Password Hashing Competition (2015), paramétrable en mémoire, temps et parallélisme | Premier choix recommandé par OWASP pour le hachage de mot de passe | Non utilisé sur ce projet | L1 |
| bcrypt | Bonne, mais coût uniquement en temps CPU, pas en mémoire, donc plus sensible à l'accélération GPU qu'un algorithme à coût mémoire | Deuxième choix recommandé par OWASP, si Argon2id n'est pas disponible | Non utilisé sur ce projet | L2 |
| PBKDF2 | Plus faible que les trois précédents, coût purement CPU, sans composante mémoire | Dernier choix recommandé par OWASP, à réserver aux environnements contraints (ex. certification FIPS) | Non utilisé sur ce projet | L2 |

**Recommandation** : scrypt n'est pas un mauvais choix, il figure dans la liste des algorithmes acceptés par l'OWASP pour le stockage de mots de passe, et il est nettement préférable à un simple hachage rapide sans dérivation de clé lente. Mais deux points méritent d'être remontés dans l'audit de sécurité (section 6) plutôt que d'être laissés tels quels : d'une part, Argon2id est la recommandation de premier rang de l'OWASP et n'a pas été retenu ici sans qu'une raison documentée n'explique ce choix ; d'autre part, les paramètres actuels sont ceux fournis par défaut par le framework, pas des valeurs choisies après une analyse du contexte de menace de ce projet précis. Un paramètre de coût non revu n'est pas nécessairement insuffisant, mais il n'a pas non plus été validé comme suffisant.

#### 5.3.6 Stockage des données sensibles hors mot de passe

**Constat vérifié dans le code** : une clé de chiffrement applicative existe déjà, utilisée par le framework pour le chiffrement des cookies et la signature d'URLs, mais aucun usage de cette clé n'a été trouvé pour chiffrer une donnée personnelle stockée en base. Les champs contenant des données personnelles des mineurs (date de naissance, téléphone, adresse, code postal, ville, nom, prénom) sont stockés en clair dans la table des comptes utilisateurs, protégés uniquement par le contrôle d'accès applicatif et par la sécurité de la base de données elle-même.

| Option | Protection si la base de données est copiée ou compromise directement | Complexité d'implémentation | Impact sur les requêtes (recherche, tri) | Niv |
|---|---|---|---|---|
| Colonnes en clair (situation actuelle) | Aucune, toute donnée est lisible dès l'accès à la base | Nulle, c'est l'état par défaut d'une colonne SQL classique | Aucun, recherche et tri natifs | L3 |
| Chiffrement au niveau champ (colonnes sensibles chiffrées avec la clé applicative) | Les colonnes chiffrées restent illisibles sans la clé applicative, même en cas de copie de la base seule | Moyenne, chiffrement/déchiffrement à chaque lecture/écriture, gestion de la clé à part de la base | Recherche et tri natifs perdus sur les colonnes chiffrées, à contourner par des colonnes dérivées ou un index séparé | L2 |
| Chiffrement au niveau disque uniquement (chiffrement du volume de la base) | Protège contre le vol physique du support, pas contre un accès applicatif ou un identifiant de base compromis | Faible, géré par l'hébergeur ou le système de fichiers, sans changement de code | Aucun impact sur les requêtes | L2 |

**Recommandation** : le chiffrement au niveau champ n'est pas systématiquement le bon choix pour toutes les colonnes, il ajoute de la complexité et casse la recherche native. Mais pour ce projet précis, où les données concernent en majorité des mineurs, il mérite d'être évalué au moins sur les colonnes les plus sensibles (date de naissance, adresse), en complément du chiffrement de disque qui protège un scénario différent (vol du support physique) et ne dispense pas d'un chiffrement applicatif si l'objectif est de résister aussi à une fuite d'identifiants de connexion à la base. Ce point est à traiter dans le plan de sécurisation (section 9), pas comme une correction immédiate isolée : il a un impact sur le modèle de données et sur les requêtes existantes.

#### 5.3.7 Communication en temps réel : le canal retenu pour la messagerie

**Constat déjà établi en 5.2.6** : un seul des deux mécanismes temps réel installés est réellement utilisé pour la messagerie privée, l'autre n'est relié à aucune route accessible. Point supplémentaire vérifié ici : le canal réellement utilisé pour la messagerie accepte des connexions depuis n'importe quelle origine, alors que l'API HTTP principale, elle, restreint ses origines à une liste explicite de domaines connus. Le cadre de sécurité existe donc dans ce projet, il n'a simplement pas été appliqué de façon uniforme sur le canal temps réel.

| Option | Origine autorisée | Cohérence avec la politique déjà en place sur l'API HTTP | Niv |
|---|---|---|---|
| Configuration actuelle du canal de messagerie | N'importe quelle origine | Incohérente avec la liste de domaines déjà utilisée sur l'API HTTP | L3 |
| Alignement sur la même liste de domaines que l'API HTTP | Domaines de l'application et de préproduction uniquement | Cohérente, un seul standard de configuration pour toute la plateforme | L2 |

**Recommandation** : restreindre l'origine du canal de messagerie à la même liste que l'API HTTP (déjà proposé comme US-08 dans le backlog, section 4.6). Ce n'est pas un arbitrage entre deux approches équivalentes, la configuration actuelle est une incohérence par rapport à un standard déjà appliqué ailleurs dans le même projet, pas un choix technique à débattre.

#### 5.3.8 Authentification : jetons d'accès plutôt que JWT ou sessions serveur

**Choix en place** : des jetons d'accès opaques stockés côté serveur, dans une table dédiée de la base de données, plutôt que des jetons auto-portants de type JWT ou des sessions serveur classiques.

| Option | Révocation immédiate d'un accès compromis | Charge portée par le serveur | Donnée exposée si le jeton est intercepté | Niv |
|---|---|---|---|---|
| Jetons d'accès en base (retenu) | Immédiate, il suffit de supprimer la ligne correspondante en base | Une vérification en base à chaque requête authentifiée | Le jeton seul ne révèle aucune information sur l'utilisateur | L2 |
| JWT auto-porté (sans vérification en base) | Impossible avant expiration naturelle du jeton, sauf liste de révocation supplémentaire à maintenir | Aucune vérification en base nécessaire, le jeton se suffit à lui-même | Le contenu du jeton (payload) est lisible par quiconque l'intercepte, sauf chiffrement supplémentaire | L1 |
| Session serveur classique (cookie + état en mémoire ou Redis) | Immédiate | Un état de session à maintenir et partager si plusieurs instances du serveur tournent en parallèle | Le cookie de session seul ne révèle aucune information | L2 |

**Recommandation** : les jetons d'accès stockés en base sont le choix le plus adapté ici, précisément parce qu'ils permettent une révocation immédiate, un point non négociable pour un compte lié à un enfant en cas de compromission (perte de téléphone d'un parent, par exemple). Un JWT auto-porté aurait été plus léger pour le serveur, mais au prix de ne pas pouvoir couper l'accès d'un jeton volé avant son expiration, sauf à reconstruire une liste de révocation, ce qui revient à recréer la vérification en base que ce choix évite justement.

#### 5.3.9 Upload et diffusion de fichiers : service dédié plutôt qu'un stockage objet cloud

**Choix en place** : un service séparé, hébergé en interne, dédié à la réception et à la diffusion des fichiers, plutôt qu'un service de stockage objet géré par un fournisseur cloud (par exemple S3, Cloudinary ou Google Cloud Storage) ou un stockage intégré directement au backend.

| Option | Isolation de la surface d'attaque liée à l'envoi de fichiers | Contrôle d'accès natif au service | Coût d'exploitation | Niv |
|---|---|---|---|---|
| Service dédié interne (retenu) | Bonne, séparé du reste de la logique métier | Aucun mécanisme d'authentification repéré sur ce service à ce jour (déjà noté en 5.2.5) | Hébergement et maintenance à la charge de VNWeb | L3 |
| Stockage objet cloud géré | Bonne, le fournisseur gère l'isolation et le durcissement de la brique de stockage | Contrôle d'accès fin natif (droits par espace de stockage, liens d'accès à durée limitée) | Facturation à l'usage, pas d'hébergement à maintenir soi-même | L2 |
| Stockage intégré au backend | Faible, une faille sur l'envoi de fichiers s'exécute dans le même processus que le reste de la logique métier et de l'accès à la base de données | Hérite du contrôle d'accès déjà en place sur le backend | Aucun service supplémentaire à héberger | L3 |

**Recommandation** : séparer le service de fichiers du reste de l'API reste une bonne décision d'isolation, elle est confirmée par les bonnes pratiques de réduction de surface d'attaque. Mais le choix d'un service interne plutôt qu'un stockage objet cloud managé laisse à la charge de VNWeb la responsabilité de sécuriser lui-même ce service, ce qui n'est pas encore fait puisqu'aucune authentification n'y a été repérée. Un service cloud managé aurait fourni ce contrôle d'accès nativement, au prix d'une dépendance à un fournisseur tiers et d'une facturation à l'usage. Ce n'est pas un point à trancher immédiatement dans ce document, mais un arbitrage à documenter dans le plan de sécurisation (section 9) : durcir le service actuel, ou migrer vers un stockage géré.

**Sources citées pour ce benchmark** : OWASP Top 10 (catégorie A03:2021, Injection), OWASP Password Storage Cheat Sheet, OWASP ASVS (contrôle d'accès et gestion de session), documentation produit officielle du framework backend, et lecture directe du code source de ce dépôt.

---

## 6. Plan d'Audit de Sécurité

### 6.1 Périmètre de l'audit

#### 6.1.1 Composants couverts

| Composant | Stack | Rôle | Exposition |
|---|---|---|---|
| Backend | AdonisJS 6, ORM Lucid, MySQL | API, authentification, données métier | Publique (API et canal temps réel) |
| Interface utilisateur | React 19, Ionic, PWA installable | Écrans utilisés par les parents, professeurs et élèves | Client, servi publiquement |
| Service de fichiers | Express, Multer | Envoi et diffusion de fichiers (photos, vidéos) | Publique, aucune authentification constatée (voir F-06) |

#### 6.1.2 Couches auditées

1. Code applicatif : logique métier, validation des entrées, requêtes vers la base de données, dans les trois composants.
2. Dépendances tierces : les bibliothèques utilisées par chaque composant (versions figées, vulnérabilités connues).
3. Secrets et configuration sensible : présence et exclusion du suivi de version des fichiers contenant des secrets, gestion de la clé de chiffrement applicative.
4. Configuration réseau : autorisations d'origine sur l'API et sur le canal temps réel, en-têtes de sécurité.
5. Contrôle d'accès : vérification de rôle, vérification de propriété (le lien entre un utilisateur et l'enfant qu'il supervise).
6. Stockage des données : champs en base, chiffrement ou absence de chiffrement des données personnelles.
7. Analyse dynamique : reportée à la disponibilité d'un environnement de test.

#### 6.1.3 Priorité aux parcours touchant les mineurs

Le cœur de gravité de cette mission de sécurisation est la protection des données des élèves mineurs (moins de 15 ans), qui n'ont pas de compte propre et sont gérés via le compte de leur parent superviseur. Deux raisons concrètes placent ces parcours en tête de liste, toutes deux vérifiées par lecture directe du code (détail en 6.3) : le lien entre un compte parent et le compte de l'enfant qu'il supervise peut être créé par un tiers qui connaît seulement l'identifiant de l'enfant (F-01), et les champs personnels de ces mineurs (date de naissance, téléphone, adresse, code postal, ville, nom, prénom) sont stockés en clair dans la table des comptes utilisateurs (F-04). Tout ce qui touche à la relation superviseur/enfant, à l'inscription d'un enfant, et au stockage de ses données, passe donc en priorité haute par défaut dans ce plan.

### 6.2 Méthodologie

Six catégories d'audit, menées dans cet ordre de dépendance logique (une catégorie amont peut invalider un contrôle en aval) :

1. **Analyse statique du code (SAST)** : recherche de vulnérabilités dans le code source sans exécution, sur les trois composants.
2. **Audit des dépendances** : recherche de vulnérabilités connues sur les bibliothèques figées utilisées par chaque composant.
3. **Détection de secrets** : recherche de clés, jetons ou mots de passe qui auraient pu être enregistrés par erreur dans l'historique du code, malgré leur exclusion prévue (voir F-13).
4. **Revue manuelle de configuration** : CORS, gestion de session, validation d'entrée, limitation de débit, en s'appuyant sur la lecture directe des fichiers de configuration.
5. **Revue de contrôle d'accès** : pour chaque route sensible, vérifier que le contrôle porte sur la propriété de la ressource et pas seulement sur le rôle de l'appelant.
6. **Analyse dynamique (DAST)** : contre un environnement de test réel, une fois disponible. Un DAST contre rien ne produit rien de valide.

Le référentiel utilisé pour juger la solidité d'un contrôle est l'OWASP ASVS (Application Security Verification Standard), le même référentiel que celui utilisé pour juger la preuve du bloc 3 du RNCP37173, ce qui garde une grille de lecture cohérente entre l'implémentation et l'évaluation. En complément, l'OWASP Top 10:2021 sert de nomenclature courte pour classer chaque constat :

| Finding | Catégorie OWASP Top 10:2021 |
|---|---|
| F-01, contrôle d'accès superviseur/enfant | A01, Broken Access Control |
| F-02, autorisation d'origine ouverte sur le canal de messagerie | A05, Security Misconfiguration |
| F-03, paramètres de hachage de mot de passe non ajustés | A02, Cryptographic Failures |
| F-04, données de mineurs en clair | A02, Cryptographic Failures |
| F-05, code mort issu de l'outil de temps réel non retenu | A05, Security Misconfiguration (surface inutile) |
| F-06, absence d'authentification sur le service de fichiers | A01, Broken Access Control |
| F-07, absence de tests et d'automatisation de vérification | Hors nomenclature Top 10, condition de A06 (composants non vérifiés) |
| F-08, redondance entre deux champs d'administration | A04, Insecure Design |
| F-09, auto-assignation aux groupes sans validation métier | A01, Broken Access Control |
| F-10, injection via le nom de fichier non filtré sur le service de fichiers | A03, Injection |
| F-11, absence de limitation de débit sur la réinitialisation de mot de passe | A07, Identification and Authentication Failures |
| F-12, absence de protection CSRF explicite | A05, Security Misconfiguration |
| F-13, exclusion de suivi de version incomplète sur le service de fichiers | A05, Security Misconfiguration |

**Outils par catégorie :**

| Catégorie | Outil retenu | Justification |
|---|---|---|
| SAST principal | CodeQL | Analyse de flux de données de bout en bout, pertinent pour tracer le chemin de la donnée de F-10, s'intègre nativement à la plateforme d'hébergement du code. |
| SAST complémentaire | Semgrep | Règles personnalisables, utile pour cibler des motifs spécifiques aux briques utilisées par ce projet, exécution locale rapide avant chaque envoi de code. |
| Audit de dépendances | Un outil de veille automatique sur les vulnérabilités connues, plus une vérification bloquante intégrée à la validation continue | Zéro infrastructure à maintenir, alerte automatique sur nouvelle vulnérabilité, cohérent avec les trois composants qui ont chacun leurs propres dépendances. |
| Détection de secrets | Un scanner de secrets versionné dans le pipeline, en complément du filet natif de la plateforme d'hébergement | Double filet, pertinent vu la lacune constatée en F-13. |
| Revue de configuration manuelle | Lecture directe et checklist du référentiel OWASP ASVS (chapitres contrôle d'accès, cryptographie du stockage, sécurité des API) | Pas d'outil automatique fiable pour juger si un contrôle d'accès porte sur la bonne propriété métier (F-01) : la lecture humaine reste nécessaire. |
| DAST | OWASP ZAP | Voir 6.2.2. |

#### 6.2.1 Choix de l'outil SAST

| Option | Intégration à la validation continue | Couverture du langage utilisé | Coût | Niveau de preuve |
|---|---|---|---|---|
| CodeQL | Native sur la plateforme d'hébergement de ce dépôt | Analyse de flux de données inter-procédurale | Gratuit sur ce type de dépôt | L2, documentation produit officielle |
| Semgrep | Intégration tierce, configuration simple | Analyse par motifs, règles communautaires et personnalisées | Gratuit en usage standard | L2, documentation produit officielle |
| Analyseur de style avec règles de sécurité complémentaires | Déjà présent si l'analyseur de style du projet est actif | Détection de motifs à risque limités, pas d'analyse de flux | Gratuit | L3, consensus technique large, pas un standard formel |

**Recommandation** : CodeQL en outil principal, Semgrep en complément ciblé. CodeQL est le seul des trois à tracer un flux de données de bout en bout, pertinent pour confirmer ou infirmer F-10, et s'intègre sans service tiers. Semgrep comble sa limite principale : des règles rapides à écrire pour des motifs propres à ce dépôt (une requête construite en contournant les protections habituelles de l'ORM, un point d'entrée sans validation des données reçues). L'analyseur de style avec règles de sécurité reste utile en filet local avant chaque envoi de code, mais ne remplace pas une analyse de flux.

#### 6.2.2 Choix de l'outil DAST

| Option | Coût | Automatisable en continu | Couverture attendue sur ce périmètre | Niveau de preuve |
|---|---|---|---|---|
| OWASP ZAP | Gratuit | Oui, des intégrations officielles existent pour un scan rapide et pour un scan complet | Bonne sur les échanges HTTP classiques ; ne couvre pas nativement le canal de messagerie en temps réel (F-02) | L2, projet officiel de la fondation OWASP |
| Nikto | Gratuit | Oui, mais moins maintenu pour les API modernes | Orienté serveur web générique, peu adapté à une API construite pour des échanges de données structurées | L3, consensus technique large, outil plus ancien |
| Burp Suite (édition communautaire) | Gratuit en usage manuel, payant pour l'automatisation continue | Non automatisable en continu dans l'édition gratuite | Bonne en usage manuel ponctuel, mais pas dans un pipeline continu | L2, documentation produit officielle |

**Recommandation** : OWASP ZAP. C'est le seul des trois à combiner gratuité et automatisation continue réelle, ce qui correspond à la contrainte de ce projet (pas de budget outillage, équipe de projet de trois personnes). Sa limite connue, la non-couverture native du canal de messagerie en temps réel, doit être compensée par un test manuel ciblé sur cette messagerie (F-02).

### 6.3 Findings identifiés

Chaque finding ci-dessous a été relu directement dans le code du dépôt avant rédaction de cette section. Une correction méthodologique au passage : un constat provisoire sur la fonctionnalité de commentaires (supposée retirée de l'interface) a été vérifié puis infirmé par une relecture du code réel de l'interface utilisateur : cette fonctionnalité est active des deux côtés, avec des contrôles d'accès en place. Ce n'est donc pas un finding, et la section 2.3 et l'objectif correspondant ont été corrigés en conséquence.

| ID | Criticité | Catégorie OWASP | Constat | Recommandation | Statut |
|---|---|---|---|---|---|
| F-01 | Haute | A01 | La fonctionnalité d'invitation d'un tiers comme superviseur d'un enfant vérifie seulement que l'appelant a un rôle autorisé à superviser des enfants en général (professeur, superviseur, élève-superviseur), pas qu'il est déjà superviseur de l'enfant précis désigné dans la demande. | Ajouter une vérification que l'appelant est déjà superviseur de l'enfant ciblé avant d'envoyer une invitation. | À faire |
| F-02 | Haute | A05 | Le canal de messagerie en temps réel accepte des connexions depuis n'importe quelle origine, alors que l'API HTTP principale restreint bien les siennes à une liste explicite de domaines connus. Le canal qui porte la messagerie privée accepte donc des connexions que l'API HTTP refuserait. | Aligner la configuration d'origine du canal de messagerie sur la même liste que l'API HTTP. | À faire |
| F-03 | Moyenne | A02 | Le hachage des mots de passe utilise l'algorithme scrypt avec les paramètres fournis par défaut par le framework, non ajustés au contexte du projet. Argon2id, premier choix recommandé par l'OWASP pour le hachage de mots de passe, n'est pas utilisé. | Revoir les paramètres de hachage après mesure de la capacité du serveur, ou migrer vers Argon2id. | À faire |
| F-04 | Haute | A02 | Les champs personnels des mineurs (date de naissance, téléphone, adresse, code postal, ville, nom, prénom) sont stockés en clair dans la table des comptes utilisateurs. Une clé de chiffrement applicative existe déjà mais n'est pas utilisée pour chiffrer ces champs. | Évaluer un chiffrement au niveau champ sur les données les plus sensibles, avec la clé applicative déjà disponible. | À faire |
| F-05 | Basse | A05 | Le contrôleur de démonstration hérité de l'outil de temps réel qui n'a finalement pas été retenu pour la messagerie n'est relié à aucune route accessible de l'application : code mort, reliquat du gabarit de démarrage de cet outil. | Retirer ce contrôleur et sa configuration associée si aucun usage n'est prévu. | À faire |
| F-06 | Haute | A01 | Le service de fichiers ne comporte aucune vérification d'identité, aucun jeton, aucune session sur les fonctions d'envoi et de nettoyage de médias. Un contrôle par origine et adresse réseau existait mais est entièrement désactivé dans le code. La taille de fichier acceptée est en outre très large (plusieurs giga-octets par fichier). | Mettre en place une authentification minimale (un jeton partagé entre les services internes) sur ce service, en priorité sur les fonctions d'envoi de fichiers. | À faire |
| F-07 | Haute | Condition de A06 | Aucun test automatisé n'a été trouvé sur les trois composants, et aucune automatisation de vérification n'est configurée. | Mettre en place une validation continue minimale (vérification de style, de types, des dépendances, et analyse statique). | À faire |
| F-08 | Moyenne | A04 | Deux champs distincts du compte utilisateur portent chacun une notion d'administration, sans mécanisme garantissant qu'ils restent synchronisés entre eux. Le contrôle d'accès administrateur ne s'appuie que sur l'un des deux, ce qui limite le risque immédiat, mais la double source de vérité complique toute revue de droits future. | Choisir une source de vérité unique pour le statut administrateur et migrer l'autre champ. | À faire |
| F-09 | Basse à moyenne | A01 | Tout utilisateur authentifié peut s'attacher lui-même à n'importe quel groupe existant, sans vérification de son rôle ni d'une règle métier d'éligibilité au groupe ciblé. | Trancher avec l'équipe produit si c'est un choix fonctionnel à documenter ou un défaut à corriger. | À trancher |
| F-10 | Haute | A03 | Lors de l'envoi d'un fichier au service de fichiers, l'extension du fichier n'est pas filtrée par une liste blanche avant d'être utilisée dans un traitement de conversion ou de compression exécuté directement sur le serveur. Un nom de fichier construit à cet effet pourrait atteindre ce traitement sans être filtré. | Confirmer par un test dédié, puis filtrer l'extension par liste blanche avant tout traitement de ce type. | À faire, priorité immédiate |
| F-11 | Moyenne | A07 | Aucune limitation du nombre de tentatives n'a été trouvée sur la fonctionnalité de réinitialisation de mot de passe, contrairement à la fonctionnalité de connexion qui en dispose déjà. | Ajouter une limitation de débit sur la fonctionnalité d'envoi du lien de réinitialisation. | À faire |
| F-12 | Basse | A05 | Aucune protection explicite contre la falsification de requête intersite (CSRF) n'est activée. L'authentification par jeton plutôt que par session réduit la surface d'exposition classique à ce type d'attaque, mais un cookie HTTP est tout de même configuré, dont l'usage réel reste à vérifier. | Confirmer l'usage réel de ce cookie, puis statuer sur le besoin d'une protection CSRF explicite. | À vérifier |
| F-13 | Basse | A05 | Le fichier qui définit les éléments exclus du suivi de version pour le service de fichiers n'exclut pas les fichiers de configuration contenant des secrets, contrairement aux deux autres composants. Aucune fuite actuelle n'a été constatée. | Corriger cette exclusion avant tout ajout futur de secret dans ce composant. | À faire |

### 6.4 Synthèse et priorisation

Ordre de priorité retenu, du plus urgent au moins urgent :

1. Corriger le contrôle d'accès sur l'invitation de superviseur (F-01) : correctif le plus direct sur la protection des mineurs.
2. Confirmer le risque d'injection sur le service de fichiers (F-10) par un test dédié, puis filtrer l'extension par liste blanche avant tout traitement à risque.
3. Mettre en place une authentification minimale sur le service de fichiers (F-06), en priorité sur les fonctions d'envoi.
4. Aligner la configuration d'origine du canal de messagerie sur celle de l'API HTTP (F-02).
5. Évaluer le chiffrement au niveau champ des données personnelles des mineurs (F-04), avec la clé applicative déjà disponible.
6. Mettre en place une validation continue minimale (F-07), condition de fiabilité de tous les correctifs suivants.
7. Ajouter la détection de secrets à la validation continue et corriger l'exclusion de suivi de version du service de fichiers (F-13).
8. Ajouter une limitation de débit sur la réinitialisation de mot de passe (F-11).
9. Revoir les paramètres de hachage des mots de passe ou migrer vers Argon2id (F-03), après mesure de la capacité réelle du serveur de production.
10. Trancher avec l'équipe produit le statut de F-09 (auto-assignation aux groupes).
11. Nettoyer le code mort (F-05) et clarifier la redondance entre les deux champs d'administration (F-08).
12. Confirmer l'usage réel du cookie HTTP et statuer sur le besoin d'une protection CSRF explicite (F-12).
13. Une fois un environnement de test disponible, lancer le premier scan de sécurité dynamique contre le backend, puis un test manuel ciblé sur le canal de messagerie (F-02).

---

## 7. AMDEC, Analyse des Modes de Défaillance, Effets et Criticité

*Périmètre : ce chapitre couvre exclusivement les défaillances accidentelles, techniques ou opérationnelles, survenant sans intention malveillante (panne, erreur de configuration, indisponibilité d'un service tiers). Les scénarios de menace intentionnelle sont traités dans le chapitre EBIOS RM (section 8).*

### Méthode et barème

IPR = G × O × D

| Critère | Définition | Échelle |
|---|---|---|
| G, Gravité | Impact sur les utilisateurs et le service | 1 (négligeable) à 10 (catastrophique) |
| O, Occurrence | Fréquence d'apparition estimée | 1 (quasi-impossible) à 10 (fréquent) |
| D, Détectabilité | Difficulté à détecter la défaillance avant impact | 1 (immédiatement détecté) à 10 (indétectable) |

Seuils : IPR inférieur à 50, acceptable. IPR entre 50 et 100, à surveiller. IPR supérieur à 100, action corrective prioritaire.

### Analyse par composant

**AM-01, Backend, crash du processus applicatif**
IPR : G 7 × O 2 × D 2 = 28, acceptable.
Effet : l'ensemble de l'application dépend de ce composant unique ; son arrêt bloque à la fois les comptes, les groupes, les posts et la messagerie. Cause : exception non gérée ou fuite mémoire sur un processus de longue durée. Action corrective : mettre en place un mécanisme de supervision du processus avec redémarrage automatique, et exposer un point de contrôle de bon fonctionnement surveillé.

**AM-02, Base de données, indisponibilité ou saturation**
IPR : G 8 × O 2 × D 3 = 48, acceptable.
Effet : toutes les fonctionnalités qui dépendent d'un accès à la base (comptes, groupes, posts, messagerie) deviennent indisponibles. Cause : pic de charge inhabituel, requête bloquante, ou panne du serveur qui héberge la base. Action corrective : surveiller la charge et les temps de réponse, prévoir une alerte en cas de dépassement de seuil.

**AM-03, Base de données, absence de sauvegarde automatisée confirmée, CRITIQUE**
IPR : G 10 × O 2 × D 8 = 160, action corrective prioritaire.
Effet : en cas de panne matérielle ou de corruption de données, perte définitive et irréversible des données de tous les utilisateurs, y compris des données personnelles de mineurs. Cause : aucune stratégie de sauvegarde automatisée régulière n'a été confirmée à ce jour [à vérifier auprès de VNWeb avant la version finale de ce dossier]. La détectabilité est jugée mauvaise, une absence de sauvegarde ne se remarque pas tant qu'aucun incident ne survient. Action corrective : mettre en place une sauvegarde automatisée régulière de la base de données, avec test de restauration périodique, avant toute autre priorité de ce plan.

**AM-04, Service de fichiers, échec du traitement de conversion des médias**
IPR : G 5 × O 4 × D 3 = 60, à surveiller.
Effet : l'envoi d'un fichier échoue pour l'utilisateur, sans qu'un message d'erreur clair n'explique la cause. Cause : fichier corrompu, format non attendu, ou traitement de compression qui échoue sur un fichier malformé. Action corrective : encadrer ce traitement d'une gestion d'erreur robuste qui échoue proprement plutôt que de bloquer le service entier, et valider le fichier avant traitement.

**AM-05, Envoi d'email de validation de compte, échec de livraison**
IPR : G 6 × O 3 × D 4 = 72, à surveiller.
Effet : un nouvel utilisateur ne reçoit pas son lien de validation, valide une heure seulement ; seul l'admin peut renvoyer un nouveau lien, ce qui ajoute un délai humain à un problème déjà temporellement contraint. Cause : panne ou lenteur du service d'envoi d'email, boîte de réception pleine côté destinataire, ou domaine expéditeur mal configuré. Action corrective : surveiller les échecs de livraison et alerter au-delà d'un seuil, envisager d'allonger la durée de validité du lien ou d'permettre un renvoi self-service après expiration.

**AM-06, Notifications push, échec silencieux d'envoi**
IPR : G 4 × O 5 × D 4 = 80, à surveiller.
Effet : un utilisateur ne reçoit pas la notification d'un nouveau message ou d'un nouveau post, sans qu'aucune alerte ne soit levée côté application ni côté utilisateur. Cause : abonnement expiré côté navigateur, ou identifiant de notification devenu invalide. Action corrective : consigner systématiquement les échecs d'envoi et surveiller leur fréquence, plutôt que de les laisser disparaître silencieusement dans les journaux.

**AM-07, Canal de messagerie en temps réel, perte de connexion silencieuse**
IPR : G 5 × O 3 × D 5 = 75, à surveiller.
Effet : les messages échangés en temps réel (indicateur de saisie, réception immédiate) cessent de circuler sans que l'utilisateur ne soit informé de la coupure ; le message reste malgré tout enregistré, mais sa remise immédiate échoue. Cause : coupure réseau, redémarrage du serveur, ou défaillance du canal temps réel. Action corrective : afficher un indicateur de connexion visible côté interface, et prévoir une reconnexion automatique.

**AM-08, Interface utilisateur (PWA), échec de mise à jour du Service Worker**
IPR : G 6 × O 2 × D 4 = 48, acceptable.
Effet : des utilisateurs restent bloqués sur une version obsolète de l'application installée, potentiellement privés d'un correctif de sécurité déjà déployé côté serveur. Cause : cache du navigateur qui ne se rafraîchit pas correctement après une mise à jour, ou erreur lors du déploiement d'une nouvelle version. Action corrective : forcer une vérification de mise à jour à l'ouverture de l'application, et informer l'utilisateur qu'une nouvelle version est disponible.

### Synthèse AMDEC

| Niveau | Composants | Action |
|---|---|---|
| Critique (IPR > 100) | Absence de sauvegarde de la base de données (AM-03, IPR 160) | Action immédiate, à traiter avant toute autre priorité de ce plan |
| À surveiller (IPR 50 à 100) | Traitement des médias (AM-04, 60), envoi d'email de validation (AM-05, 72), notifications push (AM-06, 80), messagerie temps réel (AM-07, 75) | Mesures de surveillance et d'alerte à planifier dans le plan de sécurisation (section 9) |
| Acceptable (IPR < 50) | Crash du backend (AM-01, 28), indisponibilité de la base (AM-02, 48), mise à jour du Service Worker (AM-08, 48) | Bonnes pratiques à maintenir |

---

## 8. EBIOS RM, Analyse des Menaces Intentionnelles

EBIOS RM (Expression des Besoins et Identification des Objectifs de Sécurité, Risk Manager) est la méthode d'analyse de risques publiée par l'ANSSI. Elle est alignée sur ISO 27005, le standard international qui définit le cadre de gestion du risque lié à la sécurité de l'information, structurée en cinq ateliers.

*Périmètre : ce chapitre couvre exclusivement les scénarios de menace intentionnelle impliquant un acteur malveillant. Les défaillances accidentelles sont traitées en section 7 (AMDEC). Chaque scénario identifie le pilier de la triade CIA affecté : Confidentialité, Intégrité, Disponibilité.*

### Atelier 1 : Cadrage et socle de sécurité

L'atelier 1 délimite le périmètre de l'analyse et recense les actifs à protéger.

**Valeurs métier**

| ID | Valeur métier |
|---|---|
| VM-1 | Données personnelles des mineurs (identité, date de naissance, coordonnées) |
| VM-2 | Confidentialité de la relation superviseur-enfant (qui supervise qui est une donnée sensible en soi) |
| VM-3 | Disponibilité du service, utilisé au quotidien par l'école |
| VM-4 | Intégrité des comptes, des rôles et des permissions |
| VM-5 | Confidentialité des échanges de la messagerie privée |

**Biens supports**

| ID | Bien support | VM couverte |
|---|---|---|
| BS-1 | Backend (API, authentification, logique métier) | VM-1, VM-2, VM-3, VM-4, VM-5 |
| BS-2 | Base de données (comptes, groupes, superviseurs, messages) | VM-1, VM-2, VM-4, VM-5 |
| BS-3 | Service de fichiers (photos, avatars) | VM-1, VM-3 |
| BS-4 | Interface utilisateur (PWA) | VM-1, VM-3 |
| BS-5 | Canal de messagerie en temps réel | VM-3, VM-5 |
| BS-6 | Mécanisme d'invitation de superviseur | VM-2, VM-4 |

**Événements redoutés**

| ID | Événement redouté | CIA |
|---|---|---|
| ER-1 | Exfiltration des données personnelles des mineurs | C |
| ER-2 | Accès non autorisé à la supervision d'un enfant par un tiers illégitime | C |
| ER-3 | Altération des rôles ou des permissions sans consentement | I |
| ER-4 | Indisponibilité prolongée du service | D |
| ER-5 | Interception ou accès non autorisé aux messages privés | C |

### Atelier 2 : Sources de risques et objectifs visés

| ID | Source de risque | Motivation | Capacité | Objectif visé |
|---|---|---|---|---|
| SR-1 | Cybercriminel externe opportuniste | Revente de données personnelles | Élevée, outillage automatisé | OV-1, exfiltrer les comptes et les données de mineurs |
| SR-2 | Utilisateur interne (professeur ou parent) avec de mauvaises intentions | Curiosité, conflit familial, litige de garde | Faible, accès légitime limité | OV-2, accéder à la supervision ou aux données d'un enfant qui n'est pas le sien |
| SR-3 | Attaquant opportuniste automatisé | Nuisance, défi technique | Moyenne, outils automatisés | OV-3, rendre le service indisponible |
| SR-4 | Attaquant ayant obtenu des identifiants (hameçonnage, réutilisation de mot de passe) | Accès aux données d'un compte spécifique | Faible à moyenne | OV-4, usurper l'accès à un enfant via le compte de son parent |

### Atelier 3 : Scénarios stratégiques

| ID | Scénario | Source | Chemin | CIA |
|---|---|---|---|---|
| SS-1 | Compromission de l'API pour exfiltration de masse | SR-1 | Exploitation d'une faille de contrôle d'accès ou d'injection, accès à la base de données | C |
| SS-2 | Détournement du mécanisme d'invitation de superviseur | SR-2 | Envoi d'une invitation à devenir superviseur d'un enfant qui n'est pas le sien, accès à ses données | C |
| SS-3 | Déni de service sur les points d'entrée publics | SR-3 | Saturation des routes d'inscription ou de connexion | D |
| SS-4 | Compromission d'un compte parent | SR-4 | Connexion avec des identifiants volés, accès aux enfants supervisés par ce compte | C |

### Atelier 4 : Scénarios opérationnels

**SO-1, bourrage d'identifiants sur la connexion**
Issu de : SS-4 | CIA : Confidentialité
Description : un attaquant teste des combinaisons d'adresse email et de mot de passe issues de fuites externes sur la page de connexion.
Impact : compromission d'un compte parent, accès aux enfants supervisés par ce compte. Probabilité : Moyenne, la connexion dispose déjà d'une limitation de débit constatée dans le code (F-11). Contre-mesures : cette limitation existante est un vrai frein à conserver ; une authentification à deux facteurs pour les comptes superviseurs et une alerte sur connexion depuis un nouvel appareil renforceraient la protection.

**SO-2, détournement du mécanisme d'invitation de superviseur**
Issu de : SS-2 | CIA : Confidentialité
Description : comme établi en F-01 (section 6.3), un utilisateur ayant un rôle autorisé à superviser des enfants en général peut inviter un tiers à devenir superviseur d'un enfant qui n'est pas le sien, en connaissant seulement son identifiant.
Impact : accès non autorisé aux données personnelles d'un mineur par un tiers. Probabilité : Élevée, aucune barrière technique ne s'y oppose actuellement, il suffit de connaître l'identifiant de l'enfant. Contre-mesures : le correctif est déjà identifié en F-01 et positionné en tête de la priorisation du plan d'audit (section 6.4).

**SO-3, réinitialisation de mot de passe abusive**
Issu de : SS-1 ou SS-4 | CIA : Disponibilité, et Confidentialité de façon indirecte
Description : comme établi en F-11 (section 6.3), aucune limitation de débit n'encadre la demande de réinitialisation de mot de passe, contrairement à la connexion.
Impact : un attaquant peut solliciter massivement l'envoi de liens de réinitialisation, saturer le service d'envoi d'email, ou tenter de déterminer quelles adresses email correspondent à un compte existant. Probabilité : Élevée, la route est publique et aucune protection n'a été constatée. Contre-mesures : étendre à cette route la même limitation de débit que celle déjà en place sur la connexion.

**SO-4, interception de la messagerie via une origine non autorisée**
Issu de : SS-1 | CIA : Confidentialité
Description : comme établi en F-02 (section 6.3), le canal de messagerie en temps réel accepte des connexions depuis n'importe quelle origine.
Impact : un site tiers malveillant pourrait tenter d'établir une connexion au canal de messagerie depuis le navigateur d'une victime, pendant qu'elle le consulte en étant connectée à l'application. Probabilité : Moyenne, ce scénario suppose qu'une victime visite une page malveillante pendant qu'elle est connectée. Contre-mesures : restreindre l'origine autorisée à la même liste de domaines que l'API HTTP, déjà recommandé en F-02.

**SO-5, injection de commande via l'envoi d'un fichier**
Issu de : SS-1 | CIA : Intégrité et Disponibilité, potentiellement Confidentialité si le serveur est compromis
Description : comme établi en F-10 (section 6.3), l'extension d'un fichier envoyé au service de fichiers n'est pas filtrée avant d'être utilisée dans un traitement exécuté directement sur le serveur.
Impact : exécution de commandes arbitraires sur le serveur qui héberge le service de fichiers, l'un des scénarios les plus graves de ce dossier. Probabilité : à confirmer par un test dédié, déjà positionné comme priorité immédiate en F-10, mais techniquement plausible en l'absence de validation de l'extension. Contre-mesures : filtrer l'extension par liste blanche avant tout traitement, déjà en tête de la priorisation du plan d'audit.

**SO-6, déni de service sur les routes d'inscription**
Issu de : SS-3 | CIA : Disponibilité
Description : un attaquant automatise un grand nombre de requêtes vers les routes publiques d'inscription pour saturer le service.
Impact : service ralenti ou indisponible pour les utilisateurs légitimes. Probabilité : Moyenne, la connexion dispose déjà d'une limitation de débit, mais son extension aux routes d'inscription reste à vérifier. Contre-mesures : étendre la limitation de débit à toutes les routes publiques, pas seulement à la connexion.

### Atelier 5 : Traitement des risques

| Scénario | Risque sans mesure | Mesures retenues | Risque résiduel | Priorité |
|---|---|---|---|---|
| SO-2, détournement invitation superviseur | Très élevé | Vérification du lien superviseur-enfant avant invitation | Faible | P0, bloquant |
| SO-5, injection de commande fichier | Très élevé | Filtrage de l'extension par liste blanche | Faible | P0, bloquant |
| SO-3, réinitialisation abusive | Élevé | Limitation de débit alignée sur la connexion | Faible | P1 |
| SO-4, messagerie origine ouverte | Élevé | Restriction de l'origine autorisée | Faible | P1 |
| SO-1, bourrage d'identifiants connexion | Moyen, déjà limité | Authentification à deux facteurs (superviseurs), alerte nouvel appareil | Faible | P2 |
| SO-6, déni de service inscription | Moyen | Extension de la limitation de débit | Faible | P2 |

Risques bloquants avant toute mise en production :

- SO-2 (détournement de l'invitation de superviseur) : tout envoi d'invitation doit vérifier que l'appelant est déjà superviseur de l'enfant ciblé.
- SO-5 (injection de commande via l'envoi d'un fichier) : l'extension d'un fichier envoyé doit être filtrée par liste blanche avant tout traitement exécuté sur le serveur.

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

[à rédiger, en ancrant sur le projet réel : audit et sécurisation de api-dance (AdonisJS, MySQL/Lucid), app-dance (React/Ionic) et cdn-app-dance (Express, Multer). S'appuyer sur le travail effectivement mené avec l'agent `@axel` (SAST/DAST, CI/CD) et `@jury-rncp37173` (auto-évaluation bloc par bloc) comme preuve concrète plutôt que sur une généralité.]

### 13.2 Compétences organisationnelles et transverses

[à rédiger : expériences réelles de collaboration (hackathons, équipe VNWeb) ; ne pas réutiliser telles quelles les mentions d'un projet ou d'une équipe qui ne correspondent pas à la réalité de cette alternance.]

### 13.3 Application des compétences acquises : [projet personnel, si applicable]

[à rédiger si un projet personnel distinct existe ; sinon, retirer cette sous-section plutôt que la laisser vide dans la version finale.]

### 13.4 Axes de progression

[à rédiger : axes réels et personnels, pas une généralité recopiée.]
