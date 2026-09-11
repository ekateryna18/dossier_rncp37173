# Plan d'audit, conformité RGPD

Document de travail, pas encore une section finale du dossier RNCP. Comme
`audit_serveur_infrastructure.md` comble l'angle mort machine, celui-ci comble
l'angle mort juridique : le périmètre d'audit actuel (section 6 du dossier)
regarde la sécurité technique, pas la conformité au Règlement Général sur la
Protection des Données.

**Pourquoi ce document a du sens ici en particulier** : l'application traite
des données d'enfants (les élèves) et de leurs responsables (les
superviseurs). Le RGPD traite les données de mineurs comme une catégorie qui
mérite une attention renforcée (article 8), pas comme un cas général. Ce n'est
pas une option secondaire du dossier, c'est un axe attendu dès qu'un projet
manipule ce type de public.

**Une différence importante avec l'audit serveur** : là-bas, presque tout se
vérifie avec une commande en lecture seule. Ici, une bonne partie des réponses
ne sont pas dans le code — ce sont des décisions déjà prises (ou pas encore
prises) par Vincent ou le client : durée de conservation choisie, existence
d'un délégué à la protection des données, politique de confidentialité déjà
rédigée ailleurs. Ce document sépare donc clairement, catégorie par catégorie,
ce que tu peux vérifier seule dans le code, et ce qui doit remonter comme
question à eux avant d'écrire quoi que ce soit dans le dossier — même logique
que ta remarque sur F-01 : ne pas trancher à ta place une décision qui ne
t'appartient pas.

---

## Sources officielles de référence, à ouvrir soi-même

Chaque affirmation de ce document marquée **[CLAIM]** doit pouvoir se vérifier
à la source, pas seulement se lire dans ce fichier. Trois niveaux de sources
distincts, du plus brut au plus pratique :

1. **Le texte du règlement lui-même** — `eur-lex.europa.eu`, chercher
   "Regulation (EU) 2016/679". C'est le Journal officiel de l'Union
   européenne, la source primaire pour le libellé exact de chaque article
   (32, 35, 28, 30, 8...). Niveau de preuve L1.
2. **La doctrine de la CNIL** — `cnil.fr`, section professionnels. Traduit le
   règlement en pratique concrète pour la France : liste officielle des
   traitements soumis à AIPD (sert la catégorie 10 de ce document), modèle de
   registre des traitements téléchargeable (catégorie 9), guides thématiques.
   Niveau de preuve L1-L2 selon le document précis.
3. **La loi française qui complète le règlement** — `legifrance.gouv.fr`,
   loi n° 78-17 du 6 janvier 1978 modifiée (dite "Informatique et Libertés").
   Le RGPD fixe par défaut l'âge du consentement numérique à 16 ans (article
   8) mais laisse chaque État membre l'abaisser jusqu'à 13 ans : la France a
   choisi 15 ans via cette loi nationale, pas via le règlement européen
   directement. À vérifier ici, précisément, avant de citer ce chiffre dans
   le dossier final.

Règle pratique : si une affirmation de ce document ne peut pas être reliée à
l'une de ces trois sources ouvertes personnellement, elle reste au niveau
[HYPOTHESIS] ou [CLAIM L4-L5] dans le dossier final, pas plus haut.

---

## Méthodologie, ordre de dépendance

1. Cartographie des données collectées
2. Base légale et minorité (article 8)
3. Sécurité des données (article 32) — passerelle vers l'audit déjà fait
4. Durée de conservation et droit à l'effacement
5. Droits des personnes et point de contact
6. Sous-traitants et hébergement (article 28)
7. Transferts hors Union européenne
8. Cookies et traceurs
9. Documentation et gouvernance (registre, politique de confidentialité)
10. Analyse d'impact (AIPD, article 35)

L'ordre suit la même logique que les deux audits précédents : savoir quelles
données existent (1) conditionne tout le reste — on ne peut pas juger une
durée de conservation ou un risque de transfert sans savoir précisément ce qui
est collecté.

---

## 1. Cartographie des données personnelles collectées

**Ce que j'ai déjà vérifié dans le code** (`api-dance/app/models/user.ts`) :
le modèle `User` porte `email`, `password` (haché via `scrypt`, pas en clair),
`firstName`, `lastName`, `phoneNumber`, `address`, `city`, `postalCode`,
`avatarUrl`, `birthDate`, `gender`. La date de naissance est un champ à part
entière — important pour la catégorie 2, elle permet de déterminer l'âge
exact d'un utilisateur.

