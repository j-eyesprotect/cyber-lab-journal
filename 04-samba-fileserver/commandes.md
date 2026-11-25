# Commandes : partages Samba par département (Ubuntu-DC)

## Installation et configuration de base

```bash
sudo apt install samba
```
Installe le service Samba sur Ubuntu-DC.

```bash
sudo nano /etc/samba/smb.conf
```
Ouvre le fichier de configuration principal de Samba pour y ajouter les définitions de partage.

```
[commerciaux]
path = /srv/entreprise/commerciaux
valid users = @commerciaux
read only = no
```
Bloc de configuration définissant le partage commerciaux, restreint aux membres du groupe Linux commerciaux, en lecture-écriture.

## Comptes Samba

```bash
sudo smbpasswd -a tbernard
```
Crée un compte Samba pour tbernard, avec son propre mot de passe Samba, indépendant du mot de passe Linux du compte.

```bash
sudo smbpasswd -e tbernard
```
Active le compte Samba nouvellement créé.

## Vérification et application

```bash
testparm
```
Vérifie la syntaxe du fichier smb.conf avant de redémarrer le service, pour éviter de casser un partage déjà fonctionnel avec une erreur de configuration.

```bash
sudo systemctl restart smbd
```
Redémarre le service Samba pour appliquer la nouvelle configuration.

```bash
smbclient -L localhost -U tbernard
```
Liste les partages Samba disponibles en s'authentifiant avec le compte tbernard, pour vérifier que le partage commerciaux est bien visible et accessible.

## Extension aux quatre départements

```
[direction]
path = /srv/entreprise/direction
valid users = @direction
read only = no

[administration]
path = /srv/entreprise/administration
valid users = @administration
read only = no

[informatique]
path = /srv/entreprise/informatique
valid users = @informatique
read only = no
```
Mêmes blocs de configuration répétés pour les trois autres départements, chacun restreint à son groupe Linux correspondant.
