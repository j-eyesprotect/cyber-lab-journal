# Infrastructure VirtualBox du labo CYBER-LAB

## Contexte et objectif

CYBER-LAB est un labo personnel monté sous VirtualBox, conçu comme une « entreprise fictive » pour pratiquer l'administration système, réseau et Active Directory dans des conditions proches d'un environnement professionnel réel, avec de vraies pannes à résoudre.

## Prérequis matériel

Virtualisation Intel VMX réactivée au BIOS. VirtualBox 7.2.20 installé, avec les VMs réparties entre deux disques selon l'espace disponible (disques E: puis D: au fur et à mesure de l'ajout de nouvelles machines). Le PC hôte dispose de 32 Go de RAM au total, ce qui permet de faire tourner plusieurs VMs en parallèle.

## Machines du labo

Ubuntu-DC (192.168.10.10) est le serveur principal : DNS/DHCP du réseau simulé, partages Samba par département, puis membre du domaine Active Directory après l'intégration Linux-AD. 2 Go de RAM alloués.

Ubuntu-01 (192.168.10.20) est le serveur secondaire : d'abord sans rôle actif, il reçoit ensuite la fonction de serveur de logs centralisé (rsyslog), recevant et archivant les logs d'Ubuntu-DC. 2 Go de RAM alloués.

Ubuntu-Client (192.168.10.30) est le poste de travail Linux de l'entreprise fictive.

WIN-AD-DC01 (192.168.10.50) est une VM Windows Server 2022, en évaluation gratuite, promue contrôleur de domaine Active Directory pour le domaine cyberlab.local. 4 Go de RAM alloués.

Win-Client (192.168.10.40) est une VM Windows 11 Entreprise, en évaluation gratuite 90 jours, jointe au domaine cyberlab.local comme poste client des utilisateurs du domaine. 8 Go de RAM alloués.

## Architecture réseau

Le réseau interne CYBER-LAB (plage 192.168.10.x) est totalement isolé du PC hôte : c'est le réseau de production simulé de l'entreprise fictive. Un réseau séparé, de type Host-only VirtualBox (plage 192.168.56.x), sert uniquement de canal d'administration SSH depuis le PC hôte physique vers les VMs Linux, distinct du réseau de production simulé.

Sur les VMs Windows, une carte NAT supplémentaire est ajoutée ponctuellement (désactivée par défaut) pour les mises à jour et installations de paquets, sans l'exposer en permanence.

Les adresses IP des VMs Linux sont configurées en statique via Netplan. Les Guest Additions VirtualBox sont installées sur les VMs Windows pour améliorer l'intégration (résolution d'écran, presse-papiers).

## Contrainte notable

Le copier-coller n'est pas disponible sur les VMs serveurs Ubuntu-DC et Ubuntu-01 : la connexion vers ces machines se fait en SSH depuis PowerShell sur le PC hôte Windows, avec authentification par clé ed25519 (voir le dossier 07-ssh-hardening).

## Ce que ça m'a appris

Séparer le réseau de production simulé (192.168.10.x) du canal d'administration (192.168.56.x) force à raisonner comme sur une vraie infrastructure d'entreprise, où l'accès d'administration ne doit jamais transiter par le même réseau que les flux utilisateurs. Dimensionner la RAM par VM selon son rôle réel (serveur léger vs poste de travail complet) permet de faire tourner un labo à 5 VMs sur une seule machine hôte sans la saturer.
