# Politique de sécurité des systèmes d'information

## VNWeb

*Version 0.1, document soumis à l'approbation de la direction. Ce document définit les règles de sécurité de VNWeb de façon durable : il décrit des principes et des exigences, pas une configuration technique précise. Il n'a pas à être réécrit quand un serveur change de nom, quand un outil est remplacé par un autre, ou quand l'entreprise change d'hébergeur. Le détail technique et le plan de mise en œuvre associés à cette politique vivent dans des documents séparés, revus plus souvent qu'elle.*

---

## Préambule

Une politique de sécurité des systèmes d'information fixe les objectifs, les principes et les règles qui protègent les systèmes et les données d'une organisation. Elle ne décrit pas un état technique figé : elle pose un cadre stable, dont l'application concrète s'adapte à l'outillage du moment.

Ce document se distingue de trois autres documents avec lesquels il est parfois confondu. La lettre d'engagement de la direction affirme un engagement, sans détailler des règles. La charte informatique fixe des règles de conduite individuelles pour l'usage quotidien des outils ; elle en découle et reste plus courte. Le plan de reprise d'activité organise la résilience opérationnelle en cas de sinistre ; il complète cette politique sans s'y substituer.

VNWeb adopte cette politique pour donner à son équipe un cadre de référence commun, éviter les décisions de sécurité prises au coup par coup, et poser les bases d'une démarche de conformité vis-à-vis de ses clients et de ses obligations légales.

---

## Contexte : les acteurs et les risques

Comprendre à qui s'adresse cette politique et quels risques elle vise aide à lire ses règles avec leur raison d'être, plutôt que comme une liste de contraintes.

### Les acteurs

VNWeb est une petite structure : une direction et une équipe réduite de développeurs, qui cumulent le développement, l'administration des serveurs et l'exploitation des applications, sans séparation stricte entre ces rôles. La direction décide de l'organisation et valide les livraisons, sans disposer nécessairement d'une compétence technique pour juger elle-même de leur sécurité : elle dépend de son équipe pour être informée des risques. Les clients et les prestataires externes qui interviennent sur les systèmes de l'entreprise sont présentés en détail au périmètre (section 2) ; leur seule dépendance à retenir ici est qu'ils pèsent sur les risques de l'entreprise sans être eux-mêmes soumis à cette politique.

### Les risques les plus courants pour une structure de cette taille

Une petite équipe expose des risques particuliers, indépendamment de la technologie utilisée à un instant donné :

- **La concentration des rôles.** Les mêmes personnes développent, administrent et exploitent les systèmes, sans les contrôles croisés qu'une équipe plus grande pourrait se permettre.
- **La dépendance à des tiers.** L'hébergement d'une partie des systèmes est confié à des prestataires externes, dont la sécurité n'est pas entièrement sous le contrôle de l'entreprise.
- **La mobilité des accès.** Le travail à distance, depuis un appareil personnel ou un réseau non maîtrisé, élargit la surface par laquelle un accès pourrait être compromis.
- **Les canaux informels.** Le partage d'identifiants ou d'informations sensibles par des outils de messagerie non prévus à cet effet reste une tentation naturelle, en particulier sous la pression d'une échéance.
- **Le manque de formalisation.** Sans cadre écrit, chaque personne applique ses propres pratiques de sécurité, à des niveaux de rigueur inégaux, et rien n'est vérifiable ni opposable.

Ce sont ces risques, plus que la configuration technique du moment, que cette politique cherche à réduire.

## 1. Objet

Cette politique fixe les principes et les règles de sécurité applicables à l'ensemble des systèmes d'information de VNWeb : les postes de travail, les serveurs, les applications développées ou exploitées par l'entreprise, les données qu'elle traite, et les accès à distance qui les relient. Elle s'impose à toute personne qui dispose d'un accès à ces systèmes, dès son approbation par la direction.

## 2. Périmètre

Cette politique s'applique à la direction et à l'ensemble des collaborateurs de VNWeb, ainsi qu'à toute personne qui reçoit un accès aux systèmes de l'entreprise, de façon durable ou temporaire.

Elle couvre les postes de travail utilisés par l'équipe, les serveurs internes de développement, les environnements hébergés par des tiers pour la préproduction et la production des applications, l'infrastructure infonuagique utilisée par les applications de l'entreprise, l'extranet interne, le système de gestion de versions du code source, et les outils de communication de l'équipe.

Les prestataires qui hébergent des serveurs pour le compte de VNWeb sont soumis aux exigences de sécurité que l'entreprise leur adresse par écrit, sans que cette politique leur soit directement opposable : leur relation avec VNWeb est régie par leur contrat.

Les clients de VNWeb ne sont pas soumis à cette politique. L'entreprise reste responsable de ce qu'elle leur livre et des données auxquelles ses propres accès lui donnent accès.

Le contenu fonctionnel et la qualité du code applicatif ne relèvent pas de cette politique, qui porte en revanche sur les contrôles qui entourent leur développement et leur mise en production.

## 3. Principes directeurs

Six principes guident l'application de cette politique et servent de repère dans les situations qu'elle ne prévoit pas explicitement.

- **Le moindre privilège.** Chaque personne dispose des droits nécessaires à sa mission, et seulement de ceux-là, pour la durée où ils lui sont utiles.
- **La traçabilité individuelle.** Toute action d'administration doit pouvoir être attribuée à une personne précise, pas seulement à un compte partagé.
- **La séparation des environnements.** Le développement, la préproduction et la production restent distincts, et les données réelles ne circulent pas dans un environnement de développement sans y avoir été rendues anonymes.
- **La protection des secrets.** Un identifiant, un mot de passe ou une clé d'accès ne circule pas dans un canal de discussion ordinaire : il est conservé dans un espace prévu à cet effet.
- **La proportionnalité.** Les règles de cette politique restent applicables par une petite équipe et vérifiables dans le temps. Une règle qu'il n'est pas possible de contrôler n'a pas sa place ici, même si elle paraît souhaitable en théorie.
- **La décision documentée.** Quand une règle ne peut pas être appliquée dans l'immédiat, ce choix n'est pas laissé sans suite : la direction en est informée, elle statue par écrit, pour une durée qu'elle fixe et qui ne dépasse pas un an sans nouvelle décision écrite. Cette politique fixe ainsi des objectifs réalistes plutôt qu'une sécurité théoriquement parfaite, qui coûterait plus qu'elle ne protégerait et finirait contournée par l'équipe elle-même.

