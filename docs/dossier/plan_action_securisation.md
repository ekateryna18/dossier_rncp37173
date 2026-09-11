# Plan d'action de sécurisation, Centre Art & Danse

Document de pilotage unique. Il séquence tout ce qui reste à faire sur le projet
Centre Art & Danse, à la fois le chantier technique de sécurisation et la
production des livrables documentaires manquants du dossier RNCP37173. Il ne
redécouvre rien : il ordonne des constats déjà établis dans le dossier
(`dossier_rncp_centre_art_et_danse.md`, sections 6 à 13).

Ce plan est une frise unique, pas deux plans parallèles. Chaque livrable
documentaire est rattaché à la phase technique qui le nourrit réellement : on
rédige le plan de reprise d'activité quand on a mis en place la sauvegarde, pas
avant ; on rédige la politique de sécurité quand le pipeline qui l'incarne
existe, pas avant.

---

## 1. Cadre et contraintes du plan

Rappel des contraintes qui bornent l'ordre des étapes (dossier, sections 2.4,
4.3, 4.4) :

- Application **déjà en production**, données réelles de familles, majorité de
  mineurs. Démarche corrective, pas préventive.
- **Pas de réécriture** de la stack existante (AdonisJS/MySQL pour `api-dance`,
  React/Ionic pour `app-dance`, Express/Multer pour `cdn-app-dance`).
- **Pas d'interruption de service** ; toute intervention à risque est validée en
  amont par Vincent Nilles (BNF-02).
- **Zéro régression fonctionnelle** alors qu'il n'existe **aucun test
  automatisé** (P3, F-07). Conséquence structurante : toute correction touchant
  un parcours sensible doit être **vérifiée manuellement sur ce parcours** avant
  mise en production (BNF-03), tant que la suite de tests n'existe pas.
- Équipe de **trois personnes à temps partiel** : un développeur frontend, un
  développeur backend, l'alternante en charge de la sécurité. Rythme alternance
  (4 jours entreprise, 1 jour école). Le plan n'est pas dimensionné pour un
  chantier plein temps.
- Échéance dossier : **juin 2027**.

Acteurs (abréviations utilisées dans la colonne « Qui ») :

- **SEC** : alternante sécurité (Ecaterina) — pilote de la mission, rédaction du
  dossier.
- **BE** : développeur backend VNWeb.
- **FE** : développeur frontend VNWeb.
- **VN** : Vincent Nilles — validation des orientations, priorités et
  interventions à risque.

---

## 2. Le tout premier geste, et pourquoi lui

**Premier geste concret, avant tout le reste : confirmer auprès de VNWeb l'état
réel de la sauvegarde de la base de données MySQL, et, si elle n'existe pas ou
n'est pas confirmée, la mettre en place avec un test de restauration (AM-03,
section 7).**

Ce plan doit trancher entre deux candidats au premier rang :

- **AM-03** (absence de sauvegarde base de données), marqué IPR 160, CRITIQUE,
  « à traiter avant toute autre priorité de ce plan » (synthèse AMDEC, 7).
- **F-01 / F-10** (SO-2 / SO-5), marqués **P0 bloquants** par l'EBIOS RM
  (Atelier 5, 8) et placés en tête de la priorisation de l'audit (6.4).

La décision est **AM-03 en premier**, sans ambiguïté. Justification :

1. **C'est le filet qui rend tout le reste réversible.** Les corrections
   suivantes (F-01, F-10, F-04...) modifient du code sur une application en
   production, sans aucun test automatisé pour rattraper une régression (F-07).
   Une manœuvre malheureuse pendant un de ces correctifs, sur une base sans
   sauvegarde confirmée, c'est une **perte de données irréversible** de familles
   réelles, mineurs compris. On ne bricole pas du code sensible en production
   sans point de restauration. La sauvegarde est le prérequis logique qui
   autorise à toucher au reste.

2. **F-01 et F-10 sont « bloquants avant mise en production », mais le service
   est déjà en production.** La barrière qu'ils désignent a déjà été franchie il
   y a des mois. Quelques jours de plus d'exposition, le temps de poser le filet,
   ne changent pas l'ordre de grandeur du risque déjà couru ; corriger sans filet,
   si, l'aggrave.

