# Commandes : topologie réseau sous Cisco Packet Tracer

## Configuration du routeur (CLI Cisco)

```
enable
configure terminal
```
Passe en mode privilégié puis en mode de configuration globale sur le routeur.

```
interface GigabitEthernet0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
```
Configure l'adresse IP de l'interface connectée au réseau 192.168.1.0/24 et l'active, une interface étant administrativement désactivée par défaut sur un routeur Cisco.

```
interface GigabitEthernet0/1
ip address 192.168.2.1 255.255.255.0
no shutdown
```
Même configuration pour l'interface connectée au second réseau, 192.168.2.0/24.

```
end
write memory
```
Quitte le mode de configuration et sauvegarde la configuration courante dans la configuration de démarrage du routeur.

## Vérification

```
show ip interface brief
```
Affiche l'état et l'adresse IP de chaque interface du routeur, pour confirmer que les deux interfaces sont up et correctement adressées.

```
ping 192.168.2.10
```
Depuis un PC du réseau 192.168.1.0/24, confirme que le routage de base fonctionne entre les deux réseaux distincts.

## Configuration des PC

```
IP Address: 192.168.1.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.1
```
Paramètres IP statiques configurés sur chaque PC via l'onglet Desktop puis IP Configuration de Packet Tracer, avec la passerelle par défaut pointée vers l'interface du routeur sur le même réseau.
