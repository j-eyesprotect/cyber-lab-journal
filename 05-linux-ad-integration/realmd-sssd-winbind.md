# Integration Linux-AD (realmd / sssd / winbind) sur Ubuntu-DC

## Contexte / objectif

Dans le labo CYBER-LAB, Ubuntu-DC sert des partages Samba par departement (commerciaux, direction, administration, informatique), tandis que WIN-AD-DC01 est le controleur du domaine Active Directory cyberlab.local. Les comptes Samba etaient geres localement, de maniere totalement independante de l'annuaire AD.

Objectif concret : faire fonctionner le mappage automatique de lecteurs reseau par GPO (Group Policy Preferences) sur le poste client Windows, pour que chaque utilisateur AD recupere automatiquement le lecteur reseau de son departement a la connexion.

## Probleme initial

Le mappage de lecteur GPO echouait systematiquement avec l'erreur : 0x80070056, "Le mot de passe reseau n'est pas correct" (journal Application, source Group Policy Drive Maps).

Diagnostic : l'authentification AD (Kerberos/NTLM) et l'authentification Samba locale sont deux systemes distincts. Sans integration, le partage Samba ne peut pas valider une identite qui vient d'AD, meme si le compte Windows est bien dans le domaine.

Plan en 6 etapes prepare avant de commencer : etape 1, verifier la resolution DNS et la synchro horaire entre les deux serveurs, critique pour Kerberos. etape 2, installer les paquets necessaires (realmd, sssd, winbind) sur Ubuntu-DC. etape 3, realm discover puis realm join pour rattacher Ubuntu-DC au domaine. etape 4, reconfigurer Samba en security = ads, membre du domaine au lieu d'authentification locale. etape 5, tester l'authentification AD sur le partage Samba. etape 6, re-tester le mappage de lecteur GPO.

## Mise en place

DNS pointe vers le controleur de domaine, synchro horaire forcee manuellement entre les deux machines, car un ecart d'horloge entraine un echec Kerberos garanti. Le realm join a ete execute avec succes : Ubuntu-DC a rejoint le domaine cyberlab.local. Samba a bascule en security = ads, en tant que machine membre du domaine. Un crash recurrent de winbindd a ete rencontre et corrige en cours de route.

## Blocage principal : ACCESS_DENIED malgre une authentification AD correcte

Apres le realm join et le passage en security = ads, l'acces au partage Samba renvoyait ACCESS_DENIED, alors meme que sssd resolvait correctement les identites AD (commande id sur un utilisateur fonctionnelle cote resolution de nom).

Premiere session de debug : beaucoup d'etapes reussies individuellement (join, bascule Samba, resolution sssd), mais le blocage final persistait sans explication claire. Session mise en pause, frustrant de ne pas comprendre malgre la reussite de nombreuses etapes.

### Fausse piste ecartee : l'horloge

A la reprise, decouverte que l'horloge d'Ubuntu-DC ne persistait pas apres redemarrage, le correctif manuel de synchro horaire de la veille n'etait pas permanent. Resultat : des echecs Kerberos en continu cote sssd depuis chaque boot. C'etait une cause plausible des blocages constates, a corriger en parallele (synchro horaire permanente et correction d'une mise en veille intempestive de la VM du controleur de domaine).

### Cause racine reelle

Apres correction du probleme d'horloge, le blocage ACCESS_DENIED persistait encore. Investigation plus poussee : le backend d'identite idmap_sss n'existe pas dans le paquet winbind d'Ubuntu, il a ete retire d'upstream Samba. Autrement dit, la configuration suivie, qui s'appuyait implicitement sur ce backend, ne pouvait tout simplement pas fonctionner sur cette distribution.

## Resolution

Passage a idmap config CYBERLAB : backend = rid (au lieu de idmap_sss) dans la configuration Samba. Mise a jour de /etc/nsswitch.conf pour que passwd et group pointent vers winbind. Ajout de winbind use default domain = yes. Enfin, chgrp recursif des fichiers existants sous /srv/entreprise/ pour les faire pointer vers les nouveaux GID issus du mapping rid, les anciens proprietaires et groupes Unix locaux etant devenus obsoletes apres le changement de backend d'identite.

Resultat : le mappage de lecteur reseau GPO (G:) a ete teste et confirme fonctionnel sur Win-Client pour le groupe commerciaux. L'objectif initial est atteint.

Test en cours pour les 3 autres departements : un souci restait ouvert sur direction au moment de la redaction (acces au partage OK en ligne de commande, mais lecteur GPO non mappe automatiquement), a investiguer dans une session ulterieure.

## Ce que ca m'a appris

Kerberos est extremement sensible a la synchro horaire : un ecart d'horloge, meme corrige a la main une fois, peut revenir si le correctif n'est pas persistant (NTP ou systemd-timesyncd configure durablement, et non une commande ponctuelle).

Un correctif qui semble marcher a l'instant T doit etre verifie apres un redemarrage complet avant d'etre considere comme acquis.

Une resolution d'identite reussie via sssd ne garantit pas que l'autorisation, c'est a dire le mapping vers un UID/GID Unix exploitable par Samba, fonctionne : authentification et autorisation sont deux couches distinctes, et une erreur ACCESS_DENIED peut venir de la seconde meme quand la premiere est correcte.

Il est important de verifier qu'un paquet Linux fournit reellement ce que la documentation ou les tutoriels supposent : idmap_sss est un exemple concret de fonctionnalite documentee mais absente du paquet reellement installe sur Ubuntu.

Methode a retenir : isoler les variables une par une (horloge, puis backend idmap, puis permissions fichiers) plutot que de tout reconfigurer en bloc, pour comprendre precisement quelle etape corrige quoi.
