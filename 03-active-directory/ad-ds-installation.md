# Mise en place d'Active Directory (WIN-AD-DC01)

## Contexte et objectif

Donner un rôle actif à la VM Windows 11 du labo (jusque-là sans fonction définie) en mettant en place un annuaire Active Directory, afin de centraliser l'authentification des utilisateurs de l'entreprise fictive et de pouvoir, à terme, gérer des politiques de groupe (GPO).

## Installation du contrôleur de domaine

Une VM Windows Server 2022 (évaluation gratuite) est installée dans VirtualBox pour devenir le futur contrôleur de domaine. Première tentative de mise à jour Windows bloquée (CPU à 50 pourcent, aucune activité disque) ; une VM de test en parallèle a rencontré le même problème et a été supprimée ; un simple redémarrage de la VM d'origine a permis de terminer la mise à jour avec succès.

La VM est ensuite configurée sur le réseau interne CYBER-LAB avec une IP statique (192.168.10.50) et renommée WIN-AD-DC01. Le rôle AD DS (Active Directory Domain Services) est installé, puis la promotion en contrôleur de domaine est effectuée : création d'une nouvelle forêt, nouveau domaine cyberlab.local, avec redémarrage automatique à la fin du processus.

La promotion est vérifiée fonctionnelle via la commande Get-ADDomain, ainsi que par la résolution DNS du domaine cyberlab.local vers l'adresse du contrôleur.

## Structuration de l'annuaire

Avant de rattacher le poste client Windows au domaine, l'annuaire est structuré en miroir de l'organisation de l'entreprise fictive déjà utilisée côté Linux : une unité d'organisation (OU) Entreprise contenant quatre OU filles, Direction, Administration, Commerciaux et Informatique. La protection contre la suppression accidentelle est désactivée sur ces OU pour faciliter les manipulations pendant la phase d'apprentissage.

Un premier compte utilisateur de test est créé dans l'OU Informatique ; une erreur liée à la politique de complexité des mots de passe par défaut d'AD est rencontrée puis corrigée. L'ensemble des comptes utilisateurs est ensuite créé dans les quatre OU du domaine.

Des groupes de sécurité sont créés, un par département, suivant la convention de nommage GG_ (par exemple GG_Commerciaux), et les utilisateurs sont affectés à leur groupe respectif.

## Jonction du poste client au domaine

Le poste Win-Client (Windows 11) est joint au domaine cyberlab.local, avec son DNS pointé vers le contrôleur 192.168.10.50. La jonction est testée avec succès avec un compte du département Commerciaux, puis étendue à d'autres comptes AD pour simuler plusieurs employés se connectant au même poste.

## Automatisation : réinitialisation de mots de passe en masse

Un script PowerShell interactif est développé pour faciliter la gestion des comptes : liste numérotée des utilisateurs, sélection par saisie, génération de mots de passe aléatoires et export CSV des nouveaux mots de passe. Le script est ensuite amélioré pour garantir un minimum de majuscules et de caractères spéciaux dans les mots de passe générés, plutôt qu'une sélection purement aléatoire qui pouvait échouer la politique de complexité d'AD.

## Mise en place de GPO

Une GPO de mappage automatique de lecteurs réseau par département (via les Préférences de stratégie de groupe) est mise en place sur WIN-AD-DC01. Cette étape révèle un problème d'incompatibilité entre l'authentification AD et l'authentification Samba locale sur les serveurs Linux, documenté et résolu dans 05-linux-ad-integration/realmd-sssd-winbind.md.

## Ce que ça m'a appris

Structurer l'annuaire (OU, groupes) avant de rattacher les postes clients évite d'avoir à tout redéplacer ensuite. Un script d'administration (ici PowerShell) gagne à être itératif : la première version fonctionnelle n'est pas forcément la version finale, et des contraintes découvertes à l'usage (politique de complexité des mots de passe) doivent être réintégrées dans l'outil plutôt que contournées manuellement à chaque fois.