**Ce qu'il te reste à vérifier** : ce modèle est-il le seul endroit où des
données personnelles transitent, ou y en a-t-il ailleurs (messages,
commentaires, médias) qui pourraient contenir des données sensibles de
manière incidente (une photo d'enfant dans `post.ts` ou `media.ts`, par
exemple) ?

```bash
grep -rn "column()" api-dance/app/models/media.ts api-dance/app/models/post.ts
```

**Finding attendu** : une liste précise des champs collectés, catégorie par
catégorie (identité, contact, données de mineur). Pas un jugement à ce stade —
juste l'inventaire, qui sert de base à toutes les catégories suivantes.

---

## 2. Base légale et minorité

**Pourquoi ici** : le RGPD encadre spécifiquement le cas d'un mineur (article
8) — en France, le seuil de consentement autonome pour les services en ligne
est fixé à 15 ans ; en dessous, le consentement d'un titulaire de l'autorité
parentale est requis.

**Ce que j'ai déjà vérifié dans le code** (`api-dance/app/controllers/auth_controller.ts`,
méthode `addChild`) : c'est le superviseur (le compte adulte) qui crée le
compte de l'enfant via `POST /add-child` — l'enfant ne s'auto-inscrit pas.
C'est cohérent avec l'esprit de l'article 8 : le consentement passe par
l'adulte responsable, pas par l'enfant lui-même. **[CLAIM L1]** pour le
principe de l'article 8 lui-même (texte réglementaire) ; le fait que ce
mécanisme suffise à couvrir l'obligation reste une question à valider avec
Vincent ou un juriste, pas une conclusion que je peux poser seul depuis la
lecture du code.

**Ce qu'il te reste à vérifier** :

```bash
grep -rn "signup\|register\|inscription" api-dance/app/routes/*.ts
```

Existe-t-il un chemin d'inscription autonome (l'enfant créant son propre
compte sans passer par un superviseur) ? Si oui, c'est un point à signaler,
la logique de protection actuelle ne s'appliquerait pas à ce chemin-là.

**À faire remonter à Vincent/client, pas à trancher seule** : le texte exact
présenté au superviseur au moment de la création du compte constitue-t-il un
recueil de consentement explicite (case à cocher, texte informant de l'usage
des données), ou juste un formulaire technique sans mention légale ? C'est une
question de contenu produit, pas de code.

---

## 3. Sécurité des données (article 32)

**Pourquoi ici** : l'article 32 du RGPD exige des "mesures techniques et
organisationnelles appropriées" — c'est le pont direct avec l'audit déjà fait
dans le dossier (section 6, findings F-01 à F-13) et l'audit serveur en cours.

**Ce qu'il n'y a pas à refaire** : ce document ne duplique pas l'audit
sécurité. Il se contente de le référencer : les findings déjà identifiés
(F-01 sur le contrôle d'accès, ceux à venir sur le serveur — pare-feu absent,
IPv6 exposé) sont, du point de vue RGPD, des manquements potentiels à
l'article 32, pas seulement des problèmes de sécurité générique. C'est un
angle de lecture supplémentaire à ajouter dans la colonne "Catégorie" de ces
findings quand tu les rédigeras, pas une nouvelle catégorie de recherche.

**Finding attendu** : aucune nouvelle recherche ici, juste une note de
recoupement dans le dossier final : "F-01, F-14 relèvent aussi de l'article
32 RGPD."

---

## 4. Durée de conservation et droit à l'effacement

**Pourquoi ici** : une fois les données identifiées (catégorie 1), la question
suivante est : pendant combien de temps sont-elles gardées, et que se
passe-t-il quand un élève quitte l'école ?

**Ce que tu peux vérifier dans le code** :

```bash
grep -rn "delete\|deleteChild\|archiv" api-dance/app/controllers/auth_controller.ts
```

Tu as déjà vu `deleteChild` lors de l'investigation F-01 — vérifie s'il
supprime réellement les données (suppression en base) ou seulement l'accès
(désactivation de compte, `enabled: false`). Les deux ont un sens RGPD
différent : suppression réelle vs simple archivage.

**À faire remonter à Vincent/client, pas à trancher seule** : existe-t-il une
durée de conservation définie après le départ d'un élève (par exemple, 3 ans
sans activité) ? Le RGPD n'impose pas un chiffre précis, il impose que la
durée soit définie, justifiée, et appliquée — c'est une décision de gestion,
pas une valeur technique que le code peut deviner seul.

---

## 5. Droits des personnes et point de contact

**Pourquoi ici** : le RGPD donne aux personnes concernées (ici, les
superviseurs au nom des enfants) le droit d'accéder à leurs données, de les
rectifier, de les faire effacer, de s'opposer à un traitement.

