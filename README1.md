# PowerShell & Active Directory - Atelier Pratique

## 📋 Description

Projet d'automatisation de la gestion Active Directory via PowerShell pour l'entreprise fictive **TechSecure** (200 employés).

**Problématique** : Les opérations AD manuelles via ADUC (Active Directory Users and Computers) sont chronophages et sources d'erreurs.

**Solution** : Automatiser la gestion AD avec PowerShell pour gagner en efficacité et fiabilité.

---

## 🎯 Objectifs pédagogiques

À l'issue de cet atelier, vous serez capable de :
- Installer et utiliser le module ActiveDirectory
- Gérer les utilisateurs, groupes et OU via PowerShell
- Importer des utilisateurs en masse depuis CSV
- Créer des scripts d'automatisation (onboarding/offboarding)
- Générer des rapports d'audit AD

---

## ⚙️ Prérequis

- Windows Server avec Active Directory configuré
- PowerShell 5.1 minimum
- Droits d'administration sur le domaine AD
- Connaissance de base de PowerShell

---

## 🚀 Partie 1 : Découverte du module ActiveDirectory

### Objectif
Installer et explorer le module PowerShell pour Active Directory.

### 1.1 - Vérification et installation du module
```powershell
# Vérifier si le module est déjà installé
Get-Module -ListAvailable -Name ActiveDirectory
```

**Explication** : Liste tous les modules disponibles portant le nom "ActiveDirectory". Si aucun résultat n'apparaît, le module n'est pas installé.

📸 **[Screenshot 01-verification-module.png]** - Résultat de la commande `Get-Module -ListAvailable -Name ActiveDirectory`
```powershell
# Installation si nécessaire (nécessite des droits admin)
Install-WindowsFeature RSAT-AD-PowerShell
# OU sur Windows 10/11 :
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```

**Explication** : Installe les outils RSAT (Remote Server Administration Tools) pour Active Directory, incluant le module PowerShell.

📸 **[Screenshot 02-installation-module.png]** - Processus d'installation du module

### 1.2 - Exploration du module
```powershell
# Importer le module
Import-Module ActiveDirectory

# Lister toutes les cmdlets du module
Get-Command -Module ActiveDirectory

# Compter le nombre de cmdlets
(Get-Command -Module ActiveDirectory).Count
```

**Explication** : 
- `Import-Module` charge le module en mémoire
- `Get-Command` liste toutes les commandes disponibles dans le module
- `.Count` affiche le nombre total de cmdlets

📸 **[Screenshot 03-liste-cmdlets.png]** - Liste complète des cmdlets ActiveDirectory avec le compte total
```powershell
# Filtrer les cmdlets Get-ADUser
Get-Command -Name Get-ADUser*
```

**Explication** : Utilise un wildcard (`*`) pour trouver toutes les cmdlets commençant par "Get-ADUser".

📸 **[Screenshot 04-get-aduser-cmdlets.png]** - Cmdlets liées à Get-ADUser
```powershell
# Afficher l'aide complète de Get-ADUser
Get-Help Get-ADUser -Full
# Pour des exemples pratiques :
Get-Help Get-ADUser -Examples
```

**Explication** : 
- `-Full` affiche la documentation complète (syntaxe, paramètres, exemples)
- `-Examples` montre uniquement les exemples d'utilisation

📸 **[Screenshot 05-help-get-aduser.png]** - Aide complète de Get-ADUser

### 1.3 - Connexion et informations sur le domaine
```powershell
# Récupérer les informations du domaine
Get-ADDomain
```

**Explication** : Affiche les propriétés du domaine Active Directory (nom, niveau fonctionnel, contrôleurs de domaine, etc.).

📸 **[Screenshot 06-get-addomain.png]** - Informations détaillées du domaine
```powershell
# Afficher uniquement des propriétés spécifiques
Get-ADDomain | Select-Object DNSRoot, DomainMode, PDCEmulator

# Lister les contrôleurs de domaine
Get-ADDomainController -Filter *
```

**Explication** :
- `Select-Object` filtre l'affichage pour ne garder que les propriétés spécifiées
- `Get-ADDomainController` liste tous les DC du domaine

📸 **[Screenshot 07-domain-controllers.png]** - Liste des contrôleurs de domaine

### 1.4 - Récupération d'informations utilisateur
```powershell
# Récupérer les infos de votre compte (remplacer par votre login)
Get-ADUser -Identity $env:USERNAME -Properties *
```

**Explication** :
- `-Identity` spécifie le nom du compte
- `$env:USERNAME` récupère automatiquement votre nom d'utilisateur Windows
- `-Properties *` charge TOUTES les propriétés (par défaut, seules quelques propriétés de base sont chargées)

📸 **[Screenshot 08-user-properties-all.png]** - Toutes les propriétés d'un utilisateur
```powershell
# Afficher uniquement nom, email et titre
Get-ADUser -Identity $env:USERNAME -Properties EmailAddress, Title | 
    Select-Object Name, EmailAddress, Title
```

**Explication** : 
- On ne charge que les propriétés nécessaires (`EmailAddress`, `Title`)
- `Select-Object` filtre l'affichage final

📸 **[Screenshot 09-user-specific-properties.png]** - Propriétés sélectionnées d'un utilisateur

---

## 👥 Partie 2 : Gestion des utilisateurs

### Objectif
Maîtriser les opérations CRUD (Create, Read, Update, Delete) sur les utilisateurs AD.

### 2.1 - Création d'utilisateurs
```powershell
# Créer un mot de passe sécurisé
$Password = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

# Créer l'utilisateur Alice Martin
New-ADUser -Name "Alice Martin" `
    -GivenName "Alice" `
    -Surname "Martin" `
    -SamAccountName "amartin" `
    -UserPrincipalName "amartin@techsecure.fr" `
    -EmailAddress "alice.martin@techsecure.fr" `
    -Title "Développeuse" `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $true `
    -Path "OU=Users,DC=techsecure,DC=fr"
```

**Explication détaillée** :
- `ConvertTo-SecureString` convertit le mot de passe en format sécurisé (obligatoire pour AD)
- `-Name` : nom complet de l'utilisateur
- `-GivenName` / `-Surname` : prénom et nom séparés
- `-SamAccountName` : login (format legacy Windows)
- `-UserPrincipalName` : login au format email (UPN)
- `-AccountPassword` : mot de passe initial
- `-Enabled $true` : active le compte immédiatement
- `-ChangePasswordAtLogon` : force le changement de mot de passe à la première connexion
- `-Path` : chemin DN (Distinguished Name) de l'OU de destination

📸 **[Screenshot 10-create-user-amartin.png]** - Création réussie d'Alice Martin
```powershell
# Créer Bob Dubois
New-ADUser -Name "Bob Dubois" `
    -GivenName "Bob" `
    -Surname "Dubois" `
    -SamAccountName "bdubois" `
    -UserPrincipalName "bdubois@techsecure.fr" `
    -EmailAddress "bob.dubois@techsecure.fr" `
    -Title "Administrateur Système" `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $true `
    -Path "OU=Users,DC=techsecure,DC=fr"

# Créer Claire Bernard
New-ADUser -Name "Claire Bernard" `
    -GivenName "Claire" `
    -Surname "Bernard" `
    -SamAccountName "cbernard" `
    -UserPrincipalName "cbernard@techsecure.fr" `
    -EmailAddress "claire.bernard@techsecure.fr" `
    -Title "Chef de Projet" `
    -AccountPassword $Password `
    -Enabled $true `
    -ChangePasswordAtLogon $true `
    -Path "OU=Users,DC=techsecure,DC=fr"
```

📸 **[Screenshot 11-create-multiple-users.png]** - Création des 3 utilisateurs avec confirmation

### 2.2 - Recherche d'utilisateurs
```powershell
# Lister tous les utilisateurs du domaine
Get-ADUser -Filter *
```

**Explication** : Le filtre `*` sélectionne TOUS les objets. Attention en production avec des milliers d'utilisateurs !

📸 **[Screenshot 12-list-all-users.png]** - Liste de tous les utilisateurs
```powershell
# Trouver un utilisateur spécifique par login
Get-ADUser -Identity "amartin"
# OU avec un filtre :
Get-ADUser -Filter "SamAccountName -eq 'amartin'"
```

**Explication** : 
- `-Identity` recherche par identifiant exact
- `-Filter` permet des requêtes plus complexes (opérateurs : `-eq`, `-like`, `-gt`, etc.)

📸 **[Screenshot 13-find-user-amartin.png]** - Résultat de la recherche d'amartin
```powershell
# Trouver tous les utilisateurs dont le nom commence par "B"
Get-ADUser -Filter "Surname -like 'B*'"
```

