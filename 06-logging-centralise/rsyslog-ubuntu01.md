# Logging centralise avec rsyslog (Ubuntu-01)

## Contexte / objectif

Ubuntu-01 (192.168.10.20) n'avait jusque-la aucun role actif dans le labo CYBER-LAB. Decision de lui donner la fonction de serveur de logs centralise, recevant d'abord les logs d'Ubuntu-DC, avec pour objectif a terme de s'entrainer a la recherche et au filtrage de logs dans une logique proche d'un poste SOC.

## Choix techniques

Le protocole UDP est choisi pour la reception des logs, juge suffisant pour un labo (par opposition a TCP, plus fiable mais plus complexe a mettre en oeuvre pour ce cas d'usage). Le module imudp de rsyslog est active, avec ecoute sur le port 514, et l'ecoute est verifiee fonctionnelle avec la commande ss -lunp.

## Mise en place cote reception (Ubuntu-01)

Une regle rsyslog dediee est ajoutee dans /etc/rsyslog.d/10-remote-ubuntu-dc.conf. Elle filtre sur le nom d'hote source, hostname egal a ubuntu-dc, et route les logs recus vers deux destinations en parallele, sans stopper le traitement standard : /var/log/syslog pour une vue centralisee avec les logs locaux d'Ubuntu-01, et /var/log/remote/ubuntu-dc.log pour une vue filtree, specifique a cette source, dans un dossier dedie avec des permissions restreintes (groupe syslog:adm, mode 750).

Au moment de la mise en place, le pare-feu UFW s'est avere inactif sur Ubuntu-01 (serveur cree mais jamais durci) ; point releve pour une session ulterieure de durcissement.

## Mise en place cote emission (Ubuntu-DC)

Une regle de forward est ajoutee sur Ubuntu-DC, envoyant tous les messages vers 192.168.10.20 port 514, pour router l'ensemble de ses logs vers Ubuntu-01 en UDP. Un test de bout en bout avec la commande logger confirme que les messages generes sur Ubuntu-DC arrivent bien sur Ubuntu-01, dans les deux fichiers de destination attendus.

## Retention et rotation

La retention des logs est geree via logrotate, avec une configuration dediee dans /etc/logrotate.d/remote-ubuntu-dc : rotation quotidienne, conservation de 7 jours, compression differee (pour ne pas comprimer le fichier du jour en cours). La configuration est testee en mode simulation avec logrotate -d avant d'etre validee en conditions reelles, afin de ne pas saturer le disque avec un historique illimite.

## Ce que ca m'a appris

Separer la vue centralisee (syslog global) et la vue filtree par source (fichier dedie par machine emettrice) permet de garder a la fois une vision d'ensemble et la capacite a isoler rapidement les logs d'un hote precis, ce qui est directement transposable a un contexte SOC reel. Tester une politique de retention en mode simulation (logrotate -d) avant de la valider evite de decouvrir un probleme de configuration au moment ou les logs sont deja perdus. Decouvrir un pare-feu inactif sur un serveur fraichement cree est un rappel qu'un service par defaut n'est pas forcement durci, et qu'il faut verifier explicitement chaque machine ajoutee au labo plutot que de supposer une configuration coherente avec les autres.
