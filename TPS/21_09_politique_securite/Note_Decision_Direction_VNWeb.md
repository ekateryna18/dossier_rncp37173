# Note de décision pour la direction, VNWeb
### RNCP37173, M1.4, annexe E de la politique de sécurité

| | |
|---|---|
| Date | 21/09/2026 |
| Émetteur | Le référent sécurité |
| Destinataire | La direction |
| Objet | Douze décisions à prendre pour appliquer la politique de sécurité de VNWeb |
| Réponse attendue | Par écrit, sous 30 jours à compter de la remise de cette note (règle R-03, valeur proposée) |

---

## 1. Objet de la note

La politique de sécurité de VNWeb comprend une cinquantaine de règles. La plupart ne demandent ni budget ni changement d'organisation : l'équipe les met en place dès l'approbation de la politique, et elles ne figurent pas ici.

Cette note présente les douze décisions qui dépendent de la direction, parce qu'elles coûtent du temps ou de l'argent, changent les habitudes de l'équipe, ou passent par un prestataire. La direction n'a pas à juger la technique : elle décide du niveau de risque que l'entreprise accepte, ainsi que du temps et du budget qu'elle y consacre.

## 2. Ce que l'audit a établi

**Confirmé** signifie prouvé par une vérification faite sur le serveur des projets le 17/09/26. **Déclaré** signifie décrit par l'équipe, sans vérification à ce jour.

| Constat | Statut | Source |
|---|---|---|
| Le serveur des projets (production et préproduction) autorise la connexion directe en administrateur (root) | Confirmé | Rapport d'audit J2, constat 1 |
| Aucun compte individuel sur le serveur des projets : environ une centaine de comptes, un par projet, partagés par toute personne qui travaille dessus | Confirmé | J2, constat 5 |
| Dans les journaux, on ne peut pas identifier la personne derrière un compte | Confirmé | J2, constat 6 |
| Aucune sauvegarde planifiée n'est visible sur le serveur des projets (un autre mécanisme n'a pas été vérifié) | Confirmé | J2, constat 7 |
| Les journaux de connexion sont conservés environ quatre semaines | Confirmé | J2, constat 8 |
| La connexion SSH aux serveurs est possible avec un identifiant et un mot de passe, depuis n'importe où (essai à faire) | Déclaré | Équipe, 21/09 |
| Le VPN est protégé par un simple mot de passe | Déclaré | Équipe, 21/09 |
| Des mots de passe sont partagés dans Discord | Déclaré | Équipe, analyse EBIOS (J1) |
| Les accès d'un ancien collaborateur ont été réutilisés sur les serveurs | Déclaré | Équipe, 21/09 |
| Des données personnelles de weevus sont hébergées hors de l'Union européenne | Déclaré | Analyse EBIOS (J1) |

## 3. Comment répondre

Pour chaque décision, la direction choisit l'une des trois réponses :

- **Accepter** la mesure complète.
- **Planifier** : accepter la mesure avec une date, ou accepter un **palier intermédiaire**, c'est-à-dire une première étape moins coûteuse qui réduit déjà le risque.
- **Refuser** : dans ce cas, la direction signe une fiche de décision (modèle en section 6) qui consigne le risque qu'elle accepte de garder, son motif et une date de réexamen, au plus tard 12 mois plus tard.

Refuser une mesure n'est pas un échec du plan : c'est une décision qui devient visible et datée. Le refus d'une mesure complète n'empêche pas d'obtenir ses paliers intermédiaires.

La fiche de décision atteste que la direction a été informée du risque et décide en connaissance de cause. Elle ne contient aucune clause de décharge : elle documente la décision, sans modifier les obligations légales de l'entreprise envers ses clients et la réglementation. Sa portée juridique est à faire valider par un conseil juridique.

## 4. Les douze décisions

Les efforts indiqués sont des estimations de l'équipe technique, à confirmer. Un coût marqué « à chiffrer » n'est pas connu à ce jour.

