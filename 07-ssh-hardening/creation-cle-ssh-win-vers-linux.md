# Création d'une clé SSH — authentification PC hôte (Windows) vers VM Linux

Objectif : mettre en place une connexion SSH par clé (sans mot de passe) entre le PC hôte Windows et un serveur du labo, pour un utilisateur donné (ici `admin01` sur Ubuntu-01).

  ## 1. Génération de la paire de clés

Génération d'une clé ed25519, en précisant le fichier de destination directement via `-f` pour éviter tout conflit avec une clé existante (ex. celle d'un autre utilisateur comme `admincyber`) :

```
ssh-keygen -t ed25519 -C "admin01@cyber-lab" -f "C:\Users\<user>\.ssh\id_ed25519_admin01"
  ```

  Point d'attention : sous Windows, `ssh-keygen` n'interprète pas le `~` dans le prompt interactif. Mieux vaut donc passer le chemin complet directement en argument (`-f`) plutôt que de le taper au prompt.

## 2. Copie de la clé publique vers le serveur

Pas de `ssh-copy-id` natif sous Windows ; on passe par une commande équivalente :

```
type C:\Users\<user>\.ssh\id_ed25519_admin01.pub | ssh admin01@<ip> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
  ```

  ## 3. Permissions côté serveur

  SSH ignore silencieusement une clé si les permissions sont trop ouvertes :

```
chmod 700 ~/.ssh
  chmod 600 ~/.ssh/authorized_keys
  ```

  ## 4. Alias de connexion côté PC hôte

Dans `C:\Users\<user>\.ssh\config` :

```
Host ubuntu-01
      HostName <ip>
      User admin01
    IdentityFile C:\Users\<user>\.ssh\id_ed25519_admin01
```

Permet ensuite de se connecter simplement avec `ssh ubuntu-01`.

  ## 5. Agent SSH Windows (ne plus retaper la passphrase)

En PowerShell administrateur :

```
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
  ```

  Puis, en PowerShell normal, ajout de la clé à l'agent :

  ```
ssh-add C:\Users\<user>\.ssh\id_ed25519_admin01
```

La passphrase n'est alors demandée qu'une seule fois par session Windows (`ssh-add -l` permet de vérifier les clés chargées).

  ## Point d'architecture réseau

  Le PC hôte Windows n'a pas accès au réseau interne VirtualBox (CYBER-LAB) : un réseau interne VirtualBox est isolé du PC hôte par définition, seules les VM qui y sont attachées peuvent se parler. La connexion SSH depuis le PC hôte passe donc par la seconde interface réseau de la VM, de type « Réseau privé hôte » (Host-only), et non par l'IP du réseau interne CYBER-LAB.
  


## 6. Désactiver l'authentification par mot de passe (côté serveur)

Une fois la connexion par clé validée, on désactive le mot de passe pour ne garder que l'authentification par clé — à faire sur chaque VM (ici Ubuntu-01), comme déjà fait sur Ubuntu-DC.

Depuis une session SSH déjà connectée par clé :

```
sudo nano /etc/ssh/sshd_config
```

Modifier (ou ajouter) :
```
PasswordAuthentication no
PubkeyAuthentication yes
```

Sauvegarder (`Ctrl+O`, Entrée, `Ctrl+X`), puis recharger le service :
```
sudo systemctl restart ssh
```

**Important avant de fermer la session en cours** : ouvrir un **nouveau** terminal et tester `ssh ubuntu-01` pour confirmer que la connexion par clé fonctionne toujours, avant de fermer la session déjà ouverte — pour garder un accès de secours si jamais le redemarrage du service casse quelque chose.