**Ce que tu peux vérifier** :
- Existe-t-il, dans l'application (`app-dance`), un écran de profil permettant
  à un superviseur de voir et modifier les données de l'enfant lui-même,
  sans passer par une demande manuelle ?
- Existe-t-il une adresse ou un formulaire de contact pour une demande que
  l'interface ne couvre pas (effacement complet, portabilité) ?

```bash
grep -rn "contact\|dpo@\|privacy@" app-dance/src cdn-app-dance
```

**Finding attendu** : si aucun point de contact dédié n'existe et que rien
dans l'interface ne permet à un utilisateur d'agir seul sur ses données, c'est
un manquement à documenter — un simple email de contact peut suffire comme
première étape de correction, sans nécessiter de modification du code.

---

## 6. Sous-traitants et hébergement (article 28)

**Pourquoi ici** : une fois les droits des personnes posés, la question
suivante est qui d'autre, en dehors de l'entreprise elle-même, a accès aux
données — hébergeur, service d'envoi d'email, etc.

**Ce que j'ai déjà vérifié** : le fichier `.env` de `api-dance` référence une
configuration SMTP pour l'envoi d'emails (confirmations, notifications). Un
prestataire SMTP externe est un sous-traitant au sens RGPD dès qu'il manipule
des données personnelles (au minimum l'adresse email du destinataire).

**Ce qu'il te reste à vérifier** : quel est le prestataire SMTP utilisé (nom
de l'hébergeur mail) ? Est-il basé dans l'Union européenne ? Existe-t-il un
contrat ou des conditions générales qui couvrent le traitement de données
personnelles (DPA, Data Processing Agreement) avec lui ?

**Résolu — prestataire SMTP identifié : Mailjet.** Vérification faite par
recherche web le 04/09/2026, sur la documentation officielle du fournisseur
(`mailjet.com/legal/privacy-policy`, `mailjet.com/products/data-security-and-privacy`)
**[CLAIM L2]** : Mailjet appartient aujourd'hui au groupe Sinch (société
suédoise, donc Union européenne), pas à une structure française indépendante.
L'origine française de la marque n'est donc pas, à elle seule, ce qui
garantit la conformité — c'est un point à corriger dans le raisonnement même
si la conclusion pratique reste bonne. Les données transitent par des centres
de données situés à Francfort (Allemagne) et Saint-Ghislain (Belgique),
tous deux dans l'Union européenne, et Mailjet déclare un délégué à la
protection des données et une certification ISO 27001. Sur ce point précis
(localisation du traitement), rien n'indique un manquement.

**Ce qu'il reste quand même à faire remonter à Vincent/client** : l'existence
effective d'un contrat de sous-traitance (DPA) signé avec Mailjet pour ce
projet précis — la conformité du fournisseur en général ne dispense pas
d'avoir le document contractuel qui l'engage vis-à-vis de l'école. Et plus
largement, la liste complète des autres prestataires externes (hébergement
serveur inclus) reste à vérifier, le code seul ne peut pas la révéler
entièrement.

---

## 7. Transferts hors Union européenne

**Pourquoi ici** : une fois les sous-traitants identifiés (catégorie 6), la
question suivante est où sont hébergées les données qu'ils manipulent.

**Ce que j'ai déjà vérifié dans le code** : `app-dance/package.json` déclare
`firebase` comme dépendance, mais une recherche dans `app-dance/src` ne
trouve aucun usage réel de ce paquet dans le code source — seulement dans
`package.json` et `package-lock.json`. **[CLAIM L1]**, vérifiable
directement en relançant `grep -rn "firebase" app-dance/src`. Ça suggère que
Firebase (société américaine, donc un transfert hors UE potentiel si utilisé)
n'est probablement pas actif dans le code actuel — mais "probablement" n'est
pas "confirmé" : une dépendance déclarée sans usage visible est aussi, en soi,
un point d'hygiène à corriger (la retirer si elle est inutile réduit la
confusion et la surface d'attaque).

Les notifications push, elles, passent par `api-dance/app/services/web_push_service.ts`
avec des clés VAPID — un protocole standard, indépendant d'un fournisseur
précis. **[HYPOTHESIS]** : même sans SDK Firebase, le protocole Web Push
route techniquement les notifications via le service du navigateur de
l'utilisateur final (le service de Google pour Chrome, celui d'Apple pour
Safari) — c'est une caractéristique du protocole Web Push en général, pas un
choix du projet. C'est un point à documenter dans le dossier plutôt qu'un
constat à trancher ici : creuse ce point si tu veux le citer précisément
(cherche la documentation officielle du protocole Web Push côté navigateur).

