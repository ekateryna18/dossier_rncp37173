# Sources citées dans la politique de sécurité, VNWeb

*Note de travail interne de l'auditrice, hors dossier officiel : elle liste, pour validation, chaque référence normative citée dans `Politique_Securite_VNWeb.md`, avec le texte lui-même quand il a pu être récupéré, et elle relie chaque règle (R-xx) à la ou aux sources qui la fondent. Elle ne remplace pas la lecture des textes eux-mêmes, surtout là où le texte cité ci-dessous vient d'un site tiers plutôt que de la source officielle.*

## 0. Comment lire ce fichier

Quatre niveaux de vérification :

- **Vérifié, source officielle ouverte** : le texte a été ouvert directement sur le site de l'organisme qui fait autorité (Légifrance, code.travail.gouv.fr, cyber.gouv.fr, cnil.fr, owasp.org, agefiph.fr).
- **Vérifié via un site tiers** : le texte cité ci-dessous a été récupéré sur un site qui reproduit le texte officiel (par exemple un miroir du règlement européen), pas sur le site de l'organisme d'origine lui-même. Le texte est normalement identique, mais n'a pas été recoupé avec l'original dans cette session.
- **Vérifié par recherche, pas par lecture directe** : une recherche a confirmé l'existence et le sujet du texte, sans en ouvrir le texte intégral.
- **Non vérifié (connaissance générale)** : la citation vient de ma mémoire de formation, plausible et couramment admise, mais non confrontée au texte officiel. Niveau le plus faible : à vérifier vous-même avant de vous y fier pour un dossier qui compte.

Aucune adresse n'est inventée. Un texte est mis entre guillemets seulement quand il a été lu sur la page citée à côté ; sinon, c'est marqué non vérifié.

---

## 1. RGPD (Règlement général sur la protection des données)

**Cité dans la politique** : chapitre V (article 44 et suivants), articles 9, 28, 30 ; section 7, section 13, section 15, section 22.

**Statut global** : Vérifié via un site tiers pour les cinq articles ci-dessous (textes récupérés sur `privacy-regulation.eu` et `gdpr-info.eu`, deux sites qui reproduisent le règlement, pas sur EUR-Lex directement).

### Article 9, catégories particulières de données (cité section 22)

> « Le traitement des données à caractère personnel qui révèle l'origine raciale ou ethnique, les opinions politiques, les convictions religieuses ou philosophiques ou l'appartenance syndicale, ainsi que le traitement des données génétiques, des données biométriques aux fins d'identifier une personne physique de manière unique, des données concernant la santé ou des données concernant la vie sexuelle ou l'orientation sexuelle d'une personne physique sont interdits. » (paragraphe 1, résumé de la structure ; le texte français exact des dix exceptions du paragraphe 2 n'a pas été récupéré mot pour mot dans cette session, seul un résumé de leur liste l'a été : consentement explicite, obligations en matière de droit du travail et de sécurité sociale, sauvegarde des intérêts vitaux, traitement par un organisme à but non lucratif, données rendues publiques par la personne, exercice ou défense d'un droit en justice, motif d'intérêt public important, médecine du travail et de la santé, santé publique, archivage et recherche).

### Article 28, sous-traitant (cité section 13, dans la règle R-20)

> « Lorsqu'un traitement doit être effectué pour le compte d'un responsable du traitement, celui-ci fait uniquement appel à des sous-traitants qui présentent des garanties suffisantes quant à la mise en œuvre de mesures techniques et organisationnelles appropriées de manière à ce que le traitement réponde aux exigences du présent règlement et garantisse la protection des droits de la personne concernée. » (paragraphe 1)

> « Le traitement par un sous-traitant est régi par un contrat ou un autre acte juridique au titre du droit de l'Union ou du droit d'un État membre, qui lie le sous-traitant à l'égard du responsable du traitement, définit l'objet et la durée du traitement, la nature et la finalité du traitement, le type de données à caractère personnel et les catégories de personnes concernées, et les obligations et les droits du responsable du traitement. » (début du paragraphe 3, qui se poursuit par une liste de clauses obligatoires, a) à h), non reproduite ici)

### Article 30, registre des activités de traitement (cité section 15, dans la règle R-26)

