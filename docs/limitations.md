# Limitations documentées — Projet G8Way

Ce document recense, de façon factuelle et vérifiée, les limitations rencontrées au cours du projet.
Chaque point distingue une contrainte réelle (matérielle, logicielle ou d'accès) d'une erreur de
configuration — aucune des limitations ci-dessous n'a pu être corrigée dans le temps imparti, malgré
une démarche de diagnostic systématique.

## 1. Téléphonie IP (Asterisk)

Le paquet `asterisk` n'a pas de candidat installable dans les dépôts Debian 13 (trixie) à la date du projet.

**Vérifications effectuées :**
- Recherche via `apt-cache search asterisk` — présence confirmée dans les métadonnées mais sans candidat installable.
- Vérification via `apt-cache policy asterisk` — Candidat : (aucun).
- Ajout de la composante `contrib` dans `/etc/apt/sources.list`, puis `apt update` — résultat inchangé.
- Contrôle des dépôts main, contrib, non-free-firmware, security et updates.

**Conclusion :** écart de packaging propre à Debian 13, non lié à une erreur de configuration réseau ou
système. Le VLAN dédié est pleinement opérationnel et validé (connectivité, routage, DHCP) — seule
l'installation du logiciel de téléphonie est bloquée.

**Piste de résolution future :** installation depuis les sources, backport depuis Debian 12 (bookworm),
ou conteneur/image alternative packagée par un tiers.

## 2. Réseau Wi-Fi

Interface réseau préparée côté pare-feu (VLAN dédié, plan d'adressage réservé) mais aucune borne Wi-Fi
physique disponible pour la mise en œuvre et la validation. Aucune configuration bloquante identifiée —
la mise en service ne nécessite que le raccordement physique d'une borne compatible 802.1Q.

## 3. Contrainte d'accès physique aux commutateurs

Une partie des diagnostics et corrections réseau a nécessité un accès direct (console série) aux
commutateurs Cisco physiques du laboratoire, disponible uniquement en présentiel — tâches reportées en conséquence.

## 4. Dépendance de l'environnement virtuel à l'infrastructure physique du laboratoire

L'ensemble des VM est hébergé sur un poste physique dont le commutateur virtuel externe est relié au
réseau physique du laboratoire (commutateurs Cisco en mode trunk 802.1Q).

**Constat :** lorsque ce poste change de réseau, l'ensemble des VLAN simulés perd toute cohérence, le
routage inter-VLAN et l'accès Internet reposant entièrement sur les commutateurs et le pare-feu physiques
du laboratoire.

**Axe d'amélioration identifié :** une maquette réseau totalement virtualisée (switches virtuels avec
802.1Q simulé) permettrait une administration à distance complète, indépendante du lieu.

## 5. Vérification de cohérence du compte examinateur

Un écart potentiel de nommage entre le compte AD réellement testé lors de la validation VPN/RDP et le
nom de compte spécifié dans la procédure de référence reste à confirmer. La vérification nécessite un
accès direct au contrôleur de domaine — hors du périmètre volontairement restreint du VPN (accès DMZ
uniquement, LAN explicitement refusé). Point non bloquant : le test de moindre privilège a été validé
avec succès ; correction de cohérence documentaire reportée à la prochaine séance en présentiel.