**Explication** : L'opérateur `-like` permet l'utilisation de wildcards (`*` = n'importe quelle chaîne de caractères).

📸 **[Screenshot 14-filter-surname-b.png]** - Utilisateurs avec nom commençant par B
```powershell
# Trouver les utilisateurs avec "Administrateur" dans le titre
Get-ADUser -Filter "Title -like '*Administrateur*'" -Properties Title | 
    Select-Object Name, Title
```

**Explication** : `-Properties Title` est nécessaire car `Title` n'est pas chargé par défaut.

📸 **[Screenshot 15-filter-title-admin.png]** - Utilisateurs avec titre contenant "Administrateur"
```powershell
# Compter le nombre total d'utilisateurs
(Get-ADUser -Filter *).Count
```

**Explication** : `.Count` est une propriété de tableau qui retourne le nombre d'éléments.

📸 **[Screenshot 16-count-users.png]** - Nombre total d'utilisateurs dans le domaine

### 2.3 - Modification d'utilisateurs
```powershell
# Modifier le numéro de téléphone d'amartin
Set-ADUser -Identity "amartin" -OfficePhone "01 23 45 67 89"

# Ajouter une description
Set-ADUser -Identity "amartin" -Description "Membre de l'équipe développement"

# Changer le titre
Set-ADUser -Identity "amartin" -Title "Développeuse Senior"
```

**Explication** : `Set-ADUser` modifie les propriétés d'un utilisateur existant. Seules les propriétés spécifiées sont modifiées.

📸 **[Screenshot 17-modify-user-amartin.png]** - Commandes de modification
```powershell
# Vérifier les modifications
Get-ADUser -Identity "amartin" -Properties OfficePhone, Description, Title | 
    Select-Object Name, OfficePhone, Description, Title
```

📸 **[Screenshot 18-verify-modifications.png]** - Vérification des propriétés modifiées

### 2.4 - Désactivation et suppression
```powershell
# Désactiver le compte bdubois
Disable-ADAccount -Identity "bdubois"

# Vérifier qu'il est désactivé
Get-ADUser -Identity "bdubois" | Select-Object Name, Enabled
```

**Explication** : 
- `Disable-ADAccount` désactive le compte (l'utilisateur ne peut plus se connecter)
- La propriété `Enabled` indique l'état du compte (`True` = actif, `False` = désactivé)

📸 **[Screenshot 19-disable-account.png]** - Désactivation de bdubois et vérification
```powershell
# Supprimer le compte cbernard (AVEC confirmation)
Remove-ADUser -Identity "cbernard" -Confirm:$true
```

**Explication** : 
- `Remove-ADUser` supprime définitivement l'utilisateur de l'AD
- `-Confirm:$true` demande une confirmation avant suppression (sécurité)
- **ATTENTION** : Cette action est IRRÉVERSIBLE

📸 **[Screenshot 20-delete-user-confirm.png]** - Invite de confirmation avant suppression

📸 **[Screenshot 21-delete-user-success.png]** - Confirmation de suppression

---

## 👨‍👩‍👧‍👦 Partie 3 : Gestion des groupes

### Objectif
Créer des groupes de sécurité et gérer les appartenances.

### 3.1 - Création de groupes
```powershell
# Créer le groupe GRP_Developpeurs
New-ADGroup -Name "GRP_Developpeurs" `
    -GroupScope Global `
    -GroupCategory Security `
    -Description "Équipe de développement" `
    -Path "OU=Groups,DC=techsecure,DC=fr"
```

**Explication** :
- `-GroupScope Global` : groupe utilisable dans tout le domaine et la forêt
- `-GroupCategory Security` : groupe de sécurité (pour les permissions), vs Distribution (pour emails)
- Les autres scopes possibles : `DomainLocal`, `Universal`

📸 **[Screenshot 22-create-group-dev.png]** - Création du groupe GRP_Developpeurs
```powershell
# Créer les autres groupes
New-ADGroup -Name "GRP_Admins_Systeme" `
    -GroupScope Global `
    -GroupCategory Security `
    -Description "Administrateurs système" `
    -Path "OU=Groups,DC=techsecure,DC=fr"

New-ADGroup -Name "GRP_Chefs_Projet" `
    -GroupScope Global `
    -GroupCategory Security `
    -Description "Chefs de projet" `
    -Path "OU=Groups,DC=techsecure,DC=fr"

New-ADGroup -Name "GRP_IT" `
    -GroupScope Global `
    -GroupCategory Security `
    -Description "Ensemble du département IT" `
    -Path "OU=Groups,DC=techsecure,DC=fr"
```

📸 **[Screenshot 23-create-all-groups.png]** - Création des 4 groupes

### 3.2 - Ajout de membres
```powershell
# Ajouter amartin dans GRP_Developpeurs
Add-ADGroupMember -Identity "GRP_Developpeurs" -Members "amartin"

# Ajouter bdubois dans GRP_Admins_Systeme
Add-ADGroupMember -Identity "GRP_Admins_Systeme" -Members "bdubois"
```

**Explication** : 
- `-Identity` désigne le groupe cible
- `-Members` spécifie le(s) utilisateur(s) à ajouter (peut être une liste)

📸 **[Screenshot 24-add-members.png]** - Ajout de membres aux groupes
```powershell
# Créer 2 nouveaux utilisateurs pour l'équipe dev
$Password = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

New-ADUser -Name "Sophie Leroy" `
    -SamAccountName "sleroy" `
    -AccountPassword $Password `
    -Enabled $true

New-ADUser -Name "Thomas Blanc" `
    -SamAccountName "tblanc" `
    -AccountPassword $Password `
    -Enabled $true

# Les ajouter au groupe GRP_Developpeurs
Add-ADGroupMember -Identity "GRP_Developpeurs" -Members "sleroy","tblanc"
```

📸 **[Screenshot 25-add-new-dev-members.png]** - Ajout des nouveaux développeurs
```powershell
# Ajouter les groupes Développeurs, Admins et Chefs de Projet dans GRP_IT
Add-ADGroupMember -Identity "GRP_IT" -Members "GRP_Developpeurs","GRP_Admins_Systeme","GRP_Chefs_Projet"
```

**Explication** : On peut ajouter des GROUPES comme membres d'autres groupes (nesting/imbrication).

📸 **[Screenshot 26-nested-groups.png]** - Ajout de groupes dans GRP_IT

### 3.3 - Consultation des appartenances
```powershell
# Lister tous les membres du groupe GRP_IT
Get-ADGroupMember -Identity "GRP_IT"
```

**Explication** : Affiche les membres DIRECTS du groupe (ici : les 3 groupes imbriqués).

📸 **[Screenshot 27-list-grp-it-members.png]** - Membres directs de GRP_IT
```powershell
# Lister TOUS les membres (récursif = inclut les membres des sous-groupes)
Get-ADGroupMember -Identity "GRP_IT" -Recursive
```

**Explication** : `-Recursive` descend dans les groupes imbriqués pour lister tous les utilisateurs finaux.

📸 **[Screenshot 28-list-grp-it-recursive.png]** - Membres récursifs de GRP_IT
```powershell
# Lister tous les groupes dont amartin est membre
Get-ADPrincipalGroupMembership -Identity "amartin"
```

**Explication** : Affiche tous les groupes auxquels appartient un utilisateur.

📸 **[Screenshot 29-user-group-membership.png]** - Groupes d'amartin
```powershell
# Compter les membres de chaque groupe
Get-ADGroup -Filter * | ForEach-Object {
    $count = (Get-ADGroupMember -Identity $_.Name).Count
    [PSCustomObject]@{
        GroupName = $_.Name
        MemberCount = $count
    }
} | Sort-Object MemberCount -Descending
```

**Explication** :
- `ForEach-Object` itère sur chaque groupe
- `[PSCustomObject]` crée un objet personnalisé pour affichage structuré
- `Sort-Object -Descending` trie par nombre décroissant

📸 **[Screenshot 30-group-member-count.png]** - Tableau des groupes avec leur nombre de membres

### 3.4 - Retrait de membres
```powershell
# Retirer amartin du groupe GRP_IT
Remove-ADGroupMember -Identity "GRP_IT" -Members "amartin" -Confirm:$false

# Vérifier qu'elle n'en est plus membre
Get-ADGroupMember -Identity "GRP_IT"
```

**Explication** : 
- `Remove-ADGroupMember` retire un membre d'un groupe
- `-Confirm:$false` désactive la demande de confirmation

📸 **[Screenshot 31-remove-member.png]** - Retrait d'amartin et vérification

### 3.5 - Groupes imbriqués avancés
```powershell
# Créer un groupe parent
New-ADGroup -Name "GRP_Tous_Utilisateurs" `
    -GroupScope Global `
    -GroupCategory Security `
    -Description "Tous les utilisateurs de l'entreprise"

# Ajouter GRP_IT comme membre (pas les utilisateurs individuels)
Add-ADGroupMember -Identity "GRP_Tous_Utilisateurs" -Members "GRP_IT"

# Lister les membres directs (seulement GRP_IT)
Get-ADGroupMember -Identity "GRP_Tous_Utilisateurs"

# Lister tous les membres récursifs
Get-ADGroupMember -Identity "GRP_Tous_Utilisateurs" -Recursive
```

**Explication** : Illustration de l'imbrication multi-niveaux : `GRP_Tous_Utilisateurs` > `GRP_IT` > `GRP_Developpeurs` > utilisateurs.

📸 **[Screenshot 32-nested-groups-hierarchy.png]** - Hiérarchie de groupes imbriqués

📸 **[Screenshot 33-recursive-members-all.png]** - Membres récursifs du groupe parent

---

## 🗂️ Partie 4 : Unités Organisationnelles (OU)

### Objectif
Structurer l'annuaire Active Directory avec des OU pour une meilleure organisation.

### 4.1 - Création de la structure d'OU
```powershell
# Créer l'OU racine TechSecure
New-ADOrganizationalUnit -Name "TechSecure" -Path "DC=techsecure,DC=fr"

# Créer les OU de premier niveau
New-ADOrganizationalUnit -Name "Utilisateurs" -Path "OU=TechSecure,DC=techsecure,DC=fr"
New-ADOrganizationalUnit -Name "Groupes" -Path "OU=TechSecure,DC=techsecure,DC=fr"
New-ADOrganizationalUnit -Name "Ordinateurs" -Path "OU=TechSecure,DC=techsecure,DC=fr"
```

**Explication** :
- `-Name` : nom de l'OU
- `-Path` : emplacement parent dans l'arborescence AD (notation DN)
- Les OU sont créées de haut en bas (parent avant enfants)

📸 **[Screenshot 34-create-ou-root.png]** - Création de la structure racine
```powershell
# Créer les sous-OU dans Utilisateurs
New-ADOrganizationalUnit -Name "Informatique" -Path "OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr"
New-ADOrganizationalUnit -Name "RH" -Path "OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr"
New-ADOrganizationalUnit -Name "Commercial" -Path "OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr"

# Créer les sous-OU dans Informatique
New-ADOrganizationalUnit -Name "Developpement" -Path "OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr"
New-ADOrganizationalUnit -Name "Infrastructure" -Path "OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr"
```

📸 **[Screenshot 35-create-ou-structure.png]** - Arborescence complète des OU

📸 **[Screenshot 36-ou-structure-aduc.png]** - Vue de la structure dans l'interface graphique ADUC

### 4.2 - Déplacement d'objets
```powershell
# Déplacer amartin dans l'OU Developpement
Get-ADUser -Identity "amartin" | Move-ADObject -TargetPath "OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr"
```

**Explication** :
- `Get-ADUser` récupère l'objet utilisateur
- Le pipe (`|`) passe l'objet à la commande suivante
- `Move-ADObject` déplace l'objet vers le chemin spécifié
- `-TargetPath` indique la destination complète

📸 **[Screenshot 37-move-user.png]** - Déplacement d'amartin
```powershell
# Déplacer tous les groupes créés dans l'OU Groupes
$groups = @("GRP_Developpeurs", "GRP_Admins_Systeme", "GRP_Chefs_Projet", "GRP_IT", "GRP_Tous_Utilisateurs")

foreach ($group in $groups) {
    Get-ADGroup -Identity $group | Move-ADObject -TargetPath "OU=Groupes,OU=TechSecure,DC=techsecure,DC=fr"
}
```

**Explication** :
- `$groups` stocke un tableau de noms de groupes
- `foreach` itère sur chaque élément
- Chaque groupe est récupéré puis déplacé

📸 **[Screenshot 38-move-groups.png]** - Déplacement de tous les groupes

### 4.3 - Recherche dans les OU
```powershell
# Lister tous les utilisateurs dans l'OU Informatique ET ses sous-OU
Get-ADUser -Filter * -SearchBase "OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr" -SearchScope Subtree
```

**Explication** :
- `-SearchBase` définit le point de départ de la recherche
- `-SearchScope Subtree` inclut tous les niveaux en dessous (récursif)
- Autres valeurs possibles : `Base` (juste l'OU), `OneLevel` (un niveau en dessous)

📸 **[Screenshot 39-search-ou-subtree.png]** - Utilisateurs dans Informatique (récursif)
```powershell
# Compter uniquement dans l'OU Informatique (sans sous-OU)
(Get-ADUser -Filter * -SearchBase "OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr" -SearchScope OneLevel).Count
```

📸 **[Screenshot 40-count-ou-onelevel.png]** - Comptage sans sous-OU
```powershell
# Compter avec tous les sous-niveaux
(Get-ADUser -Filter * -SearchBase "OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr" -SearchScope Subtree).Count
```

📸 **[Screenshot 41-count-ou-subtree.png]** - Comptage avec sous-OU

---

## 📂 Partie 5 : Import en masse depuis CSV

### Objectif
Automatiser la création d'utilisateurs en important les données depuis un fichier CSV.

### 5.1 - Préparation du fichier CSV
```csv
Prenom,Nom,Login,Email,Titre,Departement,OU
David,Petit,dpetit,david.petit@techsecure.fr,Développeur,Informatique,OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Emma,Roux,eroux,emma.roux@techsecure.fr,Administratrice Réseau,Informatique,OU=Infrastructure,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
François,Moreau,fmoreau,francois.moreau@techsecure.fr,Recruteur,RH,OU=RH,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Julie,Girard,jgirard,julie.girard@techsecure.fr,Développeuse,Informatique,OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Marc,Simon,msimon,marc.simon@techsecure.fr,Commercial,Commercial,OU=Commercial,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Nathalie,Laurent,nlaurent,nathalie.laurent@techsecure.fr,Chef de Projet,Informatique,OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Olivier,Michel,omichel,olivier.michel@techsecure.fr,Administrateur Système,Informatique,OU=Infrastructure,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Patricia,Lefevre,plefevre,patricia.lefevre@techsecure.fr,Assistante RH,RH,OU=RH,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Quentin,Bonnet,qbonnet,quentin.bonnet@techsecure.fr,Développeur,Informatique,OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Rachel,Fournier,rfournier,rachel.fournier@techsecure.fr,Commerciale,Commercial,OU=Commercial,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Stéphane,Rousseau,srousseau,stephane.rousseau@techsecure.fr,Responsable IT,Informatique,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Thibault,Vincent,tvincent,thibault.vincent@techsecure.fr,Développeur,Informatique,OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
Valérie,Muller,vmuller,valerie.muller@techsecure.fr,DRH,RH,OU=RH,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr
```

**Enregistrez ce fichier sous** : `C:\Scripts\AD\nouveaux_employes.csv`

📸 **[Screenshot 42-csv-file-content.png]** - Contenu du fichier CSV dans un éditeur de texte

### 5.2 - Script d'import basique

Créez le fichier `Import-ADUsersFromCSV.ps1` :
```powershell
# Import-ADUsersFromCSV.ps1
# Script d'import basique d'utilisateurs depuis CSV

# Chemin du fichier CSV
$CSVPath = "C:\Scripts\AD\nouveaux_employes.csv"

# Mot de passe par défaut
$DefaultPassword = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

# Importer le CSV
$Users = Import-Csv -Path $CSVPath -Encoding UTF8

# Parcourir chaque ligne
foreach ($User in $Users) {
    # Créer l'utilisateur
    New-ADUser -Name "$($User.Prenom) $($User.Nom)" `
        -GivenName $User.Prenom `
        -Surname $User.Nom `
        -SamAccountName $User.Login `
        -UserPrincipalName "$($User.Login)@techsecure.fr" `
        -EmailAddress $User.Email `
        -Title $User.Titre `
        -Department $User.Departement `
        -AccountPassword $DefaultPassword `
        -Enabled $true `
        -ChangePasswordAtLogon $true `
        -Path $User.OU
    
    Write-Host "✅ Utilisateur créé : $($User.Prenom) $($User.Nom)" -ForegroundColor Green
}

Write-Host "`n✨ Import terminé !" -ForegroundColor Cyan
```

**Explication ligne par ligne** :
- `Import-Csv` lit le fichier et crée des objets PowerShell (1 objet = 1 ligne)
- `-Encoding UTF8` gère les accents français
- `foreach` itère sur chaque utilisateur
- `$($User.Prenom)` accède à la colonne "Prenom" de la ligne actuelle
- `Write-Host` affiche un message avec couleur (`-ForegroundColor`)

📸 **[Screenshot 43-import-script-basic.png]** - Contenu du script dans l'éditeur

**Exécution du script** :
```powershell
.\Import-ADUsersFromCSV.ps1
```

📸 **[Screenshot 44-import-execution-basic.png]** - Exécution du script avec messages de succès

📸 **[Screenshot 45-import-users-created.png]** - Vérification des utilisateurs créés dans AD

### 5.3 - Script amélioré avec gestion d'erreurs

Créez le fichier `Import-ADUsersFromCSV-Advanced.ps1` :
```powershell
# Import-ADUsersFromCSV-Advanced.ps1
# Script d'import avancé avec gestion d'erreurs et logging

param(
    [Parameter(Mandatory=$true)]
    [string]$CSVPath,
    
    [string]$LogPath = "C:\Scripts\AD\Logs\import.log"
)

# Fonction de logging
function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $LogMessage = "[$Timestamp] [$Level] $Message"
    Add-Content -Path $LogPath -Value $LogMessage
    
    switch ($Level) {
        "SUCCESS" { Write-Host $Message -ForegroundColor Green }
        "ERROR"   { Write-Host $Message -ForegroundColor Red }
        "WARNING" { Write-Host $Message -ForegroundColor Yellow }
        default   { Write-Host $Message }
    }
}

# Vérifier que le CSV existe
if (-not (Test-Path $CSVPath)) {
    Write-Log "❌ Fichier CSV introuvable : $CSVPath" "ERROR"
    exit 1
}

# Créer le répertoire de logs si nécessaire
$LogDir = Split-Path $LogPath -Parent
if (-not (Test-Path $LogDir)) {
    New-Item -ItemType Directory -Path $LogDir -Force | Out-Null
}

Write-Log "🚀 Début de l'import depuis $CSVPath"

# Mot de passe par défaut
$DefaultPassword = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

# Compteurs
$SuccessCount = 0
$ErrorCount = 0
$SkipCount = 0

# Importer le CSV
try {
    $Users = Import-Csv -Path $CSVPath -Encoding UTF8
    Write-Log "📄 $($Users.Count) utilisateurs trouvés dans le CSV"
} catch {
    Write-Log "❌ Erreur lors de la lecture du CSV : $($_.Exception.Message)" "ERROR"
    exit 1
}

# Parcourir chaque utilisateur
foreach ($User in $Users) {
    $UserFullName = "$($User.Prenom) $($User.Nom)"
    
    try {
        # Vérifier si l'utilisateur existe déjà
        $ExistingUser = Get-ADUser -Filter "SamAccountName -eq '$($User.Login)'" -ErrorAction SilentlyContinue
        
        if ($ExistingUser) {
            Write-Log "⚠️  Utilisateur déjà existant : $UserFullName ($($User.Login))" "WARNING"
            $SkipCount++
            continue
        }
        
        # Créer l'utilisateur
        New-ADUser -Name $UserFullName `
            -GivenName $User.Prenom `
            -Surname $User.Nom `
            -SamAccountName $User.Login `
            -UserPrincipalName "$($User.Login)@techsecure.fr" `
            -EmailAddress $User.Email `
            -Title $User.Titre `
            -Department $User.Departement `
            -AccountPassword $DefaultPassword `
            -Enabled $true `
            -ChangePasswordAtLogon $true `
            -Path $User.OU `
            -ErrorAction Stop
        
        Write-Log "✅ Utilisateur créé : $UserFullName ($($User.Login))" "SUCCESS"
        $SuccessCount++
        
    } catch {
        Write-Log "❌ Erreur lors de la création de $UserFullName : $($_.Exception.Message)" "ERROR"
        $ErrorCount++
    }
}

# Résumé
Write-Log "`n📊 RÉSUMÉ DE L'IMPORT"
Write-Log "✅ Créés avec succès : $SuccessCount" "SUCCESS"
Write-Log "⚠️  Ignorés (déjà existants) : $SkipCount" "WARNING"
Write-Log "❌ Erreurs : $ErrorCount" "ERROR"
Write-Log "📄 Total traités : $($Users.Count)"

if ($ErrorCount -eq 0) {
    Write-Log "✨ Import terminé sans erreur !" "SUCCESS"
} else {
    Write-Log "⚠️  Import terminé avec des erreurs. Consultez le log : $LogPath" "WARNING"
}
```

**Améliorations apportées** :
- **Paramètres** : Chemin du CSV et du log configurables
- **Fonction Write-Log** : Centralise le logging (fichier + console avec couleurs)
- **Vérifications** : Existence du CSV, utilisateurs déjà créés
- **Try-Catch** : Gestion des erreurs pour chaque utilisateur (un échec n'arrête pas tout)
- **Compteurs** : Statistiques de succès/erreurs/ignorés
- **Résumé final** : Vue d'ensemble de l'import

📸 **[Screenshot 46-import-script-advanced.png]** - Script avancé dans l'éditeur

**Exécution** :
```powershell
.\Import-ADUsersFromCSV-Advanced.ps1 -CSVPath "C:\Scripts\AD\nouveaux_employes.csv"
```

📸 **[Screenshot 47-import-execution-advanced.png]** - Exécution avec gestion d'erreurs et résumé

📸 **[Screenshot 48-import-log-file.png]** - Contenu du fichier de log généré

### 5.4 - Import avec ajout automatique aux groupes

Modifiez votre CSV pour ajouter une colonne `Groupes` :
```csv
Prenom,Nom,Login,Email,Titre,Departement,OU,Groupes
David,Petit,dpetit,david.petit@techsecure.fr,Développeur,Informatique,OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr,GRP_Developpeurs;GRP_IT
Emma,Roux,eroux,emma.roux@techsecure.fr,Administratrice Réseau,Informatique,OU=Infrastructure,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr,GRP_Admins_Systeme;GRP_IT
```

📸 **[Screenshot 49-csv-with-groups.png]** - CSV avec colonne Groupes

Créez `Import-ADUsersWithGroups.ps1` :
```powershell
# Import-ADUsersWithGroups.ps1
# Import avec ajout automatique aux groupes

# (Reprendre le code précédent et ajouter après la création d'utilisateur)

foreach ($User in $Users) {
    try {
        # ... (code de création d'utilisateur)
        
        # Ajouter aux groupes si spécifiés
        if ($User.Groupes) {
            $Groups = $User.Groupes -split ';'
            foreach ($Group in $Groups) {
                $Group = $Group.Trim()
                try {
                    Add-ADGroupMember -Identity $Group -Members $User.Login -ErrorAction Stop
                    Write-Log "  └─ Ajouté au groupe : $Group" "SUCCESS"
                } catch {
                    Write-Log "  └─ Erreur ajout au groupe $Group : $($_.Exception.Message)" "ERROR"
                }
            }
        }
        
    } catch {
        # Gestion d'erreur
    }
}
```

**Explication** :
- `-split ';'` sépare la chaîne en tableau (si plusieurs groupes)
- `.Trim()` supprime les espaces inutiles
- Chaque groupe est ajouté individuellement avec gestion d'erreur

📸 **[Screenshot 50-import-with-groups-execution.png]** - Exécution avec ajout aux groupes

📸 **[Screenshot 51-verify-group-membership.png]** - Vérification des appartenances créées

---

## 🤖 Partie 6 : Scripts d'automatisation

### Objectif
Créer des scripts réutilisables pour les tâches courantes d'onboarding et offboarding.

### 6.1 - Script d'onboarding complet

Créez `New-Employee.ps1` :
```powershell
# New-Employee.ps1
# Script d'onboarding automatisé d'un nouvel employé

[CmdletBinding()]
param(
    [Parameter(Mandatory=$true)]
    [string]$Prenom,
    
    [Parameter(Mandatory=$true)]
    [string]$Nom,
    
    [Parameter(Mandatory=$true)]
    [string]$Titre,
    
    [Parameter(Mandatory=$true)]
    [ValidateSet("Informatique","RH","Commercial")]
    [string]$Departement,
    
    [Parameter(Mandatory=$false)]
    [string]$Manager
)

# Configuration
$DomainName = "techsecure.fr"
$LogPath = "C:\Scripts\AD\Logs\onboarding.log"

# Fonction de logging
function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $LogMessage = "[$Timestamp] [$Level] $Message"
    Add-Content -Path $LogPath -Value $LogMessage
    Write-Host "[$Level] $Message"
}

Write-Log "🚀 DÉBUT DE L'ONBOARDING" "INFO"
Write-Log "Employé : $Prenom $Nom - $Titre ($Departement)"

# 1. Générer le login (première lettre prénom + nom en minuscules)
$Login = ($Prenom.Substring(0,1) + $Nom).ToLower() -replace '[éèêë]','e' -replace '[àâä]','a' -replace '[ôö]','o'
Write-Log "Login généré : $Login"

# 2. Générer l'email
$Email = "$Login@$DomainName"
Write-Log "Email généré : $Email"

# 3. Déterminer l'OU selon le département
$OUPath = switch ($Departement) {
    "Informatique" { "OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr" }
    "RH"          { "OU=RH,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr" }
    "Commercial"  { "OU=Commercial,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=fr" }
}
Write-Log "OU cible : $OUPath"

# 4. Générer un mot de passe aléatoire sécurisé
function New-RandomPassword {
    param([int]$Length = 12)
    $Chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()"
    $Password = -join ((1..$Length) | ForEach-Object { $Chars[(Get-Random -Maximum $Chars.Length)] })
    return $Password
}

$Password = New-RandomPassword
$SecurePassword = ConvertTo-SecureString $Password -AsPlainText -Force
Write-Log "Mot de passe généré (complexité : 12 caractères)"

# 5. Créer l'utilisateur
try {
    $UserParams = @{
        Name                  = "$Prenom $Nom"
        GivenName            = $Prenom
        Surname              = $Nom
        SamAccountName       = $Login
        UserPrincipalName    = "$Login@$DomainName"
        EmailAddress         = $Email
        Title                = $Titre
        Department           = $Departement
        AccountPassword      = $SecurePassword
        Enabled              = $true
        ChangePasswordAtLogon = $true
        Path                 = $OUPath
    }
    
    if ($Manager) {
        $UserParams['Manager'] = $Manager
    }
    
    New-ADUser @UserParams -ErrorAction Stop
    Write-Log "✅ Utilisateur créé avec succès" "SUCCESS"
    
} catch {
    Write-Log "❌ Erreur lors de la création : $($_.Exception.Message)" "ERROR"
    exit 1
}

# 6. Ajouter aux groupes selon le département
$Groups = switch ($Departement) {
    "Informatique" { @("GRP_IT", "GRP_Developpeurs") }
    "RH"          { @("GRP_RH") }
    "Commercial"  { @("GRP_Commercial") }
}

foreach ($Group in $Groups) {
    try {
        Add-ADGroupMember -Identity $Group -Members $Login -ErrorAction Stop
        Write-Log "✅ Ajouté au groupe : $Group" "SUCCESS"
    } catch {
        Write-Log "⚠️  Erreur ajout au groupe $Group : $($_.Exception.Message)" "WARNING"
    }
}

# 7. Simulation d'envoi d'email de bienvenue
Write-Log "📧 SIMULATION - Email de bienvenue"
Write-Host "`n" -NoNewline
Write-Host "═══════════════════════════════════════════════" -ForegroundColor Cyan
Write-Host "📧 EMAIL DE BIENVENUE" -ForegroundColor Cyan
Write-Host "═══════════════════════════════════════════════" -ForegroundColor Cyan
Write-Host "À : $Email"
Write-Host "Objet : Bienvenue chez TechSecure !"
Write-Host "`nBonjour $Prenom,"
Write-Host "`nBienvenue dans l'équipe TechSecure !"
Write-Host "`nVoici vos identifiants de connexion :"
Write-Host "  Login : $Login" -ForegroundColor Yellow
Write-Host "  Mot de passe temporaire : $Password" -ForegroundColor Yellow
Write-Host "  (À changer à la première connexion)"
Write-Host "`nVotre manager : $Manager"
Write-Host "Département : $Departement"
Write-Host "`nCordialement,"
Write-Host "L'équipe IT TechSecure"
Write-Host "═══════════════════════════════════════════════`n" -ForegroundColor Cyan

Write-Log "✨ ONBOARDING TERMINÉ AVEC SUCCÈS" "SUCCESS"
Write-Log "Login : $Login | Email : $Email | Mot de passe : $Password"
```

**Fonctionnalités clés** :
- **Génération automatique** du login et email
- **Mot de passe aléatoire sécurisé** (12 caractères, mixte)
- **Splatting** (`@UserParams`) pour lisibilité des paramètres
- **Mapping département → OU et groupes**
- **Simulation d'email** avec affichage formaté
- **Logging complet** de toutes les opérations

📸 **[Screenshot 52-new-employee-script.png]** - Script New-Employee.ps1 complet

**Exécution** :
```powershell
.\New-Employee.ps1 -Prenom "Laura" -Nom "Durand" -Titre "Développeuse Full Stack" -Departement "Informatique" -Manager "amartin"
```

📸 **[Screenshot 53-new-employee-execution.png]** - Exécution du script d'onboarding

📸 **[Screenshot 54-new-employee-email-simulation.png]** - Simulation d'email de bienvenue

📸 **[Screenshot 55-new-employee-verification.png]** - Vérification de l'utilisateur créé dans AD

### 6.2 - Script d'offboarding

Créez `Remove-Employee.ps1` :
```powershell
# Remove-Employee.ps1
# Script d'offboarding automatisé d'un employé sortant

[CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact='High')]
param(
    [Parameter(Mandatory=$true)]
    [string]$Login
)

