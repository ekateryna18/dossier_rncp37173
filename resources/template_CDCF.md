# Cahier des Charges Fonctionnel (CDCF) — Application

> **Document de référence** : modèle à compléter pour cadrer les besoins fonctionnels d'une application web ou mobile.  
> **Version** : [0.1]  
> **Date** : [JJ/MM/AAAA]  
> **Rédigé par** : [Nom / équipe]  
> **Statut** : [Brouillon / En revue / Validé]

---

## Historique des versions

| Version | Date | Auteur | Modifications |
|---|---|---|---|
| 0.1 | [JJ/MM/AAAA] | [Nom] | Création du document |

## 1. Présentation générale du projet

### 1.1 Contexte
[Décrire l'organisation, le contexte métier et l'origine du besoin.]

### 1.2 Problématique
[Quel problème l'application doit-elle résoudre ? Quels sont les besoins ou difficultés actuels ?]

### 1.3 Objectifs du projet
- [Objectif métier 1]
- [Objectif utilisateur 1]
- [Objectif opérationnel ou commercial 1]

### 1.4 Vision du produit
[Décrire la proposition de valeur et la finalité de l'application.]

### 1.5 Périmètre du projet
**Inclus dans le périmètre**
- [Élément inclus]

**Hors périmètre**
- [Élément explicitement exclu]

### 1.6 Définitions et acronymes
| Terme | Définition |
|---|---|
| MVP | Minimum Viable Product — première version apportant une valeur minimale aux utilisateurs |
| [Terme] | [Définition] |

## 2. Utilisateurs et parties prenantes

### 2.1 Public cible
[Décrire les catégories d'utilisateurs visées.]

### 2.2 Profils utilisateurs et rôles

| Rôle | Description | Objectifs principaux | Droits / permissions |
|---|---|---|---|
| Visiteur | [À compléter] | [À compléter] | [À compléter] |
| Utilisateur inscrit | [À compléter] | [À compléter] | [À compléter] |
| Administrateur | [À compléter] | [À compléter] | [À compléter] |
| Autre | [À compléter] | [À compléter] | [À compléter] |

### 2.3 Parties prenantes
| Partie prenante | Responsabilité / contribution |
|---|---|
| Commanditaire | [À compléter] |
| Référent métier | [À compléter] |
| Product owner | [À compléter] |
| Équipe de réalisation | [À compléter] |
| Autre | [À compléter] |

## 3. Périmètre fonctionnel et fonctionnalités

### 3.1 Vue d'ensemble des modules

| ID | Module | Description | Priorité |
|---|---|---|---|
| MOD-001 | Authentification | Inscription, connexion, déconnexion, récupération du mot de passe | [Must / Should / Could / Won't] |
| MOD-002 | Profil utilisateur | Consultation et modification du profil | [Priorité] |
| MOD-003 | Tableau de bord | Affichage des informations principales | [Priorité] |
| MOD-004 | Recherche | Recherche, filtres et tri | [Priorité] |
| MOD-005 | Notifications | Réception et gestion des notifications | [Priorité] |
| MOD-006 | Administration | Gestion des utilisateurs et des contenus | [Priorité] |
| MOD-007 | [Module] | [Description] | [Priorité] |

### 3.2 Fiches détaillées des fonctionnalités

> Dupliquer cette fiche pour chaque fonctionnalité. Attribuer un identifiant unique à chaque besoin (FCT-001, FCT-002, etc.).

#### FCT-001 — [Nom de la fonctionnalité]

- **Module :** [Module associé]
- **Priorité :** [Must have / Should have / Could have / Won't have]
- **Acteur(s) :** [Rôle(s) concerné(s)]
- **Objectif :** [Résultat recherché par l'utilisateur]
- **Description :** [Comportement attendu de la fonctionnalité]
- **Préconditions :**
  - [Condition nécessaire avant le démarrage]
- **Déclencheur :** [Action ou événement qui lance le processus]

**Scénario nominal**
1. [Étape 1]
2. [Étape 2]
3. [Étape 3]
4. [Résultat attendu]

**Scénarios alternatifs et cas d'erreur**
- [Cas alternatif ou erreur] → [Comportement attendu]
- [Cas alternatif ou erreur] → [Comportement attendu]

**Règles de gestion**
- [RG-001 : règle applicable]
- [RG-002 : règle applicable]

**Données manipulées**
- [Donnée, format, caractère obligatoire ou facultatif]

**Critères d'acceptation**
- [ ] Étant donné [contexte], lorsque [action], alors [résultat attendu].
- [ ] [Autre critère vérifiable]

**Écrans concernés / références**
- [Nom de l'écran, lien vers maquette ou annexe]

---

## 4. Parcours utilisateurs et navigation

### 4.1 Parcours principaux
Décrire les parcours de bout en bout permettant aux utilisateurs d'atteindre leurs objectifs.

#### PAR-001 — [Nom du parcours]
- **Acteur :** [Rôle]
- **Objectif :** [Objectif du parcours]
- **Point d'entrée :** [Écran ou événement de départ]
- **Point de sortie :** [Écran ou état final]

**Étapes**
1. [Étape]
2. [Étape]
3. [Étape]

**Variantes / exceptions**
- [Situation et comportement attendu]

### 4.2 Structure de navigation
[Insérer un schéma de navigation ou décrire les menus, onglets, liens et retours.]

### 4.3 États particuliers
[Décrire les états vide, chargement, succès, erreur, absence de connexion, session expirée, etc.]

## 5. Exigences relatives à l'interface utilisateur (UI/UX)

### 5.1 Inventaire des écrans

| ID | Écran | Objectif | Utilisateurs concernés | Référence / maquette |
|---|---|---|---|---|
| SCR-001 | Écran d'accueil | [À compléter] | [À compléter] | [Lien] |
| SCR-002 | Connexion | [À compléter] | [À compléter] | [Lien] |
| SCR-003 | Inscription | [À compléter] | [À compléter] | [Lien] |
| SCR-004 | Tableau de bord | [À compléter] | [À compléter] | [Lien] |
| SCR-005 | Profil | [À compléter] | [À compléter] | [Lien] |
| SCR-006 | Paramètres | [À compléter] | [À compléter] | [Lien] |

### 5.2 Comportements attendus
- Navigation et retour : [À préciser]
- Formulaires et champs : [À préciser]
- Messages de confirmation : [À préciser]
- Messages d'erreur : [À préciser]
- États de chargement et états vides : [À préciser]
- Adaptation mobile / tablette / ordinateur : [À préciser]
- Accessibilité : [À préciser]

### 5.3 Identité visuelle et ressources
[Référencer la charte graphique, les composants, les icônes et les ressources disponibles.]

## 6. Règles de gestion

Lister les règles métier qui conditionnent le comportement de l'application. Elles doivent être précises, non ambiguës et vérifiables.

| ID | Règle de gestion | Fonctionnalité(s) concernée(s) | Cas d'application |
|---|---|---|---|
| RG-001 | [Ex. : une adresse e-mail ne peut être associée qu'à un seul compte] | [FCT-001] | [À compléter] |
| RG-002 | [Règle] | [ID] | [À compléter] |
| RG-003 | [Règle] | [ID] | [À compléter] |

## 7. Exigences non fonctionnelles

Préciser les niveaux attendus et, lorsque c'est possible, les rendre mesurables.

| ID | Catégorie | Exigence | Niveau / critère attendu |
|---|---|---|---|
| ENF-001 | Performance | [Temps de réponse attendu] | [Ex. : 95 % des pages chargées en moins de X secondes] |
| ENF-002 | Sécurité | [Exigences d'authentification et de protection] | [À préciser] |
| ENF-003 | Confidentialité | [Protection des données personnelles] | [Conformité applicable, notamment RGPD si concerné] |
| ENF-004 | Disponibilité | [Disponibilité attendue] | [À préciser] |
| ENF-005 | Compatibilité | [Navigateurs, systèmes et appareils supportés] | [À préciser] |
| ENF-006 | Accessibilité | [Référentiel et niveau visés] | [À préciser] |
| ENF-007 | Fiabilité | [Gestion des erreurs et cohérence des données] | [À préciser] |
| ENF-008 | Évolutivité | [Évolution du nombre d'utilisateurs ou des fonctionnalités] | [À préciser] |

## 8. Données et gestion des informations

### 8.1 Données à collecter et traiter

| Donnée | Description | Format | Obligatoire ? | Source | Sensibilité / protection |
|---|---|---|---|---|---|
| [Donnée] | [Description] | [Format] | [Oui / Non] | [Utilisateur / système / API] | [À préciser] |

### 8.2 Règles de validation
- [Règle de format]
- [Règle de cohérence]
- [Règle de caractère obligatoire]
- [Règle de déduplication]

### 8.3 Accès, conservation et suppression
- Qui peut consulter ou modifier chaque catégorie de données ?
- Combien de temps les données sont-elles conservées ?
- Dans quelles conditions peuvent-elles être exportées, rectifiées ou supprimées ?
- Quelles règles de sauvegarde et de traçabilité s'appliquent ?

### 8.4 Modèle de données
[Ajouter si utile un modèle conceptuel de données (MCD), un diagramme ou une description des principales entités et relations.]

## 9. Notifications et communications

| ID | Événement déclencheur | Destinataire | Canal | Contenu / objectif | Conditions |
|---|---|---|---|---|---|
| NOT-001 | [Ex. : compte créé] | [Utilisateur] | [E-mail] | [Confirmation] | [À préciser] |
| NOT-002 | [Événement] | [Destinataire] | [Push / SMS / in-app] | [À préciser] | [À préciser] |

Préciser également :
- Les préférences de notification configurables par l'utilisateur.
- Les notifications obligatoires ou désactivables.
- Les règles de fréquence et de regroupement.
- Le comportement en cas d'échec d'envoi.

## 10. Interfaces et intégrations avec des systèmes externes

| ID | Système / service | Objectif | Données échangées | Sens de l'échange | Contraintes |
|---|---|---|---|---|---|
| INT-001 | [Ex. : prestataire de paiement] | [Paiement en ligne] | [À préciser] | [Entrant / sortant / bidirectionnel] | [À préciser] |
| INT-002 | [Service] | [Objectif] | [À préciser] | [Sens] | [À préciser] |

Pour chaque intégration, préciser si possible :
- Les fonctionnalités concernées.
- Les événements et données échangés.
- Les conditions d'indisponibilité et les solutions de repli.
- Les responsabilités du fournisseur externe.

## 11. Critères d'acceptation et stratégie de validation

### 11.1 Principes de validation
[Décrire qui valide, sur quel environnement, à quel moment et selon quelle procédure.]

### 11.2 Cas de test d'acceptation

| ID | Fonctionnalité | Préconditions | Action / scénario | Résultat attendu | Statut |
|---|---|---|---|---|---|
| TEST-001 | [FCT-001] | [À préciser] | [À préciser] | [À préciser] | [À tester] |
| TEST-002 | [ID] | [À préciser] | [À préciser] | [À préciser] | [À tester] |

### 11.3 Conditions de validation
- [Critère de validation global]
- [Anomalies bloquantes acceptables ou non]
- [Livrables ou preuves attendus]

## 12. Priorisation et périmètre du MVP

Utiliser la méthode MoSCoW pour classer les besoins.

| Priorité | Signification | Fonctionnalités concernées |
|---|---|---|
| Must have | Indispensable à la première version | [IDs] |
| Should have | Important, mais non bloquant pour le MVP | [IDs] |
| Could have | Amélioration souhaitable | [IDs] |
| Won't have (pour cette version) | Explicitement reporté ou exclu | [IDs] |

### 12.1 Périmètre du MVP
[Décrire précisément les fonctionnalités livrées dans la première version.]

### 12.2 Évolutions envisagées
[Décrire les fonctionnalités envisagées pour les versions ultérieures.]

## 13. Contraintes, dépendances et hypothèses

### 13.1 Contraintes
- Budget : [À préciser]
- Délais et échéances : [À préciser]
- Contraintes réglementaires ou contractuelles : [À préciser]
- Contraintes organisationnelles : [À préciser]
- Contraintes techniques déjà imposées : [À préciser, le cas échéant]

### 13.2 Dépendances
[Décrire les services, équipes, données ou décisions dont le projet dépend.]

### 13.3 Hypothèses
- [Hypothèse à confirmer]
- [Hypothèse à confirmer]

### 13.4 Risques identifiés
| Risque | Impact potentiel | Probabilité | Mesure de prévention / réponse |
|---|---|---|---|
| [Risque] | [Faible / moyen / élevé] | [Faible / moyenne / élevée] | [À préciser] |

## 14. Livrables et jalons (si applicable)

| Livrable / jalon | Description | Responsable | Date cible |
|---|---|---|---|
| Validation du CDCF | [Périmètre et besoins validés] | [Nom / rôle] | [Date] |
| Maquettes validées | [Écrans et parcours] | [Nom / rôle] | [Date] |
| Recette fonctionnelle | [Tests et validation] | [Nom / rôle] | [Date] |
| Mise en production | [Lancement] | [Nom / rôle] | [Date] |

## 15. Annexes

- [Lien vers les maquettes / prototypes]
- [Diagrammes de parcours utilisateurs]
- [Diagrammes de cas d'utilisation UML]
- [Modèle conceptuel de données]
- [Charte graphique]
- [Documents réglementaires ou contractuels]
- [Glossaire complémentaire]

## 16. Validation du document

| Partie prenante | Nom | Rôle | Décision | Date |
|---|---|---|---|---|
| [Nom] | [Nom] | [Commanditaire] | [À valider / Validé] | [Date] |
| [Nom] | [Nom] | [Référent métier] | [À valider / Validé] | [Date] |

---

## Conseils d'utilisation

1. Commencer par définir les objectifs, les utilisateurs et le périmètre avant de détailler les fonctionnalités.
2. Attribuer un identifiant unique à chaque module, fonctionnalité, règle métier et exigence.
3. Décrire les comportements attendus du point de vue de l'utilisateur, sans imposer prématurément une solution technique.
4. Rédiger des critères d'acceptation observables et testables.
5. Distinguer clairement les besoins indispensables au MVP des évolutions futures.
6. Faire relire et valider le document par les parties prenantes concernées.
7. Mettre à jour l'historique des versions à chaque modification significative.

> **À retenir :** le CDCF décrit ce que l'application doit faire, pour quels utilisateurs et selon quelles règles. Les choix détaillés d'architecture, de technologies, d'hébergement et d'implémentation relèvent généralement du cahier des charges technique.
