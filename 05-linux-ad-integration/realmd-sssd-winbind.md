# Intégration Linux-AD (realmd / sssd / winbind) sur Ubuntu-DC

## Contexte et objectif

Dans le labo CYBER-LAB, Ubuntu-DC sert des partages Samba par département (commerciaux, direction, administration, informatique), tandis que WIN-AD-DC01 est le contrôleur du domaine Active Directory cyberlab.local. Les comptes Samba étaient gérés localement, de manière totalement indépendante de l'annuaire AD.

Objectif concret : faire fonctionner le mappage automatique de lecteurs réseau par GPO (Group Policy Preferences) sur le poste client Windows, pour que chaque utilisateur AD récupère automatiquement le lecteur réseau de son département à la connexion.

## Problème initial

Le mappage de lecteur GPO échouait systématiquement avec l'erreur : 0x80070056, « Le mot de passe réseau n'est pas correct » (journal Application, source Group Policy Drive Maps).

Diagnostic : l'authentification AD (Kerberos/NTLM) et l'authentification Samba locale sont deux systèmes distincts. Sans intégration, le partage Samba ne peut pas valider une identité qui vient d'AD, même si le compte Windows est bien dans le domaine.

Plan en six étapes préparé avant de commencer : étape 1, vérifier la résolution DNS et la synchro horaire entre les deux serveurs, critique pour Kerberos ; étape 2, installer les paquets nécessaires (realmd, sssd, winbind) sur Ubuntu-DC ; étape 3, realm discover puis realm join pour rattacher Ubuntu-DC au domaine ; étape 4, reconfigurer Samba en security = ads, membre du domaine au lieu d'authentification locale ; étape 5, tester l'authentification AD sur le partage Samba ; étape 6, re-tester le mappage de lecteur GPO.

## Mise en place

DNS pointé vers le contrôleur de domaine, synchro horaire forcée manuellement entre les deux machines, car un écart d'horloge entraîne un échec Kerberos garanti. Le realm join a été exécuté avec succès : Ubuntu-DC a rejoint le domaine cyberlab.local. Samba a basculé en security = ads, en tant que machine membre du domaine. Un crash récurrent de winbindd a été rencontré et corrigé en cours de route.

## Blocage principal : ACCESS_DENIED malgré une authentification AD correcte

Après le realm join et le passage en security = ads, l'accès au partage Samba renvoyait ACCESS_DENIED, alors même que sssd résolvait correctement les identités AD (commande id sur un utilisateur fonctionnelle côté résolution de nom).

Première session de debug : beaucoup d'étapes réussies individuellement (join, bascule Samba, résolution sssd), mais le blocage final persistait sans explication claire. Session mise en pause, frustrant de ne pas comprendre malgré la réussite de nombreuses étapes.

### Fausse piste écartée : l'horloge

À la reprise, découverte que l'horloge d'Ubuntu-DC ne persistait pas après redémarrage, le correctif manuel de synchro horaire de la veille n'était pas permanent. Résultat : des échecs Kerberos en continu côté sssd depuis chaque boot. C'était une cause plausible des blocages constatés, à corriger en parallèle (synchro horaire permanente et correction d'une mise en veille intempestive de la VM du contrôleur de domaine).

### Cause racine réelle

Après correction du problème d'horloge, le blocage ACCESS_DENIED persistait encore. Investigation plus poussée : le backend d'identité idmap_sss n'existe pas dans le paquet winbind d'Ubuntu, il a été retiré d'upstream Samba. Autrement dit, la configuration suivie, qui s'appuyait implicitement sur ce backend, ne pouvait tout simplement pas fonctionner sur cette distribution.

## Résolution

Passage à idmap config CYBERLAB : backend = rid (au lieu de idmap_sss) dans la configuration Samba. Mise à jour de /etc/nsswitch.conf pour que passwd et group pointent vers winbind. Ajout de winbind use default domain = yes. Enfin, chgrp récursif des fichiers existants sous /srv/entreprise/ pour les faire pointer vers les nouveaux GID issus du mapping rid, les anciens propriétaires et groupes Unix locaux étant devenus obsolètes après le changement de backend d'identité.

Résultat : le mappage de lecteur réseau GPO (G:) a été testé et confirmé fonctionnel sur Win-Client pour le groupe commerciaux. L'objectif initial est atteint.

Test en cours pour les trois autres départements : un souci restait ouvert sur direction au moment de la rédaction (accès au partage OK en ligne de commande, mais lecteur GPO non mappé automatiquement), à investiguer dans une session ultérieure.

## Ce que ça m'a appris

Kerberos est extrêmement sensible à la synchro horaire : un écart d'horloge, même corrigé « à la main » une fois, peut revenir si le correctif n'est pas persistant (NTP ou systemd-timesyncd configuré durablement, et non une commande ponctuelle).

Un correctif qui semble marcher à l'instant T doit être vérifié après un redémarrage complet avant d'être considéré comme acquis.

Une résolution d'identité réussie via sssd ne garantit pas que l'autorisation, c'est-à-dire le mapping vers un UID/GID Unix exploitable par Samba, fonctionne : authentification et autorisation sont deux couches distinctes, et une erreur ACCESS_DENIED peut venir de la seconde même quand la première est correcte.

Il est important de vérifier qu'un paquet Linux fournit réellement ce que la documentation ou les tutoriels supposent : idmap_sss est un exemple concret de fonctionnalité documentée mais absente du paquet réellement installé sur Ubuntu.

Méthode à retenir : isoler les variables une par une (horloge, puis backend idmap, puis permissions fichiers) plutôt que de tout reconfigurer en bloc, pour comprendre précisément quelle étape corrige quoi.
