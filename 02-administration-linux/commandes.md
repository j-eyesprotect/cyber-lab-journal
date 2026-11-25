# Commandes : administration Linux, permissions et sudo

## Création des groupes et des utilisateurs

```bash
sudo groupadd direction
sudo groupadd administration
sudo groupadd commerciaux
sudo groupadd informatique
```
Crée les quatre groupes représentant les départements de l'entreprise fictive.

```bash
sudo useradd -m -G commerciaux tbernard
```
Crée le compte tbernard avec son répertoire personnel et l'ajoute au groupe commerciaux.

```bash
sudo usermod -aG informatique jrat
```
Ajoute le compte jrat (support TSSR niveau 1 et 2) au groupe informatique, l'option -a évitant de retirer ses autres groupes existants.

## Arborescence partagée avec héritage de groupe

```bash
sudo mkdir -p /srv/entreprise/commerciaux /srv/entreprise/direction /srv/entreprise/administration /srv/entreprise/informatique
```
Crée l'arborescence de partage par département sous /srv/entreprise/.

```bash
sudo chown root:commerciaux /srv/entreprise/commerciaux
sudo chmod 2770 /srv/entreprise/commerciaux
```
Définit le groupe propriétaire du dossier et applique les droits 770 avec le bit SGID (le chiffre 2 initial), pour que tout nouveau fichier créé hérite automatiquement du groupe commerciaux.

## chmod, chown, chgrp : notation symbolique et octale

```bash
chmod u+rwx,g+rx,o-rwx fichier.txt
```
Notation symbolique : droits complets pour le propriétaire, lecture et exécution pour le groupe, aucun droit pour les autres.

```bash
chmod 750 fichier.txt
```
Équivalent en notation octale de la commande précédente.

```bash
sudo chown tbernard:commerciaux rapport.txt
```
Change à la fois le propriétaire et le groupe propriétaire du fichier en une seule commande.

```bash
sudo chgrp -R informatique /srv/entreprise/informatique
```
Change récursivement le groupe propriétaire de tout le contenu d'un dossier.

## Bits spéciaux et umask

```bash
chmod u+s /usr/local/bin/outil
```
Positionne le bit SUID : le programme s'exécute avec les droits de son propriétaire plutôt que ceux de l'utilisateur qui le lance.

```bash
chmod g+s /srv/entreprise/direction
```
Positionne le bit SGID sur un dossier, pour que les nouveaux fichiers créés dedans héritent du groupe du dossier plutôt que du groupe principal de l'utilisateur créateur.

```bash
chmod +t /srv/entreprise/commun
```
Positionne le sticky bit : dans un dossier partagé en écriture, seul le propriétaire d'un fichier (ou root) peut le supprimer, même si d'autres utilisateurs ont un accès en écriture au dossier.

```bash
umask
```
Affiche le masque de permissions par défaut appliqué à tout nouveau fichier ou dossier créé par l'utilisateur courant.

## Politique de privilèges sudo différenciée

```bash
sudo visudo -f /etc/sudoers.d/nlefevre
```
Ouvre un nouveau fichier dédié sous sudoers.d pour le compte administrateur système, avec la validation syntaxique automatique de visudo.

```
nlefevre ALL=(ALL) ALL
```
Donne à nlefevre un accès sudo complet, sans restriction de commandes, en cohérence avec son rôle d'administrateur système.

```bash
sudo visudo -f /etc/sudoers.d/jrat
```
Ouvre le fichier sudoers dédié au compte support niveau 1 et 2.

```
Cmnd_Alias SUPPORT_N1N2 = /usr/sbin/useradd, /usr/sbin/usermod, /bin/systemctl restart ssh, /usr/bin/tail
jrat ALL=(ALL) SUPPORT_N1N2
```
Restreint jrat à un ensemble précis de commandes de support, plutôt qu'un accès root complet, conformément au principe de moindre privilège.

## find et grep sur les journaux

```bash
find /srv/entreprise -type f -perm -002
```
Recherche les fichiers disposant d'un droit d'écriture pour tout le monde (other), utile pour repérer une permission trop permissive.

```bash
sudo grep "Failed password" /var/log/auth.log
```
Filtre les tentatives de connexion SSH échouées dans le journal d'authentification.

```bash
sudo grep "Failed password" /var/log/auth.log | grep -c jrat
```
Compte combien de tentatives échouées concernent spécifiquement le compte jrat.
