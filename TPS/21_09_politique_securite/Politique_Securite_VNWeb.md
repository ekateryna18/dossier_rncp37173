# Politique de sécurité de l'information, VNWeb
### RNCP37173, M1.4, Partie 1 : contexte et vulnérabilités

*Document de travail du 21/09/2026. Cette première partie pose le constat sur lequel la politique sera bâtie : elle liste les vulnérabilités techniques et organisationnelles de VNWeb. La politique elle-même est en partie 2, à partir de la section 6.*

---

## 1. Contexte : ce qui détermine les vulnérabilités

### 1.1 Les acteurs

| Acteur | Effectif | Ce que ça change pour la sécurité |
|---|---|---|
| **Développeurs** | 4, dont l'auditrice | Ils développent, administrent les serveurs et exploitent les applications : aucun rôle n'est séparé. Pas de RSSI (responsable de la sécurité des systèmes d'information). |
| **Patron** | 1 | Décide de l'organisation et valide les livraisons sur le plan fonctionnel. N'a pas la compétence technique pour juger la sécurité. Gère lui-même la protection des postes de travail, a accès à l'ensemble des données et détient des droits d'administration (sudo) sur les serveurs locaux. |
| **Clients** | Plusieurs | Utilisent les applications livrées par VNWeb. N'ont pas accès à l'Extranet, qui est réservé à l'entreprise. VNWeb ne maîtrise pas leurs pratiques, mais est responsable de ce qu'il leur livre et des données auxquelles ses accès lui donnent accès. Leurs sites sont hébergés chez les prestataires d'hébergement (weevus est sur AWS). |
| **Prestataires d'hébergement externes (France)** | 2 (personne ou société : à préciser ; l'un d'eux serait une personne) | Chacun gère un serveur externe où tourne la production des projets, et y est seul à disposer des droits d'administration. VNWeb y dispose d'accès de connexion, sans droits sudo. |
| **Services tiers** | AWS, Discord | Hébergeur cloud de weevus, messagerie de l'équipe. |

### 1.2 Les systèmes et où ils se trouvent

| Élément | Où | Qui le gère | Statut de l'information |
|---|---|---|---|
| Deux serveurs locaux, réservés au développement (avec Gitea et l'Extranet) ; aucun n'héberge de site en production | Dans l'entreprise | L'équipe | Déclaré (description du 21/09). Droits sudo : l'auditrice et le patron |
| Serveur des projets : production de tous les projets hors sites de voyance et hors weevus (environ une centaine de projets), et préproduction | Externe, en France | Un prestataire d'hébergement | Confirmé (J2) pour le rôle du serveur ; hébergeur externe déclaré par l'équipe |
| Serveur des sites de voyance : production | Externe, en France | Un prestataire d'hébergement | Déclaré (description du 21/09). Hors accès de l'auditrice, non audité (J1 §1.4) |
| weevus | AWS (Amplify Gen 1, Cognito, Lambda, DynamoDB, S3), région Paris sauf la table `users` en Ohio | L'équipe | Déclaré (J1 §1.2) |
| Extranet, Gitea | Internes, réservés à l'équipe (l'Extranet n'est pas accessible aux clients) | L'équipe | Déclaré (J1 §1.2, description du 21/09) |
| Postes de travail | Dans l'entreprise, ils ne sortent pas des locaux | Chaque développeur | Déclaré (description du 21/09) |
| Télétravail | À distance, via VPN (client VPN Windows) puis prise de contrôle du poste de l'entreprise | Chaque développeur | Déclaré (description du 21/09) |

### 1.3 Le trajet d'un développeur en télétravail

```
Domicile (appareil personnel)
   |  Internet
   v
VPN de l'entreprise (client VPN Windows, mot de passe seul)
   v
Poste de travail de l'entreprise (Bureau à distance Windows : on retrouve son écran)
   +--> Gitea, Extranet, serveurs locaux de dev
   +--> Connexion SSH (administration à distance) vers le serveur des projets (préproduction et production)
   +--> Console AWS / dépôt CodeCommit de weevus
```

Point important : **le poste de l'entreprise est la clé de tout.** Il concentre les accès à toutes les autres briques. S'il est compromis, l'attaquant hérite de tout ce que le développeur peut faire. C'est déjà le sens de la ligne « postes de travail » de l'atelier 1 de J1 (point d'accès pour toutes les valeurs métier).

---

## 2. Vulnérabilités techniques

*Légende du statut (sections 2 et 3) : **Confirmé** = prouvé par une vérification de l'audit sur le serveur des projets (J2, commandes du 17/09/26) ; **Déclaré** = décrit par l'équipe le 21/09 ou relevé en J1, non encore prouvé par une vérification ; **À vérifier** = hypothèse dont la vérification est prévue mais non réalisée.*

### 2.1 Accès et authentification

| Réf | Vulnérabilité | Où | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|---|
| VT-01 | Connexion SSH root directe autorisée (`PermitRootLogin yes`) | Serveur des projets (préproduction et production) | Un mot de passe root deviné ou volé donne le contrôle total du serveur ; les actions d'administration ne sont pas traçables individuellement | J2 #1 | Confirmé |
| VT-02 | Connexion SSH possible depuis n'importe où avec un identifiant et un mot de passe, sans clé ni restriction d'adresse | Serveurs locaux de développement et serveur des projets (préproduction et production) | Un mot de passe deviné ou volé donne accès au serveur depuis Internet, y compris à la production | Description du 21/09 ; J2 #2 (non qualifiable en l'état, vérification `sshd -T` en attente) | Déclaré (essai depuis un réseau extérieur à faire) |
| VT-03 | Comptes partagés : un compte système par projet (environ une centaine), aucun compte individuel ; le compte est partagé par toute personne qui travaille sur le projet | Serveur des projets | Impossible de couper l'accès d'une seule personne ; impossible de distinguer les personnes derrière un même compte | J2 #5, description du 21/09 | Confirmé |
| VT-04 | Traçabilité individuelle impossible : plusieurs comptes-projets se connectent depuis la même adresse, parfois en même temps, et les journaux ne permettent pas d'identifier la personne derrière un compte | Serveur des projets | Après un incident, impossible de savoir qui a fait quoi | J2 #6 | Confirmé |
| VT-05 | Pas de séparation entre compte d'utilisateur courant et compte d'administration | Serveurs, postes | Une connexion volée donne directement les droits d'administrateur | J1 §B, atelier 4 | À vérifier |
| VT-06 | Secrets (identifiants, mots de passe) partagés en clair sur Discord | Canal Discord | Toute personne ayant accès à l'historique du salon récupère les accès | J1 §1.2, atelier 3 (chemin A), atelier 4 | Déclaré |
| VT-07 | Double authentification (MFA) non vérifiée sur les services accessibles après le VPN | AWS, Gitea, Extranet | Un mot de passe volé suffit pour entrer sur le service | J1 atelier 5 (routines) | À vérifier |

### 2.2 Télétravail et postes de travail

| Réf | Vulnérabilité | Où | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|---|
| VT-08 | Le VPN est le seul point d'entrée vers les postes et il est protégé par un simple mot de passe, sans second facteur | Accès VPN de l'entreprise (client VPN Windows) | Un mot de passe VPN volé ou deviné ouvre l'accès au poste de bureau, donc à tous les accès qu'il détient | J1 §A « Connexion VPN », description du 21/09 | Déclaré |
| VT-09 | Une fois le VPN ouvert, on prend le contrôle du poste de l'entreprise avec le Bureau à distance Windows : les postes restent allumés en permanence, sauf le week-end, et aucune autre protection n'est connue entre le VPN et la session du poste | Postes de travail | Le VPN franchi, la session du poste est atteignable à toute heure ; un poste laissé ouvert expose une session complète | Description du 21/09 | Déclaré |
| VT-10 | Le développeur se connecte depuis son appareil personnel, dont l'état de sécurité est inconnu de l'entreprise (mises à jour, antivirus, autres utilisateurs) | Domicile | Un logiciel espion sur l'appareil personnel capte le mot de passe VPN et les frappes | Description du 21/09 | Déclaré (appareil personnel) ; état de sécurité à vérifier |
| VT-11 | Pas de séparation entre le poste de bureautique et le poste d'administration : navigation web, courriels et clés d'accès aux serveurs cohabitent sur le même poste | Postes de travail | Une pièce jointe ou un site piégé compromet le poste qui détient les clés SSH et les accès AWS | J1 §A | À vérifier |
| VT-12 | Postes protégés par une simple session (identifiant et mot de passe) ; chiffrement du disque et autres protections (antivirus) non connus de l'équipe technique, voir VO-04 | Postes de travail | Si le disque n'est pas chiffré, une personne ayant accès physique au matériel peut lire les clés et données stockées sans connaître le mot de passe de session (risque limité car les postes ne sortent pas) | J1 §B, description du 21/09 | Déclaré (session seule) ; chiffrement à vérifier |

### 2.3 Serveurs et environnements

| Réf | Vulnérabilité | Où | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|---|
| VT-13 | Aucune tâche de sauvegarde planifiée visible (le dossier `/etc/cron.d/` ne contient que de la maintenance système et un outil de supervision) | Serveur des projets | Perte définitive des données de production, ou arrêt long, après une panne, une erreur ou une attaque | J2 #7 | Confirmé (limite : une sauvegarde par un autre mécanisme n'a pas été vérifiée) |
| VT-14 | Conservation des journaux de connexion limitée à environ 4 semaines (rotation hebdomadaire, 4 copies), sans centralisation externe | Serveur des projets | Au-delà de 4 semaines, aucune investigation rétroactive n'est possible | J2 #8 | Confirmé |
| VT-15 | Pare-feu du serveur et filtrage sortant non testés | Serveur des projets, serveur de staging | Le serveur peut être atteint depuis n'importe où, ou peut communiquer librement vers l'extérieur | J1 §A/B ; J2 §2 (point non testé) | À vérifier |
| VT-16 | Niveau de mise à jour du système non qualifiable (liste de mises à jour vide, mais fraîcheur du cache non confirmée) ; inventaire des actifs et plan de reprise non vérifiés | Serveur des projets | Faille connue non corrigée ; arrêt long sans procédure de reprise | J2 #4, J1 §B | À vérifier |
| VT-17 | Les serveurs locaux de développement utilisent des bases de test ; la présence de vraies données clients (copies de production) n'est pas jugée probable par l'équipe mais n'est pas vérifiée | Serveurs locaux de dev | Si des données réelles s'y trouvent, elles sont exposées sur un serveur sans protection, avec un problème RGPD à la clé | Description du 21/09, J1 §E | À vérifier |
| VT-18 | Segmentation des environnements non confirmée : dev, préproduction et production communiquent peut-être librement | Réseau | Un accès obtenu sur un environnement ouvre les autres | J1 §A, atelier 3 chemin A | À vérifier |

### 2.4 Préproduction et production chez les prestataires d'hébergement

Toutes les applications hors weevus sont en production sur deux serveurs externes, gérés par des prestataires d'hébergement en France : l'un héberge les sites de voyance, l'autre les autres projets (serveur des projets, qui héberge aussi la préproduction). L'équipe se connecte en SSH au serveur des projets : les vulnérabilités d'accès de la section 2.1 (VT-01 à VT-04) s'y appliquent.

| Réf | Vulnérabilité | Où | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|---|
| VT-19 | Les identifiants d'accès aux serveurs des prestataires circulent probablement par les mêmes canaux non sécurisés que les autres (Discord) | Discord, serveur des projets | Fuite d'un accès vers un serveur que l'équipe ne supervise pas | J1 atelier 3 (chemin A), description du 21/09 | À vérifier |
| VT-20 | Chaque prestataire est seul à disposer des droits d'administration sur son serveur ; l'équipe n'a aucune visibilité sur ses correctifs, ses sauvegardes et ses journaux, et le serveur des sites de voyance n'a fait l'objet d'aucune vérification | Serveur des projets, serveur des sites de voyance | Faille non corrigée ou accès du prestataire non maîtrisé, sans que VNWeb puisse le savoir ni y remédier | Description du 21/09, J1 §1.4 | Déclaré |
| VT-21 | Nature des données hébergées non connue : présence de données réelles en préproduction, et données collectées par les sites de voyance (hors accès de l'auditrice) | Serveur des projets (préproduction), serveur des sites de voyance | Données personnelles, dont la sensibilité est à évaluer, hébergées chez un tiers sans cadre contractuel connu | J1 §E, description du 21/09 | À vérifier |

### 2.5 Développement et livraison

| Réf | Vulnérabilité | Où | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|---|
| VT-22 | Aucune revue de code avant mise en production | Gitea, CodeCommit | Un code défectueux ou vulnérable arrive en production sans que personne de technique ne l'ait relu | J1 atelier 3 (chemin B) | Déclaré |
| VT-23 | Aucun test systématique avant mise en production | Applications | Une modification casse un comportement existant ou ouvre une faille sans être détectée | J1 atelier 3 (chemin B) | Déclaré |
| VT-24 | Pas de CI/CD (chaîne de déploiement automatisée) | Gitea, CodeCommit, serveur des projets | Les déploiements ne laissent pas de trace outillée et les secrets de déploiement sont manipulés à la main | J1 atelier 2 | Déclaré |
| VT-25 | Secrets éventuellement commités dans l'historique du code ; dépendances non analysées ; branche de production non protégée | Gitea, CodeCommit | Fuite persistante de secrets ; bibliothèque vulnérable embarquée ; envoi direct en production | J1 §C | À vérifier |

### 2.6 Données et hébergement cloud

| Réf | Vulnérabilité | Où | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|---|
| VT-26 | Table DynamoDB `users` hébergée en Ohio (us-east-2), donc hors Union européenne | AWS | Transfert de données personnelles hors UE sans garantie identifiée : non-conformité RGPD (Art. 44 et suivants) | J1 §1.2, §1.3 | Déclaré |
| VT-27 | Amplify Gen 1 arrive en fin de support en mai 2027 | AWS (weevus) | Plus aucun correctif de sécurité après l'échéance | J1 §1.2, §B | Déclaré |
| VT-28 | Autres ressources AWS possiblement hors Paris ; chiffrement au repos et droits IAM non confirmés | AWS | Données stockées hors UE ou lisibles trop largement | J1 §B | À vérifier |
| VT-29 | Un compte AWS administrateur (racine) non protégé par double authentification, ou partagé | AWS | Perte totale de contrôle de weevus en cas de vol du mot de passe | J1 §B (IAM) | À vérifier |

---

## 3. Vulnérabilités organisationnelles

### 3.1 Gouvernance et décision

| Réf | Vulnérabilité | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|
| VO-01 | Aucune politique de sécurité écrite, aucun RSSI, aucune gouvernance formalisée | Chacun applique ses propres règles ; rien n'est vérifiable ni opposable | J1 §1.5 | Déclaré |
| VO-02 | Le décideur n'a pas la compétence technique et valide les livraisons sur le plan fonctionnel : aucune validation technique de sécurité n'est connue avant la mise en production | Une livraison techniquement fragile est validée parce qu'elle « marche » ; les mesures de sécurité qui coûtent du temps sont perçues comme du frein sans bénéfice visible | J1 atelier 3 (chemin B), description du 21/09 | Déclaré |
| VO-03 | La sécurité n'est pas traitée comme une priorité par la direction : une proposition de sécurisation de weevus n'a pas été retenue au motif qu'elle n'était pas prioritaire | Les mesures proposées par l'équipe technique n'aboutissent pas faute de décision ; les écarts restent ouverts | Description du 21/09 | Déclaré |
| VO-04 | La protection des postes de travail (antivirus, mises à jour, filtrage) est gérée par le patron, sans compétence technique, et l'équipe technique ne connaît pas l'état de ces protections | Une protection absente ou obsolète n'est pas détectée ; ceux qui sauraient l'évaluer ne peuvent pas la contrôler | Description du 21/09 | Déclaré |
| VO-05 | Le patron a accès à l'ensemble des données et détient des droits d'administration sur les serveurs locaux, sans vigilance technique pour repérer un faux courriel, un faux appel ou une usurpation | La compromission de son compte ou de son poste donne accès à tout ; une demande d'accès obtenue par ruse est exécutée | Description du 21/09 | Déclaré (accès) ; exposition à la fraude à vérifier |

### 3.2 Rôles et continuité

| Réf | Vulnérabilité | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|
| VO-06 | Concentration des rôles : les 4 développeurs cumulent développement, administration et exploitation, sans contrôle croisé | Une erreur ou un acte malveillant d'une personne n'est vu par personne | Description du 21/09, J1 atelier 3 | Déclaré |
| VO-07 | Au départ d'un collaborateur, ses accès sur les serveurs ont été réutilisés (cas réel) au lieu d'être supprimés ou renouvelés ; aucune procédure de retrait d'accès (Gitea, Extranet, AWS, VPN, serveurs, accès prestataires) | Un ancien collaborateur garde un accès valable, ses identifiants n'étant pas renouvelés | Description du 21/09, J1 atelier 3 (chemin C), J1 §D | Déclaré |
| VO-08 | Pas de revue périodique des accès : personne ne vérifie que chaque accès actif est encore justifié | Les droits s'accumulent et ne sont pas retirés faute de contrôle | J1 §D | À vérifier |
| VO-09 | Les droits d'administration (sudo) sur les serveurs locaux sont détenus par deux personnes, l'auditrice et le patron, qui n'a pas la compétence technique ; sur les serveurs externes, l'équipe n'en a pas | Une seule personne compétente peut administrer les serveurs locaux (point unique de défaillance) ; un compte volé du patron donne les droits d'administration | Description du 21/09 | Déclaré |

### 3.3 Pratiques et culture

| Réf | Vulnérabilité | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|
| VO-10 | Pas de sensibilisation ni de rappel des règles de base (le partage d'identifiants sur Discord, VT-06, en est un indice) | Les bonnes pratiques ne sont pas connues ou pas appliquées | J1 §D, atelier 5 (routines) | À vérifier |
| VO-11 | Pas de charte informatique : aucune règle écrite sur l'usage des outils, des postes et du télétravail | Chacun décide seul de ce qui est acceptable | J1 §D | À vérifier |
| VO-12 | Télétravail exceptionnel (urgence, nécessité, indisposition) mais non encadré par écrit : appareil personnel, aucune règle sur les conditions (écran visible, verrouillage, réseau domestique) | Les règles varient d'une personne à l'autre ; aucune n'est vérifiable | Description du 21/09 | Déclaré |
| VO-13 | Pas de procédure de gestion d'incident : qui prévenir, quoi couper, quoi conserver, qui décide | Réaction improvisée, preuves perdues, délai de notification RGPD (72 heures à la CNIL en cas de fuite de données personnelles) non tenu | J1 §D | À vérifier |

### 3.4 Tiers, clients et conformité

| Réf | Vulnérabilité | Ce qui pourrait arriver | Source | Statut |
|---|---|---|---|---|
| VO-14 | Aucun cadre contractuel de sécurité connu avec les prestataires d'hébergement (engagements, responsabilités, sous-traitance RGPD selon l'Art. 28), alors qu'ils hébergent la production | En cas d'incident chez un prestataire, VNWeb ne peut rien exiger et reste responsable vis-à-vis de ses clients | J1 §D « tiers » | À vérifier |
| VO-15 | Chaque serveur externe dépend d'un prestataire unique, dont l'un serait une seule personne : s'il est indisponible ou cesse son activité, VNWeb ne peut pas administrer le serveur à sa place | Arrêt d'applications en production, sans solution de secours | Description du 21/09 | Déclaré |
| VO-16 | Pas d'engagement de sécurité connu vis-à-vis des clients (comment leurs identifiants sont transmis ou stockés, où sont leurs données, qui y accède) ; ces informations ne sont pas accessibles à l'équipe technique | Les clients transmettent leurs accès par des canaux non sûrs ; leurs données sont mal protégées à leur insu | J1 §E | À vérifier |
| VO-17 | Conformité RGPD non établie : registre des traitements (Art. 30), durées de conservation, exercice des droits des personnes, rôle de VNWeb (responsable de traitement ou sous-traitant selon les projets) | Sanction ou mise en demeure, perte de confiance des clients | J1 §E, événement redouté « sanction RGPD » | À vérifier |
| VO-18 | Sécurité physique limitée à une clé : pas de badge ni de contrôle d'accès, visiteurs (livreurs) admis dans les locaux, serveurs locaux dans les locaux de l'entreprise | Accès physique non autorisé à un poste ou à un serveur local, sans trace | Description du 21/09 | Déclaré |

---

## 4. Ce qui protège déjà

Une politique ne part pas de zéro : voici ce qui existe et qu'il faut conserver.

- **fail2ban est actif sur le serveur des projets depuis plus de 3 mois** (J2 #3). Sa configuration précise (jails actives, seuils) reste à vérifier.
- **Une journalisation des connexions est en place sur le serveur des projets**, avec rotation hebdomadaire et 4 copies conservées (J2 #8), et un outil de supervision (Munin) est installé, avec des tâches de maintenance planifiées pour le disque, le RAID et PHP (J2 #7). L'existence d'alertes n'est pas vérifiée.
- **Les postes ne sortent pas de l'entreprise.** Le risque de perte ou de vol de matériel est faible, ce qui limite le risque lié à l'absence de chiffrement de disque (VT-12) sans le supprimer.
- **Les serveurs de développement sont séparés de la préproduction et de la production** (à confirmer : ils doivent l'être réellement sur le plan réseau, voir VT-18).
- **Les droits d'administration sont limités** : sur les serveurs locaux, seules deux personnes détiennent des droits sudo ; sur les serveurs externes, l'équipe n'en a pas (VO-09).
- **Le télétravail est exceptionnel** (urgence, nécessité, indisposition), ce qui limite l'exposition du VPN et des postes (VT-08 à VT-10).
- **Les serveurs locaux n'utilisent que des bases de test**, selon l'équipe (à vérifier, VT-17).
- **La préproduction et la production sont hébergées en France**, donc a priori sous le droit européen (à confirmer, notamment l'emplacement exact des données).
- **L'Extranet est réservé à l'équipe** : les clients n'y accèdent pas, ce qui réduit la surface d'exposition de ses données.
- **L'essentiel de weevus est dans la région de Paris** (eu-west-3) : la non-conformité est concentrée sur une ressource (VT-26).
- **Un audit est en cours (J1 et J2)**, avec des constats appuyés par des commandes horodatées du 17/09/26.
- **Équipe de 4 personnes** : les règles peuvent être décidées et appliquées vite, sans lourdeur.

---

## 5. Points restant à vérifier

| # | Point à vérifier | Réf. | Auprès de qui, ou comment |
|---|---|---|---|
| 1 | Rôle exact de chacun des deux serveurs locaux (développement, Gitea, Extranet) | 1.2 | Équipe technique |
| 2 | Nature de chaque prestataire (personne ou société), contrat écrit, serveur géré par chacun | VO-14, VO-15 | Patron |
| 3 | Nature des données hébergées sur le serveur des sites de voyance et en préproduction | VT-21, VO-17 | Patron, prestataires |
| 4 | Accessibilité du service SSH depuis Internet : essai de connexion depuis un réseau extérieur à l'entreprise (par exemple un partage de connexion de téléphone), sur chaque serveur | VT-02 | Auditrice |
| 5 | Configuration SSH effective (`sshd -T`) : mot de passe seul ou clé. Le compte d'audit n'ayant pas de droits sudo, un compte avec droits est nécessaire | VT-02 | Auditrice, avec le prestataire pour le serveur des projets |
| 6 | Pare-feu (`ufw` ou `iptables`) sur le serveur des projets et sur les serveurs locaux | VT-15 | Auditrice, prestataire |
| 7 | Niveau de mise à jour du serveur des projets (`apt update` puis `apt list --upgradable`) et configuration de fail2ban (jails, seuils) | VT-16 | Auditrice, prestataire |
| 8 | Audit complet des serveurs locaux et du serveur de staging (les huit vérifications de J2 ont porté sur le serveur des projets seul) | VT-15, VT-16 | Auditrice |
| 9 | Protections des postes (antivirus, mises à jour, filtrage web) et chiffrement des disques | VT-12, VO-04 | Patron |
| 10 | Informations clients : transmission de leurs accès, emplacement de leurs données | VO-16 | Patron |
| 11 | Sauvegardes sur le serveur des projets par un autre mécanisme que cron | VT-13 | Prestataire |
| 12 | Chiffrement au repos, volet réseau (segmentation, TLS), volet pipelines, volet conformité (localisation AWS, RGPD) | VT-18, VT-25, VT-28, VO-17 | Auditrice |
| 13 | Contrat d'hébergement de chaque site : conclu par VNWeb (qui le revend au client) ou par le client directement, ce qui détermine le rôle de VNWeb (sous-traitant du client, ou développeur qui intervient sur un hébergement du client) | VO-14, VO-17 | Patron |

---

# Partie 2 : la politique de sécurité

## 6. Objet, périmètre et principes

### 6.1 Objet

La présente politique de sécurité des systèmes d'information (PSSI) fixe les objectifs, le périmètre, les principes et les règles qui protègent les systèmes et les données de VNWeb. Elle s'appuie sur le constat de la partie 1 : chaque règle qu'elle contient répond à une ou plusieurs vulnérabilités identifiées (références VT et VO).

Elle s'impose à toute personne qui dispose d'un accès aux systèmes de VNWeb, à compter de son approbation par la direction.

La PSSI ne remplace pas d'autres documents, qu'elle appelle sans les contenir :

| Document | Rôle | Situation chez VNWeb |
|---|---|---|
| Engagement de la direction | Affirme l'engagement du dirigeant en faveur de la sécurité | À intégrer au bloc d'approbation de la PSSI |
| Charte informatique | Règles d'usage individuel des outils et des postes | À produire à partir de la PSSI (vulnérabilité VO-11) |
| Plan de reprise d'activité | Conduite à tenir pour redémarrer les applications après un sinistre | À produire (vulnérabilité VT-16) |
| Procédures d'arrivée, de départ et de gestion d'incident | Déroulé pas à pas, à l'usage de l'équipe | Prévues en annexe de la PSSI |

### 6.2 Objectifs

VNWeb protège quatre valeurs (partie 1 et analyse EBIOS RM de J1) : la continuité de service des applications, la confidentialité des données, l'intégrité du code et des déploiements, et sa réputation auprès des clients. Six objectifs en découlent.

| Réf | Objectif | Valeur protégée | Vulnérabilités visées |
|---|---|---|---|
| OBJ-1 | Maîtriser les accès : chaque accès est nominatif, protégé par une authentification adaptée au risque, et retiré quand il n'est plus justifié | Confidentialité, intégrité | VT-01 à VT-05, VT-07, VT-08, VO-07 à VO-09 |
| OBJ-2 | Protéger les secrets et les données : les identifiants ne circulent pas par les canaux de discussion, et les données personnelles sont hébergées et traitées conformément au RGPD | Confidentialité, réputation | VT-06, VT-26 à VT-29, VO-16, VO-17 |
| OBJ-3 | Assurer la continuité : sauvegardes testées, administration possible par plus d'une personne, prestataires d'hébergement encadrés | Continuité | VT-13, VT-16, VT-20, VO-09, VO-15 |
| OBJ-4 | Garantir l'intégrité du code et des livraisons : revue, tests et traçabilité des déploiements | Intégrité | VT-22 à VT-25 |
| OBJ-5 | Détecter et traiter les incidents : journaux exploitables, procédure connue de toute l'équipe | Continuité, réputation | VT-04, VT-14, VO-13 |
| OBJ-6 | Rendre la sécurité décidée et vérifiable : responsabilités écrites, décisions de risque signées, revue régulière | Toutes | VO-01 à VO-06, VO-10 à VO-12, VO-18 |

Chaque objectif sera assorti d'un indicateur chiffré dans la section consacrée au suivi (section 23).

### 6.3 Périmètre

**Personnes concernées**

- La direction et les quatre développeurs, ainsi que toute personne recevant un accès (arrivant, stagiaire, alternant).
- Les prestataires d'hébergement, pour les serveurs qu'ils gèrent : VNWeb ne pouvant pas modifier ces serveurs, les règles qui les concernent prennent la forme d'exigences écrites (voir section consacrée aux prestataires).
- Les clients ne sont pas soumis à la PSSI. VNWeb reste responsable de ce qu'il leur livre et des données auxquelles ses accès lui donnent accès.

**Systèmes concernés**

| Système | Application de la PSSI |
|---|---|
| Postes de travail, VPN et Bureau à distance Windows | Règles directes, y compris en télétravail |
| Serveurs locaux (développement, Gitea, Extranet) | Règles directes |
| Serveur des projets (production des projets hors sites de voyance, et préproduction) | Exigences écrites envers le prestataire ; règles directes pour les accès de l'équipe |
| Serveur des sites de voyance (production) | Exigences écrites envers le prestataire ; serveur non audité à ce jour |
| weevus sur AWS (Amplify, Cognito, Lambda, DynamoDB, S3, CodeCommit) | Règles directes |
| Discord | Règles d'usage : aucun secret ni donnée personnelle sur ce canal |
| Locaux de l'entreprise | Règles de sécurité physique |

**Données concernées** : données des clients et des utilisateurs finaux traitées par les applications que VNWeb développe et auxquelles ses accès lui donnent accès (dont des données personnelles), hébergées chez les prestataires d'hébergement ou, pour weevus, sur AWS, code source, identifiants et secrets d'accès, données de l'Extranet (tâches et projets).

**Exclusions déclarées**

- Le contenu fonctionnel du code applicatif et sa qualité (exclus de l'audit en J1). Sont en revanche couverts les contrôles qui l'entourent : revue, tests, déploiement.
- Les pratiques de sécurité internes des clients.
- Les appareils personnels, sauf lorsqu'ils servent à se connecter aux systèmes de VNWeb (télétravail).

### 6.4 Principes directeurs

Ces principes guident l'écriture de toutes les règles. En cas de situation non prévue, ils servent de repère.

| Réf | Principe | Énoncé | Vulnérabilités visées |
|---|---|---|---|
| PR-1 | Moindre privilège | Chaque personne dispose uniquement des droits nécessaires à sa mission, pour la durée nécessaire | VT-05, VO-05, VO-08, VO-09 |
| PR-2 | Traçabilité individuelle | Toute action d'administration est attribuable à une personne précise | VT-03, VT-04, VT-14 |
| PR-3 | Protection des secrets | Un identifiant ne circule pas dans un canal de discussion ; il est conservé dans un espace prévu à cet effet | VT-06, VT-19 |
| PR-4 | Séparation des environnements | Développement, préproduction et production sont distincts ; les données réelles ne sont pas utilisées en développement | VT-17, VT-18, VT-21 |
| PR-5 | Continuité des rôles | Aucun accès ou rôle critique ne repose sur une seule personne | VT-20, VO-09, VO-15 |
| PR-6 | Proportionnalité | Les règles sont peu nombreuses, applicables par une équipe de quatre personnes et vérifiables : une règle qui ne peut pas être appliquée ni contrôlée n'est pas retenue | VO-01, VO-06 |
| PR-7 | Décision explicite | Un risque que la direction choisit de ne pas traiter est consigné par écrit, avec son motif et sa date de réexamen | VO-02, VO-03 |
| PR-8 | Amélioration continue | La PSSI est revue chaque année et après tout incident majeur | VO-10, VO-11 |

---

## 7. Référentiels applicables

La PSSI s'appuie sur le socle de référence retenu en J1 (section 1.5), faute de politique interne antérieure.

| Référentiel | Usage dans la PSSI |
|---|---|
| ANSSI, *Guide d'hygiène informatique* | Base des règles organisationnelles et techniques générales |
| ANSSI, *Recommandations relatives à l'authentification multifacteur et aux mots de passe* (8 octobre 2021) ; CNIL, recommandation sur les mots de passe et autres secrets partagés (délibération n° 2022-100 du 21 juillet 2022) | Règles d'accès, de mots de passe et de double authentification (sections 9 et 10) |
| OWASP, *Code Review Guide* et *DevSecOps Guideline* | Règles de développement et de livraison |
| RGPD, notamment le chapitre V (article 44 et suivants), les articles 28, 30 et 33 | Règles de données personnelles, de sous-traitance et de gestion d'incident |
| ISO 27001:2022, Annexe A | Grille de comparaison des règles, sans visée de certification à ce stade |

Trois précisions cadrent leur usage :

- Ces référentiels guident la rédaction. Seul le RGPD est une obligation légale : en cas de divergence, la réglementation prévaut sur la présente politique.
- Les valeurs chiffrées de la PSSI (durées, fréquences, longueurs) sont des choix de VNWeb, marqués « valeur proposée » et soumis à la validation de la direction. Elles ne sont attribuées à aucun référentiel.
- La directive européenne NIS2 et le règlement européen DORA, cités dans la littérature sur les PSSI, ne sont pas retenus à ce stade : leur applicabilité à VNWeb n'est pas établie et reste à évaluer avec la direction.

---

## 8. Rôles et responsabilités

Références : ISO 27001:2022 A.5.1, A.5.2, A.5.3, A.5.4.

### 8.1 Les rôles

| Rôle | Tenu par | Responsabilités principales |
|---|---|---|
| **Direction** | Le patron | Approuve la PSSI, alloue le temps et les moyens, désigne le référent et son suppléant, décide des dérogations, gère la protection des postes de travail avec l'appui du référent |
| **Référent sécurité** | Un développeur, désigné par la direction | Tient la PSSI à jour, suit les indicateurs et les vérifications de la section 5, tient le registre des accès et des dérogations, coordonne la réponse aux incidents, rend compte à la direction |
| **Suppléant du référent** | Un autre développeur, désigné par la direction | Tient toutes les fonctions du référent en son absence, avec les mêmes accès aux informations |
| **Développeurs** | Les quatre développeurs | Appliquent la PSSI, signalent toute anomalie, participent aux revues |
| **Prestataires d'hébergement** | Les deux prestataires | Appliquent les exigences écrites de VNWeb pour leur serveur et préviennent en cas d'incident |
| **Clients** | Sans rôle dans la PSSI | Sont informés des incidents qui touchent leurs données |

Le référent coordonne, contrôle et alerte : il ne dispose pas d'une autorité hiérarchique. Toute décision de risque revient à la direction (principe PR-7). La désignation du référent ne lui transfère pas la responsabilité des décisions de la direction : il répond de l'exécution de ses missions, notamment de l'alerte, et non des décisions de risque prises par la direction. Le rôle de référent ne repose pas sur une seule personne (principe PR-5).

### 8.2 Qui décide, qui réalise

D : décide. R : réalise. C : est consulté. I : est informé. Un tiret : non concerné.

| Activité | Direction | Référent | Développeurs | Prestataires |
|---|---|---|---|---|
| Approuver la PSSI et ses révisions | D | R (prépare) | C | I |
| Attribuer ou retirer un accès | I | D et R | C | R (serveurs externes) |
| Accepter un risque (dérogation) | D | R (instruit) | C | - |
| Vérifier les protections des postes | R | C | I | - |
| Gérer un incident | D (communication, décisions de fond) | R (coordonne) | R | R (son serveur) |
| Exiger et suivre les mesures auprès des prestataires | D (contrat) | R | I | R |
| Suivre les indicateurs et rendre compte | I | R | C | - |
| Sensibiliser l'équipe | R (porte le message) | R (anime) | I | - |

### 8.3 Décisions et dérogations

Le référent et les développeurs proposent des mesures ; la direction décide. Pour que chaque décision reste explicite et vérifiable, toute proposition suit le même chemin :

1. La proposition est formulée par écrit (mesure, risque traité, effort estimé).
2. La direction répond par écrit sous 30 jours (valeur proposée) : acceptée, planifiée avec une échéance, ou refusée.
3. Pour toute mesure lourde, coûteuse ou dépendante d'un prestataire, la proposition comprend un ou plusieurs **paliers intermédiaires** : des mesures moins coûteuses qui réduisent le risque par étapes (par exemple, pour la connexion SSH : mot de passe long et unique et connexion root désactivée, puis restriction par adresse, puis clés). La direction peut retenir un palier plutôt que refuser l'ensemble.
4. Un refus, ou toute mesure de la PSSI que VNWeb choisit de ne pas appliquer, donne lieu à une **fiche de dérogation** signée par la direction : risque accepté, motif, responsable de l'acceptation, date de réexamen, et mention que la direction a été informée du risque, de la mesure proposée et des paliers intermédiaires.
5. Les fiches sont réexaminées à la date fixée, au plus tard 12 mois après la signature (valeur proposée).

Ce chemin ne contraint pas la direction à accepter une mesure. Il garantit qu'un risque connu et non traité l'est par choix, consigné et daté, et non par oubli. La fiche documente une décision : elle atteste que la direction a été informée et décide en connaissance de cause. Elle ne contient aucune clause de décharge et ne modifie pas les obligations légales de l'entreprise. Le modèle de fiche et la note de décision, qui présente à la direction les principales décisions à prendre, figurent en annexe (annexes C et E).

### 8.4 Règles

**Lecture des tableaux de règles (sections 8 à 23).** *Responsable* désigne un rôle, et non une personne. *Priorité* indique l'ordre de mise en œuvre proposé : P1 immédiat (environ 30 jours), P2 dans les 90 jours environ, P3 dans les 6 mois environ ; ces délais sont arbitrés en section 18. *Preuve ou indicateur* désigne ce qui permet de vérifier que la règle est appliquée.

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-01 | La direction désigne par écrit un référent sécurité et un suppléant parmi les développeurs | VO-01, VO-06, VO-09 | Direction | P1 | Désignation inscrite dans le bloc d'approbation |
| R-02 | Le référent remet à la direction chaque trimestre (valeur proposée) un compte rendu de sécurité d'une page : les treize indicateurs du tableau de bord (23.2) avec leur évolution, les écarts ouverts, les dérogations à réexaminer, les éléments de veille et les décisions demandées | VO-01, VO-02, VO-03, VO-04 | Référent | P2 | Un compte rendu daté par trimestre ; tableau de bord à jour |
| R-03 | Toute proposition de mesure de sécurité reçoit une réponse écrite de la direction sous 30 jours (valeur proposée). Toute mesure non appliquée fait l'objet d'une fiche de dérogation signée par la direction, qui atteste qu'elle a été informée du risque et des paliers intermédiaires proposés ; la fiche est réexaminée au plus tard 12 mois après (valeur proposée) | VO-02, VO-03 | Direction, référent | P1 | Part des propositions ayant reçu une réponse dans le délai ; registre des dérogations à jour, aucune mesure écartée sans fiche |

---

## 9. Règles d'accès et d'authentification

Références : ANSSI, *Guide d'hygiène informatique* ; ANSSI, recommandations relatives à l'authentification multifacteur et aux mots de passe (2021) ; CNIL, recommandation sur les mots de passe (délibération n° 2022-100, 2022) ; ISO 27001:2022 A.5.15, A.5.16, A.5.17, A.5.18, A.8.2, A.8.5.

Sur les serveurs externes, VNWeb ne peut pas modifier la configuration : les règles qui les concernent y sont demandées par écrit au prestataire (section 13) et, à défaut, font l'objet d'une dérogation (règle R-03).

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-04 | Chaque personne dispose d'un compte nominatif sur Gitea, l'Extranet, AWS, le VPN et les serveurs locaux ; aucun compte partagé n'est utilisé par une personne. Sur le serveur des projets, tant que subsistent des comptes partagés par projet, toute connexion à un compte-projet est inscrite dans un journal d'usage (personne, date, motif), et des comptes nominatifs sont exigés par écrit du prestataire (R-19) | VT-03, VT-04, VT-05, VT-08 | Référent, développeurs, prestataire | P1 | Liste des comptes : chacun a un titulaire identifié ; journal d'usage tenu, chaque connexion visible dans les journaux du serveur y figure ; réponse écrite du prestataire ou dérogation |
| R-05 | Les droits d'administration sont attribués nominativement, selon le besoin, avec un compte d'administration distinct du compte courant. Sur les serveurs locaux, ils sont détenus par le référent et son suppléant ; la direction dispose d'un accès de secours consigné, qui n'est pas utilisé au quotidien | VT-05, VO-05, VO-09 | Référent, direction | P2 | Liste des détenteurs de droits sudo, avec le motif ; usage du compte de secours consigné |
| R-06 | L'accès SSH aux serveurs est protégé : connexion directe en administrateur (root) désactivée ; authentification par clé personnelle protégée par une phrase secrète, à la place du mot de passe ; accès limité aux adresses de l'entreprise (valeur à confirmer selon la stabilité de l'adresse) ; protection contre les essais répétés de mot de passe (type fail2ban) documentée. Sur les serveurs externes, ces points sont exigés du prestataire (R-19) | VT-01, VT-02, VT-15 | Référent (serveurs locaux), prestataire (serveurs externes) | P1 | Configuration SSH effective : connexion root et mot de passe refusés ; essai de connexion depuis un réseau extérieur refusé ; service anti-essais actif sur chaque serveur |
| R-07 | La double authentification est activée sur le VPN, sur AWS (compte racine compris), et sur Gitea et l'Extranet lorsqu'ils la proposent. Si la solution VPN actuelle ne le permet pas, elle est complétée ou remplacée. Le compte racine AWS n'est pas utilisé au quotidien | VT-07, VT-08, VT-29 | Référent | P1 | Part des comptes AWS et VPN avec double authentification (cible : tous) ; essai de connexion VPN sans second facteur refusé |

---

## 10. Gestion des secrets

Références : ANSSI, recommandations relatives à l'authentification multifacteur et aux mots de passe (2021) ; CNIL, recommandation sur les mots de passe (délibération n° 2022-100, 2022) ; ISO 27001:2022 A.5.17.

Un secret est tout ce qui permet de s'authentifier : mot de passe, clé SSH, clé d'accès AWS, jeton d'application.

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-08 | Les secrets (mots de passe, clés, jetons) sont conservés dans le coffre-fort partagé de l'équipe, accessible par comptes nominatifs avec double authentification, et nulle part ailleurs : ni Discord, courriel ou SMS, ni code, ni dépôts. Les mots de passe sont uniques par système, générés par le coffre-fort et d'au moins 14 caractères (valeur proposée) ; ils sont renouvelés lors d'un départ, d'une fuite ou d'une suspicion, et non à date fixe. Les accès critiques (compte racine AWS, nom de domaine, comptes d'administration, coordonnées d'urgence des prestataires) y sont enregistrés et accessibles au référent, à son suppléant et à la direction | VT-06, VT-07, VT-19, VT-25, VO-05, VO-09, VO-10, VO-15 | Développeurs, référent | P1 | Nombre de secrets trouvés hors du coffre-fort (cible : zéro) ; test annuel d'accès aux accès critiques par le suppléant |
| R-09 | Toute demande d'ouverture d'accès ou de transmission de secret reçue par courriel, message ou téléphone, même au nom de la direction, d'un client ou d'un prestataire, est confirmée par un second canal connu avant d'être exécutée | VT-19, VO-05 | Développeurs, direction | P2 | Rappel inscrit au programme de sensibilisation ; demandes refusées ou confirmées, consignées |

---

## 11. Télétravail et postes de travail

Références : ANSSI, *Guide d'hygiène informatique* ; ISO 27001:2022 A.6.7, A.8.1.

Le télétravail est exceptionnel chez VNWeb : urgence, nécessité de service ou indisposition. Il se fait depuis un appareil personnel : connexion au VPN, puis prise de contrôle du poste de l'entreprise par le Bureau à distance Windows. Les postes restent allumés en permanence, sauf le week-end. Ces règles encadrent cet usage plutôt que de le supprimer.

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-10 | Le télétravail est limité à trois motifs : urgence, nécessité de service, indisposition. Il est déclaré au référent ou à son suppléant, avant ou, en cas d'urgence, dans la journée, avec la date et le motif. L'appareil personnel utilisé respecte des conditions minimales : système à jour, antivirus actif, session non partagée avec un tiers, réseau maîtrisé (pas de réseau public) ; les fichiers, clés et secrets restent sur le poste de l'entreprise | VT-10, VO-12 | Développeurs, référent | P1 | Journal de télétravail (personne, date, motif) ; attestation de conformité de l'appareil signée chaque année |
| R-11 | Le Bureau à distance Windows n'est joignable qu'à travers le VPN, et non directement depuis Internet. À la fin d'un télétravail, la session distante est fermée ; en cas d'inactivité, le poste se verrouille automatiquement après 10 minutes (valeur proposée) | VT-09 | Référent, développeurs | P1 | Essai de connexion depuis un réseau extérieur, sans VPN : refusé ; réglage de verrouillage vérifié sur chaque poste |
| R-12 | Chaque poste dispose d'un antivirus actif, de mises à jour automatiques, d'un verrouillage automatique, d'un mot de passe de session robuste et, selon le système installé, d'un disque chiffré dont la clé de récupération est conservée dans le coffre-fort. La direction, qui gère ces protections, en remet la liste au référent, qui les vérifie chaque année et à chaque nouveau poste. Les accès d'administration se font depuis un profil de navigateur ou une session dédiée, distinct de la navigation courante, avec des clés SSH protégées par une phrase secrète | VT-11, VT-12, VO-04 | Direction, référent | P2 | Fiche de contrôle par poste (date, résultat) ; état du chiffrement par poste |

---

## 12. Serveurs, sauvegardes et journalisation

Références : ANSSI, *Guide d'hygiène informatique* ; ISO 27001:2022 A.5.9, A.8.8, A.8.13, A.8.15, A.8.16, A.8.31, A.8.33.

Ces règles s'appliquent directement aux serveurs locaux (développement, Gitea, Extranet). Pour les serveurs externes, elles sont demandées aux prestataires par le cahier d'exigences de la section 13.

Sur le serveur des projets, la vérification du 17/09/26 (J2) a établi que fail2ban est actif, qu'un outil de supervision (Munin) est installé, que les journaux de connexion sont conservés environ quatre semaines et qu'aucune sauvegarde planifiée n'est visible.

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-13 | Un inventaire tenu à jour recense les serveurs, services et logiciels installés (nom, rôle, environnement, hébergeur, responsable, version du système, date de dernière mise à jour) et, pour chaque projet actif, ses dépendances applicatives (langage et version, frameworks et bibliothèques relevés dans les fichiers de verrouillage, gestionnaire de paquets). Il est revu chaque trimestre (valeur proposée) et constitué par cercles successifs, comme la couverture décrite en 18.4 | VT-16, VT-25, VT-27 | Référent, développeurs (leurs projets) | P2 | Inventaire daté, revu chaque trimestre ; part des projets actifs inventoriés (cible : tous à la fin de la vague 3) |
| R-14 | Les mises à jour de sécurité sont appliquées sur les serveurs locaux au moins une fois par mois (valeur proposée). Une faille critique qui touche un composant de l'inventaire est corrigée sous 72 heures si elle est déjà exploitée, sous 7 jours sinon (valeurs proposées) ; si le délai ne peut pas être tenu, une mesure d'atténuation est appliquée et consignée, et le report fait l'objet d'une dérogation datée. Un composant dont la fin de support est annoncée est migré avant l'échéance ou fait l'objet d'une dérogation datée. Sur un serveur externe, ces délais sont exigés du prestataire (R-19) | VT-16, VT-25, VT-27 | Référent, développeurs (leurs projets), prestataires | P2 | Nombre de paquets de sécurité en attente, relevé chaque mois (cible : zéro à l'échéance) ; délai entre la publication d'une faille et le correctif, consigné au registre de veille |
| R-15 | Un pare-feu est actif sur chaque serveur local : tout est refusé par défaut, seuls les services nécessaires sont ouverts, en entrée comme en sortie. Une fois par an (valeur proposée), le référent relève depuis l'extérieur les ports ouverts de chaque serveur, avec l'accord écrit du prestataire pour un serveur externe | VT-02, VT-15 | Référent | P2 | Règles du pare-feu relevées ; liste des ports ouverts comparée à la liste attendue, tout écart fermé ou justifié |
| R-16 | Les environnements de développement, de préproduction et de production sont séparés : les serveurs de développement n'utilisent que des données de test (aucune donnée réelle sans anonymisation validée par le référent), ne détiennent aucun identifiant de production, et les flux entre environnements sont limités à ceux qui sont listés | VT-17, VT-18 | Développeurs, référent | P3 | Contrôle par échantillon des bases de développement chaque trimestre ; liste des flux entre environnements |
| R-17 | Les données critiques (dépôts Gitea, base et fichiers de l'Extranet, configuration des serveurs locaux) sont sauvegardées automatiquement au moins chaque jour (valeur proposée), sur un support distinct du serveur, avec au moins une copie hors des locaux de l'entreprise. Chaque sauvegarde critique est testée en restauration au moins deux fois par an (valeur proposée), avec résultat et durée consignés. Un plan de reprise d'activité court (ordre de redémarrage, personnes et prestataires à contacter, emplacement des sauvegardes, durée d'interruption acceptable fixée avec la direction) est rédigé pour les applications en production et relu chaque année | VT-13, VT-16, VO-15 | Référent, direction | P1 | Tâche de sauvegarde planifiée visible ; date de la dernière sauvegarde réussie ; comptes rendus de test de restauration datés ; plan de reprise écrit, connu du référent, de son suppléant et de la direction |
| R-18 | Les journaux de connexion et d'administration des serveurs locaux sont conservés au moins 12 mois (valeur proposée) sur un emplacement distinct du serveur qui les produit ; cette durée est exigée des prestataires pour les serveurs externes. Une alerte minimale prévient le référent par courriel des échecs de connexion répétés, des connexions root et des connexions hors des horaires habituels, avec les outils déjà présents (fail2ban, Munin) avant tout nouvel outil | VT-14 | Référent, prestataire | P2 | Durée de conservation relevée (état actuel sur le serveur des projets : environ 4 semaines) ; journaux consultables hors du serveur ; alerte de test reçue |

---

## 13. Prestataires d'hébergement

Références : RGPD, article 28 ; ISO 27001:2022 A.5.19, A.5.20.

Toutes les applications hors weevus tournent en production sur deux serveurs externes, chacun géré par un prestataire d'hébergement en France : l'un héberge les sites de voyance, l'autre tous les autres projets (le « serveur des projets »). Chaque prestataire est seul à disposer des droits d'administration, et VNWeb n'a pas de droits sudo sur ces serveurs. VNWeb ne peut donc pas modifier ces serveurs lui-même : les règles prennent la forme d'exigences écrites. Si un prestataire refuse ou ne répond pas, le risque est consigné par une fiche de dérogation signée par la direction (règle R-03).

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-19 | Une fiche est établie pour chaque prestataire : identité (personne ou société), coordonnées, serveur géré, services rendus, existence d'un contrat, contact d'urgence, conservé dans le coffre-fort. Chaque prestataire reçoit par écrit un cahier d'exigences de sécurité pour chaque serveur qu'il gère : connexion root désactivée, authentification par clé, accès limité par adresse, comptes nominatifs pour l'équipe, protection contre les essais répétés, mises à jour et délais de correction des failles, conservation des journaux, sauvegardes automatiques confirmées par écrit avec un test de restauration par an, alerte à VNWeb sous 24 heures après un incident (valeur proposée), liste des comptes actifs tous les six mois (valeur proposée). Il répond par écrit sous 30 jours (valeur proposée) et fournit des relevés de configuration ; le serveur des sites de voyance, non audité à ce jour, est traité de la même manière | VT-03, VT-13, VT-20, VO-14, VO-15 | Référent (rédige), direction (transmet), prestataires | P1 | Fiche complète pour chacun des deux prestataires ; cahier remis avec accusé de réception, réponse écrite, relevés archivés ; confirmation des sauvegardes et compte rendu du test de restauration daté ; test annuel du contact d'alerte ; liste des comptes comparée à la liste des personnes en poste |
| R-20 | Les conditions de sécurité et de protection des données sont formalisées par contrat ou avenant avec chaque prestataire : niveau de service, mesures de sécurité, notification d'incident, sous-traitance de données personnelles (article 28 du RGPD), restitution des données en fin de contrat | VO-14 | Direction (signe), référent (prépare) | P2 | Contrat ou avenant signé ; à défaut, dérogation signée (R-03) |
| R-21 | Une continuité est prévue pour chaque prestataire : un contact de secours chez le prestataire (ou un dépôt de secours convenu par écrit s'il s'agit d'une personne seule) et une procédure décrite pour reprendre l'hébergement ailleurs, incluse dans le plan de reprise (R-17) | VT-20, VO-15 | Direction, référent | P3 | Contact de secours identifié ; procédure de reprise décrite |

---

## 14. Développement et livraison

Références : OWASP, *Code Review Guide* et *DevSecOps Guideline* ; ISO 27001:2022 A.8.25, A.8.29, A.8.32.

Les projets sont versionnés dans Gitea, sauf weevus qui l'est dans CodeCommit. Aucune chaîne de déploiement automatisée n'est en place : les livraisons sont manuelles. Le décideur valide les livraisons sur le plan fonctionnel ; ces règles ajoutent une validation technique, proportionnée à une équipe de quatre personnes.

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-22 | Toute modification destinée à la production est relue par un second développeur avant d'être fusionnée, dans une demande de fusion (Gitea) ou une pull request (CodeCommit) qui trace la relecture ; en cas d'urgence, la relecture a lieu dans les 48 heures (valeur proposée) et est consignée. La branche de production de chaque dépôt actif n'accepte pas de poussée directe, avec les réglages de protection de branche disponibles | VT-22, VT-25, VO-02 | Développeurs, référent | P2 | Part des fusions vers la production ayant une relecture tracée (cible : toutes) ; réglage de protection activé sur chaque dépôt actif |
| R-23 | Une liste de contrôle de mise en production est remplie pour chaque livraison : parcours principaux testés, absence de secret dans le code, dépendances vérifiées, validation en préproduction. Les projets qui traitent des données personnelles ou des authentifications reçoivent des tests automatisés, ajoutés progressivement | VT-23, VO-02 | Développeurs | P3 | Liste de contrôle jointe à chaque livraison |
| R-24 | Chaque déploiement est consigné dans un journal de déploiement (qui, quoi, quand, version, environnement), tant qu'il n'est pas tracé par un outil | VT-24 | Développeurs | P1 | Journal de déploiement tenu ; chaque déploiement visible dans les journaux du serveur a une ligne correspondante |
| R-25 | Les dépendances logicielles de chaque projet actif sont analysées pour les failles connues avant toute mise en production majeure et au moins chaque trimestre (valeur proposée), avec l'outil d'audit du gestionnaire de paquets utilisé ; les failles critiques sont traitées selon les délais de la règle R-14 | VT-25 | Développeurs | P3 | Rapport d'analyse daté par projet ; nombre de failles critiques ouvertes |

---

## 15. Données et conformité

Références : RGPD, chapitre V (article 44 et suivants), articles 28 et 30 ; ISO 27001:2022 A.5.34.

Environ une centaine de projets sont hébergés chez les deux prestataires d'hébergement, et weevus sur AWS. La nature des données de chacun n'est pas connue de l'équipe technique : les sites de voyance, weevus et l'Extranet sont examinés en premier, car ils traitent vraisemblablement des données personnelles.

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-26 | Un registre des traitements de données personnelles (article 30 du RGPD) est établi pour chaque projet hébergé : nature des données, finalité, lieu d'hébergement, destinataires, durée de conservation, rôle de VNWeb (responsable de traitement ou sous-traitant, selon le contrat). Les données sont supprimées ou anonymisées à l'échéance de leur durée de conservation, sauvegardes comprises. Le registre est commencé par weevus, les sites de voyance et l'Extranet | VT-21, VO-17 | Référent, direction | P2 | Registre renseigné ; part des projets couverts (cible : tous, en commençant par les trois cités) ; suppression testée sur un projet |
| R-27 | Toute donnée personnelle est hébergée dans l'Union européenne ; à défaut, son transfert est justifié par une base légale documentée (chapitre V du RGPD). Chaque année, le référent relève la région, le chiffrement au repos et les rôles d'accès (IAM) de chaque ressource AWS (DynamoDB, Cognito, Lambda, S3, Amplify), traite toute ressource hors de l'Union européenne ou non chiffrée, et vérifie que chaque rôle n'a que les permissions utiles | VT-26, VT-28, VT-29 | Direction, référent | P1 | Tableau des ressources daté (région, chiffrement, permissions) ; aucune donnée personnelle hors de l'Union européenne sans base légale documentée |
| R-28 | Une procédure permet de répondre aux demandes d'accès, de rectification ou de suppression de données dans le délai prévu par le RGPD, et désigne la personne qui les traite | VO-17 | Direction | P3 | Procédure écrite ; délai de traitement des demandes reçues |
| R-29 | Les contrats avec les clients précisent le rôle de VNWeb (responsable de traitement ou sous-traitant), le lieu d'hébergement, les prestataires intervenants, les mesures de sécurité et la notification d'incident. La direction remet au référent la liste des contrats et de leurs clauses. Les accès que les clients transmettent passent par un canal convenu (coffre-fort, lien temporaire) et non par la messagerie courante | VO-16, VO-17 | Direction | P3 | Clause type intégrée aux nouveaux contrats ; part des contrats actifs mis à jour ; canal de transmission défini |

---

## 16. Arrivées, départs, revue des accès et sécurité physique

Références : ANSSI, *Guide d'hygiène informatique* ; ISO 27001:2022 A.5.11, A.5.16, A.5.18, A.6.5, A.7.2, A.7.3.

Un collaborateur est déjà parti de VNWeb : ses accès sur les serveurs ont été réutilisés au lieu d'être supprimés ou renouvelés. Sur le serveur des projets, où les comptes sont partagés par projet, retirer l'accès d'une seule personne est impossible : seul le renouvellement des secrets qu'elle connaissait protège l'entreprise. C'est le sens de la règle R-31.

### 16.1 Arrivées, départs et registre des accès

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-30 | À l'arrivée d'une personne, le référent ouvre uniquement des accès nominatifs, limités à ses projets et à sa mission, et les inscrit au registre des accès : pour chaque personne, ses comptes, les systèmes concernés, ses droits, la date d'octroi et le motif, ainsi que les comptes de service indispensables, avec un responsable et des droits limités à leur usage. Dans sa première semaine, l'arrivant suit un parcours d'accueil : présentation de la PSSI et accusé de lecture, modules 1 et 2 de la formation avec leurs quiz, signature de la charte informatique ; un binôme désigné par le référent l'accompagne pendant un mois (valeur proposée) | VT-03, VT-05, VO-08, VO-09, VO-10, VO-11 | Référent | P2 | Registre des accès à jour, daté, où chaque compte réel figure ; parcours d'accueil consigné (dates, résultats des quiz) ; binôme désigné |
| R-31 | La direction prévient le référent de tout départ dès que la date est connue. Au plus tard le jour du départ, les accès de la personne sont retirés selon la liste de contrôle de départ (annexe B) : comptes Gitea, Extranet, AWS, VPN, coffre-fort et Discord désactivés, clés SSH retirées, suppression ou renouvellement de ses accès demandé aux prestataires, clés des locaux et matériel restitués, documentation de ses projets et tâches en cours transmises. La réutilisation des accès d'un ancien collaborateur est interdite : tout secret que la personne connaissait (comptes partagés par projet, mots de passe de serveurs, clés, jetons, accès au coffre-fort) est renouvelé dans les 7 jours suivant son départ (valeur proposée) | VT-03, VO-07 | Référent, direction, prestataires | P1 | Liste de contrôle de départ signée ; délai entre le départ et le retrait des accès (cible : le jour même) ; liste des secrets renouvelés, datée |

### 16.2 Revue des accès

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-32 | Chaque trimestre (valeur proposée) et à chaque changement de mission, le référent compare le registre des accès aux comptes réels de chaque système (Gitea, Extranet, AWS, VPN, serveurs locaux, coffre-fort) et aux listes fournies par les prestataires. Il retire tout accès sans titulaire ou non justifié, désactive les comptes inactifs depuis plus de 90 jours (valeur proposée), et vérifie les droits sudo, les comptes de service et les comptes VPN. Le résultat est transmis à la direction (R-02) | VT-03, VO-08, VO-09 | Référent | P2 | Compte rendu de revue daté ; nombre d'accès retirés ; part des comptes ayant un titulaire identifié (cible : tous) |

### 16.3 Sécurité physique

La sécurité physique des locaux repose aujourd'hui sur une seule clé, sans badge ni contrôle d'accès. Des visiteurs, par exemple des livreurs, entrent dans les locaux, où se trouvent les postes de travail et les serveurs locaux. Les règles suivantes restent proportionnées à des locaux de petite taille : elles portent sur les clés, la présence des visiteurs et la protection des serveurs.

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-33 | Un registre des clés des locaux est tenu : nombre de clés, détenteur, date de remise, date de restitution. Aucune clé n'est reproduite sans l'accord de la direction. En cas de perte ou de clé non restituée, le cylindre de la serrure est changé | VO-07, VO-18 | Direction (détient le registre), référent (contrôle) | P2 | Registre à jour ; nombre de clés en circulation égal au nombre de détenteurs attendus |
| R-34 | Les visiteurs (livreurs, clients, prestataires) sont accueillis par un membre de l'équipe et ne restent pas seuls dans les zones où se trouvent les postes et les serveurs ; les livraisons se font à l'entrée ; aucun secret n'est affiché ni noté à la vue. Les serveurs locaux sont installés dans une pièce ou une armoire fermant à clé, hors d'une zone de passage, dont la clé est confiée au référent, à son suppléant et à la direction. Une fois par an (valeur proposée), le référent contrôle sur place les serveurs, le registre des clés et l'absence de secrets à la vue | VO-18 | Développeurs, direction, référent | P2 | Consigne communiquée à l'équipe ; fiche de contrôle annuelle datée ; serveurs sous clé |

---

## 18. Plan de déploiement : échelle et vagues

Déployer la PSSI consiste à faire passer chaque règle de l'état « écrite » à l'état « appliquée, avec sa preuve ». Le déploiement est étalé en trois vagues à partir de la date J d'approbation de la PSSI par la direction (action A-01). Cette section arbitre les priorités P1, P2 et P3 annoncées en 8.4, décrit ce qui est mis en place à chaque étape et sur quels systèmes, et fixe comment le respect des règles est obtenu pendant le déploiement.

### 18.1 Critères d'arbitrage

Chaque règle est examinée selon six critères, du plus déterminant au moins déterminant : la gravité et les dépendances d'abord, puis les moyens et les ressources, puis l'organisation et les contraintes réglementaires (ordre de priorité retenu par l'auditrice).

| Critère | Question posée pour chaque règle |
|---|---|
| Gravité | Quel effet sur la confidentialité, l'intégrité ou la disponibilité si l'écart n'est pas traité ? Les écarts confirmés passent avant les hypothèses |
| Dépendances | D'autres règles reposent-elles sur celle-ci (elle passe tôt), ou dépend-elle d'une autre (elle passe après) ? |
| Moyens | Faut-il un outil ou un budget ? |
| Ressources | Quel temps de l'équipe, sachant qu'elle compte quatre développeurs, dont le référent ? |
| Organisation | Faut-il l'accord de la direction ou la réponse d'un prestataire ? |
| Contraintes réglementaires | La règle relève-t-elle du RGPD ? |

Une règle est placée en vague 1 lorsque sa gravité est élevée pour un effort faible, ou lorsque d'autres règles en dépendent ; en vague 2 lorsqu'elle demande un effort moyen, ou dépend d'une règle de la vague 1 ou de la réponse d'un prestataire ; en vague 3 lorsqu'elle demande un outil ou un effort important pour une gravité moindre, ou dépend de la vague 2. Les valeurs chiffrées et les échéances sont des propositions soumises à la validation de la direction.

Le plan distingue deux sortes de mesures. Les **règles** (R-xx) sont des exigences durables : elles s'appliquent tant que la PSSI est en vigueur. Les **actions** (A-xx) sont des mises à niveau ponctuelles, faites une fois pour atteindre la conformité, comme une migration ou la recherche de secrets dans un historique ; elles ne sont pas des règles. Une règle dépend parfois d'une action : le coffre-fort de mots de passe, par exemple, est une exigence durable (R-08) dont l'installation est une action (A-02). Une règle qui regroupe plusieurs points est placée dans la vague de son point le plus urgent ; les autres points s'y ajoutent progressivement.

### 18.2 Les trois vagues

| Vague | Échéance | Priorité | Objet | Règles | Actions |
|---|---|---|---|---|---|
| 1 | J + 30 jours | P1 | Fondations et risques les plus graves à faible effort : gouvernance, secrets, accès exposés, sauvegardes, retrait des accès au départ, décision RGPD sur les données hébergées hors de l'Union européenne | 16 | 3 |
| 2 | J + 90 jours | P2 | Déploiement sur l'ensemble des systèmes : exigences écrites aux prestataires, revue des accès, journalisation, développement | 24 | 2 |
| 3 | J + 6 mois | P3 | Rendre l'ensemble durable : outillage (chaîne de déploiement automatisée, séparation des environnements), continuité chez les prestataires, conformité courante au RGPD | 6 | 1 |

Nature d'une mesure : **D** décision ou document ; **T** action technique ou installation ; **E** exigence écrite envers un prestataire, dont l'avancement dépend de sa réponse ; **O** routine ou habitude à instaurer ; **Action** mise à niveau ponctuelle, qui n'est pas une règle. La colonne « Après » indique la mesure qui doit être en place avant.

**Vague 1, à J + 30 jours**

| Réf. | Mesure | Nature | Après |
|---|---|---|---|
| R-01 | Référent et suppléant désignés | D | - |
| R-03 | Réponse écrite aux propositions ; dérogations signées | D | - |
| R-04 | Comptes nominatifs ; journal d'usage des comptes partagés | T | - |
| R-06 | Accès SSH protégé : root désactivé, clés, adresses, anti-essais répétés | T | - |
| R-07 | Double authentification : VPN, AWS, Gitea, Extranet | T | - |
| R-08 | Secrets dans le coffre-fort ; mots de passe uniques ; accès critiques | T | - |
| R-10 | Télétravail limité et déclaré ; appareil personnel | O | - |
| R-11 | Bureau à distance seulement par le VPN ; verrouillage | T | - |
| R-17 | Sauvegardes, tests de restauration, plan de reprise | T | - |
| R-19 | Fiche et cahier d'exigences pour chaque prestataire | E | - |
| R-24 | Journal de déploiement | D | - |
| R-27 | Données dans l'Union européenne, relevé annuel AWS | O | - |
| R-31 | Départ : retrait des accès, secrets renouvelés | O | - |
| R-35 | Tableau de déploiement et de suivi | D | R-01 |
| R-38 | Lancement et accusé de lecture | D | A-01 |
| R-43 | Supports de formation accessibles | D | - |
| A-01 | **Approuver et signer la PSSI** | Action | - |
| A-02 | **Installer le coffre-fort et y placer les secrets ; renouveler ceux déjà exposés dans Discord** | Action | - |
| A-03 | **Données personnelles hors de l'Union européenne : décision sous 30 jours, mise en œuvre sous 6 mois** | Action | - |

**Vague 2, à J + 90 jours**

| Réf. | Mesure | Nature | Après |
|---|---|---|---|
| R-02 | Compte rendu trimestriel et tableau de bord | O | R-35 |
| R-05 | Droits d'administration nominatifs, référent et suppléant | T | R-01 |
| R-09 | Vérification par second canal des demandes d'accès | O | - |
| R-12 | Protections des postes, chiffrement des disques | O | R-01 |
| R-13 | Inventaire des serveurs, logiciels et dépendances | D | - |
| R-14 | Mises à jour, failles critiques, fins de support | O | R-13 |
| R-15 | Pare-feu et relevé des ports ouverts | T | R-13 |
| R-18 | Journaux conservés 12 mois, alerte minimale | T | - |
| R-20 | Contrat ou avenant de sécurité et de données | E | R-19 |
| R-22 | Relecture de code, branche de production protégée | O | - |
| R-26 | Registre des traitements, durées de conservation | D | - |
| R-30 | Arrivée : accès nominatifs, registre, parcours d'accueil | O | - |
| R-32 | Revue trimestrielle des accès | O | R-30 |
| R-33 | Registre des clés | D | - |
| R-34 | Visiteurs, serveurs locaux sous clé | T | R-33 |
| R-36 | Contrôles, contrôle croisé, journal d'administration | O | - |
| R-37 | Veille : sources officielles, condensé mensuel | O | - |
| R-39 | Modules de formation et quiz | O | R-38 |
| R-40 | Séance de la direction | O | - |
| R-41 | Point sécurité mensuel | O | - |
| R-42 | Charte informatique | D | - |
| R-44 | Adaptation de la formation sur demande | O | - |
| R-45 | Revue annuelle de la PSSI | O | - |
| R-46 | Temps alloué et mesure de la charge | D | - |
| A-04 | **Recherche de secrets dans l'historique des dépôts, renouvellement de ceux trouvés** | Action | A-02 |
| A-05 | **Migration d'Amplify Gen 1 vers Gen 2 : plan et environnement d'essai, bascule avant février 2027** | Action | - |

**Vague 3, à J + 6 mois**

| Réf. | Mesure | Nature | Après |
|---|---|---|---|
| R-16 | Séparation des environnements, données de test | T | R-13 |
| R-21 | Contact de secours et reprise de l'hébergement ailleurs | E | R-17, R-19 |
| R-23 | Liste de contrôle de mise en production | O | - |
| R-25 | Analyse des dépendances logicielles | T | R-22 |
| R-28 | Procédure pour les droits des personnes | D | R-26 |
| R-29 | Clauses de sécurité et de données dans les contrats clients | D | R-26 |
| A-06 | **Chaîne de déploiement automatisée minimale (CI/CD)** | Action | R-24, A-04 |

Deux échéances ne suivent pas les vagues, car elles sont fixées par un fait extérieur :

| Action | Échéance | Origine |
|---|---|---|
| A-03 | Décision à J + 30 jours ; mise en œuvre (migration ou documentation) à J + 6 mois au plus tard | Transfert de données personnelles hors de l'Union européenne (RGPD, chapitre V) |
| A-05 | Plan rédigé en vague 2 ; bascule terminée avant février 2027, quelle que soit la vague | Fin de support d'Amplify Gen 1 annoncée en mai 2027 |

### 18.3 Échelle du déploiement : quels systèmes, à quelle étape

Le tableau indique, pour chaque système, les mesures mises en place à chaque vague. Les règles sont directes lorsque l'équipe peut les appliquer elle-même. Sur les serveurs externes, VNWeb n'a pas de droits d'administration : les règles sont demandées par écrit au prestataire (section 13), et une exigence refusée ou sans réponse à l'échéance donne lieu à une dérogation signée (R-03). Les règles R-06 (accès SSH), R-14 (correctifs) et R-18 (journaux), rangées avec les serveurs locaux, sont demandées aux prestataires par le cahier d'exigences (R-19).

| Système | Mode d'application | Vague 1 | Vague 2 | Vague 3 |
|---|---|---|---|---|
| Gouvernance et suivi | Directes | R-01, R-03, R-35, A-01 | R-02, R-36, R-45, R-46 | - |
| Formation et accessibilité | Directes | R-38, R-43 | R-39, R-40, R-41, R-42, R-44 | - |
| Comptes, arrivées et départs | Directes ; demandes aux prestataires pour leurs serveurs | R-04, R-07, R-31 | R-05, R-30, R-32 | - |
| Secrets, coffre-fort, Discord | Directes | R-08, A-02 | R-09 | - |
| Postes de travail, VPN et télétravail | Directes | R-10, R-11 | R-12 | - |
| Serveurs locaux (développement, Gitea, Extranet) | Directes | R-06, R-17 | R-13, R-14, R-15, R-18 | R-16 |
| Serveurs externes : serveur des projets et sites de voyance | Exigences écrites envers les prestataires (section 13) | R-19 | R-20 | R-21 |
| weevus et AWS | Directes | R-27, A-03 | A-05 | - |
| Développement : Gitea et CodeCommit | Directes | R-24 | R-22, A-04 | R-23, R-25, A-06 |
| Données personnelles et clients | Directes | - | R-26 | R-28, R-29 |
| Veille | Directes | - | R-37 | - |
| Locaux | Directes | - | R-33, R-34 | - |

### 18.4 Projets hébergés visés à chaque étape

Environ une centaine de projets sont hébergés chez les deux prestataires d'hébergement, et weevus sur AWS. Les règles qui portent sur chaque projet (registre des traitements, relecture, protection de branche, analyse des dépendances, tests) sont déployées par cercles successifs, en commençant par les projets qui portent le plus de risque.

| Vague | Projets visés | Mesures concernées |
|---|---|---|
| 1 | La production hébergée sur le serveur des projets dans son ensemble (confirmation des sauvegardes, journal de déploiement) et les données personnelles hébergées hors de l'Union européenne | R-19, R-24, A-03 |
| 2 | Les trois ensembles les plus sensibles : weevus, les sites de voyance, l'Extranet ; puis les projets qui traitent des données personnelles ou des authentifications | R-22, R-26, A-04 |
| 3 | Tous les projets actifs, soit environ une centaine (cible : 100 %) | R-22 (extension), R-23, R-25, R-26 (complet), R-28, R-29 |

La part exacte de projets couverts à chaque étape sera chiffrée à partir du registre des traitements (R-26), dès que celui-ci recense les projets.

### 18.5 Faire respecter les règles pendant le déploiement

Quatre mécanismes garantissent que les règles sont appliquées et pas seulement écrites :

- **La signature** : la PSSI entre en vigueur à la signature de la direction (A-01), qui l'engage.
- **L'accusé de lecture** : chaque membre de l'équipe atteste avoir lu la PSSI à son approbation (R-38), et chaque arrivant à son entrée (R-30).
- **La preuve** : chaque règle a une preuve attendue, indiquée dans la colonne « Preuve ou indicateur » de son tableau ; une règle n'est déclarée appliquée que lorsque sa preuve est archivée (R-35).
- **La dérogation** : une règle non appliquée à l'échéance de sa vague est reportée avec une nouvelle date, ou fait l'objet d'une fiche de dérogation signée par la direction (R-03).

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-35 | Le référent tient un tableau de déploiement qui liste chaque règle, sa vague, son niveau de suivi (1, 2 ou 3, voir 23.1), son état (non démarrée, en cours, appliquée avec preuve, reportée ou dérogation), l'emplacement de sa preuve et la date de son dernier contrôle. Une règle n'est déclarée appliquée que lorsque sa preuve est archivée. Tout écart constaté lors d'un contrôle y est consigné, puis corrigé dans le délai proposé selon la priorité de la règle (30 jours, 90 jours ou 6 mois à compter du constat), ou fait l'objet d'une dérogation signée (R-03) ; la correction est vérifiée et sa preuve archivée | VO-01, VO-03 | Référent | P1 | Tableau à jour ; part des règles de la vague échue appliquées avec preuve ; écarts ouverts au-delà de leur délai (cible : zéro) |

---

## 20. Vérification de la conformité, veille et alignement sur ISO 27001

Références : ISO 27001:2022 A.5.6, A.5.7, A.5.31, A.5.36 ; RGPD.

Cette section répond à trois questions : comment savoir que les règles sont réellement appliquées (20.1 à 20.3), comment garder la PSSI à jour quand les textes, les logiciels et les menaces évoluent (20.4, veille), et comment la PSSI se compare à la norme ISO 27001 (20.5).

### 20.1 Qui contrôle quoi

| Niveau | Qui | Ce qui est contrôlé | Cadence |
|---|---|---|---|
| 1. Auto-contrôle | Chaque développeur | Il applique la règle et alimente sa preuve au fil de l'eau (journal d'usage R-04, journal de déploiement R-24, liste de contrôle R-23) | En continu |
| 2. Contrôle du référent | Le référent | Les contrôles du calendrier (20.2) et l'archivage de leurs preuves | Selon le calendrier |
| 3. Contrôle croisé | Le suppléant contrôle le référent ; un second développeur est prévenu des actions d'administration à fort impact | Un échantillon des preuves du référent ; les actions d'administration avant leur application | Semestriel ; avant chaque action |
| 4. Revue de la direction | La direction | Le compte rendu de sécurité (R-02) et le tableau de déploiement (R-35) | Trimestriel |

Aucun contrôle indépendant de l'équipe n'est prévu à ce stade : dans une équipe de quatre développeurs, le contrôle croisé en tient lieu (principe PR-6). Le référent ne se contrôle donc pas lui-même : c'est le rôle du suppléant. Une règle est considérée comme appliquée lorsque sa preuve est archivée (R-35) ; la colonne « Preuve ou indicateur » de chaque règle dit laquelle.

### 20.2 Calendrier des contrôles

Le référent tient le calendrier suivant. Les fréquences sont celles fixées par les règles concernées ; ce sont des valeurs proposées.

| Contrôle | Fréquence | Responsable | Règle |
|---|---|---|---|
| Lecture des fichiers de veille des thèmes 1 et 2 | Hebdomadaire | Référent | R-37 |
| Point sécurité de l'équipe | Mensuel | Référent | R-41 |
| Relevé du niveau de mise à jour des serveurs locaux | Mensuel | Référent | R-14 |
| Condensé mensuel des menaces et test de couverture de la veille | Mensuel | Référent | R-37 |
| Compte rendu de sécurité à la direction | Trimestriel | Référent | R-02 |
| Relevé des treize indicateurs du tableau de bord | Trimestriel | Référent | R-02 |
| Revue des accès | Trimestriel | Référent | R-32 |
| Revue de l'inventaire des serveurs | Trimestriel | Référent | R-13 |
| Contrôle des bases de développement (données de test) | Trimestriel | Référent | R-16 |
| Analyse des dépendances des projets actifs | Trimestriel | Développeurs | R-25 |
| Vérification d'un échantillon de preuves du référent | Semestriel | Suppléant | R-36 |
| Test de restauration des sauvegardes locales | Semestriel | Référent | R-17 |
| Liste des comptes actifs demandée aux prestataires | Semestriel | Référent | R-19 |
| Relevé des ports ouverts | Annuel | Référent | R-15 |
| Relevé des ressources AWS (région, chiffrement, permissions) | Annuel | Référent | R-27 |
| Contrôle des protections des postes | Annuel | Référent, direction | R-12 |
| Attestation de l'appareil personnel utilisé en télétravail | Annuel | Développeurs | R-10 |
| Test de restauration chez les prestataires | Annuel | Référent, prestataires | R-19 |
| Test du contact d'alerte des prestataires | Annuel | Référent | R-19 |
| Test d'accès aux accès critiques par le suppléant | Annuel | Suppléant | R-08 |
| Contrôle physique sur place | Annuel | Référent | R-34 |
| Séance de sensibilisation de la direction | Annuel | Référent, direction | R-40 |
| Réexamen des dérogations | Au plus tard 12 mois après signature | Direction, référent | R-03 |
| Revue des référentiels (section 7) et de la PSSI | Annuel | Référent, direction | R-45 |

### 20.3 Traitement des écarts

Un écart est une règle non appliquée ou une preuve manquante, constatée lors d'un contrôle ou signalée par un développeur.

| Étape | Ce qui se passe | Trace |
|---|---|---|
| 1. Consigner | L'écart est inscrit dans le tableau de déploiement (R-35) avec la date du constat | Ligne du tableau |
| 2. Décider | Correction dans un délai fixé, ou dérogation signée par la direction (R-03, §8.3) | Date d'échéance ou fiche de dérogation |
| 3. Corriger | Le responsable applique la mesure | Preuve archivée |
| 4. Vérifier | Le référent vérifie la correction, ou le suppléant si le référent est concerné | Ligne du tableau clôturée |

Le délai de correction proposé dépend de la priorité de la règle : 30 jours pour une règle P1, 90 jours pour une règle P2, 6 mois pour une règle P3, à compter du constat. Une faille de sécurité critique suit le chemin court de 20.4.

### 20.4 Veille : garder la PSSI à jour

Cette partie décrit comment VNWeb suit ce qui évolue autour de la PSSI. La PSSI s'appuie sur des textes et des composants techniques qui évoluent : nouvelle version d'un guide, décision de la CNIL, faille dans un logiciel, campagne d'attaque, fin de support d'une version. Sans veille, une règle écrite en 2026 peut être dépassée en 2027 sans que rien ne le signale.

**Quatre thèmes, un filtre.** Un thème existe seulement s'il peut déclencher une action dans la PSSI (principe PR-6). Une information qui ne peut donner lieu à aucune action n'est pas surveillée.

| Thème | Ce qu'il surveille | Ce qu'il déclenche | Cadence de lecture (valeur proposée) |
|---|---|---|---|
| 1. Conformité | RGPD et CNIL, ANSSI, ISO 27001, NIS2 et DORA | Modification de règle ou dérogation (§8.3, R-03) | Hebdomadaire |
| 2. Failles de sécurité | Systèmes, frameworks, dépendances, services AWS | Correctif (R-14) | Immédiate si faille critique, sinon hebdomadaire |
| 3. Menaces et attaques | Hameçonnage, rançongiciels visant les PME et les agences web, fraude par usurpation | Contenu de la formation (section 21), point d'attention pour la gestion d'incident (section 17) | Mensuelle ; annuelle pour le *Panorama de la cybermenace* de l'ANSSI |
| 4. Fin de support et fournisseurs | Versions qui perdent leurs correctifs ; incidents et changements chez AWS, Discord, Gitea et les hébergeurs | Migration planifiée (R-13 et A-05), revue d'une exigence envers un prestataire (section 13) | Mensuelle |

**Trois temps séparés : collecter, trier, décider.**

- La **collecte** est mécanique : un lecteur de flux RSS (fil de nouveautés publié par un site), sans intelligence artificielle, relève les nouveautés d'une liste fermée de sources officielles.
- Le **tri et le résumé** sont faits par une intelligence artificielle, à partir des seuls éléments du flux : elle ne cherche rien d'elle-même.
- La **décision** est humaine : le référent lit, qualifie et propose ; la direction décide selon le chemin de 8.3. Aucune règle n'est modifiée parce qu'un fichier de veille le suggère.

**Fiabilité.** Un fichier de veille est un signal, pas une preuve : tout élément retenu est vérifié dans le texte officiel avant qu'une décision s'y appuie. Les risques d'une intelligence artificielle (oubli, erreur de date, source secondaire) et d'une tâche qui s'arrête sans bruit sont traités par une liste blanche de sources, un lien et une date obligatoires pour chaque élément, un contrôle de complétude, un test mensuel de couverture et un journal des ratés. La procédure ne garantit pas qu'aucune évolution n'échappe à la surveillance : ce qui n'est pas dans la liste des sources n'est pas vu, et le choix de ces sources est le point sensible du dispositif.

**Traitement.** Chaque élément est classé.

| Niveau | Définition | Traitement |
|---|---|---|
| N0, sans objet | Hors du périmètre de VNWeb ou déjà connu | Aucune action |
| N1, à surveiller | Texte en projet, sans effet immédiat | Inscrit au registre de veille avec une date de réexamen |
| N2, impact possible sur la PSSI | Nouvelle obligation, nouvelle version d'un référentiel, faille touchant un composant inventorié, fin de support touchant un élément inventorié | Inscrit au registre, vérifié à la source, proposé à la direction sous 30 jours (valeur proposée) |
| Urgent | Échéance légale proche, règle devenue contraire à la loi, faille critique déjà exploitée touchant un composant utilisé | Le référent prévient la direction le jour même |

**Le chemin court pour une faille critique.** Une faille critique qui touche un composant de l'inventaire ne passe pas par la proposition écrite : on vérifie à la source qu'elle existe et si elle est exploitée, on la rapproche de l'inventaire, on prévient le même jour la direction, le développeur du projet et, pour un serveur externe, le prestataire, puis on corrige dans les délais de la règle R-14 et on consigne les dates au registre. Si une exploitation sur les systèmes de VNWeb est suspectée, la gestion d'incident s'ouvre (section 17).

**Rôles.** Le référent pilote la veille et répond devant la direction ; le suppléant le remplace, avec les mêmes accès (PR-5). Les développeurs surveillent et corrigent les composants de leurs propres projets. Toute l'équipe reçoit le condensé mensuel des menaces : cinq minutes de lecture. La charge de temps du référent est estimée à environ une heure par semaine (valeur proposée), à faire valider par la direction (R-46).

**Indicateurs de la veille** (valeurs proposées) :

| Indicateur | Cible |
|---|---|
| Fichiers de veille produits | Pas d'absence de plus de 3 jours consécutifs |
| Semaines de lecture inscrites au registre | Toutes |
| Éléments N2 ayant reçu une décision sous 30 jours | Tous |
| Délai de correction d'une faille critique | 72 heures si exploitée, 7 jours sinon |
| Âge de la dernière vérification de chaque référentiel de la section 7 | Moins de 12 mois |

**État du dispositif.** La collecte du thème 1 est déjà automatisée par des tâches quotidiennes (déclaré). Les thèmes 2 à 4 et le lecteur de flux sont une cible dont la mise en place n'est pas confirmée. Restent à établir : la liste des sources officielles et l'existence de leurs flux, les technologies et versions utilisées par les projets, le choix de l'analyseur de dépendances, la maintenance du dispositif de collecte (qui la reprend si son auteur n'est pas disponible), le seuil à partir duquel une faille est dite « critique », et la validation par la direction de la charge de temps du référent (R-46).

### 20.5 Alignement sur ISO 27001:2022

ISO 27001 est une grille de comparaison, sans visée de certification (section 7). Chaque section de règles cite en tête les contrôles de l'Annexe A dont elle s'inspire. Le tableau rapproche ces contrôles des règles qui en traitent l'objectif ; il n'affirme pas la conformité à la norme. La matrice détaillée (vulnérabilité, règle, contrôle) figure en annexe A. Les numéros et intitulés sont donnés d'après l'édition 2022 de la norme, dont le texte est payant et n'a pas été consulté pour cette rédaction [à vérifier].

| Contrôle | Objet | Règles de la PSSI |
|---|---|---|
| A.5.1 | Politiques de sécurité de l'information | R-35, R-38, A-01 |
| A.5.2 | Fonctions et responsabilités | R-01 |
| A.5.3 | Séparation des tâches | R-05 et R-36 |
| A.5.4 | Responsabilités de la direction | R-02, R-03, R-38, R-40, R-46 |
| A.5.6, A.5.7 | Contacts spécialisés, renseignement sur les menaces | R-14, R-37, R-41 |
| A.5.9 | Inventaire des actifs | R-13 et R-30 |
| A.5.11 | Restitution des actifs | R-31 et R-33 |
| A.5.15 à A.5.18 | Contrôle d'accès, identités, informations d'authentification, droits d'accès | R-04, R-05, R-08, R-30 à R-32, A-02 |
| A.5.19, A.5.20 | Fournisseurs : relations et accords | R-19 à R-21 |
| A.5.31, A.5.34 | Exigences légales, protection des données personnelles | R-26 à R-29 et R-37 |
| A.5.36 | Conformité aux politiques et aux règles | R-35 et R-36 |
| A.6.3 | Sensibilisation et formation | R-30 et R-38 à R-41 |
| A.6.5 | Responsabilités après la fin d'un emploi | R-31 |
| A.6.7 | Travail à distance | R-04, R-07, R-10, R-11 |
| A.7.2, A.7.3 | Accès physiques, sécurisation des locaux | R-33 et R-34 |
| A.8.1 | Terminaux des utilisateurs | R-10 à R-12 |
| A.8.2, A.8.5 | Droits privilégiés, authentification sécurisée | R-05 à R-07 |
| A.8.8 | Gestion des vulnérabilités techniques | R-13, R-14, R-25 |
| A.8.13 | Sauvegarde | R-17 et R-19 |
| A.8.15, A.8.16 | Journalisation, surveillance | R-18 |
| A.8.25, A.8.29, A.8.32 | Développement sécurisé, tests, gestion des changements | R-22 à R-24, R-36, A-06 |
| A.8.31, A.8.33 | Séparation des environnements, données de test | R-16 |

Certains contrôles de l'Annexe A ne sont pas ou pas entièrement traités à ce stade. Ils sont réexaminés à chaque revue de la PSSI :

| Contrôle | Situation |
|---|---|
| A.5.12 Classification de l'information | Non traité |
| A.5.14 Transfert de l'information | Partiel : interdiction des secrets dans les canaux de discussion (R-08), vérification par second canal (R-09) |
| A.5.23 Cloud (AWS) | Partiel : R-07, R-27, A-05 |
| A.5.24 à A.5.28 Gestion des incidents | Section 17 |
| A.5.29, A.5.30 Sécurité pendant une perturbation, continuité | Partiel : plan de reprise (R-17) |
| A.8.10 Suppression de l'information | Partiel : durées de conservation (R-26) |
| A.8.12 Prévention des fuites de données | Non traité |
| A.8.20, A.8.22 Sécurité et cloisonnement des réseaux | Partiel : pare-feu (R-15), séparation des environnements (R-16) ; chiffrement des flux (TLS) à vérifier |
| A.8.24 Cryptographie | Partiel : chiffrement des disques (R-12) ; chiffrement au repos à vérifier |
| A.8.28 Codage sécurisé | Partiel : relecture de code (R-22) |

### 20.6 Règles

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-36 | Le référent réalise les contrôles du calendrier (20.2) et archive la preuve de chacun. Chaque semestre (valeur proposée), le suppléant vérifie un échantillon de ces preuves et consigne le résultat. Les actions d'administration à fort impact (attribution ou retrait de droits, règles de pare-feu, suppression de données ou de comptes) sont annoncées à un second développeur avant d'être appliquées et inscrites dans un journal d'administration (qui, quoi, quand) | VO-01, VO-06 | Référent, suppléant, développeurs | P2 | Preuves archivées par contrôle ; deux comptes rendus de vérification du suppléant par an ; journal d'administration tenu |
| R-37 | Le référent applique la veille décrite en 20.4 : collecte des sources officielles de la liste blanche, lecture hebdomadaire des thèmes 1 et 2, condensé mensuel des menaces, tenue du registre de veille. Aucune règle de la PSSI n'est modifiée sans passer par le chemin de 8.3 | VT-27, VO-10, VO-17 | Référent (suppléant en son absence) | P2 | Registre de veille à jour ; part des semaines lues et inscrites (cible : toutes) |

---

## 21. Formation et accompagnement de l'équipe

Références : ANSSI, *Guide d'hygiène informatique* ; ISO 27001:2022 A.6.3.

Une politique n'est appliquée que si l'équipe comprend pourquoi elle existe et sait la mettre en pratique. Comme le retient l'analyse EBIOS de J1, la sensibilisation prend une forme légère et récurrente plutôt que celle d'une formation isolée. Le dispositif comprend une session de lancement, six modules courts calés sur les vagues de déploiement (section 18), un quiz à la fin de chaque module, des rappels réguliers et l'accompagnement des arrivants. L'accessibilité de la formation est traitée en section 22.

### 21.1 Publics

| Public | Besoin | Format |
|---|---|---|
| Développeurs | Appliquer les règles techniques et organisationnelles sur leurs propres outils | Lancement, six modules avec quiz, point mensuel |
| Direction | Décider en connaissance de cause, reconnaître la fraude par usurpation, protéger ses comptes et son poste (VO-04, VO-05) | Ouverture du lancement, séance dédiée d'une heure, sans jargon, centrée sur les décisions à prendre (note de décision) |
| Arrivants | Connaître les règles dès l'entrée | Parcours d'accueil (R-30) |
| Prestataires d'hébergement | Connaître les exigences qui les concernent | Cahier d'exigences écrit (R-19), sans formation |

### 21.2 Programme

Chaque module a lieu avant l'application des règles qu'il présente : l'équipe apprend ce qu'elle va mettre en place. J désigne la date d'approbation de la PSSI ; les échéances et les durées sont des valeurs proposées.

| Module | Contenu | Sections de la PSSI | Règles | Échéance | Durée |
|---|---|---|---|---|---|
| M0 Lancement | Les dangers et les risques, vus par des cas vécus ; présentation de la PSSI ; engagement de la direction ; accusé de lecture | 6, 8 | R-38 et A-01 | J | Une demi-journée (environ 4 heures) |
| M1 Accès, secrets, arrivées et départs | Comptes nominatifs, coffre-fort, interdiction des secrets dans Discord, retrait des accès au départ | 9, 10, 16.1, 16.2 | R-04 à R-09, R-30 à R-32, A-02 | J + 10 jours | 45 minutes |
| M2 Télétravail, postes et locaux | Télétravail exceptionnel et déclaré, appareil personnel, VPN, protections des postes, clés et visiteurs | 11, 16.3 | R-04, R-07, R-10 à R-12, R-33, R-34 | J + 20 jours | 45 minutes |
| M3 Serveurs, sauvegardes et prestataires | Mises à jour, sauvegardes, journaux, exigences envers les prestataires | 12, 13 | R-13 à R-21 | J + 45 jours | 45 minutes |
| M4 Développement et livraison | Relecture, tests, dépendances, déploiement tracé | 14 | R-22 à R-25, A-04, A-06 | J + 60 jours | 45 minutes |
| M5 Données personnelles et RGPD | Registre des traitements, localisation, conservation, droits des personnes | 15 | R-26 à R-29 et A-05 | J + 75 jours | 45 minutes |
| M6 Incident : exercice | Reconnaître, signaler et traiter un incident | 17 | Règles de la section 17 | J + 90 jours | 45 minutes |

Le total est d'environ 8 heures 30 pour chaque développeur, réparties sur trois mois, auxquelles s'ajoute une heure pour la direction (R-40).

### 21.3 Contenu des modules

Chaque module part d'un cas vécu chez VNWeb, présenté sans nom de personne, et fait pratiquer l'équipe sur ses outils réels. La pratique est aussi une étape du déploiement : former et déployer se font en même temps.

| Module | Cas vécu | Pratique sur les outils |
|---|---|---|
| M0 | Mots de passe partagés dans Discord (déclaré) ; accès d'un ancien collaborateur réutilisés (déclaré) ; connexion SSH possible avec un mot de passe depuis n'importe où (déclaré, essai à faire) ; aucune sauvegarde planifiée visible sur le serveur des projets (constaté le 17/09/26) | Chacun liste les accès qu'il détient, ce qui alimente le registre des accès (R-30) |
| M1 | Les accès d'un ancien collaborateur réutilisés au départ | Installer le coffre-fort et y placer ses mots de passe (R-08) ; activer la double authentification sur AWS (R-07) ; générer une clé SSH (R-06) |
| M2 | Connexion au VPN par mot de passe seul, depuis un appareil personnel | Tester que le Bureau à distance n'est pas joignable hors VPN (R-11) ; déclarer un télétravail (R-10) ; vérifier le verrouillage et les mises à jour d'un poste |
| M3 | Comptes partagés par projet sur le serveur des projets : impossible de savoir qui a fait quoi | Relever l'état d'un serveur local (mises à jour, ports ouverts) ; lire un journal de connexion ; rédiger la demande d'état des lieux à un prestataire (R-19) |
| M4 | Mise en production sans relecture technique | Relire une modification ; chercher un secret dans un dépôt ; remplir la liste de contrôle de mise en production (R-23) |
| M5 | Données personnelles hébergées hors de l'Union européenne | Remplir une ligne du registre des traitements pour un projet réel (R-26) |
| M6 | Un mot de passe déjà exposé dans un salon de discussion | Exercice : un secret a fuité ; qui prévenir, quoi renouveler, quoi consigner, dans quel délai informer un prestataire ou un client |

### 21.4 Format d'un module et quiz

| Temps | Durée | Contenu |
|---|---|---|
| L'enjeu | 5 minutes | Pourquoi ce sujet compte, à partir du cas vécu |
| La règle en clair | 10 minutes | Ce qu'elle impose, ce qu'elle interdit, ce qu'elle change pour chacun |
| La pratique | 20 minutes | Manipulation sur les outils réels |
| Le quiz | 10 minutes | Cinq mises en situation |

Un aide-mémoire d'une page est remis avant chaque module. Le quiz vérifie la compréhension et non la mémoire : il pose des situations plutôt que des définitions. Il ne sert pas à sanctionner : en cas d'échec, la correction est commentée et le quiz peut être repris. Exemples de situations :

- *Module 1 :* un collègue demande sur Discord le mot de passe d'un serveur pour dépanner un client. Que fait-on ? (Refuser l'envoi par ce canal, passer par le coffre-fort, confirmer la demande par un second canal : R-08 et R-09.)
- *Module 2 :* une urgence oblige à travailler de chez soi. Que faut-il faire avant de se connecter, et à quelles conditions l'appareil personnel est-il accepté ? (Déclarer au référent la date et le motif ; système à jour, antivirus actif, session non partagée, réseau maîtrisé : R-10.)
- *Module 5 :* un nouveau projet collecte des données de visiteurs. Que faut-il renseigner avant la mise en production ? (Une ligne du registre des traitements : nature des données, finalité, lieu d'hébergement, durée de conservation, rôle de VNWeb : R-26.)

### 21.5 Accompagnement et rappels

- **Le référent** est le premier contact pour toute question sur la PSSI. Un salon de discussion peut servir aux questions, mais pas aux secrets (R-08).
- **Le point sécurité mensuel** (R-41) rappelle une règle à la fois. Il s'appuie sur le condensé mensuel des menaces produit par la veille (section 20.4, thème 3), que le référent envoie à l'équipe : cinq minutes de lecture.
- **L'exercice d'incident** de M6 est répété chaque année (section 17).
- **La charte informatique** (R-42) reprend en deux pages les règles d'usage quotidien, et se signe à l'arrivée.
- **Les arrivants** suivent un parcours d'accueil dans leur première semaine (R-30).

### 21.6 Règles

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-38 | À l'approbation de la PSSI (J), la direction ouvre une session de lancement d'environ une demi-journée pour toute l'équipe : dangers et risques, présentation de la PSSI, accusé de lecture signé par chaque membre de l'équipe, direction comprise. Le module 1 (accès, secrets, arrivées et départs) se tient avant J + 30 jours | VO-01, VO-03, VO-10 | Direction (ouvre), référent (anime) | P1 | Feuille de présence ; accusés de lecture signés (toute l'équipe) ; module 1 tenu avant J + 30 jours |
| R-39 | Les modules 2 à 6 se tiennent aux échéances du programme, avant l'application de leurs règles. Chaque module suit le format en quatre temps (enjeu, règle en clair, pratique, quiz), s'accompagne d'un aide-mémoire d'une page remis à l'équipe et se termine par un quiz de mises en situation : 5 questions, seuil de réussite de 80 %, reprise possible avec correction commentée (valeurs proposées). Les résultats individuels ne sont pas diffusés ; le référent tient un bilan agrégé par module, qui sert à améliorer les modules | VO-10 | Référent | P2 | Supports et aides-mémoire archivés ; part des modules tenus à l'échéance ; bilan des quiz : taux de réussite par module |
| R-40 | La direction suit, dans les 90 jours suivant l'approbation puis chaque année (valeur proposée), une séance d'une heure animée par le référent : fraude par usurpation, protection de ses comptes et de son poste, décisions à prendre. Le contenu est actualisé avec le condensé mensuel des menaces | VO-04, VO-05 | Référent (anime), direction (participe) | P2 | Séance tenue et consignée, contenu daté |
| R-41 | Chaque mois (valeur proposée), le référent anime un point sécurité de 15 minutes : incidents et anomalies, indicateurs, dérogations en cours, et un rappel ciblé d'une règle, choisi à partir du condensé mensuel des menaces et des écarts constatés (par exemple la vérification par second canal, R-09). La direction est invitée | VO-05, VO-10 | Référent | P2 | Compte rendu d'une ligne par réunion ; nombre de points tenus sur 12 mois (cible : 12) |
| R-42 | Une charte informatique de deux pages au plus est rédigée à partir de la PSSI pour l'usage quotidien de chaque personne : postes, VPN, télétravail, Discord, coffre-fort, visiteurs. Elle est signée à l'arrivée et à chaque mise à jour | VO-11, VO-12 | Référent (rédige), direction (approuve) | P2 | Charte signée par toute l'équipe |

---

## 22. Accessibilité de la formation

Références : Code du travail, article L5213-6 ; RGPD, article 9 ; ISO 27001:2022 A.6.3.

La formation de la section 21 s'adresse à toute l'équipe, direction comprise. Elle doit pouvoir être suivie par chacun, quelle que soit sa situation : un besoin durable ou passager, connu ou non de l'entreprise. Cette section fixe comment les supports et les modalités de formation sont conçus pour être accessibles, comment une adaptation est demandée et accordée, et comment la confidentialité de la personne est protégée.

### 22.1 Cadre et principes

**Ce que dit le droit.**

- **L'employeur** : l'article L5213-6 du Code du travail impose à l'employeur de prendre, pour les salariés concernés, les mesures appropriées, notamment pour qu'une formation adaptée à leurs besoins leur soit dispensée. Ces mesures sont dues sous réserve que les charges qui en résultent ne soient pas disproportionnées. Le refus d'en prendre peut constituer une discrimination. La PSSI retient le même principe pour tout besoin d'adaptation, durable ou passager.
- **Les informations de santé** : elles appartiennent aux catégories particulières de données personnelles, dont le traitement est strictement encadré (RGPD, article 9). VNWeb en recueille le moins possible (22.4).
- **Le financement** : l'Agefiph propose une aide à l'adaptation des situations de travail, qui finance la différence de coût entre un équipement standard et un équipement adapté. L'employeur en fait la demande, et la recommandation du médecin du travail est un préalable. Le montant est fixé après étude. Cette aide est citée à titre d'information : son application à un besoin précis est à vérifier auprès de l'organisme.
- **Les supports** : les critères internationaux d'accessibilité des contenus numériques (WCAG), que le référentiel français RGAA reprend, servent de référence de conception. Les obligations légales de ce référentiel ne visent pas VNWeb pour ses supports internes [à vérifier].

**Principes retenus.**

- **Accessible par conception** : les supports et les sessions sont conçus pour être accessibles sans qu'aucune demande soit nécessaire (22.2).
- **Adaptation sur demande, au cas par cas** : une adaptation répond à un besoin concret exprimé par la personne (22.3).
- **Pas de justification médicale** : personne n'a à détailler sa situation de santé, ni devant l'équipe, ni devant le référent, ni devant la direction au-delà de ce qui est nécessaire à l'adaptation.
- **Besoin non établi à ce jour** : aucun besoin d'adaptation n'est connu de l'équipe technique (à confirmer avec la direction). Le dispositif est prévu à l'avance pour qu'un besoin futur trouve une réponse déjà prête.

### 22.2 Accessibilité par défaut

| Élément | Mesures prévues pour tous |
|---|---|
| Supports (aides-mémoire, diaporamas) | Titres structurés, police lisible et de taille suffisante, contrastes suffisants, aucune information portée par la seule couleur, texte alternatif pour les images et schémas, format modifiable (pas de texte enregistré sous forme d'image), remise avant la session |
| Sessions | Durée courte (45 minutes par module) avec pause si besoin ; salle calme ; possibilité de suivre la session à distance sur demande |
| Pratique sur les outils | Tâches réalisables au clavier ou avec les outils d'assistance du poste ; réglages d'affichage (taille, contraste) laissés à la main de chacun |
| Quiz | Questions écrites lisibles ; reformulation à l'oral possible ; pas de limite de temps stricte |

### 22.3 Adaptations possibles sur demande

Le tableau donne des exemples : les mesures sont choisies avec la personne, selon son besoin réel, et ne sont pas liées à une cause précise.

| Besoin exprimé | Adaptations possibles |
|---|---|
| Difficulté à lire à l'écran | Supports en grande taille et très contrastés, compatibles avec un lecteur d'écran ; session sans diaporama projeté |
| Difficulté à entendre | Supports écrits complets ; transcription de ce qui est dit ; salle calme ; matériel adapté si nécessaire |
| Difficulté de manipulation ou de mobilité | Pratique au clavier ; temps supplémentaire ; binôme pour les manipulations ; accès à la salle vérifié |
| Difficulté de concentration, de lecture ou d'écriture | Supports courts et structurés, police adaptée, session fractionnée, temps supplémentaire pour le quiz, réponses à l'oral |
| Situation passagère (blessure, fatigue, traitement) | Report ou fractionnement de la session, suivi à distance, rattrapage individuel |

### 22.4 Demander une adaptation

| Étape | Ce qui se passe |
|---|---|
| 1. Demande | La personne s'adresse à la direction, à l'oral ou par écrit, à tout moment, sans avoir à détailler sa situation de santé |
| 2. Réponse | La direction répond par écrit sous 10 jours ouvrés (valeur proposée) : adaptation proposée, ou refus motivé |
| 3. Mise en œuvre | Le référent adapte les modalités pratiques (format des supports, durée, aide) avant la session concernée |
| 4. Retour | Après la première session suivie, la personne indique si l'adaptation convient ; elle est ajustée si besoin |

**Confidentialité.** La direction est seule destinataire de la demande. Elle transmet au référent uniquement ce qui est nécessaire pour adapter la formation (format, durée, aide), sans la cause. Les éventuels éléments de santé, par exemple une recommandation du médecin du travail, sont conservés par la direction dans un dossier à accès restreint, sans être transmis au référent ni à l'équipe, et ne sont gardés que le temps nécessaire. Ils n'apparaissent dans aucun tableau ni registre de la PSSI.

**Coût.** L'adaptation est prise en charge par la direction, qui examine les aides disponibles (22.1). Un refus n'est possible que si les charges sont disproportionnées ; il est motivé par écrit, car le refus de mesures appropriées peut constituer une discrimination.

### 22.5 Règles

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-43 | Chaque support et chaque module de formation (section 21) est conçu pour être accessible par défaut, selon les mesures de 22.2 : structure de titres, police lisible, contrastes, texte alternatif, format modifiable, remise à l'avance, pratique réalisable au clavier, quiz sans limite de temps stricte | VO-10 | Référent | P1 | Liste de contrôle d'accessibilité renseignée pour chaque support |
| R-44 | Toute personne peut demander une adaptation de la formation à tout moment, sans détailler sa situation de santé. La demande s'adresse à la direction, qui répond par écrit sous 10 jours ouvrés (valeur proposée), en proposant une adaptation ou en motivant un refus fondé sur des charges disproportionnées ; le coût est assumé par la direction, qui examine les aides disponibles. Cette possibilité est présentée à toute l'équipe au lancement et à chaque arrivant lors du parcours d'accueil. Les informations reçues sont limitées au strict nécessaire : la direction seule conserve les éventuels éléments de santé, dans un dossier à accès restreint, et ne transmet au référent que les modalités pratiques de l'adaptation, sans la cause | VO-10 | Direction, référent | P2 | Délai de réponse ; demandes traitées et refus motivés ; dossier à accès restreint ; aucune information de santé dans les registres et tableaux de la PSSI |

---

## 23. Suivi de la politique : indicateurs, revue et révision annuelle

Références : ISO 27001:2022 A.5.1, A.5.36.

Suivre chaque règle une à une n'est pas tenable pour une équipe de quatre développeurs, et une règle qui ne peut pas être contrôlée n'est pas retenue (principe PR-6). Le suivi porte donc sur l'essentiel : un tableau de bord court, un contrôle proportionné à l'importance de chaque règle, et une revue annuelle qui retire ou fusionne ce qui ne sert pas.

### 23.1 Trois niveaux de suivi

| Niveau | Règles concernées | Comment elles sont suivies |
|---|---|---|
| 1. Règles clés | 13 règles : R-06 à R-08, R-11, R-14, R-17 à R-19, R-22, R-24, R-27, R-31, R-32 | Un indicateur relevé chaque trimestre (23.2). Ce sont les règles qui ferment les écarts confirmés ou les plus graves, et dont l'absence ferait le plus de dégâts |
| 2. Règles à contrôle périodique | 14 règles : R-02, R-03, R-10, R-12, R-13, R-15, R-16, R-25, R-34, R-36, R-37, R-40, R-41, R-45 | Un contrôle prévu au calendrier (20.2), mensuel, trimestriel, semestriel ou annuel |
| 3. Règles de mise en place | 19 règles : R-01, R-04, R-05, R-09, R-20, R-21, R-23, R-26, R-28 à R-30, R-33, R-35, R-38, R-39, R-42 à R-44, R-46 | Vérifiées une fois, à la fin de leur vague, avec leur preuve archivée (R-35), puis relues à la revue annuelle. Un écart se voit par un signalement, un incident ou la revue |

Un contrôle ne démarre qu'une fois sa règle mise en place : de J à J + 90 jours, le suivi porte surtout sur les règles clés et sur la mise en place ; les contrôles périodiques du niveau 2 démarrent progressivement à partir de J + 90 jours. Le niveau de chaque règle est inscrit dans le tableau de déploiement (R-35), qui sert aussi de tableau de suivi : il porte deux colonnes de plus, « niveau de suivi » et « date du dernier contrôle ».

### 23.2 Le tableau de bord : treize indicateurs

Chaque objectif de la section 6.2 est mesuré par un à trois indicateurs. La situation de départ vient de la partie 1 ; les cibles sont des valeurs proposées, à valider par la direction. Chaque relevé s'appuie sur une donnée qui existe déjà (configuration, journal, registre) : il demande quelques minutes, non une enquête.

| Objectif | Indicateur | Situation de départ | Cible | Comment le relever |
|---|---|---|---|---|
| OBJ-1 | Part des comptes ayant un titulaire identifié | Serveur des projets : aucun compte individuel sur environ une centaine (J2 #5) | 100 % | Comparer le registre des accès aux comptes réels (R-32) |
| OBJ-1 | Délai entre un départ et le retrait des accès | Accès réutilisés lors du dernier départ (déclaré) | Le jour même | Dates de départ et de retrait de la liste de contrôle de départ (R-31) |
| OBJ-1 | Part des accès conformes : SSH sans connexion root ni mot de passe seul, double authentification sur le VPN et sur AWS | Connexion root autorisée sur le serveur des projets (J2 #1) ; SSH par mot de passe et VPN par mot de passe seul (déclarés) | 100 %, ou dérogation signée | Configuration SSH effective de chaque serveur, réglages du VPN et d'AWS (R-06 et R-07) |
| OBJ-2 | Nombre de secrets trouvés hors du coffre-fort | Secrets partagés dans Discord, non dénombrés (déclaré) | 0 | Recherche dans l'historique des salons et dans les dépôts (R-08) |
| OBJ-2 | Avancement de la décision sur les données personnelles hébergées hors de l'Union européenne | Aucune décision | Décision à J + 30 jours, mise en œuvre à J + 6 mois | Décision écrite, état de la migration ou de la justification (A-03, R-27) |
| OBJ-3 | Ancienneté de la dernière sauvegarde réussie des données critiques locales | À établir | Moins de 24 heures | Date de la dernière sauvegarde (R-17) |
| OBJ-3 | Tests de restauration réalisés sur les 12 derniers mois | Non connus ; aucune sauvegarde planifiée visible sur le serveur des projets (J2 #7) | 2 par sauvegarde locale, 1 par prestataire | Comptes rendus de test (R-17 et R-19) |
| OBJ-4 | Part des livraisons tracées : fusion relue et déploiement inscrit au journal | Aucune relecture connue, aucun journal (déclaré) | 100 % | Demandes de fusion approuvées dans Gitea et CodeCommit, journal de déploiement (R-22 et R-24) |
| OBJ-5 | Durée de conservation des journaux de connexion | Environ 4 semaines sur le serveur des projets (J2 #8) | 12 mois | Configuration de la conservation (R-18) |
| OBJ-5 | Délai de correction d'une faille critique | Sans objet à ce jour | 72 heures si exploitée, 7 jours sinon | Dates de publication et de correction au registre de veille (R-14) |
| OBJ-5 | Part des incidents consignés avec un retour d'expérience | Aucune procédure connue | 100 % | Fiches d'incident (section 17) |
| OBJ-6 | Part des règles de la vague échue appliquées avec preuve | À établir à l'approbation | 100 %, ou report daté ou dérogation | Tableau de déploiement (R-35) |
| OBJ-6 | Décisions de la direction tracées : propositions ayant reçu une réponse écrite sous 30 jours, dérogations arrivées à échéance sans réexamen | Une proposition de sécurisation non retenue (déclaré) | Toutes les propositions ; aucune dérogation en retard | Registre des dérogations (R-03) |

### 23.3 Le compte rendu trimestriel

Le référent remet chaque trimestre à la direction un compte rendu d'une page (R-02), qui reprend le tableau de bord et ce qui demande une décision.

| Rubrique | Contenu |
|---|---|
| Indicateurs | Les treize indicateurs, avec leur évolution depuis le trimestre précédent |
| Écarts | Écarts ouverts et leur délai (R-35) |
| Dérogations | Dérogations arrivant à échéance |
| Veille | Éléments importants repérés, en cours, décidés (section 20.4) |
| Décisions demandées | Ce que la direction doit trancher ce trimestre |

### 23.4 Combien de temps cela prend

Le tableau donne une estimation du temps en régime établi, une fois les règles mises en place. Il ne comprend pas le travail de déploiement des vagues (section 18). Ce sont des valeurs proposées, à mesurer sur les trois premiers mois (R-46).

| Cadence | Ce que cela comprend | Temps par fois | Fois par an | Total annuel |
|---|---|---|---|---|
| Hebdomadaire | Lecture des fichiers de veille (R-37) | 1 h | 50 | 50 h |
| Mensuelle | Point sécurité avec sa préparation (R-41), relevé des mises à jour (R-14), condensé des menaces (R-37) | 2 h | 12 | 24 h |
| Trimestrielle | Relevé des indicateurs et compte rendu (R-02), revue des accès (R-32), inventaire (R-13), bases de développement (R-16), analyse des dépendances par les développeurs (R-25) | 6 h | 4 | 24 h |
| Semestrielle | Test de restauration locale (R-17), liste des comptes des prestataires (R-19), échantillon de preuves par le suppléant (R-36) | 4 h | 2 | 8 h |
| Annuelle | Relevés des ports, des ressources AWS, des protections des postes, attestations, tests chez les prestataires, accès critiques, contrôle physique, séance de la direction, réexamen des dérogations, revue de la PSSI (R-45) | 14 h | 1 | 14 h |
| **Total** | | | | **environ 120 h, soit 10 h par mois, dont 4 h de veille** |

Cette charge reste importante pour une équipe de quatre. Quatre leviers la réduisent :

- **Partager** : le référent et son suppléant se répartissent les contrôles, ce qui applique aussi le contrôle croisé (R-36).
- **Automatiser** : un petit script lancé chaque mois relève les éléments mesurables (niveau de mise à jour, configuration SSH, date de la dernière sauvegarde, ports ouverts, recherche de secrets) et produit une page de relevé, que le référent lit au lieu de tout vérifier à la main.
- **Regrouper** : une revue de sécurité trimestrielle de deux heures réunit les contrôles dus ce trimestre-là.
- **Étaler** : la première année, seuls les niveaux 1 et 3 sont suivis pendant les 90 premiers jours (23.1).

La direction alloue le temps nécessaire (R-46). Si la charge réelle dépasse ce temps, on ne renonce pas à un contrôle par oubli : le niveau de suivi de certaines règles est abaissé par une décision écrite (R-46).

### 23.5 Revue et révision annuelle

La PSSI est revue au moins une fois par an, et après tout incident majeur, changement important (nouveau serveur, nouveau prestataire, migration, changement de référent) ou évolution réglementaire. La revue examine :

1. les indicateurs et leur évolution sur l'année ;
2. les écarts, et les dérogations à réexaminer (R-03) ;
3. la veille : nouvelles versions des référentiels de la section 7, éléments de niveau N2 (section 20.4) ;
4. les changements survenus : serveurs, prestataires, projets, personnes ;
5. les règles à retirer ou à fusionner, faute d'utilité ou parce qu'elles ne peuvent pas être contrôlées (principe PR-6) ;
6. la partie 1 : les vulnérabilités « à vérifier » ou « déclarées » sont confirmées ou levées, et la section « points restant à vérifier » est remise à jour.

Chaque revue produit une nouvelle version, numérotée et approuvée par la direction, dont l'équipe est informée.

| Version | Date | Nature du changement | Approbation |
|---|---|---|---|
| 0.1 | 21/09/2026 | Première rédaction, soumise à la validation de la direction | En attente |

### 23.6 Règles

| ID | Règle | Vulnérabilités traitées | Responsable | Priorité | Preuve ou indicateur |
|---|---|---|---|---|---|
| R-45 | La PSSI est revue au moins une fois par an et après tout incident majeur, changement important ou évolution réglementaire, selon les points de 23.5. La nouvelle version est numérotée, approuvée par la direction, et l'équipe est informée des changements | VO-01, VO-03 | Référent (prépare), direction (approuve) | P2 | Journal des versions ; date de la dernière revue inférieure à 12 mois |
| R-46 | La direction alloue au référent et à son suppléant le temps nécessaire aux contrôles, au suivi et à la veille (valeur proposée : environ 10 heures par mois en régime établi, dont 4 pour la veille, voir 23.4). Le référent relève le temps réellement passé pendant les trois premiers mois ; la direction compare ce temps à celui qu'elle alloue et décide par écrit d'ajuster le temps alloué ou le niveau de suivi de certaines règles | VO-01, VO-03, VO-06 | Direction, référent | P2 | Décision écrite ; relevé de temps sur trois mois |

---

## Sections à venir

Chaque section sera rédigée à son tour, dans cet ordre, et validée avant de passer à la suivante. Les tableaux de règles (R-xx) courent de la section 8 à la section 23 ; la section 24 regroupe les annexes et ne contient pas de règle.

« Point n » renvoie aux huit attendus du TP : 1 définition de la politique, 2 déploiement, 3 veille, 4 vérification de la conformité (ISO 27001), 5 outils de veille, 6 formation et accompagnement, 7 accessibilité de la formation, 8 suivi de la politique. Le point 1 est traité par les sections 6 à 17.

17. **Gestion d'incident.** Réponse à VO-13.
24. **Annexes.** A : matrice de traçabilité (vulnérabilité, règle, contrôle de l'Annexe A d'ISO 27001) ; B : listes de contrôle (arrivée et départ, citées en R-30 et R-31 ; déroulé de gestion d'incident, voir §6.1) ; C : modèle de fiche de dérogation (§8.3), présent dans la note de décision (E) ; D : glossaire ; E : note de décision pour la direction (décisions à prendre, paliers intermédiaires, réponse de la direction, modèle de fiche).

---

*Document de travail : la partie 2 est rédigée section par section.*