> « Chaque responsable du traitement et, le cas échéant, le représentant du responsable du traitement tiennent un registre des activités de traitement effectuées sous leur responsabilité. » (paragraphe 1 ; le registre doit ensuite contenir sept catégories d'information : identité du responsable, finalités, catégories de personnes et de données, catégories de destinataires, transferts vers un pays tiers, durées de conservation envisagées, description des mesures de sécurité)

### Article 33, notification à l'autorité de contrôle (le « délai de 72 heures » cité à la vulnérabilité VO-13, sans que la politique nomme l'article)

> « En cas de violation de données à caractère personnel, le responsable du traitement en notifie la violation en question à l'autorité de contrôle compétente conformément à l'article 55, dans les meilleurs délais et, si possible, 72 heures au plus tard après en avoir pris connaissance, à moins que la violation en question ne soit pas susceptible d'engendrer un risque pour les droits et libertés des personnes physiques. » (paragraphe 1)

### Chapitre V, article 44, principe général applicable aux transferts (cité section 15, dans la règle R-27)

> « Un transfert, vers un pays tiers ou à une organisation internationale, de données à caractère personnel qui font ou sont destinées à faire l'objet d'un traitement après ce transfert ne peut avoir lieu que si, sous réserve des autres dispositions du présent règlement, les conditions définies dans le présent chapitre sont respectées par le responsable du traitement et le sous-traitant, y compris pour les transferts ultérieurs de données à caractère personnel au départ du pays tiers ou de l'organisation internationale vers un autre pays tiers ou à une autre organisation internationale. Toutes les dispositions du présent chapitre sont appliquées de manière à ce que le niveau de protection des personnes physiques garanti par le présent règlement ne soit pas compromis. »

**Où vérifier vous-même, sur la source officielle** : le texte consolidé du règlement (UE) 2016/679 sur EUR-Lex (`eur-lex.europa.eu`, rechercher « règlement 2016/679 ») ou les fiches thématiques de la CNIL (`cnil.fr`). Ces adresses n'ont pas été ouvertes avec succès cette session (une tentative sur EUR-Lex n'a renvoyé aucun contenu lisible) : ce sont des points de départ, pas des liens dont le contenu a été confirmé ici.

