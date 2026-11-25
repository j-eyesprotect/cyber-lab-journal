# Cyber-Lab Journal

Journal de bord technique de mon labo de cybersécurité/réseau (CYBER-LAB) et de ma reconversion vers un poste de Technicien Systèmes et Réseaux (TSSR), menée en parallèle d'une formation à la Wild Code School.

## Objectif

Documenter, au fil de l'eau, les choix d'architecture, les manipulations techniques, les pannes rencontrées et leur résolution, pour garder une trace exploitable de ce que j'apprends et pour présenter un parcours concret et vérifiable à de futurs employeurs ou formateurs. Chaque dossier contient un ou plusieurs documents au format : contexte, mise en place, problèmes rencontrés, résolution, ce que ça m'a appris.

## Stack et environnement du labo

Hyperviseur VirtualBox. Linux : Ubuntu Server (Ubuntu-DC, Ubuntu-01), réseau interne isolé du PC hôte. Windows : Windows Server 2022 (Active Directory Domain Services), Windows 11 (poste client). Annuaire Active Directory, domaine cyberlab.local. Partage de fichiers Samba, intégré à l'annuaire AD via realmd, sssd et winbind. Logging centralisé avec rsyslog et logrotate. Simulation réseau avec Cisco Packet Tracer.

## Sommaire

Le dossier 01-infrastructure couvre l'architecture VirtualBox, les réseaux internes et les VMs du labo CYBER-LAB. Le dossier 02-administration-linux couvre les utilisateurs et groupes, les permissions et les sudoers. Le dossier 03-active-directory couvre AD DS, les unités d'organisation, les groupes et les GPO. Le dossier 04-samba-fileserver couvre les partages de fichiers par département. Le dossier 05-linux-ad-integration couvre l'intégration Linux-AD via realmd, sssd et winbind. Le dossier 06-logging-centralise couvre la centralisation des logs avec rsyslog. Le dossier 07-ssh-hardening couvre l'authentification SSH par clé et le durcissement. Le dossier packet-tracer couvre la topologie réseau, le modèle OSI et le routage.

## Contexte

En reconversion professionnelle vers la cybersécurité et les réseaux, candidat admis au bootcamp TSSR (titre RNCP niveau 5) de la Wild Code School. En attendant la rentrée, je construis et fais évoluer un labo personnel complet pour mettre en pratique les notions théoriques et me confronter à de vraies pannes.

Dépôt mis à jour au fil des sessions de travail sur le labo.
