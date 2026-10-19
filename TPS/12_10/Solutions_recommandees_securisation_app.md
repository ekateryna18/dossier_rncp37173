# Solutions recommandées et besoins couverts : sécurisation de l'application VNWeb

Trois besoins de sécurité identifiés sur l'application, la solution recommandée pour chacun, et les preuves qui montrent qu'elle fonctionne.

| # | Besoin couvert | Solution recommandée |
|---|----------------|----------------------|
| 1 | Qu'un mot de passe volé ou deviné ne suffise pas à accéder aux données sensibles | Double authentification (MFA) sur les comptes |
| 2 | Qu'une librairie avec une faille connue n'arrive pas en production sans que personne ne le sache | Analyse automatique des vulnérabilités connues (CVE) dans le pipeline |
| 3 | Savoir rapidement qu'une intrusion ou une panne est en cours | Journalisation hors du serveur et alerte vers le référent sécurité |

---

## 1. Protéger les comptes : la double authentification

**Besoin couvert.** L'application contient des données personnelles, dont celles de mineurs. Aujourd'hui, un simple email et mot de passe ouvrent un compte. Si un mot de passe est réutilisé, volé ou deviné, la personne qui l'a obtenu accède aux données comme si elle était le vrai titulaire. Le risque est le plus grave sur les comptes administrateur et superviseur, qui voient le plus de choses.

**Solution recommandée.**
- Imposer la double authentification aux comptes administrateur et superviseur : mot de passe plus code à usage unique généré par une application d'authentification.
- La proposer aux autres utilisateurs, sans l'imposer. Une obligation pour tous gênerait l'usage d'un public jeune, d'où la priorité donnée aux comptes qui comptent le plus.
- Conserver les codes de secours dans le coffre-fort de l'équipe.

**Pourquoi c'est la bonne mesure.** Elle coûte peu, elle est simple à expliquer, et elle neutralise le scénario le plus courant : le mot de passe volé. Elle ne règle pas tout : un compte légitime peut voir ce qu'il ne devrait pas si les droits d'accès de l'application sont mal définis. Elle vient en renfort de la correction de ces droits, pas à sa place.

**Preuves.** 100 % des comptes à privilèges ont la double authentification, et une connexion avec le mot de passe seul est refusée.

**Coût, limites et conditions de lancement.**
- **Coût estimé** : entre 2 et 5 jours de développement, soit 2 j × 350 € = 700 € (bas) à 5 j × 350 € = 1 750 € (haut). L'écart dépend surtout de ce qu'on retient : activation seulement pour les comptes administrateur et superviseur (bas de la fourchette), ou ouverture à tous les utilisateurs avec gestion des téléphones perdus (haut de la fourchette).
- **Limite** : la double authentification ajoute une étape à chaque connexion. Elle protège le compte, pas l'autorisation d'accès aux données une fois connecté.
- **Condition de lancement** : informer et accompagner les utilisateurs avant l'activation, avec un guide court expliquant comment installer l'application d'authentification et que faire en cas de téléphone perdu. Pour les comptes administrateur et superviseur, l'activation se fait avec la personne concernée, pas à distance.

---

## 2. Bloquer les failles connues : l'analyse automatique des CVE dans le pipeline

**Besoin couvert.** Une application s'appuie sur des dizaines de librairies, et chacune peut révéler une faille publiée du jour au lendemain. Sans contrôle automatique, on déploie une librairie vulnérable sans le savoir, et on l'apprend quand quelqu'un l'exploite. Aujourd'hui, rien ne vérifie les librairies avant la mise en production.

**Solution recommandée.** Il est recommandé d'ajouter au pipeline de livraison une étape qui analyse automatiquement les librairies utilisées et signale les failles connues (CVE) avant la mise en production. Si une faille grave est détectée, la livraison est bloquée tant qu'elle n'est pas corrigée. C'est la façon la plus simple de ne plus dépendre de la mémoire de quelqu'un pour vérifier ce point.

**Preuves.** Zéro faille critique ouverte, et un rapport d'analyse daté pour chaque projet.