**Point relevé pendant ce travail, à corriger dans la politique** : la règle R-28 (répondre aux demandes d'accès, de rectification ou de suppression) porte en réalité sur les droits de la personne concernée, qui relèvent des articles 15 à 22 du RGPD. La section 15 de la politique cite pourtant seulement le chapitre V et les articles 28 et 30 : les articles 15 à 22 n'y figurent pas, alors qu'ils fondent directement cette règle.

---

## 2. Code du travail, article L5213-6

**Cité dans la politique** : section 22 (accessibilité de la formation), règle R-44.

**Statut** : Vérifié, source officielle ouverte, dans cette conversation, à la rédaction de la section 22.

**Texte** :

> « Afin de garantir le respect du principe d'égalité de traitement à l'égard des travailleurs handicapés, l'employeur prend, en fonction des besoins dans une situation concrète, les mesures appropriées pour permettre aux travailleurs mentionnés aux 1° à 4° et 9° à 11° de l'article L. 5212-13 d'accéder à un emploi ou de conserver un emploi correspondant à leur qualification, de l'exercer ou d'y progresser ou pour qu'une formation adaptée à leurs besoins leur soit dispensée. »

L'article ajoute que l'employeur doit veiller à l'accessibilité des logiciels installés sur le poste de la personne concernée et à celle du télétravail, que ces mesures sont dues sous réserve que les charges qui en résultent ne soient pas disproportionnées (compte tenu des aides pouvant compenser tout ou partie de la dépense, article L. 5213-10), et que leur refus peut constituer une discrimination au sens de l'article L. 1133-3.

**Adresse ouverte** : https://code.travail.gouv.fr/code-du-travail/l5213-6

---

## 3. Agefiph, aide à l'adaptation des situations de travail

**Cité dans la politique** : section 22.1, à titre d'information.

**Statut** : Vérifié, source officielle ouverte, dans cette conversation.

**Texte** :

> « L'aide permet de participer au financement des aménagements du poste nécessaires pour tenir compte du handicap. »

Exemple donné par la page : pour un salarié ayant besoin d'un siège ergonomique adapté à son handicap, l'Agefiph finance la différence entre le coût d'un siège ordinaire et le coût du siège ergonomique. La demande est faite par l'employeur ou le travailleur non salarié ; la recommandation du médecin du travail (pour un salarié) en est un préalable. Le montant est indiqué comme « selon étude » : la page ne fixe aucun taux ni pourcentage, et ne mentionne aucune condition particulière pour les très petites entreprises.

**Adresse ouverte** : https://www.agefiph.fr/aides-financieres/aide-ladaptation-des-situations-de-travail

---

## 4. ANSSI, *Guide d'hygiène informatique*

**Cité dans la politique** : sections 9, 11, 12, 16, 21, et section 7 comme socle général. Aucun numéro de mesure ANSSI précis n'est cité : la politique s'appuie sur le guide dans son ensemble.

**Statut** : Vérifié, source officielle ouverte, le 21/09/2026, dans le cadre de la veille du projet.

**Texte** (éléments de présentation relevés sur la page officielle) :

> Titre : « Guide d'hygiène informatique », contenant « 42 mesures d'hygiène informatique essentielles ». Publié le 23 janvier 2017.

Le guide se présente comme « une feuille de route qui épouse les intérêts de toute entité consciente de la valeur de ses données », destinée à accompagner « la sécurisation de votre système d'information ». Les 42 mesures forment « le socle minimum à respecter pour protéger les informations de votre organisation », permettant d'« interagir avec ses partenaires et de servir ses clients en respectant l'intégrité et la confidentialité ».

**Réserve** : édition 2017 confirmée par la page, mais une révision plus récente n'est pas exclue ; le contenu détaillé des 42 mesures est dans le PDF téléchargeable, non ouvert dans cette session.

**Adresse ouverte** : https://messervices.cyber.gouv.fr/guides/guide-dhygiene-informatique

---

## 5. ANSSI, *Recommandations relatives à l'authentification multifacteur et aux mots de passe* (8 octobre 2021)

**Cité dans la politique** : section 7, section 9 (règles R-06, R-07), section 10.

**Statut** : Vérifié, source officielle ouverte, le 21/09/2026.

**Texte** (recommandations relevées sur la page) :

> « Mener une analyse de risque lors de la mise en place de moyens d'authentification. »
> « Privilégier l'utilisation de l'authentification multifacteur. »
> « Adapter la robustesse d'un mot de passe à son contexte d'utilisation. »

**Réserve** : la page ouverte ne donne qu'un résumé ; les recommandations détaillées (longueur minimale par contexte, cas précis où le second facteur est obligatoire, traitement spécifique des comptes à privilèges) sont dans le PDF complet, non ouvert dans cette session.

**Adresse ouverte** : https://messervices.cyber.gouv.fr/guides/recommandations-relatives-lauthentification-multifacteur-et-aux-mots-de-passe

---

## 6. CNIL, recommandation sur les mots de passe (délibération n° 2022-100 du 21 juillet 2022)

**Cité dans la politique** : section 7, section 9, section 10 (règle R-08).

**Statut** : Vérifié, source officielle ouverte, le 21/09/2026.

**Texte** :

> « Les recommandations visent le degré de complexité du mot de passe (l'entropie) et non une longueur minimale. »
> « Les mots de passe ne doivent pas être stockés en clair », mais transformés par « une fonction cryptographique non réversible et sûre, intégrant l'utilisation d'un sel ou d'une clé ».

La recommandation abandonne l'obligation de renouvellement périodique des mots de passe pour les comptes utilisateurs classiques. Elle fixe des seuils d'entropie selon le contexte : au moins 80 bits pour un mot de passe utilisé seul, au moins 50 bits si l'accès est par ailleurs restreint (par exemple par un dispositif limitant les tentatives), au moins 13 bits si le mot de passe est combiné à un dispositif matériel avec blocage après 3 essais.

**Publication** : 14 octobre 2022. **Délibération** : n° 2022-100 du 21 juillet 2022.

**Adresse ouverte** : https://www.cnil.fr/fr/mots-de-passe-une-nouvelle-recommandation-pour-maitriser-sa-securite

---

## 7. OWASP, *Code Review Guide* et *DevSecOps Guideline*

**Cité dans la politique** : section 7, section 14 (règles R-22 à R-25).

**Statut** : Vérifié, source officielle ouverte, le 21/09/2026.

**Code Review Guide** : classé « Lab Project » (stade de maturité le plus bas de la classification OWASP), 9 contributeurs, sous licence Creative Commons Attribution-ShareAlike 4.0. La page ne donne pas de texte de présentation au-delà du titre.

**DevSecOps Guideline**, description donnée par la page officielle (en anglais, non traduite pour rester fidèle au texte) :

> « The OWASP DevSecOps Guideline explains how to build and operate a secure software delivery pipeline using a curated, vendor-neutral set of tools. »
> Objectif central : « Detect security issues, whether design flaws or application vulnerabilities, as early and as cheaply as possible, and keep detecting them continuously. »

Classé « Incubator Project » (stade intermédiaire). Les étapes de pipeline qu'il recommande incluent : détection de secrets, analyse de composants logiciels (SCA, dont l'analyse de dépendances), analyse statique (SAST), analyse d'infrastructure comme code, sécurité de la chaîne d'approvisionnement logicielle, analyse dynamique (DAST), et d'autres contrôles plus avancés.

**Réserve** : les deux projets sont explicitement non finalisés (Lab et Incubator, pas Flagship) : leur contenu peut encore changer.

**Adresses ouvertes** : https://owasp.org/www-project-code-review-guide/ et https://owasp.org/www-project-devsecops-guideline/

---

## 8. ISO 27001:2022, Annexe A

**Cité dans la politique** : en tête de presque toutes les sections de règles (8 à 23), et dans le tableau d'alignement de la section 20.5. La source la plus citée du document : 47 numéros de contrôle.

**Statut** : **Non vérifié, et il ne peut pas l'être gratuitement.** Le texte de la norme ISO/IEC 27001:2022 est payant. Je ne peux pas reproduire ici le texte de ses contrôles : ce serait soit l'inventer, soit recopier une source tierce elle-même non officielle, ce que le protocole du projet interdit de présenter comme vérifié. Les intitulés du tableau ci-dessous viennent de ma mémoire de formation, recoupée seulement par des sites tiers (ISMS.online, Advisera) lors de la veille du 21/09 : niveau L4 dans l'échelle du projet, en dessous du L1 qu'exigerait une preuve de conformité.

**C'est la source à vérifier en priorité** si ce dossier doit servir de preuve pour une démarche de certification ou d'accréditation : une seule inexactitude sur ces 47 numéros suffirait à invalider la correspondance affichée par la politique.

**Où vérifier vous-même** : le texte s'achète auprès de l'ISO (`iso.org`, rechercher « ISO/IEC 27001:2022 ») ou de l'AFNOR pour la version française. Une bibliothèque d'école ou d'entreprise y donne parfois accès sans achat individuel.

**Liste complète des contrôles cités, avec l'intitulé utilisé dans la politique (non vérifié) :**

| Contrôle | Intitulé utilisé dans la politique | Statut d'usage |
|---|---|---|
| A.5.1 | Politiques de sécurité de l'information | Base d'une règle |
| A.5.2 | Fonctions et responsabilités | Base d'une règle |
| A.5.3 | Séparation des tâches | Base d'une règle |
| A.5.4 | Responsabilités de la direction | Base d'une règle |
| A.5.6, A.5.7 | Contacts spécialisés, renseignement sur les menaces | Base d'une règle |
| A.5.9 | Inventaire des actifs | Base d'une règle |
| A.5.11 | Restitution des actifs | Base d'une règle |
| A.5.12 | Classification de l'information | Nommé, non traité par une règle |
| A.5.14 | Transfert de l'information | Nommé, traité en partie seulement |
| A.5.15 à A.5.18 | Contrôle d'accès, identités, informations d'authentification, droits d'accès | Base d'une règle |
| A.5.19, A.5.20 | Fournisseurs : relations et accords | Base d'une règle |
| A.5.23 | Cloud (AWS) | Nommé, traité en partie seulement |
| A.5.24 à A.5.28 | Gestion des incidents | Nommé, renvoyé à la section 17 |
| A.5.29, A.5.30 | Sécurité pendant une perturbation, continuité | Nommé, traité en partie seulement |
| A.5.31 | Exigences légales | Base d'une règle |
| A.5.34 | Protection des données personnelles | Base d'une règle |
| A.5.36 | Conformité aux politiques et aux règles | Base d'une règle |
| A.6.3 | Sensibilisation et formation | Base d'une règle |
| A.6.5 | Responsabilités après la fin d'un emploi | Base d'une règle |
| A.6.7 | Travail à distance | Base d'une règle |
| A.7.2, A.7.3 | Accès physiques, sécurisation des locaux | Base d'une règle |
| A.8.1 | Terminaux des utilisateurs | Base d'une règle |
| A.8.2, A.8.5 | Droits privilégiés, authentification sécurisée | Base d'une règle |
| A.8.8 | Gestion des vulnérabilités techniques | Base d'une règle |
| A.8.10 | Suppression de l'information | Nommé, traité en partie seulement |
| A.8.12 | Prévention des fuites de données | Nommé, non traité par une règle |
| A.8.13 | Sauvegarde | Base d'une règle |
| A.8.15, A.8.16 | Journalisation, surveillance | Base d'une règle |
| A.8.20, A.8.22 | Sécurité et cloisonnement des réseaux | Nommé, traité en partie seulement |
| A.8.24 | Cryptographie | Nommé, traité en partie seulement |
| A.8.25, A.8.29, A.8.32 | Développement sécurisé, tests, gestion des changements | Base d'une règle |
| A.8.28 | Codage sécurisé | Nommé, traité en partie seulement |
| A.8.31, A.8.33 | Séparation des environnements, données de test | Base d'une règle |

---

## 9. Directive NIS2 et règlement DORA

**Cité dans la politique** : section 7 uniquement, pour dire qu'ils **ne sont pas retenus** comme base : leur application à VNWeb n'est pas établie. Aucune règle ne s'appuie dessus.

**Statut** : sans objet pour la validation. Pour mémoire, la veille du projet a ouvert la page de la Commission européenne sur DORA (règlement (UE) 2022/2554), le 21/09/2026 : https://finance.ec.europa.eu/regulation-and-supervision/financial-services-legislation/implementing-and-delegated-acts/digital-operational-resilience-regulation_en

---

## 10. Résumé, par niveau de confiance

| Source | Statut |
|---|---|
| Code du travail, article L5213-6 | Vérifié, source officielle ouverte |
| Agefiph, aide à l'adaptation | Vérifié, source officielle ouverte |
| ANSSI, *Guide d'hygiène informatique* | Vérifié, source officielle ouverte ; édition 2017 confirmée, révision plus récente non exclue |
| ANSSI, recommandations authentification multifacteur | Vérifié, source officielle ouverte (résumé seulement, PDF complet non ouvert) |
| CNIL, délibération n° 2022-100 | Vérifié, source officielle ouverte |
| OWASP, *Code Review Guide* et *DevSecOps Guideline* | Vérifié, source officielle ouverte ; projets non finalisés (Lab, Incubator) |
| RGPD, articles 9, 28, 30, 33, 44 | Vérifié via un site tiers (texte repris ci-dessus), pas directement sur EUR-Lex |
| RGPD, articles 15 à 22 (droits des personnes) | Non cités par la politique alors qu'ils fondent la règle R-28 : à ajouter |
| ISO 27001:2022, Annexe A (47 contrôles) | **Non vérifié, et non vérifiable sans achat de la norme** |
| NIS2, DORA | Non utilisés comme base ; page DORA de la Commission ouverte pour mémoire |

**La priorité de vérification, si le temps manque pour tout relire** : l'ISO 27001 d'abord (47 citations, aucune vérifiée), puis recouper les cinq articles du RGPD directement sur EUR-Lex plutôt que sur un site tiers.

---

## 11. Chaque règle, reliée à ses sources

Ce tableau relie chaque règle (R-xx) et chaque action (A-xx) de la politique aux sources qui la fondent. Trois cas de figure :

- **ISO 27001** : repris du tableau d'alignement de la section 20.5 de la politique, qui associe déjà chaque contrôle à des règles précises. Quand la section 20.5 ne mentionne pas une règle pour un contrôle donné, elle n'apparaît pas ici non plus, même si la règle appartient à une section dont l'en-tête cite ce contrôle : c'est signalé par « (hérité de la section) ».
- **ANSSI, CNIL, OWASP, RGPD, Code du travail** : quand un article ou une recommandation est cité dans le texte même de la règle, c'est marqué « (cité dans la règle) », le lien est direct. Sinon, la source vient de l'en-tête de la section qui contient la règle : c'est moins précis, marqué « (hérité de la section) ».
- Une règle sans aucune ligne ci-dessous n'a pas de source normative propre au-delà de l'organisation interne de VNWeb (les règles de gouvernance et de suivi des sections 8, 18, 23, par exemple, s'appuient surtout sur les principes de la politique elle-même, PR-1 à PR-8).

| Règle | Résumé | ANSSI / CNIL / OWASP | RGPD / Code du travail | ISO 27001:2022 |
|---|---|---|---|---|
| R-01 | Référent et suppléant désignés | - | - | A.5.2 |
| R-02 | Compte rendu trimestriel à la direction | - | - | A.5.4 |
| R-03 | Réponse écrite aux propositions, dérogations | - | - | A.5.4 |
| R-04 | Comptes nominatifs | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.15 à A.5.18, A.6.7 |
| R-05 | Droits d'administration nominatifs | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.3, A.5.15 à A.5.18, A.8.2, A.8.5 |
| R-06 | Accès SSH protégé (root, clé, adresses, anti-essais répétés) | ANSSI, *Guide d'hygiène informatique* et recommandations MFA (hérités de la section) | - | A.5.15 à A.5.18 (hérité de la section) |
| R-07 | Double authentification (VPN, AWS, Gitea, Extranet) | ANSSI, recommandations MFA (cité dans la règle : « privilégier l'authentification multifacteur ») | - | A.6.7, A.8.2, A.8.5 |
| R-08 | Secrets dans le coffre-fort, mots de passe uniques | ANSSI et CNIL, mots de passe (cités dans la règle : longueur, unicité, renouvellement sur incident) | - | A.5.15 à A.5.18 |
| R-09 | Vérification par second canal | ANSSI, *Guide d'hygiène informatique* (hérité de la section, lien plus lâche : vigilance générale, pas un point précis du guide) | - | (hérité de la section) |
| R-10 | Télétravail limité et déclaré | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.6.7, A.8.1 |
| R-11 | Bureau à distance seulement par le VPN | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.6.7 |
| R-12 | Protections des postes, chiffrement | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.8.1 |
| R-13 | Inventaire des serveurs et dépendances | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.9, A.8.8 |
| R-14 | Mises à jour, failles critiques, fins de support | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.6, A.5.7, A.8.8 |
| R-15 | Pare-feu et relevé des ports | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | (hérité de la section) |
| R-16 | Séparation des environnements | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.8.31, A.8.33 |
| R-17 | Sauvegardes, tests, plan de reprise | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.8.13 |
| R-18 | Journaux conservés, alerte minimale | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.8.15, A.8.16 |
| R-19 | Fiche et cahier d'exigences pour chaque prestataire | - | RGPD article 28 (hérité de la section, lien plus lâche : la sous-traitance n'est pas encore contractualisée à ce stade) | A.5.19, A.5.20, A.8.13 |
| R-20 | Contrat ou avenant de sécurité | - | RGPD article 28 (cité dans la règle) | (hérité de la section) |
| R-21 | Contact de secours, reprise ailleurs | - | RGPD article 28 (hérité de la section) | A.5.19, A.5.20 |
| R-22 | Relecture de code avant fusion | OWASP, *Code Review Guide* (cité dans la section, correspondance directe avec l'objet de ce guide) | - | A.8.25, A.8.29, A.8.32 |
| R-23 | Liste de contrôle de mise en production | OWASP, *DevSecOps Guideline* (hérité de la section, correspond aux étapes de tests du pipeline) | - | (hérité de la section) |
| R-24 | Journal de déploiement | OWASP, *DevSecOps Guideline* (hérité de la section, correspond à la traçabilité du pipeline) | - | A.8.25, A.8.29, A.8.32 |
| R-25 | Analyse des dépendances | OWASP, *DevSecOps Guideline* (cité dans la section, correspondance directe avec l'étape « SCA » du guide) | - | A.8.8 |
| R-26 | Registre des traitements | - | RGPD article 30 (cité dans la règle) | A.5.31, A.5.34 |
| R-27 | Localisation des données, relevé AWS | - | RGPD chapitre V, article 44 (cité dans la règle) | A.5.23 (partiel, hérité de la section) |
| R-28 | Procédure pour les droits des personnes | - | RGPD articles 15 à 22 (droits des personnes) : non cités par la politique, à ajouter (voir section 1) | - |
| R-29 | Clauses de sécurité dans les contrats clients | - | RGPD, article 28 par analogie (hérité de la section) | A.5.31, A.5.34 |
| R-30 | Arrivée : accès nominatifs, parcours d'accueil | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.9, A.5.15 à A.5.18, A.6.3 |
| R-31 | Départ : retrait des accès, secrets renouvelés | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.11, A.6.5 |
| R-32 | Revue trimestrielle des accès | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.15 à A.5.18 |
| R-33 | Registre des clés des locaux | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.11, A.7.2, A.7.3 |
| R-34 | Visiteurs, serveurs sous clé | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.7.2, A.7.3 |
| R-35 | Tableau de déploiement et de suivi | - | - | A.5.1, A.5.36 |
| R-36 | Contrôles, contrôle croisé, journal d'administration | - | - | A.5.3, A.5.36, A.8.25, A.8.29, A.8.32 |
| R-37 | Veille : sources officielles, condensé mensuel | - | RGPD, en général (hérité de la section, aucun article précis) | A.5.6, A.5.7, A.5.31, A.5.34 |
| R-38 | Lancement et accusé de lecture | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.1, A.5.4, A.6.3 |
| R-39 | Modules de formation et quiz | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | (hérité de la section) |
| R-40 | Séance de la direction | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.4 |
| R-41 | Point sécurité mensuel | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | A.5.6, A.5.7, A.6.3 |
| R-42 | Charte informatique | ANSSI, *Guide d'hygiène informatique* (hérité de la section) | - | (hérité de la section) |
| R-43 | Supports de formation accessibles | - | - | A.6.3 |
| R-44 | Adaptation de la formation sur demande | - | Code du travail article L5213-6, RGPD article 9 (cités dans la règle) | A.6.3 (hérité de la section) |
| R-45 | Revue annuelle de la PSSI | - | - | A.5.1, A.5.36 |
| R-46 | Temps alloué et mesure de la charge | - | - | A.5.4 |
| A-01 | Approuver et signer la PSSI | - | - | A.5.1 |
| A-02 | Installer le coffre-fort, purger Discord | ANSSI et CNIL, mots de passe (hérités de la section) | - | A.5.15 à A.5.18 |
| A-03 | Décision sur les données hors Union européenne | - | RGPD, chapitre V (cité dans le texte de l'action) | A.5.23 (non repris dans le tableau d'alignement) |
| A-04 | Recherche de secrets dans l'historique | OWASP, *DevSecOps Guideline* (rapprochement fait ici, non cité comme tel dans la section) | - | (non repris dans le tableau d'alignement) |
| A-05 | Migration Amplify Gen 1 vers Gen 2 | - | - | A.5.23 (partiel) |
| A-06 | Chaîne de déploiement automatisée (CI/CD) | OWASP, *DevSecOps Guideline* (hérité de la section) | - | A.8.25, A.8.29, A.8.32 |

**Ce que ce tableau montre, et qui vaut la peine d'être corrigé dans la politique elle-même** :

1. **La plupart des liens ANSSI, CNIL et OWASP sont hérités de la section**, pas cités dans le texte de la règle. C'est fidèle à la façon dont la politique est écrite (une ligne « Références » par section, pas par règle), mais ça affaiblit la preuve pour une règle précise : si un jour on doit défendre la règle R-09 devant un tiers, on ne peut pas pointer une phrase exacte du guide ANSSI qui la justifie.
2. **R-28 n'a aucune source RGPD correcte à ce jour** : elle traite des droits des personnes (articles 15 à 22), que la politique ne cite à aucun endroit. C'est la correction la plus nette à apporter.
3. **A-03 et A-04 ne sont pas repris dans le tableau d'alignement ISO de la section 20.5**, alors que A-04 (recherche de secrets) correspond directement à une étape du pipeline DevSecOps de l'OWASP, et A-03 (données hors Union européenne) au même sujet que le contrôle A.5.23 partiellement traité ailleurs.
4. **Le socle ANSSI reste le plus utilisé et le moins précis** : cité pour la quasi-totalité des règles de sept sections (9, 11, 12, 16, 21, et en partie 9-10), sans le numéro de mesure exact du guide de 42 mesures pour aucune d'entre elles à ce stade. Un travail futur pourrait aller chercher, dans le PDF complet, la mesure précise qui correspond à chaque règle.

---

*À jour au 28/09/2026.*
