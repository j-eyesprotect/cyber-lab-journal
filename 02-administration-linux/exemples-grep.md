# Exemples de commandes grep

## Recherche de base

```bash
grep "ERROR" /var/log/syslog
```
Affiche toutes les lignes du fichier contenant le mot ERROR.

```bash
grep -i "error" /var/log/syslog
```
Recherche insensible à la casse (-i), trouve ERROR, error, Error, etc.

```bash
grep -n "ERROR" /var/log/syslog
```
Affiche le numéro de ligne devant chaque résultat trouvé, utile pour retrouver rapidement l'endroit exact dans le fichier.

```bash
grep -v "INFO" /var/log/syslog
```
Inverse la recherche (-v) : affiche toutes les lignes qui ne contiennent pas INFO.

## Recherche récursive dans plusieurs fichiers

```bash
grep -r "admincyber" /etc/
```
Recherche récursivement (-r) dans tous les fichiers du dossier /etc/ et de ses sous-dossiers.

```bash
grep -rl "PasswordAuthentication" /etc/ssh/
```
Recherche récursive combinée avec -l, qui affiche uniquement les noms des fichiers contenant le motif, sans afficher les lignes elles-mêmes.

```bash
grep -rn "TODO" /srv/entreprise/
```
Recherche récursive avec numéros de ligne, pratique pour retrouver tous les commentaires TODO laissés dans un ensemble de scripts.

## Compter les occurrences

```bash
grep -c "Failed password" /var/log/auth.log
```
Affiche uniquement le nombre de lignes contenant le motif, sans afficher les lignes elles-mêmes.

```bash
grep -o "Failed password" /var/log/auth.log | wc -l
```
Compte le nombre total d'occurrences du motif plutôt que le nombre de lignes, utile si le motif peut apparaître plusieurs fois sur une même ligne.

## Expressions régulières étendues

```bash
grep -E "error|warning|critical" /var/log/syslog
```
Recherche étendue (-E) permettant d'utiliser l'opérateur ou directement, sans avoir à l'échapper.

```bash
grep -E "^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" /var/log/auth.log
```
Recherche des lignes commençant par une adresse IPv4, en utilisant une expression régulière étendue.

```bash
grep -P "\d{4}-\d{2}-\d{2}" fichier.log
```
Utilise les expressions régulières compatibles Perl (-P), ici pour repérer une date au format AAAA-MM-JJ.

## Afficher le contexte autour d'un résultat

```bash
grep -A 3 "ACCESS_DENIED" /var/log/samba/log.smbd
```
Affiche la ligne correspondante ainsi que les 3 lignes suivantes (-A), utile pour voir ce qui se passe juste après l'erreur.

```bash
grep -B 3 "ACCESS_DENIED" /var/log/samba/log.smbd
```
Affiche la ligne correspondante ainsi que les 3 lignes précédentes (-B), pour voir le contexte qui a mené à l'erreur.

```bash
grep -C 3 "ACCESS_DENIED" /var/log/samba/log.smbd
```
Affiche 3 lignes avant et après chaque résultat (-C), combinant -A et -B en une seule option.

## grep combiné avec d'autres commandes (pipes)

```bash
grep "Failed password" /var/log/auth.log | sort
```
Trie alphabétiquement les lignes correspondantes, utile pour regrouper visuellement des entrées similaires.

```bash
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
```
Extrait un champ précis de chaque ligne, ici l'utilisateur ciblé, puis compte et trie les occurrences par fréquence décroissante, pour voir quel compte a été le plus ciblé par des tentatives échouées.

```bash
grep "Failed password" /var/log/auth.log | sort | uniq -c
```
Compte combien de fois chaque ligne distincte apparaît après tri, uniq -c ayant besoin d'une entrée triée pour fonctionner correctement.

```bash
grep "Failed password" /var/log/auth.log | tail -20
```
Affiche uniquement les 20 dernières lignes correspondantes, pratique pour se concentrer sur les événements les plus récents dans un fichier volumineux.

```bash
grep "Failed password" /var/log/auth.log | head -20
```
Affiche uniquement les 20 premières lignes correspondantes.

```bash
grep "Failed password" /var/log/auth.log | wc -l
```
Compte le nombre total de lignes correspondantes, équivalent plus manuel de l'option -c vue plus haut.

```bash
tail -f /var/log/syslog | grep --line-buffered "ERROR"
```
Suit un fichier de log en direct avec tail -f et filtre en temps réel les lignes contenant ERROR ; l'option --line-buffered évite que grep ne retienne les résultats en mémoire tampon avant de les afficher.
