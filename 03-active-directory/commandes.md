# Commandes : Active Directory (WIN-AD-DC01)

## Installation du rôle et promotion du contrôleur de domaine

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```
Installe le rôle Active Directory Domain Services ainsi que les outils d'administration associés.

```powershell
Install-ADDSForest -DomainName "cyberlab.local" -InstallDns
```
Promeut le serveur en contrôleur de domaine, crée une nouvelle forêt et le domaine cyberlab.local, avec le rôle DNS installé en parallèle.

```powershell
Get-ADDomain
```
Vérifie que la promotion du contrôleur de domaine a bien abouti et affiche les informations du domaine.

## Structuration de l'annuaire

```powershell
New-ADOrganizationalUnit -Name "Entreprise" -Path "DC=cyberlab,DC=local" -ProtectedFromAccidentalDeletion $false
```
Crée l'OU racine Entreprise, avec la protection contre la suppression accidentelle désactivée pour faciliter les manipulations pendant l'apprentissage.

```powershell
New-ADOrganizationalUnit -Name "Informatique" -Path "OU=Entreprise,DC=cyberlab,DC=local" -ProtectedFromAccidentalDeletion $false
```
Crée une OU fille, ici Informatique, sous l'OU Entreprise ; la même commande est répétée pour Direction, Administration et Commerciaux.

## Création des comptes et des groupes

```powershell
New-ADUser -Name "Jean Rat" -SamAccountName "jrat" -Path "OU=Informatique,OU=Entreprise,DC=cyberlab,DC=local" -AccountPassword (ConvertTo-SecureString "MotDePasse!2026" -AsPlainText -Force) -Enabled $true
```
Crée un compte utilisateur dans l'OU Informatique, avec un mot de passe respectant la politique de complexité par défaut d'AD, et l'active immédiatement.

```powershell
New-ADGroup -Name "GG_Commerciaux" -GroupScope Global -Path "OU=Commerciaux,OU=Entreprise,DC=cyberlab,DC=local"
```
Crée un groupe de sécurité global pour le département Commerciaux, suivant la convention de nommage GG_.

```powershell
Add-ADGroupMember -Identity "GG_Commerciaux" -Members "tbernard"
```
Ajoute l'utilisateur tbernard au groupe GG_Commerciaux.

## Jonction du poste client au domaine

```powershell
Add-Computer -DomainName "cyberlab.local" -Credential (Get-Credential) -Restart
```
Joint le poste Win-Client au domaine cyberlab.local, demande les identifiants d'un compte autorisé à joindre des machines au domaine, puis redémarre automatiquement.

## Script de réinitialisation de mots de passe en masse

```powershell
$users = Get-ADUser -Filter * -SearchBase "OU=Entreprise,DC=cyberlab,DC=local" | Select-Object -ExpandProperty SamAccountName
```
Récupère la liste de tous les comptes utilisateurs de l'entreprise fictive, affichée ensuite sous forme de liste numérotée dans le script interactif.

```powershell
$nouveauMdp = -join ((65..90) + (97..122) + (48..57) | Get-Random -Count 12 | ForEach-Object {[char]$_})
```
Génère un mot de passe aléatoire de 12 caractères à partir de lettres majuscules, minuscules et chiffres.

```powershell
Set-ADAccountPassword -Identity $utilisateur -NewPassword (ConvertTo-SecureString $nouveauMdp -AsPlainText -Force) -Reset
```
Applique le nouveau mot de passe généré au compte sélectionné.

```powershell
$resultats | Export-Csv -Path "C:\reset-mdp-resultats.csv" -NoTypeInformation
```
Exporte la liste des comptes traités et de leurs nouveaux mots de passe dans un fichier CSV.

## GPO de mappage de lecteurs réseau

```powershell
New-GPO -Name "Mappage-Lecteurs-Departements" | New-GPLink -Target "OU=Entreprise,DC=cyberlab,DC=local"
```
Crée une nouvelle GPO et la lie à l'OU Entreprise, avant configuration des Préférences de stratégie de groupe pour le mappage de lecteurs réseau par département via la console de gestion des stratégies de groupe (gpmc.msc).
