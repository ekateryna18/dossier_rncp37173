# Vérifications restantes, VNWeb

*Note de travail interne de l'auditrice, hors dossier officiel : elle ne fait pas partie de la politique de sécurité (`Politique_Securite_VNWeb.md`) ni du rapport d'audit (J1, J2). Elle sert à suivre les points qui restent à trancher avant de considérer le constat de la politique comme définitif.*

| # | Point à vérifier | Réf. dans la politique | Auprès de qui, ou comment |
|---|---|---|---|
| 1 | Rôle exact de chacun des deux serveurs locaux (développement, Gitea, Extranet) | 1.2 | Équipe technique |
| 2 | Nature de chaque prestataire (personne ou société), contrat écrit, serveur géré par chacun | VO-14, VO-15 | Patron |
| 3 | Nature des données hébergées sur le serveur des sites de voyance et en préproduction | VT-21, VO-17 | Patron, prestataires |
| 4 | Accessibilité du service SSH depuis Internet : essai de connexion depuis un réseau extérieur à l'entreprise (par exemple un partage de connexion de téléphone), sur chaque serveur | VT-02 | Auditrice |
| 5 | Configuration SSH effective (`sshd -T`) : mot de passe seul ou clé. Le compte d'audit n'ayant pas de droits sudo, un compte avec droits est nécessaire | VT-02 | Auditrice, avec le prestataire pour le serveur ScalarX |
| 6 | Pare-feu (`ufw` ou `iptables`) sur le serveur ScalarX et sur les serveurs locaux | VT-15 | Auditrice, prestataire |
| 7 | Niveau de mise à jour du serveur ScalarX (`apt update` puis `apt list --upgradable`) et configuration de fail2ban (jails, seuils) | VT-16 | Auditrice, prestataire |
| 8 | Audit complet des serveurs locaux et du serveur de staging au-delà de l'authentification SSH (confirmée faible, J2 écart 5) | VT-15, VT-16 | Auditrice |
| 9 | Protections des postes (antivirus, mises à jour, filtrage web) et chiffrement des disques | VT-12, VO-04 | Patron |
| 10 | Informations clients : transmission de leurs accès, emplacement de leurs données | VO-16 | Patron |
| 11 | Sauvegardes sur le serveur ScalarX par un autre mécanisme que cron | VT-13 | Prestataire |
| 12 | Segmentation réseau entre développement, préproduction et production ; volet TLS | VT-18 | Auditrice |
| 13 | Contrat d'hébergement de chaque site : conclu par VNWeb (qui le revend au client) ou par le client directement, ce qui détermine le rôle de VNWeb (sous-traitant du client, ou développeur qui intervient sur un hébergement du client) | VO-14, VO-17 | Patron |
| 14 | Points non qualifiables ou non testés listés par J2 (comptes IAM du « Groupe 0 », verrouillage Cognito, bucket S3 en PROD, le reste des tables DynamoDB, présence de secrets déjà commités, sensibilisation réelle de l'équipe) | - | Voir J2, section 2 |

---

*À jour au 28/09/2026. Quand un point est tranché, sa ligne est retirée d'ici et le statut correspondant dans la politique passe de « Déclaré » ou « À vérifier » à « Confirmé ».*