$LogPath = "C:\Scripts\AD\Logs\offboarding.log"

function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    Add-Content -Path $LogPath -Value "[$Timestamp] [$Level] $Message"
    
    $Color = switch ($Level) {
        "SUCCESS" { "Green" }
        "ERROR"   { "Red" }
        "WARNING" { "Yellow" }
        default   { "White" }
    }
    Write-Host "[$Level] $Message" -ForegroundColor $Color
}

Write-Log "🔴 DÉBUT DE L'OFFBOARDING" "WARNING"
Write-Log "Utilisateur : $Login"

# 1. Vérifier que l'utilisateur existe
try {
    $User = Get-ADUser -Identity $Login -Properties MemberOf, Description -ErrorAction Stop
    Write-Log "✅ Utilisateur trouvé : $($User.Name)"
} catch {
    Write-Log "❌ Utilisateur introuvable : $Login" "ERROR"
    exit 1
}

# 2. Demander confirmation
if (-not $PSCmdlet.ShouldProcess($User.Name, "Désactiver et archiver le compte")) {
    Write-Log "❌ Opération annulée par l'utilisateur" "WARNING"
    exit 0
}

# 3. Désactiver le compte
try {
    Disable-ADAccount -Identity $Login -ErrorAction Stop
    Write-Log "✅ Compte désactivé" "SUCCESS"
} catch {
    Write-Log "❌ Erreur lors de la désactivation : $($_.Exception.Message)" "ERROR"
}