**Coût, limites et conditions de lancement.**
- **Coût estimé** : environ 2 jours de développement (mise en place de l'analyse et premier traitement des failles remontées), soit 2 j × 350 € = 700 €. Le pipeline lui-même est un chantier à part, non compté ici.
- **Limite** : il n'existe pas encore de pipeline de livraison, donc le contrôle automatique ne peut pas bloquer une mise en production aujourd'hui.
- **Condition de lancement** : en attendant le pipeline, lancer l'analyse à la main avant chaque livraison et à un rythme fixé (par exemple chaque semaine). Une fois le pipeline en place, l'étape de sécurité est branchée dessus et le contrôle manuel disparaît.

---

## 3. Détecter un incident : la journalisation et l'alerte

**Besoin couvert.** Aujourd'hui, si quelqu'un tente de forcer une connexion ou si le service tombe, personne n'est prévenu. On le découvre par hasard, ou quand un utilisateur se plaint. Et si les journaux restent sur le serveur attaqué, l'attaquant peut les effacer ou les modifier.

**Solution recommandée.**
- Envoyer les journaux de connexion et d'administration vers un emplacement séparé du serveur qui les produit.
- Mettre en place une alerte minimale vers le référent sécurité sur trois événements : échecs de connexion répétés, connexion à un compte à privilèges, connexion à des horaires inhabituels.
- Ajouter une alerte de disponibilité, pour être prévenu quand le service ne répond plus.
- Tester l'alerte à la mise en place, puis à chaque revue périodique.

**Pourquoi c'est la bonne mesure.** Les deux premières mesures réduisent la probabilité d'un incident, celle-ci réduit le temps pendant lequel on ne sait rien. Une alerte simple sur trois cas bien choisis vaut mieux qu'un outil complet que personne ne regarde.

**Preuves.** Les journaux sont consultables hors du serveur, et le référent sécurité a bien reçu l'alerte de test.

**Coût, limites et conditions de lancement.**
- **Coût estimé** : environ 5,5 jours de développement, répartis entre la journalisation dans l'application (3 jours, valeur haute d'une fourchette de 2 à 3), l'envoi des journaux hors du serveur et les alertes de sécurité (2 jours), et l'alerte de disponibilité (0,5 jour). Calcul : (3 + 2 + 0,5) j = 5,5 j, puis 5,5 j × 350 € = 1 925 €.
- **Limite** : l'application ne produit pas de journaux exploitables aujourd'hui. Sans eux, il n'y a rien sur quoi déclencher une alerte.
- **Condition de lancement** : mettre en place la journalisation avant les alertes, dans cet ordre. Il faut aussi désigner la personne qui reçoit les alertes et qui s'engage à les lire, sinon l'alerte ne sert à rien.

---

## Comment les trois se complètent

Ces mesures agissent à trois moments différents : **empêcher** l'usurpation d'un compte (1), **prévenir** l'arrivée d'une faille connue en production (2), **détecter** ce qui passe malgré tout (3). Aucune ne suffit seule, et ensemble elles forment une base solide pour l'équipe.

## Risques résiduels, responsables et échéances

Une fois chaque solution en place, un risque subsiste. Il est identifié, rattaché à un rôle et revu dans un délai fixé. Les échéances se comptent à partir de la mise en place de la solution concernée.

| Solution | Risque résiduel | Responsable | Échéance | Prochain test | Prochaine décision |
|----------|-----------------|-------------|----------|---------------|--------------------|
| 1. Double authentification | Les utilisateurs ordinaires ne sont pas obligés d'activer la double authentification. Un compte sans MFA reste protégé par son seul mot de passe, et peut donc être usurpé si ce mot de passe est volé. | Responsable technique | Revue du taux d'activation à 3 mois | Se connecter à un compte administrateur avec le mot de passe seul : l'accès doit être refusé. Tester aussi un code de secours. | À 3 mois, selon le taux d'activation, garder le MFA volontaire pour les utilisateurs ordinaires ou le rendre obligatoire pour tous. |
| 2. Analyse des CVE | Tant que le pipeline n'est pas en place, l'analyse manuelle peut être oubliée. Une faille non encore publiée n'est pas détectée par ce type d'outil. | Développeur qui effectue la livraison | Contrôle manuel à chaque livraison, pipeline en place sous 3 mois | Lancer l'analyse à la main sur chaque projet et compter les failles critiques ouvertes. | Fixer la date de mise en place du pipeline, et décider quelles failles se corrigent avant la prochaine livraison. |
| 3. Journalisation et alertes | Une alerte reçue par une personne absente reste sans réponse. | Référent sécurité, avec un suppléant désigné | Test de l'alerte à la mise en place, puis revue à 6 mois | Provoquer volontairement des échecs de connexion répétés et vérifier que le référent sécurité reçoit l'alerte, une fois la journalisation en place. | À 6 mois, confirmer le suppléant et ajuster les seuils d'alerte s'ils se déclenchent trop ou pas assez. |

Tout risque résiduel non traité à son échéance est soit accepté par écrit par le responsable technique, soit replanifié avec une nouvelle date.

## Récapitulatif des coûts

| Solution | Jours | Tarif journalier | Calcul | Coût |
|----------|-------|------------------|--------|------|
| 1. Double authentification (bas) | 2 j | 350 € | 2 × 350 | 700 € |
| 1. Double authentification (haut) | 5 j | 350 € | 5 × 350 | 1 750 € |
| 2. Analyse des CVE | 2 j | 350 € | 2 × 350 | 700 € |
| 3. Journalisation et alertes | 5,5 j | 350 € | 5,5 × 350 | 1 925 € |
| **Total bas** | **9,5 j** | 350 € | 9,5 × 350 | **3 325 €** |
| **Total haut** | **12,5 j** | 350 € | 12,5 × 350 | **4 375 €** |

Le total bas additionne les valeurs basses (2 + 2 + 5,5 = 9,5 j), le total haut les valeurs hautes (5 + 2 + 5,5 = 12,5 j).

Ces chiffres sont des estimations de temps de développeur, calculées sur une base de 350 € par jour (hypothèse à ajuster selon le coût réel). Ils ne comprennent ni le pipeline de livraison, ni le temps de formation des utilisateurs au-delà de la préparation du guide.
