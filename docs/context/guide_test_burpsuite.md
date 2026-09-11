# Guide de test manuel avec Burp Suite : Centre Art & Danse

Document autonome, complémentaire au plan d'audit (`docs/context/plan_audit.md`)
et à la section 6 du dossier RNCP37173. Il couvre uniquement la partie test
manuel de l'audit — les constats qui exigent un jugement humain (propriété
d'une ressource, logique métier) et que ni CodeQL/Semgrep (SAST) ni OWASP ZAP
(DAST) ne peuvent juger seuls.

**Règle avant de commencer** : le périmètre autorisé pour tous les tests
ci-dessous est un environnement de test ou une instance locale. La production,
qui héberge les données réelles des mineurs, reste hors périmètre pour cette
série de tests — le test du constat 3.11 (injection) en particulier
deviendrait une intrusion non autorisée sur des données sensibles réelles s'il
était rejoué en dehors de ce périmètre.

Date de rédaction : 2026-08-21.

---

## 1. Installation et mise en place

### 1.1 Installer Burp Suite Community Edition

Téléchargement officiel : `portswigger.net/burp/communitydownload`. Édition
gratuite, suffisante pour tous les tests manuels de ce guide (le Scanner
automatisé et l'Intruder illimité sont réservés à l'édition payante Pro, non
nécessaires ici puisque le scan automatisé en continu est déjà couvert par
OWASP ZAP dans le pipeline).

### 1.2 Installer le certificat Burp (nécessaire pour intercepter le HTTPS)

1. Lancer Burp, aller dans l'onglet `Proxy` > `Intercept` (ou `Proxy Settings`
   selon la version), section `Import / export CA certificate`.
2. Exporter le certificat `Certificate in DER format`.
3. L'importer dans le magasin de certificats de confiance du navigateur ou du
   système utilisé pour tester (Chrome/Firefox : paramètres de sécurité >
   certificats > autorités > importer).
4. Sans cette étape, Burp voit bien passer le trafic HTTPS mais le navigateur
   affiche une alerte de certificat invalide à chaque requête.

### 1.3 Configurer le proxy du client testé

- **Navigateur** : extension FoxyProxy (ou équivalent) pointée sur
  `127.0.0.1:8080` (port par défaut de Burp), activée uniquement pendant les
  sessions de test.
- **`app-dance` (React/Ionic/PWA)** : le plus simple est de lancer la version
  web (`npm run dev` ou équivalent) dans le navigateur proxé, plutôt que de
  configurer le proxy sur un appareil mobile ou un émulateur — le
  comportement des routes API est identique, seul le rendu change.
- **`cdn-app-dance`** : peut être testé soit via le navigateur proxé pendant
  qu'`app-dance` l'appelle, soit en ciblant directement son URL depuis Burp
  Repeater (utile pour le constat 3.6, voir 2.3).

### 1.4 Préparer les comptes de test

Prévoir au minimum :

- **Compte A** (parent/superviseur), avec un enfant `X` qui lui est déjà
  rattaché.
- **Compte B** (parent/superviseur ou élève), avec un enfant `Y` qui n'est
  pas rattaché au compte A.
- **Compte C** (élève ou professeur), membre d'un groupe/cours auquel le
  compte A ne devrait pas avoir accès.

Ces comptes croisés sont nécessaires pour prouver les défauts de contrôle de
propriété (3.1, 3.10) : un scanner ne peut pas savoir qui devrait légitimement
accéder à quoi, seul un humain avec deux identités différentes le peut.

### 1.5 Workflow de base dans Burp

1. `Proxy` > `Intercept` désactivé (`Intercept is off`) pendant la navigation
   normale — sinon chaque requête doit être validée à la main.
2. Naviguer dans l'application en étant connecté, en déclenchant chaque
   fonctionnalité à auditer (inviter un superviseur, rejoindre un groupe,
   envoyer un fichier, réinitialiser un mot de passe, ouvrir la messagerie).
   Cela peuple `Proxy` > `HTTP history` avec toutes les routes réelles.
3. Repérer la requête pertinente dans l'historique, clic droit >
   `Send to Repeater`.
4. Dans `Repeater`, modifier le paramètre visé (identifiant, en-tête, corps de
   requête), cliquer `Send`, comparer la réponse à la réponse attendue.
5. Conserver la preuve : dans Repeater, la paire requête/réponse reste
   consultable ; faire une capture d'écran des deux panneaux (requête à
   gauche, réponse à droite) et l'enregistrer sous
   `preuve_burp_<constat>_<date>.png`. Une preuve écran par test, citée dans
   le rapport final — un test "fait à la main" sans preuve reste une
   affirmation, pas un constat démontrable.

---

## 2. Tests par constat

### 2.1 Constat 3.1 — Contrôle d'accès incomplet sur l'ajout de superviseur

**Route** : `POST /add-supervisor`
(`api-dance/app/controllers/auth_controller.ts`, méthode `addSupervisor`,
lignes 560-612).

**Ce qui est testé** : la méthode vérifie seulement le rôle de l'appelant, pas
qu'il est déjà superviseur de l'enfant ciblé.

