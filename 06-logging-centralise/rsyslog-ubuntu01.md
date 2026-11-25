# Logging centralisé avec rsyslog (Ubuntu-01)

## Contexte et objectif

Ubuntu-01 (192.168.10.20) n'avait jusque-là aucun rôle actif dans le labo CYBER-LAB. Décision de lui donner la fonction de serveur de logs centralisé, recevant d'abord les logs d'Ubuntu-DC, avec pour objectif à terme de s'entraîner à la recherche et au filtrage de logs dans une logique proche d'un poste SOC.

## Choix techniques

Le protocole UDP est choisi pour la réception des logs, jugé suffisant pour un labo (par opposition à TCP, plus fiable mais plus complexe à mettre en œuvre pour ce cas d'usage). Le module imudp de rsyslog est activé, avec écoute sur le port 514, et l'écoute est vérifiée fonctionnelle avec la commande ss -lunp.

## Mise en place côté réception (Ubuntu-01)

Une règle rsyslog dédiée est ajoutée dans /etc/rsyslog.d/10-remote-ubuntu-dc.conf. Elle filtre sur le nom d'hôte source, hostname égal à ubuntu-dc, et route les logs reçus vers deux destinations en parallèle, sans stopper le traitement standard : /var/log/syslog pour une vue centralisée avec les logs locaux d'Ubuntu-01, et /var/log/remote/ubuntu-dc.log pour une vue filtrée, spécifique à cette source, dans un dossier dédié avec des permissions restreintes (groupe syslog:adm, mode 750).

Au moment de la mise en place, le pare-feu UFW s'est avéré inactif sur Ubuntu-01 (serveur créé mais jamais durci) ; point relevé pour une session ultérieure de durcissement.

## Mise en place côté émission (Ubuntu-DC)

Une règle de forward est ajoutée sur Ubuntu-DC, envoyant tous les messages vers 192.168.10.20 port 514, pour router l'ensemble de ses logs vers Ubuntu-01 en UDP. Un test de bout en bout avec la commande logger confirme que les messages générés sur Ubuntu-DC arrivent bien sur Ubuntu-01, dans les deux fichiers de destination attendus.

## Rétention et rotation

La rétention des logs est gérée via logrotate, avec une configuration dédiée dans /etc/logrotate.d/remote-ubuntu-dc : rotation quotidienne, conservation de 7 jours, compression différée (pour ne pas comprimer le fichier du jour en cours). La configuration est testée en mode simulation avec logrotate -d avant d'être validée en conditions réelles, afin de ne pas saturer le disque avec un historique illimité.

## Ce que ça m'a appris

Séparer la vue centralisée (syslog global) et la vue filtrée par source (fichier dédié par machine émettrice) permet de garder à la fois une vision d'ensemble et la capacité à isoler rapidement les logs d'un hôte précis, ce qui est directement transposable à un contexte SOC réel. Tester une politique de rétention en mode simulation (logrotate -d) avant de la valider évite de découvrir un problème de configuration au moment où les logs sont déjà perdus. Découvrir un pare-feu inactif sur un serveur fraîchement créé est un rappel qu'un service par défaut n'est pas forcément durci, et qu'il faut vérifier explicitement chaque machine ajoutée au labo plutôt que de supposer une configuration cohérente avec les autres.
