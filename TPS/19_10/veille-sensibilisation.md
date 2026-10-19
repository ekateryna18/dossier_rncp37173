# Veille et sensibilisation

---

# Analyse de la CVE-2026-21440 : @adonisjs/bodyparser

_Cette section présente l'analyse détaillée de la vulnérabilité critique
identifiée dans la stack du projet : description, versions concernées,
conditions d'exploitation, matrice de risque et remédiation._

---

## Résumé

La vulnérabilité critique identifiée dans ce projet est référencée sous
**CVE-2026-21440** et concerne le paquet **@adonisjs/bodyparser**, tiré comme
dépendance par @adonisjs/core. Il s'agit d'une faille de traversée de
répertoire (path traversal) dans la gestion des fichiers multipart.

En l'état actuel du code, elle n'est pas directement exploitable : aucun
appel vulnérable n'est présent. Elle constitue néanmoins un risque latent,
exploitable dès lors que du code approprié serait ajouté (voir plus bas).

## Référence et version concernée

| Champ               | Valeur                                                            |
| ------------------- | ---------------------------------------------------------------- |
| Identifiant         | **CVE-2026-21440**                                               |
| Avis GitHub         | [GHSA-gvq6-hvvp-h34h](https://github.com/advisories/GHSA-gvq6-hvvp-h34h) |
| Paquet              | @adonisjs/bodyparser (npm)                                     |
| Version installée   | **10.1.0**                                                       |
| Plage vulnérable    | Versions antérieures à 10.1.2 (et préversions 11 antérieures à 11.0.0-next.6) |
| Gravité             | Critique, CVSS v4 : **9.2**                                     |
| Catégorie           | CWE-22 (Path Traversal)                                          |

Version confirmée dans le fichier de verrouillage des dépendances du projet. La
version du projet (10.1.0) se situe au sein de la plage vulnérable.

## Description de la faille

La fonction de déplacement des fichiers téléversés du paquet vulnérable, lorsqu'elle
est appelée **sans nom de fichier explicite**, reprend le nom fourni par le client
sans le nettoyer. Le chemin de destination est construit par simple concaténation
du dossier cible et de ce nom.

Un attaquant distant peut alors envoyer un fichier dont le nom contient des
séquences de remontée de répertoire. Le fichier peut ainsi être écrit
**en dehors du dossier d'upload prévu**, jusqu'à écraser des fichiers sensibles
du serveur.

## Pourquoi la faille n'est pas exploitable en l'état

- Aucune partie du projet n'appelle la fonction de déplacement des fichiers.
- Les points d'upload (publications, administration, authentification) lisent le
  fichier temporaire et le transmettent à un service d'upload externe. Les noms
  de fichiers temporaires générés par le paquet sont hors du contrôle de
  l'attaquant.

## Condition d'exploitation (risque latent)

La faille deviendrait immédiatement exploitable si du code appelait la fonction
de déplacement des fichiers sans fournir de nom nettoyé, par exemple pour
enregistrer une image téléversée dans un dossier de stockage local.

Dans ce cas, un fichier dont le nom remonte l'arborescence serait écrit hors du
répertoire de stockage prévu. Le nom fourni par le client est par ailleurs déjà
transmis en l'état au service d'upload, ce qui déplace le même risque vers ce
composant si celui-ci construit un chemin à partir de ce nom.

## Analyse de risque : méthode EBIOS Risk Manager

L'analyse de risque applique la méthode **EBIOS Risk Manager** (ANSSI) au
périmètre de la CVE-2026-21440. Elle est structurée selon les cinq ateliers de
la méthode.

**Échelles de cotation retenues**

- **Gravité** : G1 Mineure · G2 Significative · G3 Grave · G4 Critique
- **Vraisemblance** : V1 Minime · V2 Significative · V3 Forte · V4 Quasi certaine
- **Niveau de risque** = Gravité × Vraisemblance (Faible · Moyen · Élevé · Critique)

### Atelier 1 : Cadrage et socle de sécurité

- **Objet de l'étude** : l'API applicative reposant sur le framework AdonisJS,
  et sa dépendance @adonisjs/bodyparser (v10.1.0) affectée par la
  CVE-2026-21440.
- **Valeurs métier** : disponibilité et intégrité du service, confidentialité et
  intégrité des données utilisateurs (comptes, contenus, données personnelles).
- **Biens supports** : serveur applicatif Node.js, système de fichiers,
  répertoires d'upload, service d'upload externe, base de données.
- **Socle de sécurité** : validation d'entrée (VineJS), authentification et contrôle d'accès,
  analyse SAST (Semgrep) et audit de dépendances déjà présents ; écart constaté
  = dépendance vulnérable non corrigée.

### Atelier 2 : Sources de risque et objectifs visés

| Source de risque (SR)              | Objectif visé (OV)                                         | Pertinence |
| ---------------------------------- | ---------------------------------------------------------- | ---------- |
| Attaquant externe opportuniste     | Altérer le service ou prendre pied sur le serveur          | Élevée     |
| Attaquant externe ciblé            | Accéder aux données ou compromettre durablement le système | Moyenne    |
| Erreur interne (développeur)       | Introduction involontaire du code rendant la faille active | Élevée     |

### Atelier 3 : Scénarios stratégiques

Chemins d'attaque de haut niveau reliant une source de risque à une valeur
métier via l'écosystème :

- **SS1** : un attaquant externe exploite la faille de traitement des fichiers
  envoyés pour atteindre l'intégrité du système de fichiers du serveur.
- **SS2** : la faille est répercutée vers le **service d'upload externe**
  (partie prenante de l'écosystème), le nom de fichier client étant transmis en
  l'état.
- **SS3** : une erreur interne (ajout de code déplaçant un fichier sans
  nettoyer son nom) ouvre le chemin d'attaque SS1.

### Atelier 4 : Scénarios opérationnels

Déclinaison technique des scénarios stratégiques, avec cotation. Chaque scénario
est évalué selon deux axes, dont la combinaison donne le niveau de risque :

- **Gravité** : *à quel point les conséquences seraient sérieuses si le scénario
  se réalisait*. Elle mesure l'impact sur les valeurs métier (service, données),
  indépendamment de la probabilité que cela arrive.
- **Vraisemblance** : *quelle est la probabilité que le scénario se réalise
  réellement*. Elle mesure la crédibilité de l'attaque compte tenu des conditions
  nécessaires et de la capacité de l'attaquant.

**Niveau de risque = Gravité × Vraisemblance.** Un scénario très grave mais peu
vraisemblable peut rester un risque faible (cas de SO1 ci-dessous).

**Échelle de gravité**

| Cote | Signification                                                              |
| ---- | ------------------------------------------------------------------------- |
| G1   | Mineure : sans conséquence notable sur le service ou les données          |
| G2   | Significative : dégradation limitée du service                            |
| G3   | Grave : forte dégradation du service ou atteinte aux données              |
| G4   | Critique : compromission du système ou fuite massive de données           |

**Échelle de vraisemblance**

| Cote | Signification                                                             |
| ---- | ------------------------------------------------------------------------- |
| V1   | Minime : conditions d'exploitation quasi absentes, très peu crédible      |
| V2   | Significative : conditions partiellement réunies, plausible               |
| V3   | Forte : conditions largement réunies, probable                            |
| V4   | Quasi certaine : exploitation triviale, conditions totalement réunies     |

| Réf. | Scénario opérationnel                                                                 | Condition de réalisation                                             | Gravité | Vraisemblance | Risque      |
| ---- | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------- | ------- | ------------- | ----------- |
| SO1  | Exploitation directe via l'application en l'état                                      | Requiert un déplacement de fichier sans nom nettoyé, **absent du code**| G4      | V1            | **Faible**  |
| SO2  | Code vulnérable introduit (déplacement de fichier sans nom nettoyé), puis exploitation               | Erreur interne + envoi d'un nom de fichier malveillant               | G4      | V2            | **Élevé**   |
| SO3  | Traversée répercutée sur le service d'upload externe                                  | Nom client transmis tel quel ; chemin construit à partir de ce nom   | G3      | V2            | **Élevé**   |
| SO4  | Altération / écrasement de fichiers applicatifs ou de configuration                   | Consécutif à SO2 ou SO3                                              | G4      | V2            | **Moyen**   |
| SO5  | Indisponibilité du service par corruption de fichiers critiques                       | Consécutif à SO4                                                    | G3      | V2            | **Moyen**   |

### Atelier 5 : Traitement du risque

| Réf. | Décision        | Mesure de traitement                                                            | Risque résiduel |
| ---- | --------------- | ------------------------------------------------------------------------------- | --------------- |
| SO1  | Réduire         | Mise à jour de @adonisjs/bodyparser vers la version 10.1.2 ou supérieure       | Négligeable     |
| SO2  | Réduire / éviter| Mise à jour + règle de revue de code : nom de fichier généré côté serveur | Faible          |
| SO3  | Réduire         | Nettoyer le nom fourni par le client avant transmission au service d'upload| Faible          |
| SO4  | Réduire         | Mise à jour + droits d'écriture restreints sur le système de fichiers           | Faible          |
| SO5  | Réduire         | Sauvegardes et surveillance de l'intégrité des fichiers critiques               | Faible          |

**Synthèse du traitement** : la mise à jour de la dépendance est la mesure
prioritaire ; elle réduit l'ensemble des scénarios à un risque résiduel faible à
négligeable. Les mesures de revue de code et d'assainissement couvrent le risque
**latent** (introduction future de code vulnérable), qui constitue le point de
vigilance principal puisqu'une seule ligne mal écrite suffirait à activer une
faille critique (CVSS 9.2).

> Note : cette évaluation reflète l'analyse du code présent et peut évoluer
> selon la configuration réelle de déploiement.

## Remédiation recommandée

### 1. Correction immédiate de la CVE

1. Mettre à jour @adonisjs/bodyparser au-delà de la version 10.1.2 (mise à jour
   non cassante).
2. Pour tout futur déplacement de fichier téléversé, fournir un nom généré côté
   serveur, ou ne conserver que la partie finale du nom fourni par le client.
3. Nettoyer le nom de fichier fourni par le client avant de le transmettre au
   service d'upload externe.

### 2. Intégration d'un audit des dépendances dans la chaîne CI/CD

Ajouter une étape d'audit des dépendances au pipeline, exécutée à chaque envoi de
code et chaque demande de fusion (pull request), afin de détecter automatiquement
toute dépendance vulnérable connue.

- L'étape fait échouer le pipeline sur toute vulnérabilité de gravité haute ou
  critique, tout en tolérant le bruit de bas niveau.
- Elle vient compléter l'analyse SAST (Semgrep) déjà en place, qui couvre le code
  du projet mais pas les vulnérabilités connues des dépendances.
- Le rapport d'audit peut être conservé comme artefact de build pour
  historisation.

### 3. Veille automatique de la stack

Mettre en place une surveillance continue des dépendances afin d'être alerté
dès la publication d'une nouvelle CVE touchant la stack du projet :

- **Dependabot** (ou équivalent côté GitLab/Gitea) : active les alertes de
  sécurité et ouvre automatiquement des *pull requests* de mise à jour dès
  qu'un correctif est disponible pour une dépendance vulnérable.
- **Audit planifié** : une tâche périodique (par exemple chaque semaine dans la
  CI) exécutant l'audit des dépendances pour détecter les nouvelles vulnérabilités même en
  l'absence de commit récent.
- **Suivi des avis amont** : surveillance des avis de sécurité AdonisJS
  (dépôt GitHub, base d'avis GHSA) pour anticiper les correctifs.

Ces trois niveaux (correction, détection en CI, veille continue) garantissent
que la stack reste à jour et que toute nouvelle vulnérabilité est traitée au
plus tôt, sans dépendre d'un contrôle manuel ponctuel.

---

# Sources de veille

La veille s'appuie sur plusieurs sources complémentaires. Aucune ne couvre tout
à elle seule : le CERT traite les menaces à large diffusion et l'écosystème,
tandis que les bases spécialisées couvrent précisément les dépendances npm du
projet.

| Source                    | Rôle                                                              | Accès / Flux                                      |
| ------------------------- | ----------------------------------------------------------------- | ------------------------------------------------- |
| **CERT-FR** (ANSSI)       | Alertes et avis officiels sur les menaces et produits à large diffusion | cert.ssi.gouv.fr, flux RSS des avis et des alertes |
| **GitHub Advisory (GHSA)**| Base utilisée par l'audit npm ; avis précis sur les paquets npm   | github.com/advisories                           |
| **NVD** (NIST)            | Base CVE officielle, références et scores CVSS                    | nvd.nist.gov                                    |
| **OSV.dev** (Google)      | Base open-source orientée écosystèmes (npm inclus)               | osv.dev                                         |
| **Avis amont AdonisJS**   | Correctifs et annonces de sécurité du framework                   | Dépôt GitHub AdonisJS                             |

> Le CERT-FR ne publie pas nécessairement d'avis pour un paquet npm précis
> (par exemple @adonisjs/bodyparser) : il vient **en complément** de GHSA / NVD / OSV,
> qui couvrent exhaustivement les dépendances du projet.

---

# Procédure de traitement d'une vulnérabilité

Toute vulnérabilité (CVE) identifiée sur le projet, qu'elle provienne de la
veille automatique, d'un audit CI ou d'un signalement externe, suit un cycle en
quatre étapes : **Sources → Vérification → Décision → Suivi**.

## 1. Sources

Identifier l'origine de l'information et récupérer la fiche de vulnérabilité :
éditeur, CERT-FR, et bases d'avis (GHSA / NVD / OSV). Voir la section
*Sources de veille* ci-dessus.

## 2. Vérification

Confirmer que la vulnérabilité concerne réellement le projet :

- **Produit et version** : le paquet et la version installée sont-ils dans la
  plage affectée ?
- **Exposition** : le composant vulnérable est-il réellement atteignable
  (route, authentification, surface exposée) ?
- **Conditions** : quelles conditions sont nécessaires à l'exploitation ?

## 3. Décision et traitement

Pour chaque vulnérabilité confirmée, statuer sur :

- **Priorité** : déduite de la gravité (CVSS) et de la matrice de risque.
- **Mesure** : correction (mise à jour de la dépendance, automatique ou manuelle, puis
  vérification de la construction et des tests), mitigation temporaire si aucun correctif
  n'est disponible (désactivation de la fonctionnalité exposée, filtrage en
  amont), ou acceptation documentée du risque.
- **Responsable** : le **responsable de projet** est garant du traitement (il
  peut déléguer la correction, mais reste redevable de sa réalisation).
- **Échéance** : délai cible selon la gravité.

### Grille d'échéances

| Gravité            | Échéance de traitement cible        |
| ------------------ | ----------------------------------- |
| Critique / Haute   | **48 h**                            |
| Moyenne            | 7 jours                             |
| Faible             | Prochaine mise à jour planifiée     |

> Justification d'une échéance courte : les correctifs de dépendances sont en
> général rapides à appliquer, et toute attente laisse
> s'accumuler du code s'appuyant sur une dépendance obsolète ou vulnérable.

### Cas particulier : vérifier une éventuelle fuite de données

Lorsque la vulnérabilité a pu être **exploitée avant sa correction**, il faut
déterminer si des données personnelles ont été exposées. Pistes concrètes :

- **Journaux d'accès** (serveur web, serveur intermédiaire) : requêtes anormales, pour
  la CVE-2026-21440, des noms de fichiers contenant des séquences de remontée de
  répertoire, volumes inhabituels, codes de réponse inattendus.
- **Journaux applicatifs** : erreurs d'upload, écritures hors des répertoires
  attendus, accès à des routes sensibles.
- **Système de fichiers** : fichiers inattendus dans ou hors des répertoires
  d'upload, dates de modification suspectes.
- **Base de données** : accès ou modifications anormaux, requêtes volumineuses,
  connexions hors plages horaires habituelles.
- **Corrélation temporelle** : croiser la date de divulgation de la CVE avec les
  journaux pour cibler la fenêtre d'exposition.

> Remarque : la fiabilité de cette vérification dépend de la journalisation en
> place. Conserver des journaux horodatés, centralisés et protégés contre
> l'altération pour pouvoir investiguer a posteriori.

### En cas de fuite de données personnelles : notification CNIL (72 h)

Si la vérification conclut (ou laisse un doute sérieux) à une **violation de
données à caractère personnel**, le RGPD s'applique :

- **Notification à la CNIL sous 72 heures** après en avoir pris connaissance
  (article 33 du RGPD), sauf si la violation est peu susceptible d'engendrer un
  risque pour les personnes concernées. Passé ce délai, la notification doit
  être justifiée.
- **Information des personnes concernées** (article 34) lorsque la violation est
  susceptible d'engendrer un **risque élevé** pour leurs droits et libertés.
- **Documenter** la violation dans un registre interne (nature, effets, mesures
  prises), que la CNIL soit notifiée ou non.

La notification s'effectue via le téléservice dédié de la CNIL
(notification en ligne).

## 4. Suivi

Conserver la **preuve du traitement** et planifier la **prochaine revue**. Le
tableau suivant est renseigné pour chaque vulnérabilité traitée :

| Champ                | Description                                               |
| -------------------- | --------------------------------------------------------- |
| Vulnérabilité        | Identifiant (CVE / GHSA) et paquet concerné               |
| Responsable          | Responsable de projet (garant)                            |
| Décision             | Mesure retenue (correction / mitigation / acceptation)    |
| Échéance             | Délai cible fixé                                          |
| Preuve de traitement | Référence de la modification, date, audit sans faille critique, tests réussis |
| Fuite de données     | Investiguée ? Violation avérée ? CNIL notifiée ?          |
| Statut               | À traiter / En cours / Traité                             |
| Prochaine revue      | Date de la prochaine vérification                         |

### Exemple : CVE-2026-21440

| Champ                | Valeur                                                    |
| -------------------- | --------------------------------------------------------- |
| Vulnérabilité        | CVE-2026-21440 : @adonisjs/bodyparser                   |
| Responsable          | Responsable de projet                                     |
| Décision             | Correction (mise à jour vers la version 10.1.2 ou supérieure)|
| Échéance             | 48 h (gravité critique)                                   |
| Preuve de traitement | _à compléter : référence de la modification + résultat de l'audit_|
| Fuite de données     | _à compléter : aucune exploitation possible en l'état_    |
| Statut               | _à compléter_                                             |
| Prochaine revue      | _à compléter_                                             |

---

# Conformité aux normes ISO/IEC 27000

Les pratiques décrites dans ce document s'inscrivent dans la famille de normes
**ISO/IEC 27000** (sécurité de l'information) : **27001** (système de management,
certifiable), **27002** (mesures de sécurité) et **27005** (gestion du risque).
L'essentiel est déjà couvert sans être nommé ; le tableau ci-dessous relie
chaque pratique au contrôle correspondant et signale ce qui reste à formaliser.

## Correspondance pratiques du projet → contrôles ISO/IEC 27002:2022

| Pratique en place / décrite ici                          | Contrôle ISO/IEC 27002:2022                              | État     |
| -------------------------------------------------------- | ------------------------------------------------------- | -------- |
| Analyse EBIOS (ateliers 1→5), matrice de risque          | 5.7 Renseignement sur les menaces · (ISO 27005)         | Oui |
| Audit des dépendances en CI + veille (Dependabot, CERT…)     | 8.8 Gestion des vulnérabilités techniques               | Oui |
| Procédure de traitement CVE (décision, échéance, suivi)  | 8.8 · 5.26 Réponse aux incidents                        | Oui |
| Validation d'entrée (VineJS), SAST (Semgrep)             | 8.25-8.28 Développement sécurisé                        | Oui |
| Investigation fuite via journaux                         | 8.15 Journalisation · 8.16 Surveillance                 |  Partiel |
| Notification CNIL / registre des violations              | 5.34 Protection des données · 6.8 Signalement           | Oui |
| Responsable de projet garant du traitement               | 5.2 Rôles et responsabilités                            |  Partiel |

## Points à formaliser pour viser la conformité

- **Politique de sécurité écrite** (contrôle 5.1) : formaliser ce document et
  les règles associées dans une politique validée et diffusée à l'équipe.
- **Registre des risques** (ISO 27005) : tenir un registre central des
  vulnérabilités et de leur traitement, au-delà du cas de cette CVE.
- **Journalisation et surveillance** (8.15 / 8.16) : mettre en place une
  journalisation horodatée, centralisée et protégée, aujourd'hui principale
  limite pour investiguer une fuite.
- **Revue périodique** (amélioration continue, cœur d'ISO 27001) : planifier une
  revue régulière des risques et des mesures, et non au seul fil des incidents.

> Note : la certification ISO/IEC 27001 relève d'une démarche organisationnelle
> et d'un audit externe ; ce document contribue au volet technique, sans s'y
> substituer.