3. **Coût et risque quasi nuls, aucune régression possible.** Mettre en place une
   sauvegarde ne touche pas au code applicatif : zéro risque de casser un
   parcours. Le geste peut donc démarrer immédiatement et en parallèle du cadrage,
   sans bloquer personne.

4. **C'est le risque le moins détectable du dossier** (AM-03, D=8, G=10) : une
   absence de sauvegarde ne se voit pas tant qu'aucun incident ne survient. La
   traiter tôt, c'est fermer l'angle mort avant qu'il ne coûte tout.

Nuance opérationnelle : le tout premier acte n'est pas « installer un outil »
mais **lever l'incertitude marquée entre crochets dans le dossier** (« aucune
stratégie de sauvegarde automatisée régulière n'a été confirmée à ce jour [à
vérifier auprès de VNWeb] », 7, AM-03). Si une sauvegarde existe déjà, on la
teste par une restauration réelle et on documente ; sinon, on la met en place.
Dans les deux cas, la phase 0 ne se clôt qu'avec **une restauration testée**, pas
sur une simple déclaration.

Les corrections P0 F-01 et F-10 ne sont pas repoussées loin : elles ouvrent la
phase 1, juste après le filet.

---

## 3. La frise, vue d'ensemble

Le plan compte **huit phases** (une phase 0 de mise en sécurité et de cadrage,
puis sept phases de travail). Les livrables documentaires du dossier sont
répartis dans les phases qui les nourrissent, pas regroupés à la fin.

| Phase | Intitulé | Cœur technique | Livrables dossier rattachés |
|---|---|---|---|
| 0 | Filet de survie et cadrage factuel | Sauvegarde BDD (AM-03), Gitleaks, exclusions | Section 9 « Sans coût », amorce section 11, données manquantes |
| 1 | Correctifs P0 sur les mineurs | F-01, F-10 | Mise à jour statuts 6.3 |
| 2 | Durcissement du service de fichiers exposé | Docker cdn, F-06, F-02 | Arbitrage 5.3.9, section 9 « Faible coût » |
| 3 | Pipeline DevSecOps et confinement | F-07 (Semgrep, linter, CodeQL, Renovate), Docker api + app | Section 9 « Investissement structurant », section 10 |
| 4 | Corrections de moyenne priorité et durabilité | F-11, F-03, F-04, amorce tests | Section 9 « Synthèse », suite de tests |
| 5 | Arbitrages produit et réduction de surface | F-09, F-05, F-08, F-12 | Mise à jour 6.3, section 12 (matière) |
| 6 | Environnement de test et analyse dynamique | Préprod, OWASP ZAP, test manuel messagerie | Complément section 9, section 11 finale |
| 7 | Finalisation documentaire du dossier | (aucun code) | Sections 10, 11, 12, 13, page de titre |

---

## 4. Détail des phases

Chaque étape précise : (a) ce qui est fait, (b) pourquoi à ce moment et pas
avant/après, (c) qui est concerné, (d) la section du dossier concernée.

### Phase 0 — Filet de survie et cadrage factuel

Objectif : rendre le reste du chantier réversible et lever les incertitudes qui
bloquent la rédaction. Aucune de ces actions ne modifie un parcours applicatif,
donc aucune ne peut provoquer de régression : la phase peut démarrer sans
validation de risque lourde.

**0.1 — Confirmer / mettre en place la sauvegarde automatisée de la base, avec
test de restauration (le premier geste)**
- (a) Vérifier auprès de VNWeb si une sauvegarde MySQL régulière existe. Sinon,
  mettre en place une sauvegarde automatisée périodique. Dans tous les cas,
  prouver la restauration par un essai réel et consigner la procédure.
- (b) Prérequis absolu : rend réversible tout correctif de code des phases
  suivantes, sur une base sans tests automatisés. C'est le point le moins
  détectable et le plus grave du dossier.
- (c) SEC pilote, BE pour l'accès serveur/BDD, VN valide la stratégie et
  l'emplacement des sauvegardes.
- (d) Section 7, AM-03 ; lève la donnée manquante « [à vérifier auprès de VNWeb] ».

**0.2 — Poser Gitleaks en porte avant commit (détection de secrets)**
- (a) Installer Gitleaks en pré-commit sur les trois dépôts ; scan de
  l'historique complet existant.
