# Infrastructure VirtualBox du labo CYBER-LAB

## Contexte / objectif

CYBER-LAB est un labo personnel monte sous VirtualBox, concu comme une "entreprise fictive" pour pratiquer l'administration systeme, reseau et Active Directory dans des conditions proches d'un environnement professionnel reel, avec de vraies pannes a resoudre.

## Prerequis materiel

Virtualisation Intel VMX reactivee au BIOS. VirtualBox 7.2.20 installe, avec les VMs reparties entre deux disques selon l'espace disponible (disques E: puis D: au fur et a mesure de l'ajout de nouvelles machines). Le PC hote dispose de 32 Go de RAM au total, ce qui permet de faire tourner plusieurs VMs en parallele.

## Machines du labo

Ubuntu-DC (192.168.10.10) est le serveur principal : DNS/DHCP du reseau simule, partages Samba par departement, puis membre du domaine Active Directory apres l'integration Linux-AD. 2 Go de RAM alloues.

Ubuntu-01 (192.168.10.20) est le serveur secondaire : d'abord sans role actif, il recoit ensuite la fonction de serveur de logs centralise (rsyslog), recevant et archivant les logs d'Ubuntu-DC. 2 Go de RAM alloues.

Ubuntu-Client (192.168.10.30) est le poste de travail Linux de l'entreprise fictive.

WIN-AD-DC01 (192.168.10.50) est une VM Windows Server 2022, en evaluation gratuite, promue controleur de domaine Active Directory pour le domaine cyberlab.local. 4 Go de RAM alloues.

Win-Client (192.168.10.40) est une VM Windows 11 Entreprise, en evaluation gratuite 90 jours, jointe au domaine cyberlab.local comme poste client des utilisateurs du domaine. 8 Go de RAM alloues.

## Architecture reseau

Le reseau interne CYBER-LAB (plage 192.168.10.x) est totalement isole du PC hote : c'est le reseau de production simule de l'entreprise fictive. Un reseau separe, de type Host-only VirtualBox (plage 192.168.56.x), sert uniquement de canal d'administration SSH depuis le PC hote physique vers les VMs Linux, distinct du reseau de production simule.

Sur les VMs Windows, une carte NAT supplementaire est ajoutee ponctuellement (desactivee par defaut) pour les mises a jour et installations de paquets, sans l'exposer en permanence.

Les adresses IP des VMs Linux sont configurees en statique via Netplan. Les Guest Additions VirtualBox sont installees sur les VMs Windows pour ameliorer l'integration (resolution d'ecran, presse-papiers).

## Contrainte notable

Le copier/coller n'est pas disponible sur les VMs serveurs Ubuntu-DC et Ubuntu-01 : la connexion vers ces machines se fait en SSH depuis PowerShell sur le PC hote Windows, avec authentification par cle ed25519 (voir le dossier 07-ssh-hardening).

## Ce que ca m'a appris

Separer le reseau de production simule (192.168.10.x) du canal d'administration (192.168.56.x) force a raisonner comme sur une vraie infrastructure d'entreprise, ou l'acces d'administration ne doit jamais transiter par le meme reseau que les flux utilisateurs. Dimensionner la RAM par VM selon son role reel (serveur leger vs poste de travail complet) permet de faire tourner un labo a 5 VMs sur une seule machine hote sans la saturer.