# 4. Retirer de tous les groupes (sauf Domain Users)
$Groups = $User.MemberOf | Where-Object { $_ -notlike "*Domain Users*" }
foreach ($GroupDN in $Groups) {
    try {
        $GroupName = (Get-ADGroup -Identity $GroupDN).Name
        Remove-ADGroupMember -Identity $GroupDN -Members $Login -Confirm:$false -ErrorAction Stop
        Write-Log "✅ Retiré du groupe : $GroupName" "SUCCESS"
    } catch {
        Write-Log "⚠️  Erreur retrait du groupe : $($_.Exception.Message)" "WARNING"
    }
}

# 5. Créer l'OU "Comptes Désactivés" si elle n'existe pas
$DisabledOU = "OU=Comptes Desactives,OU=TechSecure,DC=techsecure,DC=fr"
if (-not (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$DisabledOU'" -ErrorAction SilentlyContinue)) {
    try {
        New-ADOrganizationalUnit -Name "Comptes Desactives" -Path "OU=TechSecure,DC=techsecure,DC=fr"
        Write-Log "✅ OU 'Comptes Désactivés' créée" "SUCCESS"
    } catch {
        Write-Log "⚠️  Erreur création OU : $($_.Exception.Message)" "WARNING"
    }
}

# 6. Déplacer vers l'OU des comptes désactivés
try {
    Move-ADObject -Identity $User.DistinguishedName -TargetPath $DisabledOU -ErrorAction Stop
    Write-Log "✅ Déplacé vers 'Comptes Désactivés'" "SUCCESS"
} catch {
    Write-Log "⚠️  Erreur déplacement : $($_.Exception.Message)" "WARNING"
}

# 7. Réinitialiser le mot de passe
function New-RandomPassword {
    param([int]$Length = 16)
    $Chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()"
    -join ((1..$Length) | ForEach-Object { $Chars[(Get-Random -Maximum $Chars.Length)] })
}

$NewPassword = New-RandomPassword
$SecurePassword = ConvertTo-SecureString $NewPassword -AsPlainText -Force

try {
    Set-ADAccountPassword -Identity $Login -NewPassword $SecurePassword -Reset -ErrorAction Stop
    Write-Log "✅ Mot de passe réinitialisé" "SUCCESS"
} catch {
    Write-Log "⚠️  Erreur réinitialisation mot de passe : $($_.Exception.Message)" "WARNING"
}

# 8. Ajouter une note dans la description
$DateDesactivation = Get-Date -Format "yyyy-MM-dd HH:mm"
$NewDescription = if ($User.Description) {
    "$($User.Description) | Désactivé le $DateDesactivation"
} else {
    "Désactivé le $DateDesactivation"
}

try {
    Set-ADUser -Identity $Login -Description $NewDescription -ErrorAction Stop
    Write-Log "✅ Description mise à jour avec date de désactivation" "SUCCESS"
} catch {
    Write-Log "⚠️  Erreur mise à jour description : $($_.Exception.Message)" "WARNING"
}

Write-Log "✨ OFFBOARDING TERMINÉ" "SUCCESS"
Write-Log "Compte désactivé, archivé et sécurisé."
```

**Fonctionnalités clés** :
- **`[CmdletBinding(SupportsShouldProcess)]`** : Active `-WhatIf` et `-Confirm`
- **Confirmation obligatoire** avant toute action critique
- **Retrait de TOUS les groupes** (sauf Domain Users qui est non-supprimable)
- **Création dynamique** de l'OU d'archivage si nécessaire
- **Réinitialisation du mot de passe** pour sécurité
- **Timestamp dans la description** pour traçabilité

📸 **[Screenshot 56-remove-employee-script.png]** - Script Remove-Employee.ps1

**Exécution** :
```powershell
.\Remove-Employee.ps1 -Login "dpetit"
```

📸 **[Screenshot 57-remove-employee-confirmation.png]** - Demande de confirmation

📸 **[Screenshot 58-remove-employee-execution.png]** - Exécution complète de l'offboarding

📸 **[Screenshot 59-remove-employee-verification.png]** - Vérification du compte désactivé et déplacé

### 6.3 - Script de réinitialisation de mot de passe

Créez `Reset-EmployeePassword.ps1` :
```powershell
# Reset-EmployeePassword.ps1
# Réinitialisation sécurisée du mot de passe

[CmdletBinding()]
param(
    [Parameter(Mandatory=$true)]
    [string]$Login
)

$LogPath = "C:\Scripts\AD\Logs\password_reset.log"

function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    Add-Content -Path $LogPath -Value "[$Timestamp] [$Level] $Message"
    
    $Color = switch ($Level) {
        "SUCCESS" { "Green" }
        "ERROR"   { "Red" }
        "WARNING" { "Yellow" }
        default   { "White" }
    }
    Write-Host "[$Level] $Message" -ForegroundColor $Color
}

