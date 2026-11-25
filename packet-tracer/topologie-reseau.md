# Topologie réseau sous Cisco Packet Tracer

## Contexte et objectif

En complément du labo CYBER-LAB sous VirtualBox, Cisco Packet Tracer est utilisé pour apprendre et visualiser les bases du réseau (modèle OSI, routage) dans un simulateur dédié, avant de progresser vers des notions plus avancées comme le DHCP, les VLAN, les ACL et le NAT.

## Première topologie

Une première topologie est montée avec deux réseaux distincts, 192.168.1.0/24 et 192.168.2.0/24, reliés par un routeur. Le routeur est configuré à la fois via l'interface graphique et en ligne de commande (CLI Cisco), avec attribution des adresses IP et activation des interfaces (no shutdown). Les passerelles par défaut sont configurées sur les PC de chaque réseau, et un ping fonctionnel entre les deux réseaux confirme que le routage de base fonctionne correctement.

## Progression prévue

La suite logique de l'apprentissage est planifiée dans l'ordre : DHCP, puis VLAN, puis ACL, puis NAT, avec à chaque étape une explication concrète plutôt que purement théorique, pour bien comprendre le rôle de chaque brique avant de l'empiler sur la précédente.

## Lien avec le labo CYBER-LAB

Décision est prise de recréer dans Packet Tracer la topologie réelle du labo CYBER-LAB (VirtualBox), afin de mieux visualiser le rôle de chaque machine (Ubuntu-DC, Ubuntu-01, WIN-AD-DC01, Win-Client) et leurs interactions, en la représentant sous une forme simplifiée et purement réseau, séparée des détails de configuration système propres à chaque VM.

## Ce que ça m'a appris

Pratiquer en parallèle sur un simulateur dédié (Packet Tracer) et sur un labo de VMs réelles (VirtualBox) permet de séparer les deux dimensions d'un même apprentissage : la logique réseau pure, plus facile à visualiser et à faire évoluer rapidement dans un simulateur, et la complexité de configuration système réelle, propre à chaque système d'exploitation. Recréer une topologie déjà connue (celle du labo CYBER-LAB) dans un nouvel outil est aussi un bon moyen de vérifier qu'elle est bien comprise, au-delà de sa simple mise en œuvre pratique.