*Les tableaux qui suivent citent des référentiels par sigle : ANSSI pour l'Agence nationale de la sécurité des systèmes d'information, dont le guide d'hygiène informatique est noté « hyg. » ; CNIL pour la Commission nationale de l'informatique et des libertés ; OWASP pour la communauté internationale de référence en sécurité des applications ; ISO pour l'Organisation internationale de normalisation, à travers l'Annexe A de la norme ISO 27001:2022. Le détail de chaque référentiel figure en section 20.*

## 4. Politique de gouvernance

Cette politique fixe qui décide, qui pilote la sécurité au quotidien et qui l'applique. Elle protège l'entreprise contre les décisions de sécurité prises au hasard, par une seule personne, ou sans que la direction en soit informée.

| N° | Rôle | Responsabilités | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R4.1 | La direction | Approuve cette politique et en assure la diffusion. Elle alloue le temps et les moyens nécessaires à son application, désigne un référent sécurité parmi les développeurs ainsi qu'un suppléant, afin que la fonction ne repose pas sur une seule personne, et décide des dérogations à cette politique lorsqu'une règle ne peut pas être appliquée, dont elle assume la responsabilité, consignée par écrit. | Désignation inscrite dans le bloc d'approbation ; registre des dérogations à jour ; décision écrite sur le temps alloué | ISO A.5.2, A.5.4 |
| R4.2 | Le référent sécurité | Anime l'application de la politique au quotidien : il en tient le contenu à jour, suit les indicateurs de sécurité, tient le registre des accès, coordonne la réponse aux incidents, et rend compte à la direction à intervalle régulier. Il ne dispose pas d'une autorité hiérarchique sur ses collègues, et sa désignation ne le rend pas responsable des décisions que seule la direction peut prendre. | Un compte rendu daté par trimestre ; tableau de bord à jour | ISO A.5.4 |
| R4.3 | Le suppléant | Reprend l'ensemble des fonctions du référent en son absence, afin que le pilotage de la sécurité ne repose pas sur la seule disponibilité d'une personne. | Suppléant identifié, actif en l'absence du référent | - |
| R4.4 | Chaque collaborateur | Applique cette politique dans son travail quotidien, signale toute anomalie de sécurité qu'il constate, et participe aux actions de sensibilisation prévues à cet effet. | Signalements enregistrés ; participation aux actions de sensibilisation | - |
| R4.5 | Les prestataires | Appliquent, pour les systèmes qu'ils hébergent pour le compte de VNWeb, les exigences de sécurité qui leur sont transmises par écrit, et informent l'entreprise sans délai de tout incident touchant ces systèmes. | Cahier des exigences remis avec accusé de réception ; réponse écrite ; alerte testée | RGPD Art. 28 ; ISO A.5.19, A.5.20, A.8.13 |

## 5. Politique de contrôle d'accès et d'authentification

Cette politique fixe qui peut se connecter à quoi, et comment cette identité est vérifiée. Elle protège les systèmes de l'entreprise contre un accès obtenu par une personne non autorisée ou par un mot de passe compromis.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R5.1 | Un compte par personne | Chaque personne dispose d'un compte qui lui est propre pour accéder aux systèmes de l'entreprise. Le partage d'un même compte par plusieurs personnes est évité ; là où un tel partage subsiste pour des raisons historiques, son usage est journalisé et son remplacement par des comptes individuels reste un objectif à atteindre. | Liste des comptes : chacun a un titulaire identifié ; journal d'usage tenu pour les comptes encore partagés | ANSSI (hyg.) ; ISO A.5.15 à A.5.18, A.6.7 |
| R5.2 | Des droits d'administration nominatifs | Les droits d'administration sont accordés nominativement, distincts du compte d'usage courant, et limités aux personnes qui en ont réellement besoin. | Liste des détenteurs de droits d'administration, avec le motif | ANSSI (hyg.) ; ISO A.5.3, A.5.15 à A.5.18, A.8.2, A.8.5 |
| R5.3 | Pas d'accès administrateur direct | La connexion directe à un compte disposant de tous les droits, sans passer par un compte nominatif, est évitée sur les systèmes que l'entreprise administre elle-même, et cette exigence est également transmise à ses prestataires. | Configuration d'accès : connexion administrateur directe refusée | ANSSI (hyg., authentification forte) ; ISO A.5.15 à A.5.18 |
| R5.4 | Une authentification forte à distance | L'accès à distance aux serveurs se fait par une authentification robuste, fondée sur une clé plutôt que sur un mot de passe seul, et restreinte autant que possible aux réseaux de l'entreprise. Une protection contre les tentatives de connexion répétées est active sur chaque serveur exposé. | Essai de connexion par mot de passe seul refusé ; protection contre les tentatives répétées active | ANSSI (hyg., authentification forte) ; ISO A.5.15 à A.5.18 |
| R5.5 | La double authentification | Elle est activée sur les services qui la proposent, en particulier pour les accès à privilèges et pour l'accès distant aux systèmes de l'entreprise (réseau privé virtuel). Un compte administrateur disposant de tous les pouvoirs sur un service donné n'est pas utilisé pour un usage quotidien. | Part des comptes avec double authentification (cible : tous) | ANSSI (authentification forte) ; ISO A.6.7, A.8.2, A.8.5 |

## 6. Politique de gestion des secrets