Write-Log "🔑 RÉINITIALISATION DE MOT DE PASSE" "INFO"

# 1. Vérifier que l'utilisateur existe
try {
    $User = Get-ADUser -Identity $Login -Properties LockedOut, PasswordExpired -ErrorAction Stop
    Write-Log "✅ Utilisateur trouvé : $($User.Name)"
} catch {
    Write-Log "❌ Utilisateur introuvable : $Login" "ERROR"
    exit 1
}

# 2. Générer un nouveau mot de passe sécurisé
function New-RandomPassword {
    param([int]$Length = 14)
    
    # Assurer la complexité : majuscule, minuscule, chiffre, symbole
    $Lowercase = "abcdefghijklmnopqrstuvwxyz"
    $Uppercase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    $Numbers = "0123456789"
    $Symbols = "!@#$%^&*"
    
    $Password = ""
    $Password += $Lowercase[(Get-Random -Maximum $Lowercase.Length)]
    $Password += $Uppercase[(Get-Random -Maximum $Uppercase.Length)]
    $Password += $Numbers[(Get-Random -Maximum $Numbers.Length)]
    $Password += $Symbols[(Get-Random -Maximum $Symbols.Length)]
    
    $AllChars = $Lowercase + $Uppercase + $Numbers + $Symbols
    for ($i = 0; $i -lt ($Length - 4); $i++) {
        $Password += $AllChars[(Get-Random -Maximum $AllChars.Length)]
    }
    
    # Mélanger les caractères
    $Password = -join ($Password.ToCharArray() | Get-Random -Count $Password.Length)
    
    return $Password
}

