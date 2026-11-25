# Commandes : intégration Linux-AD (realmd / sssd / winbind)

## Préparation : DNS et synchronisation horaire

```bash
sudo nano /etc/resolv.conf
```
Pointe la résolution DNS d'Ubuntu-DC vers le contrôleur de domaine (192.168.10.50), indispensable pour que Kerberos et la découverte du domaine fonctionnent.

```bash
sudo timedatectl set-ntp true
```
Active la synchronisation horaire automatique, critique pour Kerberos qui tolère très peu d'écart d'horloge entre client et contrôleur de domaine.

```bash
timedatectl status
```
Vérifie que l'horloge est bien synchronisée et que la synchronisation est active, pas seulement corrigée ponctuellement.

## Installation et jonction au domaine

```bash
sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin winbind
```
Installe l'ensemble des paquets nécessaires à l'intégration d'Ubuntu-DC à l'annuaire Active Directory.

```bash
sudo realm discover cyberlab.local
```
Vérifie que le domaine cyberlab.local est bien détecté et affiche les paquets requis pour la jonction.

```bash
sudo realm join cyberlab.local -U administrateur
```
Rattache Ubuntu-DC au domaine cyberlab.local, avec un compte disposant des droits nécessaires pour joindre une machine au domaine.

```bash
realm list
```
Confirme que la machine est bien jointe au domaine et affiche les détails de la configuration réalisée (sssd, krb5, etc.).

## Bascule de Samba en membre de domaine

```
security = ads
realm = CYBERLAB.LOCAL
workgroup = CYBERLAB
```
Remplace l'authentification locale de Samba par une authentification déléguée à Active Directory, en déclarant Ubuntu-DC comme machine membre du domaine.

```bash
sudo systemctl restart smbd winbind
```
Redémarre Samba et winbind pour appliquer le nouveau mode d'authentification.

## Débogage : identité résolue mais accès refusé (ACCESS_DENIED)

```bash
id jrat@cyberlab.local
```
Vérifie que sssd résout correctement l'identité d'un utilisateur AD côté Linux ; une résolution réussie ici ne garantit pas encore que l'accès au partage Samba fonctionnera.

```bash
sudo systemctl status winbindd
```
Vérifie que le service winbind est bien actif, après un crash récurrent rencontré en cours de mise en place.

```bash
wbinfo -u
```
Liste les utilisateurs du domaine vus par winbind, pour confirmer que la communication entre winbind et le contrôleur de domaine fonctionne.

```bash
klist
```
Affiche les tickets Kerberos actifs, utile pour confirmer ou écarter un problème d'authentification Kerberos lors du diagnostic d'un ACCESS_DENIED.

```bash
sudo tail -f /var/log/samba/log.smbd
```
Suit en direct le journal Samba pendant une tentative de connexion, pour voir apparaître l'erreur ACCESS_DENIED au moment précis où elle se produit.

## Cause racine : backend idmap_sss absent et correction

```bash
apt list --installed | grep winbind
```
Vérifie la version du paquet winbind réellement installée, après avoir découvert que le backend idmap_sss documenté dans certains tutoriels a été retiré d'upstream Samba sur cette distribution.

```
idmap config * : backend = tdb
idmap config * : range = 10000-19999
idmap config CYBERLAB : backend = rid
idmap config CYBERLAB : range = 20000-29999
winbind use default domain = yes
```
Remplace le backend idmap_sss, indisponible, par le backend rid dans la configuration Samba, pour que les identités AD soient correctement mappées vers des UID/GID Unix exploitables.

```bash
sudo nano /etc/nsswitch.conf
```
Met à jour nsswitch.conf pour que les bases passwd et group interrogent aussi winbind, et pas seulement les fichiers locaux.

```
passwd: files winbind
group: files winbind
```
Lignes modifiées dans nsswitch.conf pour intégrer winbind à la résolution des utilisateurs et des groupes.

```bash
sudo systemctl restart smbd winbind sssd
```
Redémarre l'ensemble des services concernés après le changement de backend idmap.

```bash
sudo chgrp -R "CYBERLAB\commerciaux" /srv/entreprise/commerciaux
```
Fait pointer récursivement le groupe propriétaire des fichiers existants vers le nouveau GID issu du mapping rid, les anciens groupes Unix locaux étant devenus obsolètes après le changement de backend.

## Vérification finale

```bash
smbclient //localhost/commerciaux -U jrat@cyberlab.local
```
Teste l'accès au partage avec un compte AD, pour confirmer que l'authentification et l'autorisation fonctionnent désormais de bout en bout, avant de retester le mappage de lecteur réseau via GPO sur Win-Client.
