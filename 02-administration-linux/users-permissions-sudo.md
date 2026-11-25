# Administration Linux : utilisateurs, permissions et sudo

## Contexte et objectif

Dans le labo CYBER-LAB, Ubuntu-DC héberge une « entreprise fictive » complète, pensée pour pratiquer l'administration Linux dans des conditions proches du réel : gestion des comptes, des droits d'accès et des privilèges d'administration.

## Utilisateurs et groupes

L'entreprise fictive est structurée autour de quatre groupes représentant les départements : direction, administration, commerciaux et informatique. Six comptes utilisateurs sont créés avec useradd et affectés à leur groupe avec usermod et groupadd. Une arborescence dédiée est mise en place sous /srv/entreprise/, avec des permissions 770 et le bit SGID positionné sur les dossiers de département, pour que les nouveaux fichiers créés héritent automatiquement du bon groupe propriétaire. Des tests de collaboration en SSH entre comptes de différents départements ont permis de valider le cloisonnement des accès.

## Permissions : chmod, chown, chgrp

Travail approfondi sur la notation symbolique et octale de chmod, ainsi que sur la syntaxe propriétaire:groupe de chown et chgrp, à travers des exercices pratiques directs sur /srv/entreprise/. Les bits spéciaux SUID, SGID et sticky bit sont étudiés et appliqués concrètement, de même que la notion d'umask par défaut pour les nouveaux fichiers et dossiers.

## Politique de privilèges avec sudoers

Une politique de privilèges différenciée est mise en place dans l'entreprise fictive, via des fichiers dédiés sous /etc/sudoers.d/ avec des Cmnd_Alias : un compte (nlefevre) joue le rôle d'administrateur système avec des droits sudo étendus, tandis qu'un autre compte (jrat) représente un technicien TSSR de support niveau 1 et 2, avec des droits sudo volontairement restreints à un ensemble de commandes précis plutôt qu'un accès root complet.

## Recherche et analyse : find et grep

Session dédiée à la consolidation de find et grep, avec de nombreux exercices pratiques menés directement sur le labo CYBER-LAB et sur des fichiers de logs réels comme /var/log/auth.log, pour s'entraîner à retrouver rapidement une information précise dans un volume de données plus conséquent, une compétence directement transposable à un poste d'analyste SOC.

## Ce que ça m'a appris

Mettre en place une vraie séparation des privilèges (administrateur système versus support N1/N2) plutôt que de tout faire depuis un unique compte root oblige à réfléchir concrètement à quelles commandes un rôle donné doit réellement pouvoir exécuter, ce qui est une compétence directement liée au principe de moindre privilège en cybersécurité. Construire une arborescence de fichiers partagés avec SGID dès le départ évite d'avoir à corriger après coup des propriétaires de fichiers incohérents au fil des créations successives.