- (b) Sans coût, non intrusif, aucun impact sur le service en production. Doit
  exister **avant** de manipuler des fichiers de configuration dans les phases
  suivantes, pour éviter d'introduire un secret dans l'historique pendant qu'on
  travaille. Un secret entré dans l'historique y reste lisible même après
  suppression (9, « Pourquoi cet ordre n'est pas interchangeable »).
- (c) SEC met en place, BE et FE l'adoptent sur leur poste.
- (d) Section 9 (pipeline, phase « avant le commit »), F-13.

**0.3 — Corriger l'exclusion de suivi de version du service de fichiers**
- (a) Ajouter au fichier d'exclusion de `cdn-app-dance` les fichiers de
  configuration contenant des secrets, à l'image des deux autres composants.
- (b) Sans coût, à faire avant tout futur ajout de secret dans ce composant (le
  correctif d'authentification F-06, phase 2, en ajoutera un). Aucune fuite
  actuelle constatée, donc pas d'urgence critique, mais le geste doit précéder
  F-06.
- (c) SEC ou BE.
- (d) Section 6.3, F-13.

**0.4 — Collecter les données factuelles manquantes du dossier**
- (a) Recueillir et intégrer : noms des développeurs frontend et backend (2.2),
  nom de l'école cliente (2.2), année en page de titre, et **définir avec VNWeb
  les cibles chiffrées** de non-fonctionnel (couverture de tests, disponibilité,
  temps de réponse) laissées en suspens (2.3, 4.2, BNF-07).
- (b) Ces valeurs sont des prérequis de rédaction : elles apparaissent dans
  plusieurs sections et ne doivent pas être inventées. Les fixer tôt évite de
  reprendre chaque section à la fin. Les cibles chiffrées orientent aussi le
  chantier de tests (phase 4).
- (c) SEC recueille, VN valide les cibles chiffrées et le périmètre client.
- (d) Sections 2.2, 2.3, 4.2, page de titre.

**0.5 — Rédiger la sous-section 9 « Sans coût, à engager immédiatement »**
- (a) Formaliser dans le dossier les actions 0.1 à 0.3 comme le lot « sans coût »
  du plan de migration.
- (b) Ces actions viennent d'être décidées et engagées : les rédiger maintenant,
  tant que le raisonnement est frais, plutôt que de reconstituer plus tard.
- (c) SEC.
- (d) Section 9, sous-section « Sans coût » (actuellement vide).

**0.6 — Amorcer la section 11 (PRA) avec le volet sauvegarde/restauration**
- (a) Rédiger le dispositif de sauvegarde et la procédure de restauration testée
  en 0.1 comme premier bloc du plan de reprise d'activité.
- (b) Le PRA ne peut se rédiger dans le vide : son cœur est précisément le
  dispositif de sauvegarde qu'on vient de mettre en place. On capture la matière
  réelle au moment où elle est produite. Le reste de la section 11 sera complété
  en phase 6, quand l'environnement de test et la supervision existeront.
- (c) SEC.
- (d) Section 11.2, 11.3, 11.4 (articulation avec l'AMDEC / AM-03).

### Phase 1 — Correctifs P0 sur les données des mineurs

Objectif : traiter les deux risques marqués **P0 bloquants** par l'EBIOS
(Atelier 5) et en tête de la priorisation d'audit (6.4). On les fait juste après
le filet, avant le pipeline : ce sont des correctifs ciblés, urgents, qui ne
peuvent attendre la mise en place d'une chaîne d'automatisation.

**1.1 — F-01 : contrôle d'accès sur l'invitation de superviseur**
- (a) Ajouter la vérification que l'appelant est **déjà superviseur de l'enfant
  ciblé** avant d'envoyer une invitation (aujourd'hui, seul le rôle général est
  vérifié). Vérification manuelle du parcours d'invitation avant mise en
  production.
- (b) Correctif le plus direct sur la protection des mineurs (SO-2, probabilité
  élevée, aucune barrière technique actuelle). Positionné #1 en 6.4. On le fait
  en premier parmi les correctifs car il ne dépend d'aucun autre et ferme un
  accès non autorisé aux données d'un mineur.
- (c) BE implémente, SEC spécifie et vérifie le parcours, VN valide l'intervention.
- (d) Sections 6.3 (F-01), 8 (SO-2), backlog US-07.

**1.2 — F-10 : confirmer puis fermer l'injection de commande sur le service de
fichiers**
- (a) D'abord **confirmer le risque par un test dédié** (nom de fichier construit
  pour atteindre le traitement de conversion/compression), puis filtrer
  l'extension par **liste blanche** avant tout traitement exécuté sur le serveur.
  Vérification manuelle de l'upload légitime après correctif.
- (b) Scénario le plus grave du dossier (SO-5 : exécution de commande arbitraire
  sur le serveur), P0 bloquant, #2 en 6.4. Le test de confirmation précède le
  correctif parce que le dossier le demande explicitement (« confirmer par un
  test dédié, puis filtrer »), et parce qu'il documente la réalité du risque pour
  le bilan.
- (c) BE implémente, SEC conçoit le test de confirmation et valide la liste
  blanche, VN informé.
- (d) Sections 6.3 (F-10), 8 (SO-5), backlog US-05.

À l'issue de la phase 1, mettre à jour les statuts des findings dans le tableau
6.3 (F-01, F-10 passent de « À faire » à « Corrigé, vérifié le [date] »).

### Phase 2 — Durcissement du service de fichiers exposé

Objectif : `cdn-app-dance` est le composant le plus exposé (aucune
authentification F-06, injection F-10 tout juste traitée). On le confine et on
l'authentifie. F-02 (canal de messagerie) est traité ici car c'est le dernier des
findings « Haute » restants et il relève de la même logique de configuration
réseau.

**2.1 — Dockeriser `cdn-app-dance`**
- (a) Conteneuriser le service de fichiers (un conteneur dédié, système de
  fichiers et réseau isolés).
- (b) Placé avant le pipeline complet et juste après F-10 : le confinement limite
  ce qu'une compromission de ce service précis peut atteindre sur le reste de la
  plateforme, **sans attendre** que tous ses correctifs soient déployés (5.3.10).
  C'est aussi le prérequis naturel du pipeline CI/CD (phase 3) : une image se
  déploie à l'identique du poste de dev à la production. On commence par ce
  composant car c'est le plus exposé.
- (c) BE, VN valide l'intervention sur l'infrastructure de production.
- (d) Section 5.3.10, référence F-06 et F-10.

**2.2 — F-06 : authentification minimale sur le service de fichiers**
- (a) Mettre en place un jeton partagé entre services internes sur les fonctions
  d'envoi et de nettoyage de médias ; restreindre la taille de fichier acceptée
  (aujourd'hui plusieurs Go). Vérification manuelle des uploads légitimes.
- (b) #3 en 6.4. Vient après la dockerisation (le conteneur limite déjà les
  dégâts) et après 0.3 (l'exclusion de secrets est corrigée avant d'introduire le
  secret partagé du jeton). Ferme l'accès anonyme au composant le plus exposé.
- (c) BE implémente côté `cdn` et côté `api` (appelant), SEC vérifie, VN validé.
- (d) Sections 6.3 (F-06), 5.2.5, 5.3.9.

**2.3 — F-02 : restreindre l'origine du canal de messagerie**
- (a) Aligner la liste des origines autorisées du canal temps réel de messagerie
  sur celle, déjà restreinte, de l'API HTTP. Vérification manuelle que la
  messagerie fonctionne depuis les domaines légitimes.
- (b) #4 en 6.4 (SO-4, P1). Ce n'est pas un arbitrage technique mais la correction
  d'une incohérence par rapport à un standard déjà appliqué ailleurs dans le même
  projet (5.3.7). Regroupé ici avec les autres correctifs de configuration
  d'exposition.
- (c) BE implémente, FE vérifie côté interface, SEC valide.
- (d) Sections 6.3 (F-02), 8 (SO-4), 5.3.7, backlog US-08.

**2.4 — Rédiger la sous-section 9 « Faible coût, valeur élevée »**
- (a) Formaliser F-06, F-02, F-11 (à venir) et les mesures de surveillance AMDEC
  comme le lot « faible coût » du plan.
- (b) Rédigé maintenant que les premiers correctifs de ce lot sont en cours, avec
  le recul de leur mise en œuvre réelle.
- (c) SEC.
- (d) Section 9, sous-section « Faible coût » (vide).

### Phase 3 — Pipeline DevSecOps et confinement complet

Objectif : F-07, « condition de fiabilité de tous les correctifs suivants »
(6.4, #6). Le pipeline vient **après** les P0 (qu'on ne pouvait pas attendre)
mais **avant** les corrections restantes, qu'il fiabilisera et dont il préviendra
la régression. Gitleaks est déjà posé (phase 0.2) ; on complète la chaîne dans
l'ordre imposé par le dossier.

**3.1 — Dockeriser `api-dance` et `app-dance`**
- (a) Conteneuriser les deux composants restants.
- (b) Complète le confinement entamé en 2.1 et fournit au pipeline des images
  déployables à l'identique à chaque étape. Fait avant de câbler les étapes de
  déploiement du pipeline, qui s'appuient sur ces images.
- (c) BE (api), FE (app), VN valide.
- (d) Section 5.3.10.

**3.2 — Semgrep + analyseur de style à chaque envoi de code**
- (a) Câbler l'analyseur de style avec règles de sécurité, puis Semgrep, en
  porte bloquante sur sévérité haute à chaque envoi vers le dépôt partagé.
- (b) Deuxième étage du pipeline (après Gitleaks). Semgrep est rapide : il élimine
  les motifs évidents en quelques secondes, ce qui réserve CodeQL, plus lent, aux
  demandes de fusion (9). Placé avant CodeQL pour cette raison de coût.
- (c) SEC configure les règles, BE et FE adaptent leur flux de travail.
- (d) Section 9 (pipeline), 6.2.1, F-07.

**3.3 — CodeQL + Renovate à la demande de fusion**
- (a) Câbler CodeQL (analyse de flux de données, bloquant sur sévérité haute) et
  Renovate (audit de dépendances, proposition de mise à jour, bloquant sur
  sévérité haute/critique) à l'ouverture et la mise à jour d'une demande de
  fusion. Activer Renovate en veille continue une fois en place.
- (b) Troisième étage. CodeQL trace les flux complexes, coûteux, donc réservé au
  filtre plus tardif de la fusion. Renovate est retenu car le dépôt est
  autohébergé (Gitea) : Dependabot n'y est techniquement pas utilisable (9).
- (c) SEC configure, BE et FE traitent les propositions de mise à jour.
- (d) Section 9 (pipeline + benchmarks Renovate), 6.2.1, F-07, backlog US-01.

**3.4 — Rédiger la sous-section 9 « Investissement structurant » et la section 10
(politique de sécurité)**
- (a) Documenter le pipeline et la dockerisation comme investissement structurant
  (9). Rédiger la politique de sécurité (10) : objectifs, périmètre, règles de
  conception et de codage, gouvernance, responsabilités, processus de validation.
- (b) La politique de sécurité **décrit les règles que le pipeline vient
  d'incarner** (analyse statique bloquante, détection de secrets, audit de
  dépendances, revue de contrôle d'accès). La rédiger maintenant, c'est décrire un
  dispositif réel et non une intention. Le « processus de validation » (10.6) est
  exactement le pipeline construit en 3.1-3.3.
- (c) SEC rédige, VN valide la gouvernance et les responsabilités.
- (d) Section 9 « Investissement structurant », section 10 (6 sous-sections vides).

### Phase 4 — Corrections de moyenne priorité et durabilité

Objectif : traiter les findings restants de priorité moyenne, désormais
fiabilisés par le pipeline, et lancer le chantier de fond (tests, chiffrement).

**4.1 — F-11 : limitation de débit sur la réinitialisation de mot de passe**
- (a) Étendre à la route de réinitialisation la même limitation de débit que
  celle déjà en place sur la connexion. Vérification manuelle du parcours de
  réinitialisation.
- (b) #8 en 6.4 (SO-3, P1). Faible coût, réutilise un mécanisme existant. Fait
  après le pipeline pour bénéficier de la vérification automatique, et parce qu'il
  est moins urgent que les findings « Haute » des phases 1-2.
- (c) BE implémente, SEC vérifie.
- (d) Sections 6.3 (F-11), 8 (SO-3), 5.2.1.

**4.2 — F-03 : paramètres de hachage des mots de passe**
- (a) Mesurer la capacité réelle du serveur de production, puis revoir les
  paramètres de scrypt, ou migrer vers Argon2id (premier choix OWASP).
- (b) #9 en 6.4. Explicitement conditionné à une **mesure préalable de la capacité
  du serveur** : on ne peut pas fixer un coût de calcul sans connaître la machine.
  Placé après les correctifs plus urgents. Une migration d'algorithme se fait de
  façon transparente (re-hachage à la prochaine connexion), à cadrer avec VN.
- (c) SEC mène la mesure et l'arbitrage, BE implémente, VN validé.
- (d) Sections 6.3 (F-03), 5.3.5, backlog (implicite US sécurité).

**4.3 — F-04 : chiffrement au niveau champ des données de mineurs**
- (a) Évaluer puis appliquer un chiffrement au niveau champ sur les colonnes les
  plus sensibles (date de naissance, adresse...), avec la clé applicative déjà
  disponible. Migration de données existantes.
- (b) #5 en 6.4, mais volontairement traité ici et non plus tôt : c'est un
  **investissement structurant** qui touche le modèle de données et casse la
  recherche/tri natifs sur les colonnes chiffrées (5.3.6). Il exige donc le filet
  de sauvegarde (phase 0, migration de données à risque), le pipeline (phase 3,
  pour fiabiliser un changement large), et idéalement l'amorce de tests (4.4). Le
  faire avant aurait été le geste le plus risqué du plan sur une base sans filet.
- (c) BE implémente la migration, SEC conçoit et vérifie, VN valide l'intervention
  à risque sur la donnée de production.
- (d) Sections 6.3 (F-04), 5.3.6, BNF-01, BNF-05.

**4.4 — Amorcer la suite de tests automatisés**
- (a) Démarrer une suite de tests, en priorité sur les parcours sensibles déjà
  corrigés (invitation superviseur, upload de fichier, authentification).
- (b) Chantier à part entière (P3, F-07), pas un prérequis bloquant des correctifs
  urgents — c'est pourquoi il vient ici et non au début. Il capitalise sur les
  parcours déjà éprouvés manuellement en phases 1-2 : on transforme la
  vérification manuelle en test automatisé pérenne, ce qui prépare la couverture
  de tests fixée comme cible en 0.4.
- (c) BE et FE écrivent les tests, SEC priorise les parcours sécurité.
- (d) Sections 4.4 (P3), 6.3 (F-07), objectifs 2.3.

**4.5 — Rédiger la sous-section 9 « Synthèse »**
- (a) Consolider les quatre lots du plan de migration (sans coût, faible coût,
  investissement structurant) en une synthèse.
- (b) Rédigeable une fois les trois lots précédents documentés (phases 0, 2, 3).
- (c) SEC.
- (d) Section 9, sous-section « Synthèse » (vide).

### Phase 5 — Arbitrages produit et réduction de surface

Objectif : traiter les findings qui demandent une décision produit ou qui
relèvent du nettoyage/réduction de surface, moins urgents que les correctifs de
sécurité directs.

**5.1 — F-09 : trancher l'auto-assignation aux groupes**
- (a) Décider avec l'équipe produit si l'auto-assignation libre à un groupe est un
  choix fonctionnel à documenter ou un défaut à corriger ; appliquer la décision.
- (b) #10 en 6.4, statut « À trancher » dans le dossier. Ce n'est pas une faille
  tranchée mais un arbitrage métier : il requiert une décision de VN, pas un
  correctif immédiat. Placé après les corrections de sécurité avérées.
- (c) VN tranche, SEC documente, BE implémente si correction retenue.
- (d) Sections 6.3 (F-09), 8, backlog US-04.

**5.2 — F-05 et F-08 : nettoyage et source de vérité unique**
- (a) Retirer le contrôleur de démonstration mort issu de l'outil temps réel non
  retenu (F-05) ; choisir une source de vérité unique pour le statut
  administrateur et migrer l'autre champ (F-08).
- (b) #11 en 6.4. Réduction de surface d'attaque et de dette, à faible risque
  immédiat (le contrôle d'accès admin ne s'appuie que sur un des deux champs). Le
  pipeline (phase 3) sécurise ces changements de code contre une régression. F-08
  demande une migration : elle profite du filet de sauvegarde.
- (c) BE implémente, SEC vérifie.
- (d) Sections 6.3 (F-05, F-08), 5.2.6, backlog US-06, US-09.

**5.3 — F-12 : statuer sur la protection CSRF**
- (a) Confirmer d'abord l'usage réel du cookie HTTP configuré, puis statuer sur le
  besoin d'une protection CSRF explicite.
- (b) #12 en 6.4, statut « À vérifier ». L'authentification par jeton réduit déjà
  la surface CSRF classique ; la décision dépend d'une vérification factuelle
  préalable (l'usage du cookie), d'où le placement tardif. Le correctif éventuel
  est mineur.
- (c) SEC vérifie, BE implémente si nécessaire.
- (d) Sections 6.3 (F-12), 5.3.8.

### Phase 6 — Environnement de test et analyse dynamique

Objectif : lever la dépendance qui bloque l'analyse dynamique (DAST) depuis le
début (« un DAST contre rien ne produit rien de valide », 6.2), et compléter le
pipeline par son dernier étage.

**6.1 — Mettre en place un environnement de préproduction**
- (a) Déployer un environnement de test/préproduction représentatif, à partir des
  images Docker des phases 2-3.
- (b) Prérequis technique de l'analyse dynamique et du dernier étage du pipeline
  (OWASP ZAP après déploiement préprod). Impossible plus tôt sans les images
  conteneurisées. C'est aussi le point qui débloque la validation avant promotion
  en production.
- (c) BE et FE, VN valide.
- (d) Sections 6.1.2 (couche 7, analyse dynamique reportée), 9 (étape ZAP).

**6.2 — OWASP ZAP et test manuel ciblé de la messagerie**
- (a) Lancer le premier scan dynamique (ZAP) contre le backend en préproduction,
  intégré au pipeline après chaque déploiement préprod. Compléter par un **test
  manuel ciblé** du canal de messagerie temps réel, que ZAP ne couvre pas
  nativement.
- (b) #13 en 6.4, dernier de la priorisation car conditionné à un environnement de
  test réel. Le test manuel compense la limite connue de ZAP (F-02).
- (c) SEC mène le scan et le test manuel, BE traite les constats.
- (d) Sections 6.2.2, 6.3 (F-02), 6.4 (#13), 9 (étape ZAP).

**6.3 — Compléter la section 11 (PRA) et le dispositif de surveillance**
- (a) Finaliser le plan de reprise d'activité : services critiques et objectifs de
  reprise, scénarios de sinistre et procédures, dispositif de surveillance et de
  déclenchement, articulation complète avec l'AMDEC. Câbler les mesures de
  surveillance AMDEC « à surveiller » (AM-04 à AM-07 : traitement médias, email de
  validation, notifications push, messagerie temps réel).
- (b) La section 11 ne peut être complète qu'une fois l'environnement de test et
  la supervision en place. Son amorce (0.6) portait la sauvegarde ; sa fin porte
  la reprise et la surveillance, désormais outillées.
- (c) SEC rédige, BE met en place la supervision, VN valide.
- (d) Sections 11.1 à 11.5, 7 (synthèse AMDEC, mesures « à surveiller »).

### Phase 7 — Finalisation documentaire du dossier

Objectif : produire les livrables réflexifs restants, qui ne peuvent se rédiger
qu'une fois le chantier technique largement mené (ils prennent appui sur du
travail réel).

**7.1 — Section 12 : bilan critique**
- (a) Rédiger les trois décisions à challenger : une décision technique
  (ex. scrypt vs Argon2id, F-03), une décision de conformité ou d'architecture
  (ex. chiffrement de champ tardif, F-04, ou service interne vs cloud managé,
  5.3.9), une décision de service tiers.
- (b) Un bilan critique honnête suppose d'avoir vécu les décisions et leurs
  effets : il se rédige après coup, pas au milieu du chantier. La matière est
  accumulée tout au long des phases 1 à 6 (notamment les arbitrages 4.2, 4.3,
  5.1).
- (c) SEC.
- (d) Section 12 (3 sous-sections vides), s'appuie sur 5.3.5, 5.3.6, 5.3.9.

**7.2 — Section 13 : bilan de l'année et compétences**
- (a) Rédiger compétences techniques acquises (ancrées sur le travail réel mené
  sur les trois composants), compétences organisationnelles et transverses, axes
  de progression. Retirer la sous-section « projet personnel » si aucun n'existe,
  plutôt que la laisser vide.
- (b) Bilan de fin de parcours : dernier livrable, il synthétise tout ce qui
  précède.
- (c) SEC.
- (d) Section 13 (4 sous-sections).

**7.3 — Revue finale de cohérence et page de titre**
- (a) Mettre à jour tous les statuts de findings (6.3), vérifier la cohérence des
  renvois entre sections, compléter l'année en page de titre, s'assurer qu'aucune
  donnée entre crochets ne subsiste.
- (b) Ultime passe avant remise, échéance juin 2027.
- (c) SEC, VN relit.
- (d) Ensemble du dossier.

---

## 5. Traçabilité, findings et AMDEC vers phases

| Élément | Priorité source | Phase du plan |
|---|---|---|
| AM-03 sauvegarde BDD | AMDEC critique, IPR 160 | 0.1 |
| F-13 exclusion secrets cdn | 6.4 #7 (partie) | 0.3 |
| Gitleaks pré-commit | 9, pipeline | 0.2 |
| F-01 invitation superviseur | 6.4 #1, EBIOS P0 (SO-2) | 1.1 |
| F-10 injection fichier | 6.4 #2, EBIOS P0 (SO-5) | 1.2 |
| F-06 auth service fichiers | 6.4 #3 | 2.2 |
| F-02 origine messagerie | 6.4 #4, EBIOS P1 (SO-4) | 2.3 |
| Docker cdn (puis api, app) | 5.3.10 | 2.1, 3.1 |
| F-07 pipeline (Semgrep, CodeQL, Renovate) | 6.4 #6 | 3.2, 3.3 |
| F-11 rate limit reset | 6.4 #8, EBIOS P1 (SO-3) | 4.1 |
| F-03 hachage | 6.4 #9 | 4.2 |
| F-04 chiffrement champ | 6.4 #5 | 4.3 |
| F-09 auto-assignation | 6.4 #10 | 5.1 |
| F-05 code mort, F-08 double champ | 6.4 #11 | 5.2 |
| F-12 CSRF | 6.4 #12 | 5.3 |
| DAST ZAP + test manuel messagerie | 6.4 #13 | 6.2 |
| AM-04 à AM-07 surveillance | AMDEC à surveiller | 6.3 |

Note sur l'écart apparent avec l'ordre 6.4 : le dossier 6.4 ordonne les
**findings de sécurité intentionnelle**. AM-03 vient de l'AMDEC (défaillances
accidentelles), qui n'apparaît pas dans la liste 6.4 mais porte une consigne
explicite « avant toute autre priorité de ce plan ». Ce plan résout la tension en
plaçant AM-03 en phase 0 (filet non intrusif) sans repousser F-01/F-10, qui
restent les premiers **correctifs de code** (phase 1). F-04 est le seul finding
volontairement déplacé plus tard que son rang 6.4 (#5 → phase 4), pour une raison
de dépendance technique explicite : son impact sur le modèle de données en fait
un investissement structurant qui exige au préalable la sauvegarde, le pipeline et
l'amorce de tests.

---

## 6. Livrables documentaires, où ils se rédigent

| Section du dossier | État actuel | Phase de rédaction | Pourquoi là |
|---|---|---|---|
| 9, « Sans coût » | vide | 0.5 | Décrit les actions sans coût qu'on vient d'engager |
| 9, « Faible coût » | vide | 2.4 | Décrit le lot en cours de mise en œuvre |
| 9, « Investissement structurant » | vide | 3.4 | Décrit le pipeline et la dockerisation réalisés |
| 9, « Synthèse » | vide | 4.5 | Consolide les trois lots documentés |
| 10, politique de sécurité (6 sous-sections) | vides | 3.4 | Décrit les règles que le pipeline incarne |
| 11, PRA (5 sous-sections) | vides | 0.6 (amorce) + 6.3 (fin) | Sauvegarde d'abord, reprise/surveillance ensuite |
| 12, bilan critique (3 sous-sections) | vides | 7.1 | Suppose d'avoir vécu les décisions |
| 13, bilan de l'année (4 sous-sections) | vides | 7.2 | Bilan de fin de parcours |
| Données entre crochets (2.2, 2.3, 4.2, titre) | manquantes | 0.4 | Prérequis de rédaction transverse |

---

## 7. Principe de vérification, rappelé à chaque correctif

Tant que la suite de tests automatisés (4.4) n'existe pas, **chaque correction
touchant un parcours sensible est vérifiée manuellement sur ce parcours avant
mise en production** (BNF-03, P3). Cela concerne en particulier : invitation de
superviseur (1.1), upload de fichier (1.2, 2.2), authentification (2.2),
réinitialisation de mot de passe (4.1), migration de chiffrement (4.3), migration
du champ administrateur (5.2). Toute intervention à risque sur la production est
validée par Vincent Nilles au préalable (BNF-02).
