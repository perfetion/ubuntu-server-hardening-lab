# Déploiement et sécurisation d'un serveur Linux sous Ubuntu

Projet personnel d'administration système et de cybersécurité : installation, durcissement, vérification et documentation d'un serveur **Ubuntu Server 26.04.1 LTS** dans une machine virtuelle **VirtualBox**, administré uniquement par SSH depuis un poste Ubuntu Desktop 24.04 LTS.

**Auteur :** MBWINI MUGBAKRANDE Geni · **Date :** octobre 2026

> 📄 **Rapport complet (37 pages, 57 captures) :** [`docs/Rapport_Securisation_Serveur_Ubuntu.pdf`](docs/Rapport_Securisation_Serveur_Ubuntu.pdf)

## Problématique

Comment déployer un serveur Ubuntu administrable à distance, puis réduire sa surface d'attaque et le rendre résistant aux tentatives d'intrusion courantes, sans jamais perdre l'accès d'administration, et en prouvant l'efficacité de chaque mesure ?

## Architecture

| Élément | Configuration |
|---|---|
| Hôte | HP ProBook 640 G2, Ubuntu Desktop 24.04 LTS, ~8 Go de RAM |
| Virtualisation | Oracle VirtualBox |
| Serveur | Ubuntu Server 26.04.1 LTS (noyau 7.0.0-38-generic), 4 Go de RAM, 2 vCPU, 25 Go |
| Réseau | Adaptateur 1 NAT (`enp0s3`, mises à jour) · Adaptateur 2 Host-Only (`enp0s8`, 192.168.56.101/24, administration) |
| Comptes | `genius_24` (administrateur), `analyste` (standard, sans sudo) |

## Mesures mises en œuvre

| Risque | Mesure | Preuve |
|---|---|---|
| Scan de ports | UFW : refus par défaut, SSH limité à `192.168.56.0/24` | Nmap : 1 023 ports filtrés, seul le 22 est ouvert |
| Force brute SSH | Fail2ban (3 échecs, ban 15 min) + `MaxAuthTries 3` | Adresse bannie puis débannie, journaux |
| Vol de mot de passe | Clés Ed25519, mots de passe désactivés | `Permission denied (publickey)` |
| Compromission de root | `PermitRootLogin no` | `sshd -T` |
| Compte non autorisé | `AllowGroups sshusers`, moindre privilège | `sudo -l` refusé pour `analyste` |
| Perte de données | Archive datée, restauration testée, copie hors serveur | `diff` identique, fichier restauré |
| Configuration faible | Audit Lynis avant/après | Indice 66 → 67, aucun warning |

## Résultats

- **Ports exposés :** 22/tcp uniquement, et seulement depuis le réseau d'administration.
- **Authentification :** clés SSH uniquement ; root interdit ; accès limité au groupe `sshusers`.
- **Détection :** Fail2ban opérationnel (détection, bannissement, débannissement prouvés dans les journaux).
- **Audit Lynis 3.1.6 :** indice de durcissement **66 → 67**, 257 tests, aucun warning.

## Contenu du dépôt

```
.
├── README.md
├── docs/            Rapport complet (PDF)
├── captures/        57 captures d'écran numérotées dans l'ordre du rapport
├── configs/         Configurations appliquées (SSH, Fail2ban, UFW)
├── commandes.md     Journal des commandes par étape
├── .gitignore
└── LICENSE
```

## Démarche

Étape par étape : observer avant de modifier, modifier de façon réversible, vérifier par un test. Le réseau et SSH ont été validés avant le pare-feu, les clés testées avant la désactivation des mots de passe, et `sshd -t` utilisé avant chaque rechargement. Les incidents rencontrés et leur résolution figurent au chapitre 7 du rapport.

## Limites et pistes d'amélioration

- Sauvegarde manuelle (pas de planification `cron`) ; copie hors serveur stockée sur l'hôte.
- Pas de mot de passe GRUB ni de `/home` séparé ; `jail.local` non créé.
- Laboratoire isolé : les paramètres (ex. `ignoreip`) sont à réexaminer pour un usage en production.

## Avertissement

Projet pédagogique réalisé dans un réseau de laboratoire isolé. Les adresses IP et noms de comptes sont ceux du laboratoire. Aucune clé privée ni mot de passe n'est publié dans ce dépôt.

## Licence

[MIT](LICENSE)
