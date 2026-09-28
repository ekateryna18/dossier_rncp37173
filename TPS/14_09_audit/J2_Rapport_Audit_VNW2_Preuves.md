# J2, Rapport d'audit des écarts, basé sur les preuves collectées
### VNWeb, serveur vnw2 (prod/preprod), RNCP37173, M1.4

*Ce document ne couvre que ce qui a été réellement testé à ce stade (8 commandes exécutées sur le serveur vnw2, hébergeant prod et preprod). Ce n'est pas l'intégralité de l'audit prévu en J1 : chaque point du plan d'audit non encore vérifié est listé en fin de document plutôt que passé sous silence, conformément à la méthode du cours.*

---

## 1. Rapport d'audit des écarts (C3)

| # | Volet | Procédure attendue | Observation | Preuve d'audit | Écart qualifié |
|---|---|---|---|---|---|
| 1 | Serveur (vnw2) | Connexion administrateur via compte nominatif + sudo, pas de connexion root directe (ANSSI, guide d'hygiène informatique) | `grep` sur `sshd_config` retourne `PermitRootLogin yes` | Commande exécutée par l'auditrice le 17/09/26 15:36, serveur vnw2 | Connexion SSH root directe autorisée, absence de traçabilité individuelle sur les actions d'administration |
| 2 | Serveur (vnw2) | Authentification forte (clé SSH) sur accès distant (ANSSI/CNIL) | Le `grep` ciblait aussi `PasswordAuthentication` et `PubkeyAuthentication`, mais seule la ligne `PermitRootLogin` a été retournée | Commande exécutée le 17/09/26 15:36, serveur vnw2 | **Non qualifiable en l'état** : l'absence de ces deux directives dans le fichier ne prouve pas leur valeur réelle (elles peuvent être héritées d'une valeur par défaut). Une vérification complémentaire (`sshd -T \| grep -i authentication`) est nécessaire avant de qualifier cet écart |
| 3 | Serveur (vnw2) | Ce point ne correspond à aucun écart si la protection existe déjà | `systemctl status fail2ban` : service actif depuis plus de 3 mois | Commande exécutée le 17/09/26 15:38, serveur vnw2 | **Pas d'écart sur ce serveur** : fail2ban est présent et actif. Contredit l'hypothèse initiale ("pas de fail2ban") posée en J1 à partir des déclarations d'équipe ; à corriger dans le plan d'audit. Reste à vérifier la configuration précise (jails actives, seuils) |
| 4 | Serveur (vnw2) | Système maintenu à jour, correctifs de sécurité appliqués (ANSSI) | `apt list --upgradable` ne retourne aucun paquet | Commande exécutée le 17/09/26 15:42, serveur vnw2 | **Non qualifiable en l'état** : le résultat dépend d'un cache apt dont la fraîcheur n'a pas pu être confirmée (`sudo apt update` non exécutable, compte sans droits sudo). Ni conformité ni écart ne peuvent être affirmés sans cette confirmation |
| 5 | Organisationnel | Comptes d'accès nominatifs, individuels par collaborateur (ANSSI ; ISO 27001:2022 Annexe A) | `awk` sur `/etc/passwd` : une centaine de comptes système, chacun correspondant à un projet, aucun compte individuel visible pour les collaborateurs | Commande exécutée le 17/09/26 15:50, serveur vnw2 | Architecture par conception sans compte individuel : un compte système = un projet, partagé par toute personne y travaillant |
| 6 | Organisationnel | Traçabilité individuelle des connexions et actions (ANSSI) | `last -a` : plusieurs comptes-projets différents connectés depuis la même IP, parfois simultanément | Commande exécutée le 17/09/26 16:09, serveur vnw2 | Impossible d'identifier, à partir des logs, quelle personne physique a agi derrière un compte-projet donné |
| 7 | Serveur (vnw2) | Sauvegardes régulières et documentées des données critiques (ANSSI) | `ls -la /etc/cron.d/` : aucune tâche de sauvegarde présente, seules des tâches de maintenance système (RAID, disque, PHP) et un outil de supervision (Munin) déjà installé | Commande exécutée le 17/09/26 16:13, serveur vnw2 | Aucune sauvegarde visible via cron système sur ce serveur. Limite : une sauvegarde pourrait exister par un autre mécanisme (crontab root inaccessible, service externe) non vérifié ici |
| 8 | Serveur/SI (vnw2) | Journalisation permettant une investigation sur une durée raisonnable (ANSSI) | `ls -la /var/log/auth.log*` et `cat /etc/logrotate.d/rsyslog` : rotation hebdomadaire, 4 copies conservées | Commandes exécutées le 17/09/26 16:16, serveur vnw2 | Fenêtre de conservation des logs limitée à environ 4 semaines ; au-delà, aucune investigation rétroactive possible, aucune centralisation externe ne compense cette limite |

**Écart potentiellement réévalué par ces preuves** : l'écart initial "pas de fail2ban" (posé en J1 sur la base de déclarations d'équipe, sans vérification serveur) doit être retiré ou reformulé pour le serveur vnw2, où fail2ban est confirmé actif. Ce point illustre l'intérêt de la vérification directe par rapport à la seule déclaration.

---

## 2. Éléments du plan d'audit non encore testés

Conformément au principe du cours ("déclarer ce qui n'est pas couvert"), les points suivants du plan d'audit (J1) n'ont pas encore de preuve associée à ce stade :

- Serveur de staging (les 8 vérifications ci-dessus n'ont porté que sur vnw2, prod/preprod)
- Pare-feu hôte (`ufw`/`iptables`) sur vnw2 et sur le staging
- Effective configuration SSH (`sshd -T`) pour confirmer `PasswordAuthentication`/`PubkeyAuthentication`
- Comptes génériques, séparation compte utilisateur/admin (au-delà du constat d'architecture déjà fait)
- Chiffrement au repos
- Tout le volet Réseau (VPN, segmentation, TLS, cartographie)
- Tout le volet Pipelines (déploiement, secrets, dépendances, protection des branches)
- Tout le volet Conformité (localisation AWS, RGPD)
- L'ensemble des points organisationnels au-delà de la traçabilité déjà observée (communication, gouvernance, sensibilisation, gestion des tiers)

---

## 3. Plan d'action, pour les écarts déjà qualifiés uniquement

| Écart # | Mesure | Type | Justification | Preuve attendue |
|---|---|---|---|---|
| 1 | Désactiver `PermitRootLogin`, imposer un compte nominatif + sudo | Corrective | ANSSI, guide d'hygiène informatique | `PermitRootLogin no` confirmé dans `sshd_config`, test de connexion via compte nominatif |
| 5 | Étudier la faisabilité de comptes individuels en complément des comptes-projets (ou a minima une traçabilité applicative) | Préventive | ANSSI ; ISO 27001:2022 Annexe A 5.3/6.5 | Solution documentée, même si l'architecture par projet est conservée pour des raisons techniques |
| 6 | Découle directement de la mesure de l'écart #5 | - | - | - |
| 7 | Mettre en place une sauvegarde planifiée et vérifiée pour vnw2 | Corrective | ANSSI, guide d'hygiène informatique | Tâche de sauvegarde visible et testée (restauration) |
| 8 | Étendre la fenêtre de rétention des logs ou mettre en place une centralisation externe | Préventive | ANSSI | Configuration modifiée et documentée |

*Les écarts #2, #3, #4 ne sont pas encore qualifiés (preuve insuffisante), donc pas de mesure associée pour l'instant : les qualifier d'abord (`sshd -T`, configuration fail2ban, `sudo apt update`) avant de proposer une mesure.*

---

*Document de travail, base réelle de commandes exécutées le 17/09/26. À compléter au fur et à mesure de l'avancement de l'audit sur le staging, le réseau, les pipelines et la conformité.*