Cette politique fixe où et comment les mots de passe et les clés d'accès sont conservés. Elle protège l'entreprise contre la fuite d'un identifiant, qui donnerait à quiconque le même accès qu'un collaborateur légitime.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R6.1 | Un coffre-fort partagé | Les mots de passe, les clés d'accès et les jetons d'authentification sont conservés dans un coffre-fort numérique partagé, accessible par des comptes individuels protégés par une double authentification. | Coffre-fort en service, accès individuels avec double authentification | ANSSI (authentification forte), CNIL ; ISO A.5.15 à A.5.18 |
| R6.2 | Aucun secret hors du coffre-fort | Les secrets ne circulent pas par messagerie instantanée, par courriel, ni par tout autre canal de discussion ordinaire. | Nombre de secrets trouvés hors du coffre-fort (cible : zéro) | ANSSI (authentification forte), CNIL ; ISO A.5.15 à A.5.18 |
| R6.3 | Des mots de passe uniques et renouvelés à bon escient | Chaque mot de passe est propre à un système, suffisamment long ou généré automatiquement, et renouvelé dès qu'un départ, une fuite ou un doute le justifie, plutôt qu'à une fréquence fixe qui n'apporterait pas de protection réelle. | Mots de passe uniques par système ; renouvellement tracé après un départ ou un doute | ANSSI (authentification forte), CNIL ; ISO A.5.15 à A.5.18 |
| R6.4 | La vérification par un second canal | Toute demande d'accès ou de transmission d'un secret reçue par un canal inhabituel, même en apparence légitime, est vérifiée par un second moyen de contact avant d'être exécutée. Cette vigilance protège aussi bien l'équipe technique que la direction, dont la position expose particulièrement à ce type de tentative. | Rappel inscrit au programme de sensibilisation ; demandes refusées ou confirmées, consignées | ANSSI (hyg.) |

## 7. Politique des postes de travail et du télétravail

Cette politique fixe les conditions de sécurité des postes utilisés au bureau et à distance. Elle protège l'entreprise contre un poste compromis, en particulier lors d'une connexion depuis un domicile ou un appareil personnel, qui donnerait accès à l'ensemble des systèmes.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R7.1 | Un socle de protection sur chaque poste | Chaque poste de travail dispose d'une protection antivirus active, de mises à jour appliquées régulièrement, d'un verrouillage automatique et d'un mot de passe de session robuste. Le chiffrement du disque est mis en œuvre lorsque le système le permet. | Fiche de contrôle par poste (date, résultat) ; état du chiffrement par poste | ANSSI (hyg.) ; ISO A.8.1 |
| R7.2 | Un télétravail encadré, pas un usage par défaut | Il répond à un besoin précis, connu du référent sécurité. La connexion à distance passe par un accès sécurisé et, dès que la solution technique le permet, protégé par une double authentification. | Journal de télétravail (personne, date, motif) | ANSSI (hyg.) ; ISO A.6.7, A.8.1 |
| R7.3 | Des conditions minimales sur l'appareil utilisé | Qu'il appartienne à l'entreprise ou à la personne elle-même, l'appareil utilisé pour se connecter respecte des conditions minimales de sécurité : système à jour, protection active, session non partagée, réseau maîtrisé. | Attestation de conformité de l'appareil signée chaque année | ANSSI (hyg.) ; ISO A.6.7, A.8.1 |
| R7.4 | Une séparation des usages sur les tâches sensibles | Les accès d'administration aux serveurs se font, autant que possible, depuis un contexte distinct de la navigation et de la messagerie courantes, pour limiter l'exposition d'un poste compromis par un usage ordinaire. | Comptes ou profils d'administration distincts de l'usage courant, vérifiés par échantillon | ANSSI (hyg.) ; ISO A.6.7 |

## 8. Politique de sécurité des serveurs et de l'infrastructure

