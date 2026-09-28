# Segmentation réseau — Projet G8Way

## Plan VLAN

| VLAN | Réseau | Fonction |
|---|---|---|
| 1 | — (désactivé) | VLAN natif par défaut, volontairement inutilisé (bonne pratique sécurité) |
| 10 | 10.10.10.0/24 | LAN interne — contrôleurs de domaine, serveur GLPI/Zabbix, administration |
| 20 | 10.10.20.0/24 | Direction — postes clients |
| 30 | 10.10.30.0/24 | DMZ — serveur web (service exposé publiquement) |
| 40 | 10.10.40.0/24 | VOIP — téléphonie (voir [limitations](limitations.md)) |
| 90 | 10.0.0.0/30 | Sync — lien pfsync dédié entre les deux nœuds pare-feu (réplication d'état CARP) |
| — | 10.10.99.0/24 | Réseau tunnel VPN (OpenVPN) — accès distant tiers, isolé, restreint à la DMZ |

## Principe de segmentation retenu

La conception applique le **principe de moindre privilège** : chaque segment réseau n'a accès qu'aux
ressources strictement nécessaires à sa fonction.

- Une DMZ n'est pas isolée d'Internet — elle doit rester joignable en entrant (HTTP/HTTPS), c'est sa
  fonction. Ce qui est restreint : sa capacité à initier des connexions vers le LAN interne (rebond
  latéral) et son trafic sortant vers Internet (limité aux besoins de mise à jour).
- Un accès distant tiers (VPN) suit la même logique que la DMZ : autorisé uniquement vers la ressource
  pour laquelle il est habilité, refusé explicitement vers le LAN.
- Les contrôleurs de domaine, ressources les plus critiques du SI, n'ont besoin que de résolution DNS
  et de synchronisation horaire (NTP) vers l'extérieur — aucun accès Internet général nécessaire.

## Matrice de règles pare-feu (état cible)

| Source | Besoin identifié | Règle appliquée |
|---|---|---|
| VLAN 20 (utilisateurs classiques) | Accès Internet + accès LAN pour les services AD (DNS, Kerberos, SMB) | Pass vers LAN (AD/DNS/SMB) et WAN ; DMZ et VOIP refusés |
| Poste d'administration | Internet, ICMP (diagnostic), SSH vers les serveurs | Pass large : WAN, ICMP vers tous les segments, SSH (port 22) vers les serveurs |
| DMZ (serveur web) | Hébergement du site web uniquement | Entrant WAN → DMZ (80/443) ; sortant DMZ → WAN limité (mises à jour) ; DMZ → LAN et DMZ → VLAN 20 refusés |
| VOIP | Non traité — service non fonctionnel (voir limitations) | Règles existantes conservées sans durcissement pour le moment |
| Contrôleurs de domaine | DNS et NTP uniquement | Pass sortant limité aux ports 53 (DNS) et 123 (NTP) vers WAN ; reste bloqué |
| Réseau VPN (10.10.99.0/24) | Accès DMZ uniquement, aucun accès LAN | Pass vers DMZ ; Block explicite vers LAN ; Block vers tout le reste |

## Scénario de test — accès examinateur via VPN

Validation en moins de 5 minutes, sans configuration requise côté testeur :

1. Connexion VPN avec un profil client dédié.
2. Test d'accès autorisé : navigation vers le site hébergé en DMZ → doit s'afficher normalement.
3. Test de refus attendu : ping vers un contrôleur de domaine du LAN → aucune réponse (timeout).

| Test | Résultat attendu | Signification |
|---|---|---|
| Connexion VPN | Tunnel établi, IP obtenue dans le sous-réseau VPN | Authentification VPN fonctionnelle |
| Accès DMZ (site web) | Page affichée avec succès | L'utilisateur distant atteint les services autorisés |
| Accès LAN (ping) | Aucune réponse (timeout) | Le réseau interne reste hermétique à cet accès externe |

**Interprétation :** la combinaison « accès DMZ réussi + accès LAN refusé » démontre l'application
effective du principe de moindre privilège pour un accès distant tiers.