$NewPassword = New-RandomPassword
$SecurePassword = ConvertTo-SecureString $NewPassword -AsPlainText -Force
Write-Log "✅ Nouveau mot de passe généré (14 caractères, complexité élevée)"

# 3. Réinitialiser le mot de passe
try {
    Set-ADAccountPassword -Identity $Login -NewPassword $SecurePassword -Reset -ErrorAction Stop
    Write-Log "✅ Mot de passe réinitialisé avec succès" "SUCCESS"
} catch {
    Write-Log "❌ Erreur lors de la réinitialisation : $($_.Exception.Message)" "ERROR"
    exit 1
}

# 4. Forcer le changement à la prochaine connexion
try {
    Set-ADUser -Identity $Login -ChangePasswordAtLogon $true -ErrorAction Stop
    Write-Log "✅ Changement obligatoire à la prochaine connexion activé" "SUCCESS"
} catch {
    Write-Log "⚠️  Erreur configuration changement obligatoire : $($_.Exception.Message)" "WARNING"
}

# 5. Déverrouiller le compte si nécessaire
if ($User.LockedOut) {
    try {
        Unlock-ADAccount -Identity $Login -ErrorAction Stop
        Write-Log "✅ Compte déverrouillé" "SUCCESS"
    } catch {
        Write-Log "⚠️  Erreur déverrouillage : $($_.Exception.Message)" "WARNING"
    }
} else {
    Write-Log "ℹ️  Compte non verrouillé, aucune action nécessaire"
}

# 6. Afficher le nouveau mot de passe
Write-Host "`n" -NoNewline
Write-Host "═══════════════════════════════════════════════" -ForegroundColor Cyan
Write-Host "🔑 NOUVEAU MOT DE PASSE" -ForegroundColor Cyan
Write-Host "═══════════════════════════════════════════════" -ForegroundColor Cyan
Write-Host "Utilisateur : $($User.Name) ($Login)" -ForegroundColor Yellow
Write-Host "Mot de passe : $NewPassword" -ForegroundColor Green
Write-Host "`n⚠️  À communiquer de manière sécurisée à l'utilisateur" -ForegroundColor Yellow
Write-Host "⚠️  L'utilisateur devra le changer à sa prochaine connexion" -ForegroundColor Yellow
Write-Host "═══════════════════════════════════════════════`n" -ForegroundColor Cyan

Write-Log "✨ Opération terminée avec succès"
Write-Log "Mot de passe pour $Login : $NewPassword"
```

**Fonctionnalités clés** :
- **Génération de mot de passe complexe** : garantit au moins 1 minuscule, 1 majuscule, 1 chiffre, 1 symbole
- **Mélange aléatoire** des caractères pour imprévisibilité
- **Changement obligatoire** à la prochaine connexion
- **Déverrouillage automatique** si le compte était verrouillé
- **Affichage sécurisé** du mot de passe avec avertissements

📸 **[Screenshot 60-reset-password-script.png]** - Script Reset-EmployeePassword.ps1

**Exécution** :
```powershell
.\Reset-EmployeePassword.ps1 -Login "amartin"
```

📸 **[Screenshot 61-reset-password-execution.png]** - Exécution avec affichage du nouveau mot de passe

📸 **[Screenshot 62-reset-password-verification.png]** - Vérification des propriétés du compte

---

## 📊 Partie 7 : Rapports et audits

### Objectif
Créer des scripts de reporting pour auditer l'Active Directory et identifier les problèmes potentiels.

### 7.1 - Rapport des utilisateurs inactifs

Créez `Get-InactiveUsers.ps1` :
```powershell
# Get-InactiveUsers.ps1
# Rapport des utilisateurs n'ayant pas changé leur mot de passe depuis 90+ jours

param(
    [int]$InactiveDays = 90,
    [string]$ExportPath = "C:\Scripts\AD\Reports\InactiveUsers.csv"
)

Write-Host "🔍 Recherche des utilisateurs inactifs (>$InactiveDays jours)..." -ForegroundColor Cyan

# Calculer la date limite
$DateLimit = (Get-Date).AddDays(-$InactiveDays)

# Récupérer tous les utilisateurs avec la date de dernier changement de mot de passe
$InactiveUsers = Get-ADUser -Filter {Enabled -eq $true} -Properties PasswordLastSet, PasswordNeverExpires |
    Where-Object {
        $_.PasswordLastSet -and
        $_.PasswordLastSet -lt $DateLimit -and
        -not $_.PasswordNeverExpires
    } |
    Select-Object @{
        Name = 'Login'
        Expression = {$_.SamAccountName}
    },
    @{
        Name = 'Nom'
        Expression = {$_.Name}
    },
    @{
        Name = 'DernierChangementMDP'
        Expression = {$_.PasswordLastSet}
    },
    @{
        Name = 'JoursDepuisChangement'
        Expression = {[math]::Round((New-TimeSpan -Start $_.PasswordLastSet -End (Get-Date)).TotalDays)}
    } |
    Sort-Object JoursDepuisChangement -Descending

# Afficher les résultats
Write-Host "`n📊 RÉSULTATS" -ForegroundColor Yellow
Write-Host "═══════════════════════════════════════════════════════════════" -ForegroundColor Gray

