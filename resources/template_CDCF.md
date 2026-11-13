# Cahier des Charges Fonctionnel (CDCF) : [Nom de l'application]

> **Objet** : [Une phrase décrivant l'application et ses utilisateurs.]
> **Version** : [1.0]
> **Date** : [JJ/MM/AAAA]
> **Rédigé par** : [Nom, fonction]

---

## 1. Présentation du projet

### 1.1 Contexte
[Organisation, contexte métier, origine du besoin.]

### 1.2 Problématique
[Difficultés actuelles que l'application doit résoudre.]

### 1.3 Objectifs
- [Objectif métier]
- [Objectif utilisateur]
- [Objectif opérationnel]

### 1.4 Périmètre
**Inclus**
- [Élément inclus]

**Hors périmètre**
- [Élément explicitement exclu]

### 1.5 Définitions
| Terme | Définition |
|---|---|
| [Terme] | [Définition] |

## 2. Utilisateurs et parties prenantes

### 2.1 Rôles
| Rôle | Description | Droits principaux |
|---|---|---|
| [Rôle] | [Qui est-ce ?] | [Ce qu'il peut faire] |

### 2.2 Parties prenantes
| Partie prenante | Responsabilité |
|---|---|
| Commanditaire | [Nom, rôle dans le projet] |
| Équipe de réalisation | [Nom, rôle dans le projet] |

## 3. Fonctionnalités

> Regrouper les fonctionnalités par domaine (un tableau par domaine). Une ligne par fonctionnalité : ce qu'elle fait, qui l'utilise, et les cas d'erreur importants. Les règles détaillées vont en section 6, les tests en section 11.

### 3.1 [Domaine, ex. : Comptes et accès]
| ID | Fonctionnalité | Comportement attendu | Acteurs |
|---|---|---|---|
| F-01 | [Nom] | [Description courte, cas d'erreur importants] | [Rôles] |

## 4. Parcours utilisateurs et navigation

### 4.1 Parcours principaux
**[Nom du parcours]** ([acteur]) : [étape 1] → [étape 2] → [étape 3]. [Variante ou exception éventuelle.]

### 4.2 Navigation
[Menus, onglets, pages accessibles sans compte.]

### 4.3 États particuliers
[Liste vide, chargement, succès, erreur, absence de connexion, session expirée.]

## 5. Interface utilisateur

### 5.1 Écrans
| Écran | Objectif | Utilisateurs |
|---|---|---|
| [Nom] | [Objectif] | [Rôles] |

### 5.2 Comportements et identité visuelle
- Formulaires et messages : [À préciser]
- Adaptation mobile / tablette / ordinateur : [À préciser]
- Identité visuelle : [Charte, logo, ressources]

## 6. Règles de gestion

> Règles métier précises et vérifiables. Citer chaque fonctionnalité par son nom suivi de son identifiant.

| ID | Règle | Fonctionnalité concernée |
|---|---|---|
| RG-01 | [Règle] | [Nom (F-xx)] |

## 7. Exigences non fonctionnelles

| ID | Catégorie | Exigence | Critère mesurable |
|---|---|---|---|
| ENF-01 | [Sécurité, performance, disponibilité, compatibilité, accessibilité, fiabilité, évolutivité, protection des données] | [Exigence] | [Critère] |

## 8. Données

### 8.1 Données traitées
| Donnée | Format | Obligatoire | Sensibilité |
|---|---|---|---|
| [Donnée] | [Format] | [Oui / Non] | [Faible / moyenne / élevée] |

### 8.2 Règles de validation
- [Format, cohérence, unicité]

### 8.3 Accès, conservation et suppression
- [Qui accède à quoi, durée de conservation, suppression, sauvegarde]

### 8.4 Modèle de données
[Renvoi vers le modèle conceptuel en annexe.]

## 9. Notifications

| Événement | Destinataire | Canal | Contenu |
|---|---|---|---|
| [Événement] | [Destinataire] | [Courriel / push] | [Contenu] |

[Préférences, échec d'envoi.]

## 10. Services externes

| Service | Objectif | Données échangées | En cas d'indisponibilité |
|---|---|---|---|
| [Service] | [Objectif] | [Données] | [Comportement attendu] |

## 11. Recette

### 11.1 Organisation
[Qui valide, sur quel environnement, à quel moment.]

### 11.2 Tests d'acceptation
| ID | Fonctionnalité | Situation et action | Résultat attendu |
|---|---|---|---|
| T-01 | [Nom (F-xx)] | [Étant donné..., lorsque...] | [Alors...] |

### 11.3 Conditions de validation
- [Critère global, anomalies bloquantes, preuves attendues]

## 12. Priorisation et évolutions

### 12.1 Priorités (méthode MoSCoW)
| Priorité | Fonctionnalités |
|---|---|
| Must have (indispensable) | [Noms] |
| Should have (important) | [Noms] |
| Could have (souhaitable) | [Noms] |
| Won't have (exclu) | [Noms] |

### 12.2 Évolutions prévues
| ID | Évolution | Exigence ou fonctionnalité liée | Priorité |
|---|---|---|---|
| EV-01 | [Évolution] | [Nom (ID)] | [Priorité] |

### 12.3 Jalons
| Jalon | Responsable | Échéance |
|---|---|---|
| [Jalon] | [Nom] | [Date ou condition] |

## 13. Contraintes, hypothèses et risques

### 13.1 Contraintes
- [Budget, délais, réglementation, organisation, contraintes techniques imposées]

### 13.2 Dépendances et hypothèses
- [Dépendance ou hypothèse]

### 13.3 Risques
| Risque | Impact | Probabilité | Réponse |
|---|---|---|---|
| [Risque] | [Faible / moyen / élevé] | [Faible / moyenne / élevée] | [Mesure] |

## 14. Annexes
- [Modèle conceptuel de données, matrice des droits, maquettes, documents de référence]

## 15. Validation du document

| Partie prenante | Nom | Rôle | Décision | Date |
|---|---|---|---|---|
| [Organisation] | [Nom] | [Rôle] | | |

---

## Conseils d'utilisation

1. Le CDCF décrit ce que l'application doit faire, pour qui et selon quelles règles, sans imposer de solution technique.
2. Rester court : une ligne par fonctionnalité, les détails vont dans les règles de gestion et les tests.
3. Donner un identifiant seulement aux éléments cités ailleurs dans le document (fonctionnalités, règles, exigences, tests, évolutions), et l'écrire à côté du nom : « Inscription (F-01) » plutôt que « F-01 » seul.
4. Rédiger des critères mesurables et des tests vérifiables.
5. Garder les questions ouvertes et les notes de travail dans un document séparé : le CDCF livré est un document propre.