| N° | Décision demandée et ce qu'elle change | Effort estimé | Palier intermédiaire | Risque qui reste en cas de refus | Règles |
|---|---|---|---|---|---|
| D1 | **Remplacer Discord pour les mots de passe par un coffre-fort partagé.** Les mots de passe ne circulent plus dans les messages ; ceux déjà exposés sont renouvelés | Quelques jours pour installer l'outil et renouveler les secrets ; l'outil peut être gratuit ou payant (à chiffrer) | Interdire dès maintenant Discord pour les mots de passe et renouveler les plus critiques (aucun coût) ; choisir l'outil ensuite | Les mots de passe restent lisibles par toute personne ayant accès à l'historique du salon | R-08 et A-02 |
| D2 | **Sécuriser la connexion SSH aux serveurs.** La connexion directe en administrateur est désactivée ; on se connecte avec une clé personnelle plutôt qu'un mot de passe, et seulement depuis l'entreprise ou le VPN | Serveurs locaux : de quelques heures à 2 jours pour l'équipe ; serveurs externes : selon la réponse du prestataire | 1) Mot de passe long et unique dans le coffre-fort, connexion root désactivée ; 2) accès limité aux adresses de l'entreprise ; 3) clés personnelles | Un mot de passe deviné ou volé donne accès au serveur depuis Internet, y compris à la production | R-06 |
| D3 | **Demander à chaque prestataire un état des lieux et des engagements écrits** : sauvegardes, comptes, journaux, alerte en cas d'incident, réponse sous 30 jours. La direction, qui détient la relation, transmet la demande | Un courriel par prestataire, puis le suivi ; un service supplémentaire pourrait être facturé (à vérifier) | Obtenir d'abord la confirmation écrite de l'existence des sauvegardes de production, puis le reste | Perte de données de production sans savoir si une sauvegarde existe ; incident chez le prestataire non signalé | R-19 |
| D4 | **Créer des comptes individuels sur le serveur des projets et formaliser la relation par un contrat ou un avenant** : sécurité, sauvegardes, alerte, protection des données personnelles, contact de secours | Travail du prestataire pour les comptes ; relecture d'un avenant ; coût éventuel à chiffrer | Journal d'usage des comptes partagés tenu par l'équipe, et renouvellement des mots de passe au départ d'une personne ; comptes individuels et avenant ensuite | Impossible de savoir qui a fait quoi ; un ancien collaborateur peut garder un accès ; responsabilités mal cadrées avec le prestataire | R-04 et R-19 à R-21 |
| D5 | **Ajouter un second facteur au VPN** : à la connexion à distance, un code sur téléphone en plus du mot de passe | Réglage simple si la solution actuelle le permet, remplacement sinon (à vérifier) | Télétravail limité à trois motifs et déclaré, Bureau à distance joignable seulement par le VPN, mot de passe VPN individuel, long et unique | Un mot de passe VPN volé ouvre l'accès au poste de bureau, donc à tout ce qu'il détient | R-07, R-10, R-11 |
| D6 | **Réserver les droits d'administrateur des serveurs locaux au référent et à son suppléant.** La direction garde un accès de secours, non utilisé au quotidien | Une décision et une configuration : moins d'une journée | La direction garde ses droits ; chaque utilisation est consignée et le référent est prévenu | Un compte compromis donne les droits d'administration ; une seule personne technique en cas d'absence du référent | R-05 |
| D7 | **Sauvegarder les données critiques locales** (code source dans Gitea, Extranet, configuration) chaque jour, avec une copie hors des locaux, et tester la restauration deux fois par an | Environ 1 à 2 jours de mise en place ; stockage hors des locaux à chiffrer | Sauvegarde hebdomadaire au départ, puis quotidienne ; copie hors des locaux dans un second temps | Perte définitive du code source ou des données de l'Extranet en cas de panne, d'erreur ou de sinistre | R-17 |
| D8 | **Faire relire chaque modification par un second développeur avant la mise en production.** Le patron continue de valider le fonctionnement ; la relecture ajoute une validation technique | Quelques minutes de relecture par modification, réparties entre les développeurs | Commencer par weevus, les sites de voyance et l'Extranet | Un code défectueux ou vulnérable arrive en production sans relecture technique | R-22 |
| D9 | **Mettre en conformité les données personnelles hébergées hors de l'Union européenne** : les migrer vers l'Union européenne, ou documenter la base légale du transfert. Décision sous 30 jours ; mise en œuvre sous six mois | À chiffrer : temps de migration, arrêt possible pendant la bascule ; avis juridique pour la base légale | Documenter d'abord la situation et demander un avis juridique, puis migrer | Non-conformité au RGPD sur un transfert hors de l'Union européenne : mise en demeure ou sanction possibles, perte de confiance des utilisateurs concernés | R-27, A-03 |
| D10 | **Migrer weevus d'Amplify Gen 1 vers Gen 2 avant février 2027**, la fin de support étant annoncée en mai 2027 | Chantier à planifier avec un environnement d'essai ; effort à chiffrer | Rédiger le plan et tester en environnement d'essai avant de décider de la bascule | Plus de correctifs de sécurité après mai 2027 sur la plateforme qui héberge weevus | A-05 |
| D11 | **Communiquer au référent les protections en place sur les postes** (antivirus, mises à jour, filtrage), pour une vérification annuelle ; le chiffrement des disques est étudié | Un échange d'une heure, puis une vérification par an | Se limiter à la liste des protections en place ; étudier le chiffrement ensuite | Une protection absente ou obsolète n'est pas détectée | R-12 |
| D12 | **Renforcer la sécurité des locaux** : registre des clés, cylindre changé si une clé est perdue ou non restituée, serveurs locaux dans une pièce ou une armoire fermant à clé | Registre : une heure ; cylindre et armoire : coût à chiffrer | Registre des clés d'abord ; cylindre changé seulement en cas de perte avérée | Entrée non autorisée dans les locaux sans trace ; accès physique aux serveurs locaux | R-33 et R-34 |

