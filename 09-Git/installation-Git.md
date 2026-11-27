# Installation de Git sur Windows — choix effectués

Documentation des options choisies lors de l'installation de **Git 2.56.1** sur Windows (via l'installeur officiel [git-scm.com](https://git-scm.com/downloads)).

## 1. Éditeur par défaut

**Choix : Use Visual Studio Code as Git's default editor**

Définit VS Code comme éditeur utilisé par Git lorsqu'il a besoin d'ouvrir un éditeur de texte (message de commit détaillé, résolution de conflits, rebase interactif, etc.), à la place d'un éditeur en ligne de commande comme Vim ou Nano.

Prérequis : VS Code doit déjà être installé et accessible dans le PATH.

## 2. Nom de la branche initiale

**Choix : Override the default branch name for new repositories → `main`**

Les nouveaux dépôts créés avec `git init` auront une branche par défaut nommée `main` plutôt que `master`. C'est devenu le standard de fait sur GitHub, GitLab et dans la majorité des projets récents (GitHub a basculé master → main par défaut en 2020).

Ce réglage n'affecte pas les dépôts existants.

## 3. Intégration au PATH

**Choix : Git from the command line and also from 3rd-party software (recommandé)**

Permet d'utiliser la commande `git` depuis :
- Git Bash
- l'invite de commandes Windows (cmd)
- PowerShell
- tout logiciel tiers qui cherche Git dans le PATH (ex. VS Code)

Sans ajouter les outils Unix optionnels au PATH, ce qui évite les conflits avec des commandes Windows natives comme `find` ou `sort`.

## 4. Client SSH

**Choix : Use bundled OpenSSH**

Utilise le client SSH (`ssh.exe`) fourni avec Git plutôt qu'une installation externe. Simple et suffisant pour la majorité des usages (authentification SSH vers GitHub/GitLab, etc.).

## 5. Bibliothèque HTTPS/TLS

**Choix : Use the native Windows Secure Channel library**

La validation des certificats serveur se fait via le magasin de certificats de Windows plutôt que via le fichier `ca-bundle.crt` d'OpenSSL. Adapté à un usage Windows standard, et compatible avec des certificats d'autorité internes (utile en entreprise).

## 6. Gestion des fins de ligne (line endings)

**Choix : Checkout Windows-style, commit Unix-style line endings (`core.autocrlf=true`)**

- À la récupération (checkout) d'un fichier : conversion `LF` → `CRLF` (format Windows).
- Au commit : conversion `CRLF` → `LF` (format Unix, standard Git/GitHub).

Évite que des fichiers apparaissent comme "modifiés" uniquement à cause d'une différence de fin de ligne entre contributeurs Windows et Linux/Mac. Réglage recommandé pour les projets multiplateformes développés depuis Windows.

## 7. Terminal pour Git Bash

**Choix : Use MinTTY (terminal par défaut de MSYS2)**

Terminal avec fenêtre redimensionnable, sélection de texte non rectangulaire et bon support Unicode. Nécessite `winpty` pour lancer des programmes interactifs Windows natifs (ex. Python interactif), ce qui n'est généralement pas un problème au quotidien.

## 8. Comportement par défaut de `git pull`

**Choix : Merge**

Quand des commits locaux et distants divergent, `git pull` crée un commit de fusion combinant les deux historiques, en préservant la trace du travail effectué en parallèle.

Alternatives possibles (non choisies ici) :
- **Rebase** : rejoue les commits locaux par-dessus la branche distante → historique linéaire, mais réécrit les commits locaux.
- **Fast-forward only** : refuse la fusion si un fast-forward n'est pas possible.

## 9. Gestionnaire d'identifiants (credential helper)

**Choix : Git Credential Manager**

Mémorise de façon sécurisée les identifiants (ou tokens) utilisés pour les opérations HTTPS (push/pull/clone vers GitHub, etc.), évitant d'avoir à les ressaisir à chaque opération.

## 10. Options supplémentaires

**Choix :**
- ✅ Enable file system caching (`core.fscache=true`) — amélioration des performances, activé par défaut sans inconvénient connu.
- ⬜ Enable symbolic links — non activé (nécessite une permission Windows spécifique et n'est utile que pour des projets l'exploitant explicitement).

## 11. Exceptions de sécurité ASLR

**Choix : Add mandatory ASLR security exceptions → non coché**

Réglage réservé aux cas où Windows empêche Git Bash de se lancer correctement à cause de l'ASLR (protection mémoire). Laissé désactivé par défaut car l'activer peut introduire une vulnérabilité de sécurité et ralentit le chargement des paramètres de protection Windows. À activer seulement en cas de problème de lancement avéré.

---

## Vérification post-installation

```bash
git --version
```

## Configuration de l'identité (à faire avant le premier commit)

```bash
git config --global user.name "ton nom"
git config --global user.email "ton email"
```