**Étapes** :

1. Connecté en tant que Compte A, déclencher l'invitation d'un tiers comme
   superviseur pour l'enfant `X` (le sien).
2. Repérer la requête `POST /add-supervisor` dans `HTTP history`, l'envoyer
   dans Repeater.
3. Dans le corps de la requête, remplacer l'identifiant de l'enfant `X` par
   l'identifiant de l'enfant `Y` (rattaché au Compte B, pas au Compte A).
4. Envoyer.

**Résultat attendu si sécurisé** : erreur 401/403, l'appelant n'est pas
superviseur de l'enfant `Y`.

**Résultat qui confirme le constat** : réponse 200, l'invitation part quand
même pour l'enfant `Y`. C'est la preuve directe de 3.1.

### 2.2 Constat 3.10 — Auto-assignation aux groupes sans validation métier

**Route** : `POST /group/join`
(`api-dance/app/controllers/group_controller.ts`, méthode `join`,
lignes 33-86).

**Étapes** :

1. Connecté en tant que Compte A (élève, par exemple), rejoindre un groupe
   auquel il a normalement droit, capturer la requête.
2. Envoyer dans Repeater.
3. Remplacer le `groupId` par celui d'un groupe/cours qui ne concerne pas le
   Compte A (niveau différent, cours réservé à d'autres élèves).
4. Envoyer.

**Résultat qui confirme le constat** : 200, l'utilisateur rejoint quand même
le groupe. À noter dans le rapport comme "à trancher avec l'équipe produit"
(voir 3.10 du plan d'audit) plutôt que comme correctif automatique, puisque
ça peut être un choix fonctionnel assumé.

### 2.3 Constat 3.6 — Absence d'authentification sur `cdn-app-dance`

**Routes** : `POST /upload/group`, `POST /upload/user`, `POST /upload/post`,
`POST /medias/clean` (`cdn-app-dance/src/app.ts`).

**Étapes** :

1. Capturer une requête d'upload normale (déclenchée depuis `app-dance`) dans
   `HTTP history`.
2. L'envoyer dans Repeater.
3. Supprimer tout en-tête d'authentification, cookie ou jeton présent dans la
   requête (s'il y en a un — le constat indique qu'il n'y en a probablement
   aucun à retirer).
4. Envoyer directement contre l'URL de `cdn-app-dance`, sans repasser par
   `app-dance`.

**Résultat qui confirme le constat** : 200, le fichier est accepté et stocké
sans aucune vérification d'identité. C'est le test le plus rapide de ce
guide : il ne nécessite même pas d'être connecté au préalable, juste de
rejouer la requête telle quelle vers `cdn-app-dance`.

**Note** : profiter de ce test pour vérifier aussi la taille de fichier
acceptée (constat 3.6 mentionne une limite très large, ~5 Go) — un fichier de
quelques dizaines de Mo suffit à confirmer l'absence de blocage, inutile
d'aller jusqu'à la limite réelle.

### 2.4 Constat 3.11 — Injection via l'extension de fichier non filtrée

**Route** : `POST /upload/post` (`cdn-app-dance/src/app.ts`, lignes 129-256).

**Point d'attention** : ce test prouve une exécution de commande sur le
serveur. Périmètre autorisé : un environnement de test isolé uniquement. Le
payload doit rester à délai (time-based, ex. `sleep`), pas destructeur
(pas de `rm`, pas de shell inversé), pour prouver l'injection sans endommager
le serveur.

**Étapes** :

1. Capturer une requête `POST /upload/post` normale (upload multipart) dans
   `HTTP history`, l'envoyer dans Repeater.
2. Repérer le paramètre `filename` dans le corps multipart de la requête
   (ligne du type `Content-Disposition: form-data; name="file";
   filename="video.mp4"`).
3. Construire un nom de fichier qui place une substitution de commande shell
   juste avant l'extension, par exemple :
   `video$(sleep 10).mp4` ou `` video`sleep 10`.mp4 ``
   (l'objectif est un délai mesurable, pas une action destructrice).
4. Envoyer, mesurer le temps de réponse.
5. Comparer avec le temps de réponse d'un upload normal (sans le payload) sur
   le même type de fichier — un écart d'environ 10 secondes confirme que la
   commande shell a été exécutée avec la substitution interprétée, donc que
   l'injection décrite en 3.11 est confirmée et pas seulement théorique.

**Résultat attendu si corrigé** : l'extension est filtrée par liste blanche
avant tout traitement, la requête est rejetée (400/415) avant même d'atteindre
l'appel `exec()`.

**Preuve à conserver** : capture des deux requêtes (normale et avec payload)
avec leurs temps de réponse visibles dans Burp (`Response` > onglet
`Timing` ou simplement le temps affiché en bas du panneau réponse).

### 2.5 Constat 3.2 — CORS incohérent sur le canal WebSocket (socket.io)

**Fichier concerné** : `api-dance/start/ws.ts`, lignes 6-11
(`cors: { origin: '*' }`).

Burp (versions récentes) capture aussi le trafic WebSocket dans
`Proxy` > `WS history`, mais l'origine se négocie sur la requête HTTP de
handshake (upgrade), avant le passage en WebSocket pur.

**Étapes** :

1. Ouvrir la messagerie dans l'app, capturer la requête de handshake
   (`GET` avec en-têtes `Upgrade: websocket`, `Origin: ...`) dans
   `HTTP history`.
2. L'envoyer dans Repeater.
3. Remplacer l'en-tête `Origin` par un domaine qui ne figure pas dans la liste
   blanche de l'API HTTP principale (`api-dance/config/cors.ts`, lignes
   11-17) — par exemple `https://site-quelconque.example`.
4. Envoyer.

**Résultat qui confirme le constat** : la connexion s'établit quand même
(réponse 101 Switching Protocols), alors que la même origine serait rejetée
sur l'API HTTP classique. Confirme l'incohérence relevée en 3.2.

**Alternative si Repeater rejoue mal le handshake WebSocket** :
un petit fichier HTML local avec un script `socket.io-client` pointé sur le
serveur, ouvert directement dans le navigateur (donc avec une origine
`file://` ou un autre domaine local), donne le même résultat de façon plus
simple à démontrer.

### 2.6 Constat 3.12 — Absence de limitation de débit sur la réinitialisation de mot de passe

**Route** : contrôleur `password_reset_controller.ts` (route d'envoi du lien
de réinitialisation).

**Étapes** :

1. Capturer la requête de demande de réinitialisation (formulaire "mot de
   passe oublié"), l'envoyer dans Repeater.
2. Cliquer `Send` 15 à 20 fois de suite (ou utiliser la fonction d'envoi en
   série si la version de Burp la propose).
3. Observer les codes de réponse de chaque tentative.

**Résultat qui confirme le constat** : toutes les requêtes renvoient 200 sans
qu'aucune ne soit bloquée (pas de 429), contrairement à la route de connexion
qui utilise déjà `getLoginLimiter()`
(`auth_controller.ts`, ligne 158).

### 2.7 Constat 3.13 — Cookie et protection CSRF, à vérifier avant conclusion

**Fichier concerné** : `api-dance/config/app.ts`, lignes 32-39
(cookie configuré avec `sameSite: 'lax'`).

**Étapes** :

1. Dans `HTTP history`, repérer une réponse de connexion (`login`) et
   observer l'en-tête `Set-Cookie` renvoyé — noter les attributs présents
   (`HttpOnly`, `Secure`, `SameSite`).
2. Prendre une requête d'action sensible après connexion (par exemple
   `POST /add-supervisor` déjà utilisée en 2.1) et vérifier dans l'onglet
   `Request` si le cookie est effectivement envoyé, et si le serveur répond
   différemment selon qu'on retire ce cookie tout en gardant le jeton
   `Authorization` (guard par jeton, `api-dance/config/auth.ts`, ligne 6).
3. Si retirer le cookie ne change rien au comportement de l'API (seul le
   jeton compte), le cookie est un résidu sans usage d'authentification réel
   — la protection CSRF explicite reste alors une priorité basse, comme noté
   dans le plan d'audit.

**Résultat à documenter** : usage réel confirmé ou non du cookie, pour lever
le `[à vérifier]` du constat 3.13.

---

## 3. Récapitulatif

| Constat | Route testée | Comptes nécessaires | Preuve attendue |
|---|---|---|---|
| 3.1 | `POST /add-supervisor` | A + enfant d'un autre compte (B) | Réponse 200 sur un enfant non supervisé |
| 3.10 | `POST /group/join` | A + groupe hors périmètre | Réponse 200 sur un groupe non autorisé |
| 3.6 | `POST /upload/*`, `/medias/clean` (cdn-app-dance) | Aucun (test sans authentification) | Upload accepté sans jeton |
| 3.11 | `POST /upload/post` (cdn-app-dance) | Un compte suffit | Écart de temps de réponse mesurable (payload `sleep`) |
| 3.2 | Handshake WebSocket (socket.io) | Un compte suffit | Connexion acceptée avec une origine non autorisée |
| 3.12 | Route de réinitialisation de mot de passe | Aucun compte requis | 15-20 requêtes acceptées sans blocage |
| 3.13 | Cookie de session après connexion | Un compte suffit | Confirmation de l'usage réel (ou non) du cookie |

Les constats non couverts par ce guide (3.3, 3.4, 3.5, 3.7, 3.8, 3.9, 3.14)
relèvent de la lecture de code, de la configuration ou de l'audit de
dépendances — pas d'un test de route en direct, donc pas de Burp à prévoir
pour eux.

## 4. Après les tests

Chaque test réalisé doit être reporté dans le tableau des findings du
dossier RNCP (section 6.3) avec :

- La preuve écran associée (`preuve_burp_<constat>_<date>.png`).
- Le statut réel constaté (confirmé / infirmé), en particulier pour 3.11 où
  le plan d'audit demandait explicitement une confirmation par test dédié
  avant de le traiter comme exploité.
- La date du test, pour tracer quand la preuve a été produite par rapport à
  la version du code auditée.
