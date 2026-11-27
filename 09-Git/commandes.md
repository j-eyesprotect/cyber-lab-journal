# Commandes Git

Référence des commandes Git utilisées et rencontrées, classées par usage.

## Configuration initiale (une fois par machine)

```
git config --global user.name "ton nom"
git config --global user.email "ton email"
git config --global --list
git config --global --unset user.mail
```

user.name / user.email : identité associée à chaque commit. L'email doit correspondre à celui du compte GitHub utilisé pour que les commits soient liés au profil. --list vérifie la configuration actuelle. --unset <clef> supprime un réglage erroné (ex. faute de frappe comme user.mail au lieu de user.email).

## Créer ou récupérer un dépôt

```
git init
git clone <url>
```

init transforme le dossier courant en dépôt Git (crée le dossier caché .git). clone récupère une copie locale complète d'un dépôt distant existant.

## Cycle de travail quotidien

```
git status
git add <fichier>
git add .
git commit -m "message"
git push
git pull
```

status : état actuel (fichiers modifiés, en staging, non suivis). add : prépare des changements pour le prochain commit (staging area), le point prépare tout le dossier courant. commit -m : enregistre un snapshot permanent dans l'historique local. push : envoie les commits locaux vers le dépôt distant. pull : récupère et fusionne les changements du dépôt distant.

## Relier un dépôt local à GitHub (une seule fois par dépôt)

```
git remote add origin <url-du-depot>
git branch -M main
git push -u origin main
```

remote add origin : enregistre l'adresse du dépôt distant sous le nom origin. branch -M main : force le nom de la branche courante à main. push -u : premier envoi, et lie la branche locale à origin/main (les push suivants n'ont plus besoin de préciser la destination).

## Historique et différences

```
git log
git diff
```

## Branches

```
git branch
git checkout -b <nom>
git merge <branche>
```

branch liste les branches existantes. checkout -b crée une nouvelle branche et bascule dessus (pour tester sans toucher au code stable). merge fusionne une branche dans la branche courante.

## Dépannage rencontré

"Could not fork child process: Resource temporarily unavailable" à l'ouverture de Git Bash (voir installation-Git.md) : lié à l'ASLR obligatoire de Windows interférant avec l'émulation fork() de MSYS2. Résolu en réinstallant Git avec la case "Add mandatory ASLR security exceptions" cochée à la dernière étape de l'installeur.

"Author identity unknown" au premier git commit : user.name/user.email pas encore définis, voir section Configuration initiale ci-dessus.

Git Bash introuvable au clic droit (Windows 11) : l'option est masquée dans "Afficher plus d'options" du menu contextuel, ou s'ouvre directement depuis le menu Démarrer puis cd vers le dossier du projet.
