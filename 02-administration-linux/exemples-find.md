# Exemples de commandes find

## Recherche de base par nom

```bash
find /srv/entreprise -name "*.txt"
```
Recherche tous les fichiers se terminant par .txt dans l'arborescence /srv/entreprise, recherche sensible à la casse.

```bash
find /srv/entreprise -iname "*rapport*"
```
Recherche insensible à la casse (iname), ici tous les fichiers ou dossiers contenant "rapport" dans leur nom, quelle que soit la casse.

```bash
find / -maxdepth 2 -name "*.conf"
```
Limite la profondeur de recherche à deux niveaux sous la racine, utile pour éviter une recherche trop longue sur tout le système de fichiers.

## Recherche par type

```bash
find /srv/entreprise -type f
```
Ne retourne que les fichiers réguliers, en excluant les dossiers et les liens symboliques.

```bash
find /srv/entreprise -type d
```
Ne retourne que les dossiers.

```bash
find /srv/entreprise -type l
```
Ne retourne que les liens symboliques, utile pour repérer des liens cassés ou à vérifier.

## Recherche par permissions et propriétaire

```bash
find /srv/entreprise -type f -perm -002
```
Recherche les fichiers disposant d'un droit d'écriture pour tout le monde (other), utile pour repérer une permission trop permissive.

```bash
find /srv/entreprise -perm 777
```
Recherche les fichiers ou dossiers ayant exactement les droits 777, c'est-à-dire tous les droits pour tout le monde.

```bash
find /srv/entreprise -user tbernard
```
Recherche tous les fichiers appartenant à l'utilisateur tbernard.

```bash
find /srv/entreprise -group commerciaux
```
Recherche tous les fichiers appartenant au groupe commerciaux.

```bash
find /srv/entreprise -nouser
```
Recherche les fichiers dont le propriétaire ne correspond plus à aucun utilisateur existant, souvent le signe d'un compte supprimé sans nettoyage de ses fichiers.

## Recherche par taille et par date

```bash
find /var/log -size +100M
```
Recherche les fichiers de plus de 100 Mo, utile pour repérer un journal qui grossit anormalement.

```bash
find /var/log -size -1k
```
Recherche les fichiers de moins de 1 Ko, par exemple pour repérer des fichiers vides ou quasiment vides.

```bash
find /var/log -mtime -7
```
Recherche les fichiers modifiés au cours des 7 derniers jours.

```bash
find /var/log -mtime +30
```
Recherche les fichiers non modifiés depuis plus de 30 jours, utile avant un nettoyage ou un archivage.

```bash
find /tmp -atime +14
```
Recherche les fichiers non accédés depuis plus de 14 jours, basé sur la date de dernier accès plutôt que de modification.

## Combiner find et une action avec -exec

```bash
find /srv/entreprise -type f -name "*.tmp" -exec rm {} \;
```
Supprime chaque fichier .tmp trouvé ; {} représente le fichier courant et le point-virgule final termine la commande -exec, exécutée une fois par fichier trouvé.

```bash
find /srv/entreprise -type f -name "*.sh" -exec chmod +x {} \;
```
Rend exécutables tous les scripts .sh trouvés dans l'arborescence.

```bash
find /srv/entreprise -type f -name "*.log" -exec cp {} /srv/archive/ \;
```
Copie chaque fichier .log trouvé vers un dossier d'archive, en conservant le nom original du fichier.

```bash
find /srv/entreprise -type f -name "*.tmp" -exec rm {} +
```
Variante de -exec se terminant par un signe plus plutôt qu'un point-virgule : regroupe les fichiers trouvés en un minimum d'appels à la commande rm, plus efficace quand il y a beaucoup de résultats.

```bash
find /srv/entreprise -type f -newer /srv/entreprise/reference.txt
```
Recherche les fichiers plus récents qu'un fichier de référence donné, pratique pour retrouver ce qui a changé depuis un point donné.

## Suppression directe avec -delete

```bash
find /tmp -type f -name "*.tmp" -delete
```
Supprime directement les fichiers trouvés sans passer par -exec rm, plus rapide mais à utiliser avec prudence puisqu'il n'y a pas de confirmation.

## find combiné avec xargs

```bash
find /srv/entreprise -type f -name "*.log" | xargs grep -l "ERROR"
```
Transmet la liste des fichiers .log trouvés à grep, qui affiche ensuite uniquement les noms des fichiers contenant le mot ERROR.

```bash
find /srv/entreprise -type f -name "*.bak" -print0 | xargs -0 rm
```
Utilise des séparateurs nuls, via print0 côté find et l'option 0 côté xargs, pour gérer correctement les noms de fichiers contenant des espaces ou des caractères spéciaux.

## find combiné avec plusieurs conditions

```bash
find /srv/entreprise -type f -name "*.conf" -not -path "*/backup/*"
```
Recherche les fichiers de configuration tout en excluant ceux situés dans un dossier backup.

```bash
find /srv/entreprise \( -name "*.log" -o -name "*.txt" \)
```
Recherche les fichiers correspondant à l'une ou l'autre des deux extensions, grâce à l'opérateur logique -o (ou) entouré de parenthèses échappées.
