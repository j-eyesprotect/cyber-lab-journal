# Logging centralisé avec rsyslog (Ubuntu-01)

## Contexte et objectif

Ubuntu-01 (192.168.10.20) n'avait jusque-là aucun rôle actif dans le labo CYBER-LAB. Décision de lui donner la fonction de serveur de logs centralisé, recevant d'abord les logs d'Ubuntu-DC, avec pour objectif à terme de s'entraîner à la recherche et au filtrage de logs dans une logique proche d'un poste SOC.

## Choix techniques

Le protocole UDP est choisi pour la réception des logs, jugé suffisant pour un labo (par opposition à TCP, plus fiable mais plus complexe à mettre en œuvre pour ce cas d'usage). Le module imudp de rsyslog est activé, avec écoute sur le port 514, et l'écoute est vérifiée fonctionnelle avec la commande ss -lunp.

## Mise en place côté réception (Ubuntu-01)

Une règle rsyslog dédiée est ajoutée dans /etc/rsyslog.d/10-remote-ubuntu-dc.conf. Elle filtre sur le nom d'hôte source, hostname égal à ubuntu-dc, et route les logs reçus vers deux destinations en parallèle, sans stopper le traitement standard : /var/log/syslog pour une vue centralisée avec les logs locaux d'Ubuntu-01, et /var/log/remote/ubuntu-dc.log pour une vue filtrée, spécifique à cette source, dans un dossier dédié avec des permissions restreintes (groupe syslog:adm, mode 750).

Au moment de la mise en place, le pare-feu UFW s'est avéré inactif sur Ubuntu-01 (serveur créé mais jamais durci) ; durci dans une session ultérieure (voir la section dédiée plus bas).

## Mise en place côté émission (Ubuntu-DC)

Une règle de forward est ajoutée sur Ubuntu-DC, envoyant tous les messages vers 192.168.10.20 port 514, pour router l'ensemble de ses logs vers Ubuntu-01 en UDP. Un test de bout en bout avec la commande logger confirme que les messages générés sur Ubuntu-DC arrivent bien sur Ubuntu-01, dans les deux fichiers de destination attendus.

## Rétention et rotation

La rétention des logs est gérée via logrotate, avec une configuration dédiée dans /etc/logrotate.d/remote-ubuntu-dc : rotation quotidienne, conservation de 7 jours, compression différée (pour ne pas comprimer le fichier du jour en cours). La configuration est testée en mode simulation avec logrotate -d avant d'être validée en conditions réelles, afin de ne pas saturer le disque avec un historique illimité.

## Durcissement du pare-feu (UFW)

Le pare-feu UFW, trouvé inactif lors de la mise en place initiale, est activé dans une session dédiée. Avant toute activation, une règle autorisant SSH (22/tcp) est ajoutée en priorité, pour ne pas couper l'accès distant à la VM au moment du passage en mode actif — un filet de sécurité supplémentaire (accès via la console VirtualBox) est identifié avant de valider, au cas où. Une seconde règle restreint ensuite la réception des logs au seul réseau du labo (192.168.10.0/24) sur le port 514/UDP, plutôt qu'une ouverture globale à toute source. Une fois UFW activé (politique par défaut : deny en entrée, allow en sortie, routed désactivé), la connexion SSH et la réception des logs sont toutes deux revérifiées pour confirmer qu'aucun flux légitime n'a été cassé par le passage en mode actif.

## Extension à Ubuntu-Client (deuxième source)

Une deuxième machine, Ubuntu-Client (192.168.10.30), est ajoutée comme source de logs, en suivant le même schéma qu'Ubuntu-DC : une règle de forward sur Ubuntu-Client vers 192.168.10.20:514 en UDP, et côté Ubuntu-01 une règle dédiée dans /etc/rsyslog.d/11-remote-ubuntu-client.conf qui filtre cette fois sur $fromhost-ip (l'adresse IP source du paquet) plutôt que sur le hostname contenu dans le message, et route vers /var/log/remote/ubuntu-client.log. Le dossier /var/log/remote existant déjà avec les bonnes permissions, aucune création supplémentaire n'est nécessaire. La règle UFW existante (192.168.10.0/24 sur 514/UDP) couvrait déjà cette nouvelle source sans modification.

Une rotation logrotate dédiée est ajoutée pour ce nouveau fichier, avec le même format que celle d'Ubuntu-DC.

## Ce que ça m'a appris

Séparer la vue centralisée (syslog global) et la vue filtrée par source (fichier dédié par machine émettrice) permet de garder à la fois une vision d'ensemble et la capacité à isoler rapidement les logs d'un hôte précis, ce qui est directement transposable à un contexte SOC réel. Tester une politique de rétention en mode simulation (logrotate -d) avant de la valider évite de découvrir un problème de configuration au moment où les logs sont déjà perdus. Découvrir un pare-feu inactif sur un serveur fraîchement créé est un rappel qu'un service par défaut n'est pas forcément durci, et qu'il faut vérifier explicitement chaque machine ajoutée au labo plutôt que de supposer une configuration cohérente avec les autres. Activer un pare-feu sur une machine administrée à distance demande de préparer la règle d'accès distant (SSH) avant l'activation, sous peine de se couper soi-même l'accès ; et restreindre une ouverture de port à la plage réseau réellement concernée, plutôt que de l'ouvrir largement, est une pratique directement transposable à un contexte professionnel. Une règle de pare-feu pensée par plage réseau plutôt que par machine individuelle évite d'avoir à la retoucher à chaque nouvelle source ajoutée au labo. Oublier de redémarrer rsyslog après avoir créé ou modifié un fichier de config reste le piège le plus fréquent : la configuration est valide mais inactive tant que le service n'a pas rechargé. Dans le même esprit, une règle logrotate sans le bloc postrotate qui indique à rsyslog de rouvrir ses fichiers fait certes tourner le nom du fichier (renommage en .1), mais rsyslog continue d'écrire dans l'ancien fichier renommé au lieu de basculer vers un nouveau fichier — un problème silencieux qui ne se voit qu'en vérifiant concrètement qu'un nouveau fichier vide apparaît après rotation.
