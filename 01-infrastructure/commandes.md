# Commandes : infrastructure VirtualBox

## Réseau interne et réseau host-only

```bash
VBoxManage list hostonlyifs
```
Liste les interfaces réseau host-only disponibles sur l'hôte, utile pour identifier celle utilisée comme canal d'administration SSH (plage 192.168.56.x).

```bash
VBoxManage hostonlyif ipconfig "VirtualBox Host-Only Ethernet Adapter" --ip 192.168.56.1 --netmask 255.255.255.0
```
Configure l'adresse IP de l'interface host-only côté hôte, pour qu'elle serve de passerelle d'administration vers les VMs Linux.

```bash
VBoxManage modifyvm "Ubuntu-DC" --nic2 hostonly --hostonlyadapter2 "VirtualBox Host-Only Ethernet Adapter"
```
Ajoute une deuxième carte réseau host-only à la VM Ubuntu-DC, séparée de la carte réseau interne de production (192.168.10.x).

## Adressage IP statique avec Netplan

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      addresses: [192.168.10.10/24]
      gateway4: 192.168.10.1
      nameservers:
        addresses: [192.168.10.1]
```
Fichier de configuration Netplan définissant une adresse IP statique pour Ubuntu-DC sur le réseau interne du labo.

```bash
sudo netplan apply
```
Applique la nouvelle configuration réseau sans redémarrer la machine.

```bash
ip a
```
Vérifie que l'adresse IP statique est bien active sur l'interface réseau.

## Carte NAT ponctuelle sur les VMs Windows

```bash
VBoxManage modifyvm "WIN-AD-DC01" --nic3 nat
```
Active une carte NAT supplémentaire pour les mises à jour Windows, à désactiver ensuite pour ne pas l'exposer en permanence.

```bash
VBoxManage modifyvm "WIN-AD-DC01" --nic3 none
```
Désactive la carte NAT une fois les mises à jour terminées.
