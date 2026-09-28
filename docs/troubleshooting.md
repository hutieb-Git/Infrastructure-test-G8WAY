# Diagnostic & résolution de pannes — Projet G8Way

## Méthodologie
Utilisation systématique d'outils de test réseau pour éviter les faux positifs (ex. distinguer un
chemin LAN parasite d'un vrai tunnel VPN) : `ping -S` (choix de l'interface source), `Test-NetConnection`
(test de port ciblé), analyse du TTL pour identifier le chemin réel emprunté par un paquet.

## Chaîne d'incidents résolus

| # | Symptôme | Cause racine | Correction |
|---|---|---|---|
| 1 | Partage AD : dossiers/groupes vides, mauvaise affectation des utilisateurs | Groupes AD jamais peuplés + faux négatif dû à un artefact d'affichage console PowerShell (boucle `ForEach-Object`) | Reconstruction complète du modèle AGDLP + ABE, commandes isolées non bouclées pour vérification fiable |
| 2 | Accès au partage refusé malgré des permissions NTFS correctes | Permission de partage SMB plafonnée à "Lecture" + racine NTFS sans droit minimal de listage | `Grant-SmbShareAccess` + `icacls` racine en lecture seule non héritée |
| 3 | Déploiement VPN+RDP pour un poste examinateur externe : accès refusé | Certificat créé mais jamais lié au compte utilisateur (deux systèmes indépendants) | Association manuelle certificat ↔ utilisateur, Client Specific Override, export client |
| 4 | IP fixe VPN non appliquée (attribution DHCP au lieu du CSO) | Format `/30` incompatible avec la topologie serveur en Subnet (`/24` attendu) | Correction du masque dans le Client Specific Override |
| 5 | RDP fonctionnel depuis un poste, échec systématique depuis tout nouveau poste externe | Route réseau manquante (option "IPv4 Local Networks" du serveur OpenVPN ne poussait pas le sous-réseau cible) | Ajout de la route dans la configuration du serveur OpenVPN |
| 6 | Doublon de Client Specific Override pour la même identité | Résidu `/30` d'une correction précédente jamais supprimé | Suppression de l'entrée obsolète |
| 7 | Déconnexions VPN périodiques | Cause probable réseau/MTU | Ajustement préventif `tun-mtu` / `mssfix` |
| 8 | RDP toujours en échec après corrections réseau serveur | Pare-feu Windows (+ antivirus tiers) classait l'adaptateur VPN en "Réseau public", bloquant RDP sortant par défaut | `Set-NetConnectionProfile -NetworkCategory Private` |
| 9 | Accès GLPI/Zabbix impossible depuis un poste interne, fonctionnel depuis le sous-réseau du serveur | Routage asymétrique à travers un pare-feu à état : absence de route directe du serveur vers le VLAN client, réponses déviées via pfSense et rejetées (aucun état correspondant) | Route statique directe vers le VLAN via la passerelle du switch |
