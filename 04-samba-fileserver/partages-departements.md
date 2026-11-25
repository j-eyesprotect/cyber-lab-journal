# Partages de fichiers Samba par département (Ubuntu-DC)

## Contexte et objectif

Donner à l'entreprise fictive du labo CYBER-LAB un véritable espace de partage de fichiers entre départements, accessible aussi bien depuis les postes Linux que depuis les postes Windows, en s'appuyant sur l'arborescence /srv/entreprise/ déjà structurée avec des permissions Unix par département.

## Première mise en place : partage commerciaux

Samba est installé et configuré sur Ubuntu-DC. Un premier partage, nommé commerciaux, est créé sur /srv/entreprise/commerciaux, restreint au groupe Linux commerciaux. Un compte Samba distinct est créé pour l'utilisateur tbernard, car Samba gère ses propres identifiants d'authentification, indépendants des mots de passe Linux. L'accès au partage est ensuite testé depuis le poste client Windows (Win-Client), confirmant que le partage est bien accessible et correctement restreint.

## Extension aux quatre départements

Les partages Samba sont étendus aux quatre départements (commerciaux, direction, administration, informatique), en miroir exact de l'arborescence /srv/entreprise/. Des comptes Samba sont créés pour l'ensemble des utilisateurs de l'entreprise fictive. L'accès est confirmé fonctionnel pour tous les comptes depuis Win-Client, chaque utilisateur ne pouvant accéder qu'au partage de son propre département, conformément aux permissions Unix sous-jacentes.

## Une authentification à part entière

Un point important mis en évidence au fil de cette mise en place : Samba possède son propre système d'authentification, distinct à la fois des comptes Linux locaux et, plus tard, de l'annuaire Active Directory. Un utilisateur peut exister côté Linux sans être automatiquement utilisable côté Samba, et inversement. Cette distinction est au cœur de la panne résolue ultérieurement lors de l'intégration Linux-AD (voir le dossier 05-linux-ad-integration), où l'authentification AD et l'authentification Samba locale se sont révélées incompatibles sans configuration explicite.

## Ce que ça m'a appris

Un service comme Samba qui semble n'être « qu'un partage de fichiers » embarque en réalité sa propre couche d'authentification, séparée du système d'exploitation sous-jacent. Ne pas en tenir compte dès la conception (en supposant qu'un compte Linux suffit) mène directement aux problèmes d'intégration rencontrés plus tard avec Active Directory. Partir d'une arborescence de permissions Unix déjà propre (groupes, SGID) facilite énormément la configuration des partages Samba, qui peuvent alors s'appuyer sur des permissions déjà cohérentes plutôt que de devoir tout redéfinir.