if ($InactiveUsers.Count -eq 0) {
    Write-Host "✅ Aucun utilisateur inactif trouvé." -ForegroundColor Green
} else {
    Write-Host "⚠️  $($InactiveUsers.Count) utilisateur(s) inactif(s) détecté(s):`n" -ForegroundColor Yellow
    $InactiveUsers | Format-Table -AutoSize
    
    # Exporter en CSV
    $InactiveUsers | Export-Csv -Path $ExportPath -NoTypeInformation -Encoding UTF8
    Write-Host "`n💾 Rapport exporté : $ExportPath" -ForegroundColor Green
}
```

**Explication détaillée** :
- **`-Filter {Enabled -eq $true}`** : Ne sélectionne que les comptes actifs
- **`Where-Object`** : Filtre après récupération (pour logique complexe)
- **`Select-Object @{Name='...'; Expression={...}}`** : Crée des colonnes calculées personnalisées
- **`[math]::Round()`** : Arrondit le nombre de jours
- **`Format-Table -AutoSize`** : Affichage tabulaire adaptatif
- **`Export-Csv`** : Génère un fichier CSV exploitable dans Excel

📸 **[Screenshot 63-inactive-users-script.png]** - Script Get-InactiveUsers.ps1

**Exécution** :
```powershell
.\Get-InactiveUsers.ps1 -InactiveDays 90
```

📸 **[Screenshot 64-inactive-users-results.png]** - Résultats affichés dans la console

📸 **[Screenshot 65-inactive-users-csv.png]** - Fichier CSV généré ouvert dans Excel

### 7.2 - Rapport des comptes désactivés

Créez `Get-DisabledAccounts.ps1` :
```powershell
# Get-DisabledAccounts.ps1
# Liste tous les comptes utilisateurs désactivés

param(
    [string]$ExportPath = "C:\Scripts\AD\Reports\DisabledAccounts.csv"
)

Write-Host "🔍 Recherche des comptes désactivés..." -ForegroundColor Cyan

# Récupérer les comptes désactivés
$DisabledAccounts = Get-ADUser -Filter {Enabled -eq $false} -Properties Description, WhenChanged |
    Select-Object @{
        Name = 'Login'
        Expression = {$_.SamAccountName}
    },
    @{
        Name = 'Nom'
        Expression = {$_.Name}
    },
    @{
        Name = 'OU'
        Expression = {($_.DistinguishedName -split ',',2)[1]}
    },
    @{
        Name = 'Description'
        Expression = {$_.Description}
    },
    @{
        Name = 'DerniereModification'
        Expression = {$_.WhenChanged}
    }

# Afficher les résultats
Write-Host "`n📊 RÉSULTATS" -ForegroundColor Yellow
Write-Host "═══════════════════════════════════════════════════════════════" -ForegroundColor Gray
Write-Host "🔴 $($DisabledAccounts.Count) compte(s) désactivé(s):`n" -ForegroundColor Red

if ($DisabledAccounts.Count -gt 0) {
    $DisabledAccounts | Format-Table -AutoSize
    
    # Exporter
    $DisabledAccounts | Export-Csv -Path $ExportPath -NoTypeInformation -Encoding UTF8
    Write-Host "`n💾 Rapport exporté : $ExportPath" -ForegroundColor Green
}
```

**Explication** :
- **`($_.DistinguishedName -split ',',2)[1]`** : Extrait le chemin de l'OU (enlève le CN de l'utilisateur)
- **`WhenChanged`** : Propriété AD indiquant la dernière modification de l'objet

📸 **[Screenshot 66-disabled-accounts-execution.png]** - Exécution et résultats

📸 **[Screenshot 67-disabled-accounts-csv.png]** - Export CSV

### 7.3 - Rapport des groupes et membres

Créez `Get-GroupsReport.ps1` :
```powershell
# Get-GroupsReport.ps1
# Rapport détaillé des groupes avec leurs membres

param(
    [string]$ExportPath = "C:\Scripts\AD\Reports\GroupsReport.html"
)

Write-Host "🔍 Génération du rapport des groupes..." -ForegroundColor Cyan

# Récupérer tous les groupes de sécurité
$Groups = Get-ADGroup -Filter {GroupCategory -eq 'Security'} -Properties Description, Members

# Préparer les données
$GroupData = foreach ($Group in $Groups) {
    $Members = Get-ADGroupMember -Identity $Group.SamAccountName -ErrorAction SilentlyContinue
    
    [PSCustomObject]@{
        NomGroupe = $Group.Name
        Description = $Group.Description
        NombreMembres = $Members.Count
        Membres = ($Members | ForEach-Object { $_.Name }) -join ', '
        EstVide = if ($Members.Count -eq 0) { "Oui" } else { "Non" }
    }
}

# Identifier les groupes vides
$EmptyGroups = $GroupData | Where-Object { $_.EstVide -eq "Oui" }