## 5. Réponse de la direction

Cocher une case par décision. Un refus s'accompagne d'une fiche de décision (section 6).

| N° | Accepte la mesure complète | Accepte un palier (lequel) | Planifie (date) | Refuse (fiche signée) |
|---|---|---|---|---|
| D1 | [ ] | [ ] | [ ] | [ ] |
| D2 | [ ] | [ ] | [ ] | [ ] |
| D3 | [ ] | [ ] | [ ] | [ ] |
| D4 | [ ] | [ ] | [ ] | [ ] |
| D5 | [ ] | [ ] | [ ] | [ ] |
| D6 | [ ] | [ ] | [ ] | [ ] |
| D7 | [ ] | [ ] | [ ] | [ ] |
| D8 | [ ] | [ ] | [ ] | [ ] |
| D9 | [ ] | [ ] | [ ] | [ ] |
| D10 | [ ] | [ ] | [ ] | [ ] |
| D11 | [ ] | [ ] | [ ] | [ ] |
| D12 | [ ] | [ ] | [ ] | [ ] |

| Direction | Référent sécurité |
|---|---|
| Nom : | Nom : |
| Date : | Date : |
| Signature : | Signature : |

## 6. Fiche de décision (modèle de dérogation)

Une fiche par mesure refusée ou reportée (règle R-03). Elle est conservée dans le registre des dérogations et réexaminée à la date indiquée.

| Champ | À renseigner |
|---|---|
| Référence | D... / R... |
| Date | |
| Rédigée par | Le référent sécurité |
| Mesure proposée | |
| Risque traité et constat | (vulnérabilité VT ou VO, source de la preuve) |
| Paliers intermédiaires proposés | |
| Décision de la direction | [ ] Accepte la mesure complète [ ] Accepte le palier : ... [ ] Planifie pour le : ... [ ] Refuse et accepte le risque qui reste |
| Motif de la décision | |
| Risque qui reste accepté | |
| Date de réexamen | Au plus tard 12 mois après la signature |

La direction déclare avoir été informée du risque, de la mesure proposée et des paliers intermédiaires, et décide en connaissance de cause.

| Direction | Référent sécurité |
|---|---|
| Nom : | Nom : |
| Date : | Date : |
| Signature : | Signature : |
