# Commandes : authentification SSH par clé et durcissement

## Génération et déploiement de la clé

```bash
ssh-keygen -t ed25519 -C "admincyber@cyber-lab"
```
Génère une paire de clés ed25519 côté PC hôte Windows, avec un commentaire identifiant la clé.

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh admincyber@192.168.10.10 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```
Déploie la clé publique sur Ubuntu-DC via un pipe SSH depuis PowerShell, sans avoir besoin de copier-coller manuellement le contenu de la clé.

```bash
ssh admincyber@192.168.10.10
```
Teste la connexion SSH par clé, qui ne doit plus demander de mot de passe mais seulement, le cas échéant, la passphrase de la clé privée.

## ssh-agent pour contourner le bug de collage de passphrase

```powershell
Start-Service ssh-agent
```
Démarre le service ssh-agent sur la VM Windows 11, pour éviter de ressaisir la passphrase à chaque connexion.

```bash
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```
Charge la clé privée dans l'agent une fois par session Windows ; la passphrase n'est alors demandée qu'une seule fois.

```bash
ssh-add -l
```
Liste les clés actuellement chargées dans l'agent, pour vérifier que le chargement a bien fonctionné.

## Durcissement ciblé avec Match User

```bash
sudo nano /etc/ssh/sshd_config
```
Ouvre la configuration du serveur SSH sur Ubuntu-DC pour y ajouter un bloc de configuration spécifique à un compte.

```
Match User admincyber
    PasswordAuthentication no
```
Désactive l'authentification par mot de passe uniquement pour le compte admincyber, les autres comptes (jrat, nlefevre, etc.) conservant l'authentification par mot de passe pour de futurs exercices.

```bash
sudo sshd -t
```
Vérifie la syntaxe de la configuration SSH avant de redémarrer le service, pour éviter de se retrouver bloqué hors de la machine en cas d'erreur.

```bash
sudo systemctl restart sshd
```
Redémarre le service SSH pour appliquer le durcissement ciblé.
