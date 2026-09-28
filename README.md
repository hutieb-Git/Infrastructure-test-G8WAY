# Projet G8WAY — Infrastructure et Systèmes (projet de fin de formation TSSR)

## Contexte
Projet scolaire réalisé dans le cadre du titre Technicien Supérieur Systèmes et Réseaux (Niveau 5,
ADRAR Pôle Numérique) : réponse fictive à un appel d'offres pour la société **G8Way — Infrastructure
et Systèmes**, pour le compte d'un client fictif du secteur industriel.

Couvre les deux blocs de compétences du titre :
- **CCP1** — Exploiter les éléments de l'infrastructure et assurer le support aux utilisateurs
- **CCP2** — Maintenir l'infrastructure et contribuer à son évolution et à sa sécurisation

## Stack technique démontrée
- **Active Directory** : modèle AGDLP, GPO, partages SMB avec Access-Based Enumeration, GPP Drive Maps
- **Réseau** : VLAN, routage inter-VLAN sur switches Cisco réels, HSRP (haute disponibilité)
- **Sécurité périmétrique** : pfSense (règles par interface), CARP (HA pare-feu), OpenVPN (certificats,
  Client Specific Override, tunnel scindé)
- **Serveurs** : Debian 13, virtualisation Hyper-V (2 hôtes physiques)
- **Supervision / ITSM** : Zabbix, GLPI
- **Automatisation** (prévue au cahier des charges) : Python, Bash

## Architecture

**Hôte 1**

| VM | Rôle | Zone réseau |
|---|---|---|
| VM Client | Windows 10 — poste utilisateur | VLAN 20 |
| VMDC01 | Windows Server 2019 — contrôleur de domaine principal | VLAN 10 |
| VM pfSense 1 | Pare-feu — MASTER (CARP), OpenVPN | Interconnexion |
| VM Serveur Web | Debian 13 — hébergement web | VLAN 30 (DMZ) |
| SRV-LINUX01 | Debian 13 — GLPI + Zabbix | VLAN 10 |

**Hôte 2 (redondance / haute disponibilité)**

| VM | Rôle | Zone réseau |
|---|---|---|
| VMDC02 | Windows Server 2019 — contrôleur de domaine secondaire (réplication AD) | VLAN 10 |
| VM pfSense 2 | Pare-feu — BACKUP (CARP), OpenVPN | Interconnexion |

[Insère ici ton schéma d'architecture — export PNG depuis draw.io/diagrams.net]

## Documentation détaillée
- [Segmentation réseau & règles pare-feu](docs/segmentation.md)
- [Diagnostic & résolution de pannes](docs/troubleshooting.md)
- [Limitations documentées](docs/limitations.md)

## Objectifs du projet
- Isoler les flux entre services via VLAN et la segmentation DMZ/LAN, selon le principe de moindre privilège
- Assurer la continuité de service via la réplication AD, le HSRP et le CARP pfSense
- Fournir un accès distant sécurisé et filtré via OpenVPN (tunnel scindé, certificats par utilisateur)
- Superviser l'infrastructure en temps réel avec Zabbix et centraliser la gestion de parc avec GLPI

## Sécurité de la publication
Toutes les captures d'écran et extraits de commandes publiés ont été passés en revue : aucun mot de
passe réel, aucun identifiant nominatif. Les adresses IP conservées sont des plages privées (RFC 1918),
non routables publiquement.

## Compétences mobilisées
Active Directory (AGDLP, GPO, ABE, GPP), réseau Cisco (VLAN, routage inter-VLAN, HSRP), sécurité
périmétrique (pfSense, CARP), VPN (OpenVPN, certificats, tunnel scindé), diagnostic réseau avancé
(routage asymétrique, pare-feu à état, analyse TTL), supervision (Zabbix), gestion de parc (GLPI),
virtualisation Hyper-V, administration Debian, PowerShell
