# Premier dépôt versionné : plugin Ableton (AcidTB303)

27 nov 2026

## Contexte

Premier cas pratique d'utilisation de Git en dehors de ce journal, après l'installation documentée dans installation-Git.md : un plugin audio pour Ableton, développé en C++ avec le framework JUCE 9.0.6 et CMake 4.5.11. Le projet est mené en "vibe coding" : les spécifications et les choix viennent de moi, le code est écrit par Claude.

## Problème rencontré avant de commencer : Git Bash ne s'ouvrait pas

À la première tentative d'ouverture de Git Bash depuis le dossier du projet, erreur bloquante :

```
Error: Could not fork child process: Resource temporarily unavailable (-1).
DLL rebasing may be required; see 'rebaseall / rebase --help'.
```

Cause identifiée : l'ASLR obligatoire de Windows interfère avec l'émulation fork() de MSYS2 (sur lequel repose Git Bash). Un simple redémarrage de l'ordinateur n'a pas suffi. Solution appliquée : relancer l'installeur Git, refaire les mêmes choix qu'à l'installation initiale, et cette fois cocher "Add mandatory ASLR security exceptions" à la toute dernière étape ("Security options"). Après réinstallation, Git Bash s'est ouvert normalement.

## Préparation du projet : .gitignore

Un .gitignore adapté C++/JUCE/CMake a été créé avant tout commit, pour exclure les dossiers de build générés par CMake, les fichiers compilés, les fichiers d'IDE et les fichiers temporaires Windows. Contenu détaillé dans le dépôt du plugin lui-même.

## Étapes suivies

1. git status a confirmé que seuls les fichiers utiles (.gitignore, CMakeLists.txt, Source/) étaient détectés, aucun dossier build/ ne traînait dans la liste : le .gitignore fonctionnait dès le départ.
2. git add . puis git commit -m "..." ont d'abord échoué avec l'erreur "Author identity unknown" : l'identité Git n'avait jamais été configurée sur cette machine.
3. Configuration de l'identité (voir commandes.md, section Configuration initiale) avec le compte GitHub j-eyesprotect : git config --global user.name et git config --global user.email. Une faute de frappe initiale (user.mail au lieu de user.email) n'a pas empêché le fonctionnement, l'entrée erronée a simplement été laissée de côté.
4. Le commit est passé : premier commit du dépôt, 6 fichiers, 223 lignes.
5. Création du dépôt distant sur GitHub (compte j-eyesprotect, dépôt privé AcidTB303inspiration), sans README/gitignore/licence initialisés côté GitHub pour éviter un conflit avec l'historique local déjà existant.
6. Liaison du dépôt local au dépôt distant : git remote add origin, git branch -M main, git push -u origin main. Authentification via Git Credential Manager (configuré à l'installation), fenêtre de connexion ouverte dans le navigateur. Push réussi : branche main créée côté distant.

## Décision prise : un seul compte GitHub

Les deux comptes GitHub existants sont fusionnés en un seul (j-eyesprotect), y compris pour les projets personnels comme ce plugin. Un projet perso peut rester en dépôt privé si je ne veux pas le rendre public, sans que ça pose de problème pour l'utiliser depuis le même compte que le portfolio pro/cyber.

## Cycle désormais applicable pour ce projet

```
git add .
git commit -m "description du changement"
git push
```

Plus besoin de remote add ni du -u sur ce dépôt : fait une seule fois à l'étape 6.

## Ce que ça m'a appris

Lien direct entre un message d'erreur Git Bash au démarrage et une option de sécurité vue (et ignorée) lors de l'installation initiale de Git. Importance de configurer l'identité Git avant le tout premier commit sur une nouvelle machine. Le cycle add / commit / push, une fois le remote en place, est identique quel que soit le langage ou le projet.
