# Plan d'audit, couche serveur et infrastructure

Document de travail, pas encore une section finale du dossier RNCP. Il comble un
angle mort du périmètre d'audit actuel (`dossier_rncp_centre_art_et_danse.md`,
section 6.1.1) : celui-ci ne couvre que les trois composants applicatifs
(backend, interface, service de fichiers), pas la machine qui les héberge.

Portée de ce document : le serveur `SERVER02` (staging), là où l'accès existe
déjà. Prod et preprod vivent sur d'autres serveurs (confirmé) : chaque
vérification ci-dessous devra être répétée là-bas séparément, avec l'accord de
Vincent avant d'y toucher, une fois cette première passe validée sur staging.

Toutes les commandes listées sont en lecture seule : elles consultent l'état du
système, elles ne le modifient pas. Aucune ne redémarre un service, ne change
une configuration, ne touche à la base de données.

---

## Méthodologie, ordre de dépendance

Comme pour l'audit applicatif (section 6.2 du dossier), l'ordre n'est pas
arbitraire : une catégorie amont peut changer la lecture d'une catégorie aval.
Par exemple, connaître la version exacte de l'OS (catégorie 1) conditionne
quelles CVE chercher (catégorie 2) ; savoir qui a accès SSH (catégorie 3)
conditionne la portée réelle d'un compte compromis (catégorie 6).

1. Identification du système
2. État des correctifs de sécurité
3. Accès et authentification (SSH, comptes, sudo)
4. Réseau et pare-feu
5. Services exposés et surface d'attaque
6. Isolation entre les trois applications
7. Journalisation et détection
8. Sauvegarde et reprise (niveau machine, au-delà de la base de données déjà couverte par AM-03)
9. Durcissement général (Lynis)

---

## 1. Identification du système

**Pourquoi en premier** : toute la suite dépend de savoir précisément quel OS,
quelle version, quel noyau tournent ici. Une CVE se vérifie contre une version
précise, pas contre une famille de distribution supposée.

```bash
cat /etc/os-release        # nom et version exacte de la distribution
uname -a                   # noyau complet, architecture
uptime -p                  # depuis quand le serveur tourne sans redémarrage
```

Point de vigilance déjà identifié dans l'échange avec Ecaterina : ne pas
confondre la version du noyau (`6.1.135-1`, cohérente avec Debian 12) avec la
version de la distribution elle-même. `cat /etc/os-release` donne la réponse
sans ambiguïté.

**Finding attendu** : aucun si la version est une version stable maintenue
(Debian 12 l'est). Si la distribution s'avère en fin de vie (EOL), c'est un
finding Haute priorité immédiat, avant tout le reste de ce document.

---

## 2. État des correctifs de sécurité

**Pourquoi ici** : une fois la version connue, vérifier si elle est à jour de
ses propres correctifs est la question suivante la plus déterminante.

```bash
sudo apt update
apt list --upgradable 2>/dev/null                    # tout ce qui peut être mis à jour
apt list --upgradable 2>/dev/null | grep -i security  # ce qui est spécifiquement un correctif de sécurité
dpkg -l | grep unattended-upgrades                    # mises à jour de sécurité automatiques, configurées ou non
cat /etc/apt/apt.conf.d/20auto-upgrades 2>/dev/null   # si le paquet existe, est-il activé
```

**Finding attendu** : si des mises à jour de sécurité sont en attente sans
mécanisme automatique pour les appliquer, c'est comparable à F-07 dans l'audit
applicatif (absence d'automatisation), mais au niveau du système d'exploitation
plutôt que du code.

---

## 3. Accès et authentification

**Pourquoi ici** : c'est le point déjà repéré (authentification par mot de
passe SSH active). Cette catégorie est la plus directement comparable à SO-1
dans l'EBIOS RM (bourrage d'identifiants), mais sur l'accès au serveur
lui-même plutôt que sur un compte applicatif.

```bash
sudo grep -E "^PasswordAuthentication|^PermitRootLogin|^Port|^PubkeyAuthentication|^MaxAuthTries" /etc/ssh/sshd_config
sudo grep -E "^PasswordAuthentication|^PermitRootLogin" /etc/ssh/sshd_config.d/*.conf 2>/dev/null

cat /etc/passwd | grep -E "/bin/bash|/bin/sh"   # comptes disposant d'un shell interactif
sudo cat /etc/sudoers.d/* 2>/dev/null            # qui a des droits sudo, et sur quoi
getent group sudo                                # membres du groupe sudo

dpkg -l | grep -i fail2ban                       # protection contre le bourrage d'identifiants, présente ou non
sudo systemctl status fail2ban 2>/dev/null
```

**Findings attendus, déjà pressentis** :
- `PasswordAuthentication yes` sans authentification par clé : Haute priorité,
  comparable à SO-1, la porte d'entrée la plus exposée de toute la machine.
- Absence de fail2ban ou équivalent : aggrave le point précédent, rien ne
  ralentit une tentative automatisée.
- `PermitRootLogin yes` : Haute priorité si présent, un compte root accessible
  directement en SSH est une cible de choix.

---

## 4. Réseau et pare-feu

**Pourquoi ici** : une fois qui peut s'authentifier connu, la question suivante
est ce qui est joignable depuis le réseau, point d'entrée par point d'entrée.

```bash
sudo ufw status verbose 2>/dev/null || sudo iptables -L -n -v || sudo nft list ruleset 2>/dev/null
sudo ss -tlnp                                    # tous les ports en écoute, pas seulement ceux des trois applis
sudo ss -ulnp                                    # idem en UDP
```

**Finding attendu** : si aucun pare-feu actif n'est configuré (ni ufw, ni
iptables, ni nftables avec des règles réelles), tous les ports en écoute sont
potentiellement joignables depuis n'importe quelle machine du réseau local, pas
seulement celles qui en ont besoin. À croiser avec la liste des ports du point
suivant.

