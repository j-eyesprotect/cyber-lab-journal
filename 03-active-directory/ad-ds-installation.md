# Mise en place d'Active Directory (WIN-AD-DC01)

## Contexte / objectif

Donner un role actif a la VM Windows 11 du labo (jusque-la sans fonction definie) en mettant en place un annuaire Active Directory, afin de centraliser l'authentification des utilisateurs de l'entreprise fictive et de pouvoir, a terme, gerer des politiques de groupe (GPO).

## Installation du controleur de domaine

Une VM Windows Server 2022 (evaluation gratuite) est installee dans VirtualBox pour devenir le futur controleur de domaine. Premiere tentative de mise a jour Windows bloquee (CPU a 50 pourcent, aucune activite disque) ; une VM de test en parallele a rencontre le meme probleme et a ete supprimee ; un simple redemarrage de la VM d'origine a permis de terminer la mise a jour avec succes.

La VM est ensuite configuree sur le reseau interne CYBER-LAB avec une IP statique (192.168.10.50) et renommee WIN-AD-DC01. Le role AD DS (Active Directory Domain Services) est installe, puis la promotion en controleur de domaine est effectuee : creation d'une nouvelle foret, nouveau domaine cyberlab.local, avec redemarrage automatique a la fin du processus.

La promotion est verifiee fonctionnelle via la commande Get-ADDomain, ainsi que par la resolution DNS du domaine cyberlab.local vers l'adresse du controleur.

## Structuration de l'annuaire

Avant de rattacher le poste client Windows au domaine, l'annuaire est structure en miroir de l'organisation de l'entreprise fictive deja utilisee cote Linux : une unite d'organisation (OU) Entreprise contenant quatre OU filles, Direction, Administration, Commerciaux et Informatique. La protection contre la suppression accidentelle est desactivee sur ces OU pour faciliter les manipulations pendant la phase d'apprentissage.

Un premier compte utilisateur de test est cree dans l'OU Informatique ; une erreur liee a la politique de complexite des mots de passe par defaut d'AD est rencontree puis corrigee. L'ensemble des comptes utilisateurs est ensuite cree dans les quatre OU du domaine.

Des groupes de securite sont crees, un par departement, suivant la convention de nommage GG_ (par exemple GG_Commerciaux), et les utilisateurs sont affectes a leur groupe respectif.

## Jonction du poste client au domaine

Le poste Win-Client (Windows 11) est joint au domaine cyberlab.local, avec son DNS pointe vers le controleur 192.168.10.50. La jonction est testee avec succes avec un compte du departement Commerciaux, puis etendue a d'autres comptes AD pour simuler plusieurs employes se connectant au meme poste.

## Automatisation : reinitialisation de mots de passe en masse

Un script PowerShell interactif est developpe pour faciliter la gestion des comptes : liste numerotee des utilisateurs, selection par saisie, generation de mots de passe aleatoires et export CSV des nouveaux mots de passe. Le script est ensuite ameliore pour garantir un minimum de majuscules et de caracteres speciaux dans les mots de passe generes, plutot qu'une selection purement aleatoire qui pouvait echouer la politique de complexite d'AD.

## Mise en place de GPO

Une GPO de mappage automatique de lecteurs reseau par departement (via les Preferences de strategie de groupe) est mise en place sur WIN-AD-DC01. Cette etape revele un probleme d'incompatibilite entre l'authentification AD et l'authentification Samba locale sur les serveurs Linux, documente et resolu dans 05-linux-ad-integration/realmd-sssd-winbind.md.

## Ce que ca m'a appris

Structurer l'annuaire (OU, groupes) avant de rattacher les postes clients evite d'avoir a tout redeplacer ensuite. Un script d'administration (ici PowerShell) gagne a etre iteratif : la premiere version fonctionnelle n'est pas forcement la version finale, et des contraintes decouvertes a l'usage (politique de complexite des mots de passe) doivent etre reintegrees dans l'outil plutot que contournees manuellement a chaque fois.