# Génération du HTML
$HTML = @"
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rapport des Groupes AD - $(Get-Date -Format 'dd/MM/yyyy')</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }
        h1 {
            color: #2c3e50;
            border-bottom: 3px solid #3498db;
            padding-bottom: 10px;
        }
        .summary {
            background-color: #ecf0f1;
            padding: 15px;
            border-radius: 5px;
            margin-bottom: 20px;
        }
        .summary-item {
            display: inline-block;
            margin-right: 30px;
            font-size: 1.1em;
        }
        .summary-number {
            font-weight: bold;
            color: #3498db;
            font-size: 1.5em;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            background-color: white;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        th {
            background-color: #3498db;
            color: white;
            padding: 12px;
            text-align: left;
        }
        td {
            padding: 10px;
            border-bottom: 1px solid #ddd;
        }
        tr:hover {
            background-color: #f5f5f5;
        }
        .empty {
            background-color: #ffe6e6;
        }
        .warning {
            color: #e74c3c;
            font-weight: bold;
        }
        .footer {
            margin-top: 30px;
            text-align: center;
            color: #7f8c8d;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <h1>📊 Rapport des Groupes Active Directory</h1>
    <p><strong>Date de génération :</strong> $(Get-Date -Format 'dd/MM/yyyy HH:mm:ss')</p>
    
    <div class="summary">
        <div class="summary-item">
            <div>Total de groupes</div>
            <div class="summary-number">$($GroupData.Count)</div>
        </div>
        <div class="summary-item">
            <div>Groupes vides</div>
            <div class="summary-number warning">$($EmptyGroups.Count)</div>
        </div>
    </div>
    
    <h2>📋 Liste des groupes</h2>
    <table>
        <thead>
            <tr>
                <th>Nom du groupe</th>
                <th>Description</th>
                <th>Nombre de membres</th>
                <th>Membres</th>
            </tr>
        </thead>
        <tbody>
"@

# Ajouter les lignes du tableau
foreach ($Group in ($GroupData | Sort-Object NombreMembres -Descending)) {
    $RowClass = if ($Group.EstVide -eq "Oui") { " class='empty'" } else { "" }
    $MembersDisplay = if ($Group.Membres) { $Group.Membres } else { "<span class='warning'>Groupe vide</span>" }
    
    $HTML += @"
            <tr$RowClass>
                <td><strong>$($Group.NomGroupe)</strong></td>
                <td>$($Group.Description)</td>
                <td style="text-align: center;">$($Group.NombreMembres)</td>
                <td>$MembersDisplay</td>
            </tr>
"@
}

$HTML += @"
        </tbody>
    </table>
    
    <div class="footer">
        <p>Généré automatiquement par Get-GroupsReport.ps1</p>
    </div>
</body>
</html>
"@

# Sauvegarder le fichier HTML
$HTML | Out-File -FilePath $ExportPath -Encoding UTF8

Write-Host "`n✅ Rapport généré avec succès !" -ForegroundColor Green
Write-Host "📄 Fichier : $ExportPath" -ForegroundColor Cyan
Write-Host "`n📊 Statistiques :" -ForegroundColor Yellow
Write-Host "  • Total de groupes : $($GroupData.Count)"
Write-Host "  • Groupes vides : $($EmptyGroups.Count)" -ForegroundColor $(if ($EmptyGroups.Count -gt 0) {"Red"} else {"Green"})

# Ouvrir le rapport dans le navigateur
Start-Process $ExportPath
```

**Explication** :
- **Here-String (`@"..."@`)** : Permet de créer des chaînes multi-lignes (idéal pour HTML)
- **CSS inline** : Styles intégrés pour rapport autonome
- **Mise en évidence conditionnelle** : Groupes vides colorés en rouge
- **`Start-Process`** : Ouvre automatiquement le HTML dans le navigateur par défaut

📸 **[Screenshot 68-groups-report-script.png]** - Script de génération du rapport HTML

📸 **[Screenshot 69-groups-report-execution.png]** - Exécution du script

📸 **[Screenshot 70-groups-report-html.png]** - Rapport HTML généré dans le navigateur

### 7.4 - Rapport d'audit complet

Créez `Get-ADHealthReport.ps1` - **Script final complet disponible dans le dépôt**

Ce script génère un rapport HTML exhaustif incluant :
- Statistiques générales (utilisateurs actifs/désactivés, groupes, OU)
- Top 10 des groupes par nombre de membres
- Utilisateurs avec mots de passe non-expirants
- Utilisateurs avec mots de passe expirés
- Répartition par département
- Design moderne avec gradient CSS et cartes interactives

📸 **[Screenshot 71-health-report-execution.png]** - Exécution du script d'audit

📸 **[Screenshot 72-health-report-html-full.png]** - Vue complète du rapport HTML

📸 **[Screenshot 73-health-report-stats-cards.png]** - Section des statistiques en cartes

📸 **[Screenshot 74-health-report-alerts.png]** - Section des alertes de sécurité

📸 **[Screenshot 75-health-report-top-groups.png]** - Top 10 des groupes

---

## 🎯 Mémo des commandes essentielles

### Utilisateurs

| Commande | Description | Exemple |
|----------|-------------|---------|
| `Get-ADUser` | Récupérer un/des utilisateurs | `Get-ADUser -Identity "amartin"` |
| `New-ADUser` | Créer un utilisateur | `New-ADUser -Name "John Doe" -SamAccountName "jdoe"` |
| `Set-ADUser` | Modifier un utilisateur | `Set-ADUser -Identity "jdoe" -Title "Manager"` |
| `Remove-ADUser` | Supprimer un utilisateur | `Remove-ADUser -Identity "jdoe" -Confirm:$false` |
| `Disable-ADAccount` | Désactiver un compte | `Disable-ADAccount -Identity "jdoe"` |
| `Enable-ADAccount` | Activer un compte | `Enable-ADAccount -Identity "jdoe"` |
| `Unlock-ADAccount` | Déverrouiller un compte | `Unlock-ADAccount -Identity "jdoe"` |
| `Set-ADAccountPassword` | Changer le mot de passe | `Set-ADAccountPassword -Identity "jdoe" -NewPassword $pwd -Reset` |

### Groupes

| Commande | Description | Exemple |
|----------|-------------|---------|
| `Get-ADGroup` | Récupérer un/des groupes | `Get-ADGroup -Filter "Name -like 'GRP_*'"` |
| `New-ADGroup` | Créer un groupe | `New-ADGroup -Name "GRP_Test" -GroupScope Global` |
| `Remove-ADGroup` | Supprimer un groupe | `Remove-ADGroup -Identity "GRP_Test"` |
| `Add-ADGroupMember` | Ajouter un membre | `Add-ADGroupMember -Identity "GRP_IT" -Members "jdoe"` |
| `Remove-ADGroupMember` | Retirer un membre | `Remove-ADGroupMember -Identity "GRP_IT" -Members "jdoe"` |
| `Get-ADGroupMember` | Lister les membres | `Get-ADGroupMember -Identity "GRP_IT"` |
| `Get-ADPrincipalGroupMembership` | Groupes d'un utilisateur | `Get-ADPrincipalGroupMembership -Identity "jdoe"` |

### Unités Organisationnelles

| Commande | Description | Exemple |
|----------|-------------|---------|
| `Get-ADOrganizationalUnit` | Récupérer des OU | `Get-ADOrganizationalUnit -Filter *` |
| `New-ADOrganizationalUnit` | Créer une OU | `New-ADOrganizationalUnit -Name "IT" -Path "DC=domain,DC=com"` |
| `Remove-ADOrganizationalUnit` | Supprimer une OU | `Remove-ADOrganizationalUnit -Identity "OU=IT,DC=domain,DC=com"` |
| `Move-ADObject` | Déplacer un objet | `Move-ADObject -Identity "CN=User..." -TargetPath "OU=..."` |

### Domaine et recherche

| Commande | Description | Exemple |
|----------|-------------|---------|
| `Get-ADDomain` | Infos du domaine | `Get-ADDomain` |
| `Get-ADForest` | Infos de la forêt | `Get-ADForest` |
| `Get-ADDomainController` | Lister les DC | `Get-ADDomainController -Filter *` |
| `-Filter` | Recherche côté serveur | `-Filter "Name -like 'A*'"` |
| `-SearchBase` | Définir l'OU de recherche | `-SearchBase "OU=Users,DC=domain,DC=com"` |
| `-SearchScope` | Profondeur de recherche | `-SearchScope Subtree` (Base/OneLevel/Subtree) |

---

## 📚 Ce que j'ai appris

### Compétences techniques acquises

✅ **Gestion des utilisateurs AD via PowerShell**
- Création, modification, suppression et désactivation de comptes
- Gestion des propriétés (email, titre, département, manager)
- Manipulation des mots de passe de manière sécurisée
- Déverrouillage de comptes et réinitialisation de mots de passe

✅ **Gestion des groupes et appartenances**
- Création de groupes de sécurité (Global, DomainLocal, Universal)
- Ajout et retrait de membres
- Gestion de groupes imbriqués (nesting)
- Audit des appartenances utilisateurs et groupes

✅ **Organisation de l'annuaire avec les OU**
- Création d'une structure hiérarchique d'OU
- Déplacement d'objets entre OU
- Recherche ciblée par OU avec `-SearchBase` et `-SearchScope`

✅ **Import en masse et automatisation**
- Lecture et traitement de fichiers CSV
- Gestion d'erreurs robuste avec try-catch
- Logging des opérations pour traçabilité
- Scripts paramétrés et réutilisables

✅ **Génération de rapports**
- Export de données en CSV
- Création de rapports HTML professionnels
- Visualisation de données avec tableaux et graphiques
- Identification d'anomalies de sécurité

### Bonnes pratiques apprises

🔐 **Sécurité**
- Ne jamais stocker de mots de passe en clair
- Utiliser `ConvertTo-SecureString` pour les mots de passe
- Forcer le changement de mot de passe à la première connexion
- Générer des mots de passe complexes aléatoires

🛡️ **Fiabilité**
- Toujours utiliser `try-catch` pour gérer les erreurs
- Vérifier l'existence des objets avant modification
- Demander confirmation (`-Confirm`) pour les actions destructives
- Tester avec `-WhatIf` avant exécution réelle

📝 **Maintenabilité**
- Commenter le code de manière claire
- Utiliser des noms de variables explicites
- Structurer les scripts avec des fonctions
- Logger toutes les opérations importantes

⚡ **Performance**
- Utiliser `-Filter` plutôt que `Where-Object` (filtrage côté serveur)
- Limiter les propriétés chargées avec `-Properties`
- Éviter les boucles inutiles sur de gros volumes

---

## 🔧 Améliorations possibles

Pour une utilisation en production, les scripts pourraient être enrichis avec :

1. **Intégration avec un système de ticketing** (ServiceNow, Jira) pour automatiser les demandes
2. **Notifications par email** réelles (via `Send-MailMessage`) pour les onboarding/offboarding
3. **Synchronisation avec Azure AD** pour environnements hybrides
4. **Gestion des certificats** et authentification MFA
5. **Tableau de bord Power BI** alimenté par les exports CSV
6. **Intégration CI/CD** pour déploiement automatisé des scripts
7. **Tests unitaires** avec Pester pour validation du code
8. **Backup automatique** avant toute modification critique
9. **Gestion multi-domaines** et multi-forêts
10. **API REST** pour exposer les fonctionnalités aux applications tierces

---

## 📂 Structure des fichiers du projet
```
C:\Scripts\AD\
├── Import-ADUsersFromCSV.ps1
├── Import-ADUsersFromCSV-Advanced.ps1
├── Import-ADUsersWithGroups.ps1
├── New-Employee.ps1
├── Remove-Employee.ps1
├── Reset-EmployeePassword.ps1
├── Get-InactiveUsers.ps1
├── Get-DisabledAccounts.ps1
├── Get-GroupsReport.ps1
├── Get-ADHealthReport.ps1
├── nouveaux_employes.csv
├── Logs\
│   ├── import.log
│   ├── onboarding.log
│   ├── offboarding.log
│   └── password_reset.log
└── Reports\
    ├── InactiveUsers.csv
    ├── DisabledAccounts.csv
    ├── GroupsReport.html
    └── AD_Health_Report.html
```

---

## 🎓 Ressources complémentaires

- [Documentation officielle Microsoft ActiveDirectory](https://docs.microsoft.com/powershell/module/activedirectory/)
- [PowerShell Gallery - Module ActiveDirectory](https://www.powershellgallery.com/)
- [Active Directory Best Practices](https://docs.microsoft.com/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory)
- [PowerShell Style Guide](https://poshcode.gitbook.io/powershell-practice-and-style/)

---

## 📸 Organisation des Screenshots

Tous les screenshots doivent être placés dans le dossier `/docs/screenshots/` avec la nomenclature suivante :
```
/docs/screenshots/
├── 01-verification-module.png
├── 02-installation-module.png
├── 03-liste-cmdlets.png
├── ...
└── 75-health-report-top-groups.png
```

**Convention de nommage** : `[numéro séquentiel]-[description-courte].png`

---

**Auteur** : Votre Nom  
**Date** : Janvier 2026  
**Contexte** : Atelier PowerShell & Active Directory - TechSecure
