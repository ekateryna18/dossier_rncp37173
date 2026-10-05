# Grille de conformité — ISO/IEC 27001:2022, Annexe A

### VNWeb

*Ce document sert à vérifier, contrôle par contrôle, que les citations ISO 27001 de `Politique_Securite_VNWeb.md` et `PSSI_VNWeb.md` correspondent vraiment au texte de la norme. Il complète `Sources_Citees_VNWeb.md`, qui documente déjà les autres référentiels (ANSSI, CNIL, OWASP, RGPD) avec un niveau de vérification plus élevé. Pour l'ISO 27001, le texte est payant et n'a pas été ouvert en session : chaque numéro de contrôle mentionné dans la politique vient d'une connaissance générale, pas d'une lecture directe. Cette grille existe pour fermer cet écart.*

*Rappel du cadre (section 20 de `PSSI_VNWeb.md`) : VNWeb utilise l'Annexe A de l'ISO 27001:2022 comme grille de comparaison, pas comme objectif de certification. Cette grille vérifie donc une cohérence déclarée, pas une conformité certifiable par un organisme accrédité.*

---

## Comment remplir cette grille

1. Se procurer le texte réel de la norme (achat sur iso.org ou la boutique AFNOR, ou accès via une bibliothèque universitaire — à demander d'abord à l'établissement).
2. Pour chaque ligne, ouvrir la norme au numéro de contrôle indiqué et reporter son intitulé exact dans la colonne prévue.
3. Comparer cet intitulé à la ou les règles VNWeb qui s'y réfèrent : la correspondance est-elle réelle, approximative, ou le numéro est-il erroné ?
4. Cocher la colonne « Vérifié », et noter tout écart dans la dernière colonne.
5. Une fois la grille complète, reporter les corrections nécessaires dans `PSSI_VNWeb.md`, `Politique_Securite_VNWeb.md` et `Sources_Citees_VNWeb.md`, puis conserver cette grille comme preuve de vérification dans le dossier.

Une ligne où le numéro de contrôle s'avère faux ou inexistant doit être corrigée dans la politique avant l'approbation : une source erronée dans un document destiné à la direction affaiblirait tout le reste du dossier.

---

## La grille

| N° | Contrôle(s) ISO cité(s) | Règle(s) VNWeb concernées | Intitulé exact du contrôle (à reporter depuis le texte réel) | Vérifié | Écart constaté |
|---|---|---|---|---|---|
| 1 | A.5.2 | R4.1 | | | |
| 2 | A.5.3 | R5.2 | | | |
| 3 | A.5.4 | R4.1, R4.2 | | | |
| 4 | A.5.6 | R8.2 | | | |
| 5 | A.5.7 | R8.2 | | | |
| 6 | A.5.9 | R8.1, R12.1 | | | |
| 7 | A.5.11 | R12.2, R12.3, R12.4, R13.1 | | | |
| 8 | A.5.15 à A.5.18 | R5.1, R5.2, R5.3, R5.4, R6.1, R6.2, R6.3, R12.1, R12.5 | | | |
| 9 | A.5.19 | R4.5, R9.1, R9.2, R9.4 | | | |
| 10 | A.5.20 | R4.5, R9.1, R9.2, R9.4 | | | |
| 11 | A.5.23 | R11.2 | | | |
| 12 | A.5.24 à A.5.28 | R14.1, R14.2, R14.3, R14.4, R14.5 | | | |
| 13 | A.5.31 | R11.1, R11.4 | | | |
| 14 | A.5.34 | R11.1, R11.4 | | | |
| 15 | A.5.36 | R19.1, R19.2 | | | |
| 16 | A.6.3 | R12.1 | | | |
| 17 | A.6.5 | R12.2, R12.3, R12.4 | | | |
| 18 | A.6.7 | R5.1, R5.5, R7.2, R7.3, R7.4 | | | |
| 19 | A.7.2 | R13.1, R13.2, R13.3 | | | |
| 20 | A.7.3 | R13.1, R13.2, R13.3 | | | |
| 21 | A.8.1 | R7.1, R7.2, R7.3 | | | |
| 22 | A.8.2 | R5.2, R5.5 | | | |
| 23 | A.8.5 | R5.2, R5.5 | | | |
| 24 | A.8.8 | R8.1, R8.2, R10.5 | | | |
| 25 | A.8.13 | R4.5, R8.4, R8.5, R9.1, R9.2 | | | |
| 26 | A.8.15 | R8.6 | | | |
| 27 | A.8.16 | R8.6 | | | |
| 28 | A.8.25 | R10.1, R10.2, R10.4 | | | |
| 29 | A.8.29 | R10.1, R10.2, R10.4 | | | |
| 30 | A.8.31 | R8.3 | | | |
| 31 | A.8.32 | R10.1, R10.2, R10.4 | | | |
| 32 | A.8.33 | R8.3 | | | |

30 numéros de contrôle distincts, regroupant les 46 règles de `PSSI_VNWeb.md` qui citent l'ISO comme source.

---

## Ce que cette grille ne remplace pas

Elle vérifie que les citations sont exactes, pas que VNWeb applique réellement chaque contrôle : ça, c'est le rôle du tableau de bord et des indicateurs de la section 17 de `PSSI_VNWeb.md`. Les deux documents sont complémentaires : cette grille valide la source, le tableau de bord valide l'application.

---

*À jour au 28/09/2026. Grille à remplir une fois le texte de la norme obtenu ; les corrections qui en résultent sont reportées dans `PSSI_VNWeb.md`, `Politique_Securite_VNWeb.md` et `Sources_Citees_VNWeb.md`.*