**À faire remonter à Vincent/client** : confirmer si `firebase` est
réellement inutilisé (et donc à retirer des dépendances), et où est hébergé
le serveur de base de données et de fichiers physiquement (Union européenne
ou non) — ça, le code ne peut pas te le dire, c'est un choix d'hébergeur.

---

## 8. Cookies et traceurs

**Pourquoi ici** : distinct du reste, régi par la directive ePrivacy en plus
du RGPD — les cookies strictement nécessaires au fonctionnement (session,
authentification) sont exemptés de consentement ; les cookies de mesure
d'audience ou de publicité ne le sont pas.

**Ce que j'ai déjà vérifié** : `app-dance/src/services/cookie_service.ts`
pose des cookies avec `SameSite=Lax` et `Secure` en production — l'usage
visible dans ce fichier est un cookie technique (probablement de session),
pas un traceur. Aucune bannière de consentement n'a été trouvée dans le
périmètre déjà parcouru, mais cette recherche n'a pas été exhaustive.

**Ce qu'il te reste à vérifier** :

```bash
grep -rln "CookieService.create" app-dance/src
grep -rn "analytics\|gtag\|google-analytics\|matomo\|hotjar" app-dance/src
```

**Finding attendu** : si tous les usages de `CookieService.create` sont
strictement fonctionnels (session, préférences d'affichage) et qu'aucun outil
de mesure d'audience n'apparaît, aucune bannière de consentement n'est requise
— à documenter comme un point conforme, pas seulement comme une absence de
recherche.

---

## 9. Documentation et gouvernance

**Pourquoi ici** : une fois la réalité technique cartographiée (catégories
1 à 8), la question devient : est-ce que cette réalité est écrite quelque
part, de façon consultable ?

**Ce qu'il te reste à vérifier** :

```bash
grep -rln "politique de confidentialit\|mentions.l[ée]gales\|privacy-policy" app-dance/src -i
```

**À faire remonter à Vincent/client, entièrement organisationnel** :
- Un registre des traitements (article 30) existe-t-il, même sous forme
  simple (tableur) ? C'est une obligation pour la plupart des structures,
  quelle que soit leur taille, dès qu'elles traitent des données de mineurs
  de façon régulière.
- Une politique de confidentialité est-elle publiée quelque part (site
  vitrine, mention dans l'app) ?
- Un délégué à la protection des données (DPO) est-il désigné, ou l'entreprise
  a-t-elle évalué qu'elle n'en a pas l'obligation ?

Ce sont des questions à poser telles quelles, pas des lacunes à combler par
toi dans le code.

---

## 10. Analyse d'impact relative à la protection des données (AIPD, article 35)

**Pourquoi en dernier** : cette analyse ne se fait qu'une fois tout le reste
posé (catégories 1 à 9) — elle synthétise le risque global du traitement.

**Ce que dit la doctrine de la CNIL (autorité française) [CLAIM L1]** : une
AIPD est obligatoire quand un traitement remplit au moins deux critères parmi
une liste (dont : données de personnes vulnérables, comme les mineurs ; suivi
systématique ; grande échelle). Le traitement de données d'enfants coche déjà
un critère à lui seul — ça ne rend pas l'AIPD automatiquement obligatoire,
mais ça la rend une question sérieuse à trancher, pas à ignorer par défaut.

**À faire remonter à Vincent/client** : la décision de mener une AIPD formelle
(document structuré : description du traitement, nécessité et
proportionnalité, risques, mesures) revient à l'entreprise, éventuellement
avec un accompagnement juridique. Ton rôle ici s'arrête à signaler que la
question se pose légitimement au vu de la catégorie 1 (données de mineurs).

---

## Format de restitution

Même logique que l'audit serveur — chaque écart réel devient un finding à la
suite des précédents, avec une colonne "Catégorie" qui référence l'article du
RGPD concerné plutôt qu'une catégorie OWASP :

| ID | Criticité | Article RGPD | Constat | Recommandation | Statut |
|---|---|---|---|---|---|

---

## Ce que ce document ne couvre pas

- Toute décision de fond (durée de conservation, désignation d'un DPO,
  contenu exact d'une politique de confidentialité, nécessité d'une AIPD
  formelle) : ce sont des choix de gouvernance qui reviennent à Vincent et au
  client, pas des constats techniques à déduire seule du code.
- Une revue juridique complète du texte d'un futur formulaire de consentement
  ou d'une politique de confidentialité : relève d'un avis juridique, pas
  d'un audit technique.