---

## 5. Services exposés et surface d'attaque

**Pourquoi ici** : maintenant que la liste des ports ouverts est connue
(catégorie 4), identifier ce que chacun fait réellement, et si tout ce qui
tourne est nécessaire.

```bash
sudo systemctl list-units --type=service --state=running
sudo systemctl list-unit-files --state=enabled     # tout ce qui démarre automatiquement au boot
ps aux --sort=-%mem | head -20                     # vue d'ensemble des processus actifs
```

**Finding attendu** : tout service en écoute réseau qui n'est ni l'une des
trois applications, ni un service d'infrastructure identifié et nécessaire
(SSH, par exemple), est une surface d'attaque à justifier ou à couper.

---

## 6. Isolation entre les trois applications

**Pourquoi ici** : ce point recoupe directement la section 5.3.10 du dossier
(recommandation de dockerisation). Avant même Docker, vérifier l'état actuel de
l'isolation donne la mesure exacte du risque déjà couru aujourd'hui.

```bash
ps -eo user,pid,cmd | grep -E "node|node22"        # sous quel compte système tourne chaque appli
ls -la ~/www/                                       # permissions des dossiers de chaque appli
id                                                   # le compte utilisé pour l'admin est-il partagé entre les trois déploiements
```

**Finding attendu** : si les trois applications tournent sous le même compte
système, sans séparation de permissions, une faille sur l'une (par exemple
F-06 ou F-10 sur `cdn-app-dance`) peut potentiellement lire ou modifier les
fichiers des deux autres. C'est l'argument déjà présent dans le dossier pour
justifier la dockerisation (section 5.3.10), avec ici une preuve concrète de
l'état actuel plutôt qu'un raisonnement théorique.

---

## 7. Journalisation et détection

**Pourquoi ici** : une fois la surface connue, la question suivante est de
savoir si une intrusion ou une tentative serait seulement visible après coup.

```bash
sudo systemctl status rsyslog 2>/dev/null
ls -la /var/log/auth.log* 2>/dev/null              # journal des authentifications, tentatives échouées incluses
sudo grep -c "Failed password" /var/log/auth.log 2>/dev/null   # nombre de tentatives échouées déjà enregistrées
dpkg -l | grep -i auditd                            # audit système plus fin, présent ou non
```

**Finding attendu** : si `/var/log/auth.log` n'existe pas ou n'est pas
surveillé, une campagne de bourrage d'identifiants sur SSH (catégorie 3)
risque de ne pas être détectée avant qu'un accès n'ait déjà abouti, plutôt que
d'être repérée pendant qu'elle est en cours.

---

## 8. Sauvegarde et reprise, niveau machine

**Pourquoi ici, distinct d'AM-03** : AM-03 (section 7 du dossier) couvre la
sauvegarde de la base de données. Ce point-ci est plus large : la machine
elle-même (configuration système, fichiers déployés hors base de données,
certificats) a-t-elle un mécanisme de sauvegarde ou de snapshot, indépendamment
des données applicatives ?

```bash
crontab -l 2>/dev/null                              # tâches planifiées de l'utilisateur courant
sudo crontab -l 2>/dev/null                         # idem pour root
ls -la /etc/cron.d/ /etc/cron.daily/ 2>/dev/null
```

**Finding attendu** : si aucune tâche planifiée ni mécanisme de snapshot n'est
identifié, une panne matérielle ou une erreur de configuration nécessiterait
probablement une reconstruction manuelle complète du serveur, pas seulement une
restauration de base de données. À rapprocher du chapitre PRA (section 11 du
dossier, actuellement à rédiger).

---

## 9. Durcissement général, Lynis

**Pourquoi en dernier** : Lynis recoupe une bonne partie des catégories
précédentes en un seul rapport structuré, avec un score. Le lancer en dernier
permet de comparer ses résultats à ce que tu as déjà trouvé manuellement, ce
qui est aussi une bonne vérification croisée de tes propres constats.

```bash
sudo apt install lynis
sudo lynis audit system
```

Le rapport complet est aussi disponible ensuite dans `/var/log/lynis.log` et
`/var/log/lynis-report.dat` pour être relu et cité précisément dans le dossier.

**Source** : Lynis est un outil d'audit open source pour systèmes Linux/Unix,
documentation officielle sur cisofy.com. Niveau de preuve L2 (documentation
produit officielle), cohérent avec le niveau déjà utilisé pour les autres
outils du benchmark (section 6.2 du dossier).

---

## Format de restitution, pour rester cohérent avec le dossier existant

Une fois les neuf catégories parcourues, chaque écart réel se formule comme un
nouveau finding, à la suite de F-13, avec le même format que la table de la
section 6.3 :

| ID | Criticité | Catégorie OWASP / ANSSI | Constat | Recommandation | Statut |
|---|---|---|---|---|---|

Pour la colonne Criticité, garder le même raisonnement que le reste du dossier :
un accès serveur compromis touche potentiellement les trois applications à la
fois, donc les findings de cette catégorie partent rarement en dessous de
Moyenne, et montent à Haute dès qu'ils concernent l'authentification ou le
pare-feu (catégories 3 et 4).

---

## Ce que ce document ne couvre pas

- Preprod et production, sur d'autres serveurs : à auditer séparément, une fois
  cette première passe terminée et validée, avec l'accord explicite de Vincent
  avant tout accès (ce sont des machines qui exposent de vraies données de
  familles).
- Un scan réseau actif (type Nmap) depuis l'extérieur du réseau de l'entreprise :
  hors périmètre de ce document, ça relève d'un test d'intrusion formalisé, pas
  d'un audit de configuration en lecture seule.
