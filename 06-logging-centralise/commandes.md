# Commandes : logging centralisé avec rsyslog

## Réception des logs (Ubuntu-01)

```bash
sudo nano /etc/rsyslog.conf
```
Ouvre la configuration principale de rsyslog pour activer le module de réception UDP.

```
module(load="imudp")
input(type="imudp" port="514")
```
Active le module imudp et ouvre l'écoute sur le port 514 en UDP, pour recevoir les logs envoyés par Ubuntu-DC.

```bash
sudo nano /etc/rsyslog.d/10-remote-ubuntu-dc.conf
```
Crée une règle dédiée pour filtrer et router les logs provenant spécifiquement d'Ubuntu-DC.

```
if $fromhost-ip == '192.168.10.10' then {
    action(type="omfile" file="/var/log/remote/ubuntu-dc.log")
}
```
Filtre les messages selon l'hôte source et les écrit en plus dans un fichier dédié, sans bloc d'arrêt du traitement, afin que les mêmes messages continuent aussi vers /var/log/syslog pour la vue centralisée.

```bash
sudo mkdir -p /var/log/remote
sudo chown syslog:adm /var/log/remote
sudo chmod 750 /var/log/remote
```
Crée le dossier dédié aux logs distants avec des permissions restreintes au groupe adm.

```bash
sudo systemctl restart rsyslog
```
Redémarre rsyslog pour appliquer la nouvelle configuration de réception.

```bash
ss -lunp | grep 514
```
Vérifie que rsyslog écoute bien en UDP sur le port 514.

## Émission des logs (Ubuntu-DC)

```bash
sudo nano /etc/rsyslog.d/90-forward-ubuntu01.conf
```
Crée la règle de forward sur Ubuntu-DC.

```
*.* action(type="omfwd" target="192.168.10.20" port="514" protocol="udp")
```
Envoie l'ensemble des messages de log générés sur Ubuntu-DC vers Ubuntu-01, port 514, en UDP.

```bash
sudo systemctl restart rsyslog
```
Redémarre rsyslog sur Ubuntu-DC pour activer le forward.

```bash
logger "Test de forward depuis Ubuntu-DC"
```
Génère un message de test, à vérifier ensuite côté Ubuntu-01 dans les deux fichiers de destination attendus.

## Rétention avec logrotate

```bash
sudo nano /etc/logrotate.d/remote-ubuntu-dc
```
Crée la configuration de rotation dédiée aux logs distants d'Ubuntu-DC.

```
/var/log/remote/ubuntu-dc.log {
    daily
    rotate 7
    delaycompress
    missingok
    notifempty
}
```
Rotation quotidienne, conservation de 7 jours, compression différée pour ne pas comprimer le fichier du jour en cours.

```bash
sudo logrotate -d /etc/logrotate.d/remote-ubuntu-dc
```
Teste la configuration en mode simulation, sans rien modifier réellement, avant de la valider en conditions réelles.

```bash
sudo logrotate -f /etc/logrotate.d/remote-ubuntu-dc
```
Force une rotation réelle pour valider que la configuration fonctionne correctement une fois le test en simulation concluant.

## Durcissement du pare-feu (UFW)

```bash
sudo ufw allow 22/tcp
```
Autorise SSH en priorité, avant toute activation du pare-feu, pour ne pas couper l'accès distant à la VM.

```bash
sudo ufw allow from 192.168.10.0/24 to any port 514 proto udp
```
Restreint la réception des logs au seul réseau interne du labo, plutôt que d'ouvrir le port 514/UDP à toute source.

```bash
sudo ufw show added
```
Liste les règles enregistrées avant activation (ufw status n'affiche rien tant que le pare-feu est inactif).

```bash
sudo ufw enable
```
Active le pare-feu avec la politique par défaut (deny en entrée, allow en sortie).

```bash
sudo ufw status verbose
```
Vérifie l'état et le détail des règles actives, puis revalidation de la connexion SSH et de la réception des logs pour confirmer qu'aucun flux légitime n'a été cassé.