Cette politique fixe comment les serveurs sont maintenus, cloisonnés, sauvegardés et surveillés. Elle protège la disponibilité et l'intégrité des applications et des données de l'entreprise contre une panne, une erreur ou une intrusion qui passerait inaperçue.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R8.1 | Un inventaire à jour | Les serveurs, les services et les logiciels utilisés sont recensés, avec leur rôle, leur responsable et leur niveau de mise à jour. | Inventaire daté, revu chaque trimestre | ANSSI (hyg.) ; ISO A.5.9, A.8.8 |
| R8.2 | Des mises à jour régulières | Les mises à jour de sécurité sont appliquées à intervalle régulier, sans attendre en cas de faille connue et déjà exploitée. | Nombre de paquets de sécurité en attente, relevé chaque mois (cible : zéro à l'échéance) | ANSSI (hyg.) ; ISO A.5.6, A.5.7, A.8.8 |
| R8.3 | Un filtrage réseau minimal | Chaque serveur n'ouvre que les services nécessaires. Les environnements de développement, de préproduction et de production restent séparés, tant sur le plan des accès que sur celui des données qu'ils contiennent. | Règles du pare-feu relevées ; contrôle par échantillon des bases de développement | ANSSI (hyg.) ; ISO A.8.31, A.8.33 |
| R8.4 | Des sauvegardes testées | Les données critiques sont sauvegardées automatiquement, à une fréquence adaptée à leur importance, avec au moins une copie conservée en dehors du lieu où elles sont produites. Chaque sauvegarde fait l'objet d'un test de restauration au moins une fois par semestre, condition pour qu'elle offre une garantie réelle plutôt qu'une simple présomption. | Date de la dernière sauvegarde réussie ; comptes rendus de test de restauration datés | ANSSI (hyg.) ; ISO A.8.13 |
| R8.5 | Un plan de reprise | Il décrit, pour chaque application en production, la conduite à tenir en cas d'indisponibilité prolongée. | Plan de reprise écrit, connu du référent, de son suppléant et de la direction | ANSSI (hyg.) ; ISO A.8.13 |
| R8.6 | Une journalisation exploitable | Les journaux de connexion et d'administration sont conservés sur une durée suffisante pour permettre une investigation rétroactive, et stockés à l'écart du serveur qui les produit. Une alerte minimale prévient le référent sécurité des anomalies les plus significatives : échecs de connexion répétés, connexions à un compte à privilèges, connexions à des horaires inhabituels. | Durée de conservation relevée ; journaux consultables hors du serveur ; alerte de test reçue | ANSSI (hyg.) ; ISO A.8.15, A.8.16 |

## 9. Politique relative aux prestataires et à la sous-traitance d'hébergement

Cette politique fixe ce que VNWeb exige des prestataires qui hébergent ses systèmes. Elle protège l'entreprise contre un maillon externe mal sécurisé et contre une dépendance sans solution de secours.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R9.1 | Une identification claire de chaque prestataire | VNWeb identifie chaque prestataire auquel elle confie l'hébergement d'un système ou de données : son identité, le périmètre de ce qu'il gère, et un contact d'urgence. | Fiche complète pour chacun des prestataires | RGPD Art. 28 ; ISO A.5.19, A.5.20, A.8.13 |
| R9.2 | Des exigences transmises par écrit | VNWeb adresse par écrit à chaque prestataire les exigences de sécurité applicables à ce qu'il héberge, portant notamment sur l'authentification, les sauvegardes, les mises à jour et la conservation des journaux, et suit sa réponse. | Cahier des exigences remis avec accusé de réception ; réponse écrite ; relevés archivés | RGPD Art. 28 ; ISO A.5.19, A.5.20, A.8.13 |
| R9.3 | Des engagements contractuels | Les engagements de sécurité et de protection des données sont formalisés par contrat avec chaque prestataire, y compris lorsqu'il traite des données à caractère personnel pour le compte de VNWeb. | Contrat ou avenant signé ; à défaut, dérogation signée | RGPD Art. 28 |
| R9.4 | Une continuité prévue | Une solution de secours est prévue en cas d'indisponibilité d'un prestataire, sous la forme d'un contact d'urgence et d'une capacité à reprendre l'hébergement ailleurs. | Contact de secours identifié ; procédure de reprise décrite | RGPD Art. 28 ; ISO A.5.19, A.5.20 |

## 10. Politique de développement et de mise en production

Cette politique fixe les contrôles qui entourent l'écriture et la livraison du code. Elle protège les applications contre un code défectueux, vulnérable ou malveillant qui atteindrait la production sans avoir été relu.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R10.1 | Une relecture systématique | Toute modification destinée à la production est relue par une personne autre que son auteur avant d'être intégrée, et cette relecture est tracée. | Part des fusions vers la production ayant une relecture tracée (cible : toutes) | OWASP (Code Review Guide) ; ISO A.8.25, A.8.29, A.8.32 |
| R10.2 | Une branche de production protégée | Elle n'accepte pas de modification directe, hors de la relecture prévue. | Réglage de protection activé sur chaque dépôt actif | OWASP (Code Review Guide) ; ISO A.8.25, A.8.29, A.8.32 |
| R10.3 | Une vérification minimale avant mise en production | Elle porte sur les parcours essentiels, l'absence de secret dans le code, et la validation préalable en préproduction. Les projets qui traitent des données personnelles ou des authentifications font l'objet d'une attention renforcée, avec des tests automatisés introduits progressivement. | Liste de contrôle jointe à chaque livraison | OWASP (DevSecOps Guideline) |
| R10.4 | Un déploiement traçable | Qui a déployé quoi, quand, et sur quel environnement, reste connu. | Journal de déploiement tenu | OWASP (DevSecOps Guideline) ; ISO A.8.25, A.8.29, A.8.32 |
| R10.5 | Une veille sur les dépendances | Les dépendances logicielles utilisées par chaque projet actif sont examinées à intervalle régulier pour des vulnérabilités connues, et une faille critique déjà exploitée est corrigée sans délai. | Rapport d'analyse daté par projet ; nombre de failles critiques ouvertes | OWASP (DevSecOps Guideline) ; ISO A.8.8 |

## 11. Politique de protection des données à caractère personnel

Cette politique fixe comment les données des utilisateurs et des clients sont recensées, hébergées et protégées. Elle protège les personnes concernées contre un usage non maîtrisé de leurs données, et l'entreprise contre le non-respect de la réglementation.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R11.1 | Un registre des traitements | VNWeb tient un registre des traitements de données à caractère personnel qu'elle opère, précisant la nature des données, leur finalité, leur lieu d'hébergement, leur durée de conservation, et son propre rôle au regard de la réglementation, responsable de traitement ou sous-traitant selon les projets. | Registre renseigné ; part des projets couverts | RGPD Art. 30 ; ISO A.5.31, A.5.34 |
| R11.2 | Une localisation dans l'Union européenne | Les données à caractère personnel y sont hébergées ; à défaut, leur transfert hors de l'Union repose sur une base légale documentée. Une revue régulière des ressources d'hébergement vérifie leur localisation, leur chiffrement, et les droits d'accès qui leur sont associés. | Tableau des ressources daté (région, chiffrement, permissions) | RGPD chapitre V, Art. 44 ; ISO A.5.23 |
| R11.3 | Le respect des droits des personnes | Une procédure permet de répondre, dans les délais prévus par la réglementation, aux demandes d'une personne concernant ses propres données : accès, rectification ou suppression. | Procédure écrite ; délai de traitement des demandes reçues | RGPD Art. 15 à 22 |
| R11.4 | Des contrats clairs avec les clients | Ils précisent le rôle de VNWeb à l'égard de leurs données, les mesures de sécurité appliquées, et la façon dont un incident leur serait notifié. | Clause type intégrée aux nouveaux contrats | RGPD Art. 28 ; ISO A.5.31, A.5.34 |

## 12. Politique des arrivées, des départs et de la revue des accès

Cette politique fixe comment les accès sont ouverts, retirés et vérifiés dans le temps. Elle protège l'entreprise contre un accès qui reste actif après qu'il n'est plus justifié, en particulier au départ d'un collaborateur.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R12.1 | Des accès nominatifs à l'arrivée | Seuls les accès nécessaires à la mission de la personne lui sont ouverts, sous une forme nominative, et consignés dans un registre des accès. Elle prend connaissance de cette politique et en atteste par écrit. | Registre des accès à jour ; parcours d'accueil consigné | ANSSI (hyg.) ; ISO A.5.9, A.5.15 à A.5.18, A.6.3 |
| R12.2 | Un retrait immédiat au départ | L'ensemble des accès d'une personne est retiré au plus tard le jour de son départ : comptes, clés, accès au coffre-fort de secrets, matériel et accès physiques restitués. | Liste de contrôle de départ signée ; délai entre le départ et le retrait des accès (cible : le jour même) | ANSSI (hyg.) ; ISO A.5.11, A.6.5 |
| R12.3 | Un filet pour les accès partagés | Un accès qui n'a pas pu être retiré immédiatement, par exemple sur un système partagé, fait l'objet d'un renouvellement des secrets concernés dans les jours qui suivent. | Liste des secrets renouvelés, datée | ANSSI (hyg.) ; ISO A.5.11, A.6.5 |
| R12.4 | Aucune réutilisation d'un accès laissé vacant | Un accès resté ouvert après le départ d'une personne n'est pas repris par quelqu'un d'autre : il est fermé. | Aucun accès d'une personne partie encore valable lors de la revue suivante | ANSSI (hyg.) ; ISO A.5.11, A.6.5 |
| R12.5 | Une revue périodique | Le référent sécurité compare à intervalle régulier le registre des accès aux comptes réellement actifs sur chaque système, et retire tout accès qui n'est plus justifié. | Compte rendu de revue daté ; nombre d'accès retirés | ANSSI (hyg.) ; ISO A.5.15 à A.5.18 |

## 13. Politique de sécurité physique

Cette politique fixe qui peut entrer dans les locaux et comment les équipements y sont protégés. Elle protège les postes de travail et les serveurs contre un accès physique non autorisé.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R13.1 | Un accès aux locaux limité à l'équipe | Les personnes extérieures à l'équipe n'entrent pas librement dans les locaux qui hébergent des postes de travail ou des serveurs. | Registre des clés à jour | ANSSI (hyg.) ; ISO A.5.11, A.7.2, A.7.3 |
| R13.2 | Des visiteurs accompagnés | Ils ne restent pas seuls dans les zones où se trouvent ces équipements. | Consigne communiquée à l'équipe | ANSSI (hyg.) ; ISO A.7.2, A.7.3 |
| R13.3 | Des serveurs sous protection | Les serveurs installés dans les locaux de l'entreprise sont placés dans un espace dont l'accès est restreint. | Fiche de contrôle annuelle datée ; serveurs sous clé | ANSSI (hyg.) ; ISO A.7.2, A.7.3 |

## 14. Politique de gestion des incidents de sécurité

Cette politique fixe la conduite à tenir face à un incident de sécurité soupçonné ou confirmé. Elle protège la capacité de l'entreprise à réagir vite, à limiter les conséquences d'un incident, et à en tirer les leçons. Dans les tableaux qui suivent, la mention « aucune règle propre à ce jour » signale qu'une plage de l'Annexe A de la norme couvre le sujet sans qu'aucune règle publiée ne la précise encore : elle n'affaiblit pas l'obligation posée par la règle elle-même.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R14.1 | Un signalement immédiat | Un incident de sécurité soupçonné ou confirmé, qu'il s'agisse d'une intrusion, d'une fuite de données, d'une perte d'accès ou d'un comportement anormal, est signalé sans délai au référent sécurité. | Fiche d'incident ouverte le jour du signalement | ISO A.5.24 à A.5.28 (aucune règle propre à ce jour) |
| R14.2 | Une qualification et une réaction rapides | Le référent sécurité qualifie la situation, prend les mesures immédiates nécessaires pour en limiter l'effet, et en informe la direction. | Délai entre le signalement et la première mesure de confinement | ISO A.5.24 à A.5.28 (aucune règle propre à ce jour) |
| R14.3 | Une information proportionnée à la gravité | Selon la nature de l'incident, les personnes concernées sont informées, y compris les clients dont les données seraient touchées, et l'autorité compétente est notifiée lorsque la réglementation l'exige et dans le délai qu'elle prévoit. | Personnes et autorité informées, avec la date | RGPD Art. 33, Art. 34 ; ISO A.5.24 à A.5.28 |
| R14.4 | Des preuves conservées | Les éléments de preuve sont conservés le temps nécessaire à l'analyse de l'incident. | Éléments de preuve archivés avec l'incident | ISO A.5.24 à A.5.28 (aucune règle propre à ce jour) |
| R14.5 | Un retour d'expérience systématique | Chaque incident donne lieu à une analyse de ce qui a permis sa survenue et de ce qui doit changer pour éviter sa répétition. | Fiche de retour d'expérience par incident | ISO A.5.24 à A.5.28 (aucune règle propre à ce jour) |

## 15. Sensibilisation et formation

Cette section ne fixe pas un domaine de règles comme les précédentes : elle décrit comment le reste de cette politique devient réel pour l'équipe, au lieu de rester un texte que personne ne lit. Une politique non comprise ne protège personne : cette section protège donc, indirectement, chacun des domaines couverts plus haut, contre les erreurs qui viennent d'une règle mal connue plutôt que d'une intention malveillante.

### À qui elle s'adresse

Les développeurs suivent le programme complet, pour appliquer les règles techniques et organisationnelles sur leurs propres outils. La direction suit le lancement puis une séance qui lui est propre, centrée sur les décisions de sécurité qui lui reviennent et sur la reconnaissance des tentatives de fraude, en raison de sa position et de l'étendue de ses propres accès. Les arrivants suivent un parcours d'accueil dès leur première semaine, plutôt que d'attendre le prochain cycle collectif. Les prestataires ne suivent pas cette formation : ils reçoivent par écrit les exigences qui les concernent.

### Le programme

Un lancement d'environ deux heures, puis six modules courts de quarante-cinq minutes chacun, placés avant que les règles qu'ils présentent ne s'appliquent, pour que l'équipe apprenne ce qu'elle va mettre en place plutôt que ce qu'elle aurait dû faire. La durée totale avoisine sept heures pour chaque développeur, réparties sur trois mois, à laquelle s'ajoute une séance d'une heure propre à la direction. Le jour de l'approbation de cette politique sert de point de départ, noté J ; les échéances ci-dessous sont des valeurs indicatives, ajustées par la direction selon le rythme réel de mise en œuvre.

| N° | Étape | Ce qu'elle couvre | Échéance |
|---|---|---|---|
| M0 | Lancement | Les dangers et les risques, présentés à partir de cas vécus ; présentation de cette politique ; engagement de la direction ; accusé de lecture signé par toute l'équipe | Jour de l'approbation (J) |
| M1 | Accès et secrets | Comptes individuels, coffre-fort de mots de passe, interdiction des secrets en clair, retrait des accès au départ | J + 10 jours |
| M2 | Postes et télétravail | Conditions du télétravail, appareil utilisé, protections des postes, sécurité des locaux | J + 20 jours |
| M3 | Serveurs et prestataires | Mises à jour, sauvegardes, journaux, exigences transmises aux prestataires | J + 45 jours |
| M4 | Développement et livraison | Relecture avant mise en production, tests, traçabilité des déploiements | J + 60 jours |
| M5 | Données personnelles | Registre des traitements, localisation des données, durées de conservation | J + 75 jours |
| M6 | Incident, exercice | Reconnaître, signaler et traiter un incident de sécurité | J + 90 jours |

La séance propre à la direction a lieu vers J + 15 jours, après le lancement et avant les premiers modules techniques de l'équipe.

### Le contenu d'un module

Chaque module part d'un cas concret, si possible vécu par l'entreprise et présenté sans nommer personne, plutôt que d'une liste de recommandations abstraites : un secret partagé par erreur, un accès resté ouvert après un départ, une connexion tentée depuis l'extérieur. Il se poursuit par une mise en pratique sur les outils réels de l'équipe, ce qui fait de la formation une étape du déploiement plutôt qu'un exercice à part : installer le coffre-fort de mots de passe et y transférer ses identifiants, activer une double authentification, vérifier qu'un accès à distance est bien protégé, remplir une ligne du registre des traitements pour un projet réel.

### Le déroulé et le contrôle des connaissances

Chaque module suit le même format en quatre temps : l'enjeu, à partir du cas vécu (cinq minutes) ; la règle expliquée en clair (dix minutes) ; la mise en pratique sur les outils (vingt minutes) ; un quiz de mises en situation, pas de définitions à réciter (dix minutes). Un aide-mémoire d'une page est remis avant chaque module.

Le quiz vérifie la compréhension, pas la mémoire : il pose une situation concrète et demande la bonne réaction, avec une reprise possible et une correction commentée en cas d'échec. Les résultats individuels ne sont pas diffusés ; un bilan global par module sert à améliorer son contenu, pas à évaluer les personnes.

### L'accompagnement continu

Le référent sécurité reste le premier contact pour toute question sur cette politique. Un point sécurité court, une fois par mois, rappelle une règle à la fois, choisie selon les anomalies constatées et l'actualité des menaces. L'exercice d'incident du module 6 est répété chaque année. Une charte informatique de deux pages reprend les règles d'usage quotidien et se signe à l'arrivée. Le parcours d'accueil d'un nouvel arrivant associe un collègue désigné, qui l'accompagne pendant son premier mois.

## 16. Accessibilité

Cette section fixe comment la formation reste utilisable par toute personne, quelle que soit sa situation. Elle protège le droit de chacun à suivre cette formation dans des conditions adaptées, sans avoir à se justifier.

### Le cadre

Le code du travail impose à l'employeur de prendre, pour un salarié qui en a besoin en raison de sa situation, les mesures appropriées pour qu'une formation adaptée lui soit dispensée, sous réserve que les charges qui en résultent ne soient pas disproportionnées ; refuser ces mesures sans motif peut constituer une discrimination. Cette obligation légale vise une situation précise ; cette politique va plus loin et retient le même principe pour tout besoin d'adaptation, durable ou passager, qu'il soit ou non déclaré à l'entreprise.

Les informations de santé qu'une adaptation peut révéler appartiennent aux catégories de données les plus protégées par la réglementation ; l'entreprise en recueille le moins possible, et seulement ce qui est nécessaire pour répondre à la demande. Un financement public peut exister pour couvrir le surcoût d'un aménagement lié à une situation de handicap ; son application à un besoin précis reste à vérifier auprès de l'organisme compétent le moment venu.

### Une accessibilité par défaut

Les supports et les sessions sont conçus pour rester accessibles sans qu'aucune demande soit nécessaire : structure de titres claire, police lisible, contrastes suffisants, aucune information portée par la seule couleur, texte de remplacement pour les images, format modifiable plutôt qu'un texte capturé en image, remise à l'avance. Les sessions restent courtes, avec une pause si besoin, dans un lieu calme, et peuvent être suivies à distance sur demande. La pratique reste réalisable au clavier, et le quiz ne s'appuie sur aucune limite de temps stricte.

### Une adaptation sur demande

| Besoin exprimé | Adaptations possibles |
|---|---|
| Difficulté à lire à l'écran | Supports en grande taille et très contrastés, compatibles avec un lecteur d'écran ; session sans support projeté |
| Difficulté à entendre | Supports écrits complets, transcription de ce qui est dit, lieu calme, matériel adapté si nécessaire |
| Difficulté de manipulation ou de mobilité | Pratique entièrement au clavier, temps supplémentaire, accompagnement pour les manipulations, accès à la salle vérifié |
| Difficulté de concentration, de lecture ou d'écriture | Supports courts et structurés, session fractionnée, temps supplémentaire pour le quiz, réponses à l'oral possibles |
| Situation passagère (blessure, fatigue, traitement) | Report ou fractionnement de la session, suivi à distance, rattrapage individuel |

Ces exemples ne sont pas une liste fermée : chaque adaptation se choisit avec la personne concernée, selon son besoin réel.

### Comment une adaptation est demandée

La personne s'adresse à la direction, à l'oral ou par écrit, à tout moment, sans avoir à détailler un motif médical. La direction répond par écrit, dans un délai annoncé à l'avance, en proposant une adaptation ou en motivant un refus fondé sur des charges disproportionnées. Le référent sécurité adapte ensuite les modalités pratiques avant la session concernée, et un retour après la première session suivie permet d'ajuster l'adaptation si besoin.

La direction reste seule destinataire de la demande. Elle ne transmet au référent que ce qui est nécessaire à l'adaptation elle-même, sans la cause. Les éventuels éléments de santé sont conservés séparément, dans un accès restreint, pour la durée strictement nécessaire, et n'apparaissent dans aucun registre de cette politique. Le coût de l'adaptation est assumé par l'entreprise, qui peut examiner les aides disponibles.

| N° | Mise en œuvre | Échéance |
|---|---|---|
| 16.1 | Rendre les supports de formation accessibles par défaut, selon les critères ci-dessus | Avant le lancement (M0), jour de l'approbation (J) |
| 16.2 | Faire connaître à l'équipe la possibilité de demander une adaptation | Jour de l'approbation (J) |

## 17. Contrôle, indicateurs et révision de la politique

Cette section fixe comment on vérifie que la politique est réellement appliquée, et comment elle évolue dans le temps. Elle protège l'entreprise contre une politique qui resterait sur le papier sans être suivie dans les faits.

### Un contrôle proportionné

Suivre chaque règle une à une n'est pas tenable pour une petite équipe, et une règle qui ne peut pas être contrôlée n'a pas sa place dans cette politique (principe de proportionnalité, section 3). Le contrôle se concentre donc sur l'essentiel : les exigences dont l'absence aurait le plus de conséquences reçoivent un indicateur suivi à intervalle rapproché ; les autres, un contrôle périodique moins fréquent, ou une simple vérification lors de leur mise en place.

### Le tableau de bord

Un petit nombre d'indicateurs suit, en continu, l'état réel de la sécurité de l'entreprise. Chacun se relève à partir d'une donnée qui existe déjà, une configuration, un journal ou un registre, en quelques minutes, sans enquête à part.

| Ce que l'entreprise veut savoir | Indicateur | Cible |
|---|---|---|
| Les comptes sont-ils individuels ? | Part des comptes ayant un titulaire identifié | 100 %, ou usage journalisé pour les comptes encore partagés |
| Un départ est-il traité à temps ? | Délai entre le départ d'une personne et le retrait de ses accès | Le jour même |
| Les accès distants sont-ils protégés ? | Part des accès protégés par une clé ou une double authentification | 100 %, ou décision documentée |
| Les secrets restent-ils dans le coffre-fort ? | Nombre de secrets trouvés ailleurs | Zéro |
| Les sauvegardes fonctionnent-elles vraiment ? | Ancienneté de la dernière sauvegarde réussie, et du dernier test de restauration | Moins de 24 heures ; un test par sauvegarde critique chaque semestre |
| Le code est-il relu avant la production ? | Part des mises en production ayant fait l'objet d'une relecture tracée | 100 % |
| Les incidents laissent-ils une trace exploitable ? | Durée de conservation des journaux de connexion | Suffisante pour une investigation rétroactive |
| Les décisions de la direction sont-elles suivies d'effet ? | Part des propositions ayant reçu une réponse écrite | 100 % |

Ces indicateurs sont un point de départ, pas une liste fermée : ils se complètent et s'ajustent à mesure que la politique est appliquée.

### Le compte rendu régulier

Le référent sécurité remet à la direction, à intervalle régulier, un compte rendu court qui reprend le tableau de bord, les écarts constatés et leur délai de correction, les dérogations arrivées à échéance, et ce qui nécessite une décision de la direction pour la période à venir.

### Combien de temps cela demande

Un contrôle qui prend trop de temps finit par ne plus être fait. Le tableau suivant donne un ordre de grandeur, une fois les règles mises en place, pour une équipe de la taille de celle de VNWeb.

| Cadence | Ce qu'elle comprend | Temps estimé |
|---|---|---|
| Hebdomadaire | Suivi des évolutions réglementaires et techniques qui pourraient toucher cette politique | Une heure |
| Mensuelle | Point sécurité de l'équipe, relevé des mises à jour en attente | Deux heures |
| Trimestrielle | Relevé complet du tableau de bord, compte rendu à la direction, revue des accès, mise à jour de l'inventaire des serveurs et logiciels | Une demi-journée |
| Semestrielle | Test de restauration d'une sauvegarde, vérification par le suppléant d'un échantillon de contrôles | Une demi-journée |
| Annuelle | Contrôle physique, séance de sensibilisation de la direction, revue complète de la politique, vérification des textes de référence cités en section 20 | Une à deux journées |

Cette charge reste réelle pour une petite équipe. Elle se réduit en la partageant entre le référent et son suppléant, en automatisant ce qui peut l'être, et en regroupant plusieurs contrôles dans un même moment plutôt que de les éparpiller.

### La révision de la politique

Elle a lieu au moins une fois par an, et sans attendre cette échéance après un incident important, un changement significatif dans l'organisation ou les systèmes de l'entreprise, ou une évolution de la réglementation applicable. Chaque révision reprend six points : l'évolution des indicateurs sur la période, les écarts et les dérogations à réexaminer, les évolutions réglementaires repérées depuis la dernière revue, les changements survenus dans l'entreprise, les règles à retirer ou à fusionner faute d'utilité ou de possibilité de contrôle, et la mise à jour des documents associés à cette politique.

Chaque révision produit une nouvelle version, numérotée, approuvée par la direction et portée à la connaissance de toute l'équipe.

| N° | Mise en œuvre | Échéance |
|---|---|---|
| 17.1 | Premier relevé complet du tableau de bord | J + 90 jours |
| 17.2 | Premier compte rendu trimestriel à la direction | J + 90 jours |
| 17.3 | Première revue complète de la politique | Au plus tard J + 12 mois |

## 18. Veille réglementaire et technique

Cette section fixe comment l'entreprise se tient informée de ce qui pourrait rendre une règle de cette politique dépassée ou insuffisante. Elle protège l'entreprise contre le risque qu'une politique écrite à un instant donné reste inchangée alors que les menaces, les textes de loi ou les technologies utilisées, eux, continuent d'évoluer.

### Quatre domaines surveillés

- **La conformité** : les évolutions du règlement sur la protection des données, des recommandations de la Commission nationale de l'informatique et des libertés et de l'Agence nationale de la sécurité des systèmes d'information, et de la norme utilisée comme grille de comparaison.
- **Les failles de sécurité** : les vulnérabilités connues qui touchent les logiciels, les services et les dépendances utilisés par l'entreprise.
- **Les menaces et les attaques** : l'hameçonnage, les rançongiciels et les techniques de fraude qui visent en particulier les petites structures.
- **La fin de support et les fournisseurs** : les versions et les services qui cesseront d'être maintenus, et les changements chez les prestataires et les fournisseurs utilisés par l'entreprise.

### Comment l'information est collectée et traitée

La collecte est aussi automatisée que possible, à partir d'un nombre limité de sources officielles reconnues pour chaque domaine. Le tri et le résumé de ce qui est collecté peuvent s'appuyer sur un outil, mais la décision reste humaine : le référent sécurité lit, vérifie à la source ce qui semble pertinent, et propose une suite à la direction.

Un élément de veille reste un signal, pas une preuve : il est vérifié à la source avant qu'une décision s'appuie dessus. Aucune règle de cette politique n'est modifiée sur la seule foi d'un tel signal : toute modification suit le chemin de décision décrit en section 4, avec une proposition écrite et une réponse de la direction, ou une dérogation documentée si la mesure n'est pas retenue.

### Le rythme

| Fréquence | Qui fait quoi, concrètement |
|---|---|
| Chaque semaine | Le référent sécurité consulte les sites officiels retenus pour la conformité (la CNIL, l'ANSSI) et pour les failles de sécurité des logiciels et services utilisés par l'entreprise, et note ce qui pourrait la concerner. |
| Chaque mois | Le référent rassemble en quelques lignes ce qu'il a repéré sur les menaces et les arnaques du moment, et sur les fournisseurs (fin de support annoncée, changement de service), et le présente pendant le point sécurité mensuel déjà prévu en section 15. |
| Dès qu'elle est repérée | Une faille de sécurité déjà exploitée par des attaquants, qui touche un logiciel ou un service utilisé par l'entreprise, est corrigée sans attendre le prochain point hebdomadaire ou mensuel. |
| Une fois par an | Le référent vérifie que les textes et la norme cités comme référence de cette politique (section 20) n'ont pas changé de version depuis la dernière vérification. |

### Qui fait quoi

Le référent sécurité pilote la veille, avec son suppléant en son absence. Chaque développeur surveille les composants et les dépendances de ses propres projets. L'ensemble de l'équipe reçoit le condensé mensuel, qui demande quelques minutes de lecture.

| N° | Mise en œuvre | Échéance |
|---|---|---|
| 18.1 | Mettre en place la collecte des sources de conformité et de failles de sécurité | J + 30 jours |
| 18.2 | Premier condensé mensuel diffusé à l'équipe | J + 45 jours |

## 19. Manquements

Cette politique fixe la réponse apportée à un manquement constaté à la politique. Elle protège l'équilibre entre la nécessité de faire respecter les règles et le droit de chacun à signaler une erreur sans crainte.

| N° | Règle | Explication | Preuve ou indicateur | Source normative |
|---|---|---|---|---|
| R19.1 | Un signalement, pas une sanction automatique | Tout manquement constaté à cette politique est signalé au référent sécurité, qui en informe la direction. | Manquement consigné, avec la date du signalement | ISO A.5.36 (aucune règle propre à ce jour) |
| R19.2 | Une réponse graduée | Elle commence par un rappel de la règle, puis, en cas de répétition ou de gravité, une mesure adaptée, dans le respect du cadre légal applicable. | Nature de la réponse consignée (rappel ou mesure) | ISO A.5.36 (aucune règle propre à ce jour) |
| R19.3 | Le signalement spontané n'est pas une faute | Une personne qui signale elle-même une erreur ou un incident qu'elle a commis n'est pas traitée comme fautive pour ce signalement. | Aucune sanction enregistrée pour un signalement spontané | - |

## 20. Cadre normatif

Cette politique s'appuie sur des référentiels reconnus, choisis en fonction de la taille et de l'activité de VNWeb : le guide d'hygiène informatique et la recommandation relative à l'authentification multifacteur et aux mots de passe de l'Agence nationale de la sécurité des systèmes d'information (ANSSI), les recommandations de la Commission nationale de l'informatique et des libertés (CNIL), les guides *Code Review Guide* et *DevSecOps Guideline* de la communauté OWASP pour le développement logiciel, le règlement général sur la protection des données (RGPD), et l'Annexe A de la norme ISO 27001:2022 utilisée comme grille de comparaison. Seul le règlement sur la protection des données constitue une obligation légale directe ; les autres référentiels guident la rédaction de cette politique sans l'y contraindre.

## 21. Approbation

| Champ | Valeur |
|---|---|
| Version | 0.1 |
| Date | À compléter |
| Rédigée par | Le référent sécurité |
| Référent sécurité désigné | À compléter |
| Suppléant désigné | À compléter |
| Approuvée par | La direction |
| Prochaine révision | Au plus tard 12 mois après l'approbation |

*La désignation du référent et de son suppléant, mentionnée en section 4, est actée par les deux lignes ci-dessus au moment de l'approbation ; le document est rédigé en amont par la personne pressentie pour cette fonction.*

---

*Fin de la politique de sécurité des systèmes d'information de VNWeb.*
