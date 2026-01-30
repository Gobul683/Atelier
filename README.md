# 📂 Atelier PowerShell & Active Directory : Gestion des Utilisateurs

Ce dépôt documente mon apprentissage de l'automatisation des tâches d'administration système avec PowerShell. Le projet simule la gestion de l'infrastructure d'une entreprise fictive, **TechSecure**.

**Objectif des parties 1 & 2** : Passer de l'interface graphique (ADUC) à la ligne de commande pour l'installation, la découverte, et la gestion du cycle de vie des utilisateurs (CRUD).

---

## 🛠️ Partie 1 : Découverte et Configuration

Avant de manipuler l'annuaire, nous avons mis en place l'environnement et exploré le module Active Directory.

### 1. Vérification des outils

Le module AD doit être chargé pour que PowerShell comprenne les commandes.
```powershell
# Vérifier si le module est disponible
Get-Module -ListAvailable ActiveDirectory

# Lister toutes les commandes disponibles (environ 140+)
Get-Command -Module ActiveDirectory
```

![Vérification du module ActiveDirectory](screenshots/01-module-verification.png)

### 2. Connexion et Audit

Vérification de la connexion au contrôleur de domaine et récupération des infos de l'utilisateur courant.
```powershell
# Informations sur le domaine (Niveau fonctionnel, Contrôleurs...)
Get-ADDomain

# Récupérer mes propres infos
# Note : Par défaut, AD n'affiche que 10 propriétés. "-Properties *" force l'affichage complet.
Get-ADUser -Identity $env:USERNAME -Properties *
```

![Informations du domaine](screenshots/02-domain-info.png)

![Propriétés utilisateur courant](screenshots/03-current-user-properties.png)

---

## 👤 Partie 2 : Gestion des Utilisateurs (CRUD)

Mise en pratique des opérations **Create, Read, Update, Delete**.

### 2.1 Création d'utilisateurs (Create)

La création nécessite la gestion sécurisée du mot de passe. L'Active Directory n'accepte pas les mots de passe en texte clair.
```powershell
# 1. Conversion du mot de passe en chaîne sécurisée (SecureString)
$pw = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

# 2. Création de l'utilisateur "Alice Martin"
# Important : Remplir GivenName et Surname dès la création pour faciliter les recherches futures.
New-ADUser -Name "Alice Martin" `
           -SamAccountName "amartin" `
           -GivenName "Alice" `
           -Surname "Martin" `
           -UserPrincipalName "amartin@techsecure.fr" `
           -EmailAddress "alice.martin@techsecure.fr" `
           -Title "Développeuse" `
           -AccountPassword $pw `
           -Enabled $true `
           -ChangePasswordAtLogon $true
```

![Création de l'utilisateur Alice Martin](screenshots/04-create-user.png)

### 2.2 Recherche et Filtrage (Read)

L'utilisation du paramètre `-Filter` permet de trier la base de données efficacement.
```powershell
# Rechercher par login exact
Get-ADUser -Filter "SamAccountName -eq 'amartin'"

# Rechercher par motif (Tous les noms commençant par 'M')
Get-ADUser -Filter "Surname -like 'M*'"

# Lister tous les utilisateurs (Nom et Login uniquement)
Get-ADUser -Filter * | Select-Object Name, SamAccountName
```

![Recherche d'utilisateurs avec filtres](screenshots/05-search-filter.png)

### 2.3 Modification (Update)

Mise à jour des attributs d'un compte existant.
```powershell
# Ajout du téléphone, modification du titre et description
Set-ADUser -Identity "amartin" `
           -OfficePhone "01 23 45 67 89" `
           -Title "Développeuse Senior" `
           -Description "Membre de l'équipe développement"
```

![Modification des attributs utilisateur](screenshots/06-update-user.png)

### 2.4 Désactivation et Suppression (Delete)

Gestion de la fin de vie d'un compte.
```powershell
# 1. Désactiver un compte (Offboarding)
Disable-ADAccount -Identity "bdubois"

# 2. Vérification technique (Le compte doit afficher Enabled = False)
Get-ADUser -Identity "bdubois" -Properties Enabled | Select-Object Name, Enabled

# 3. Suppression définitive (avec demande de confirmation)
Remove-ADUser -Identity "cbernard" -Confirm:$true
```

![Désactivation d'un compte](screenshots/07-disable-account.png)

![Suppression d'un utilisateur](screenshots/08-delete-user.png)

---

## 👥 Partie 3 : Gestion des Groupes de Sécurité

Dans un environnement professionnel, on ne gère pas les permissions utilisateur par utilisateur, mais par groupe.

### 3.1 Création des Groupes

Utilisation de `New-ADGroup` pour créer des conteneurs de sécurité.

- **Scope Global** : Le groupe est visible dans tout le domaine.
- **Category Security** : Sert à gérer les droits d'accès (fichiers, applis).
```powershell
# Création de plusieurs groupes d'un coup
New-ADGroup -Name "GRP_Developpeurs" -GroupScope Global -GroupCategory Security -Description "Équipe Dev"
New-ADGroup -Name "GRP_Admins_Systeme" -GroupScope Global -GroupCategory Security
New-ADGroup -Name "GRP_IT" -GroupScope Global -GroupCategory Security
```

![Création des groupes de sécurité](screenshots/09-create-groups.png)

### 3.2 Gestion des Membres et Pipelines

Pour ajouter des membres, on utilise `Add-ADGroupMember`. J'ai appris à gérer l'ajout de masse.

**Méthode 1 : Ajout direct**
```powershell
Add-ADGroupMember -Identity "GRP_Developpeurs" -Members "amartin", "bdubois"
```

**Méthode 2 : Ajout dynamique (Le défi du Pipeline)**

L'objectif était de mettre tous les Développeurs dans le groupe IT.

- **Problème rencontré** : Le pipe `|` simple échouait parfois à passer les objets.
- **Solution** : Utiliser les parenthèses pour forcer l'exécution de la recherche en premier.
```powershell
# On cherche d'abord les membres, PUIS on les ajoute au groupe cible
Add-ADGroupMember -Identity "GRP_IT" -Members (Get-ADGroupMember -Identity "GRP_Developpeurs")
```

![Ajout de membres aux groupes](screenshots/10-add-group-members.png)

### 3.3 Groupes Imbriqués (Nested Groups)

Création d'une hiérarchie : Le groupe "IT" est membre du groupe "Tous_Utilisateurs".
```powershell
# Ajout d'un groupe dans un autre
Add-ADGroupMember -Identity "GRP_Tous_Utilisateurs" -Members "GRP_IT"

# Vérification récursive (Indispensable pour voir les vrais utilisateurs au fond des groupes)
Get-ADGroupMember -Identity "GRP_Tous_Utilisateurs" -Recursive | Select-Object Name
```

![Groupes imbriqués et vérification récursive](screenshots/11-nested-groups.png)

---

## 📂 Partie 4 : Organisation (OUs) et Structure

Passage d'une liste plate à une structure hiérarchique organisée (Arbre LDAP).

### 4.1 Architecture et Chemins LDAP

Création de l'arborescence avec `New-ADOrganizationalUnit`.

- **Challenge technique** : Comprendre le "Distinguished Name" (DN).
- **Erreur corrigée** : Mon script utilisait `DC=fr` alors que mon domaine était `DC=local`. Correction via `(Get-ADDomain).DistinguishedName`.
```powershell
# Définition du chemin racine correct
$rootPath = "DC=techsecure,DC=local"

# Création de la structure
New-ADOrganizationalUnit -Name "TechSecure" -Path $rootPath
New-ADOrganizationalUnit -Name "Utilisateurs" -Path "OU=TechSecure,$rootPath"
```

![Création de la structure OU](screenshots/12-create-ou-structure.png)

### 4.2 Déménagement des Objets

Les utilisateurs créés par défaut se trouvent dans le conteneur "Users". Nous les avons déplacés vers leur nouvelle structure.
```powershell
# Définition de la destination (Variable pour éviter les erreurs de frappe)
$targetDev = "OU=Developpement,OU=Informatique,OU=Utilisateurs,OU=TechSecure,DC=techsecure,DC=local"

# Déplacement via Pipe
Get-ADUser -Identity "amartin" | Move-ADObject -TargetPath $targetDev
```

![Déplacement des utilisateurs vers les OUs](screenshots/13-move-users.png)

### 4.3 Audit et Statistiques (SearchScope)

Comment compter les utilisateurs dans une structure complexe ?

- **SearchScope 'OneLevel'** : Cherche uniquement dans le dossier courant (Résultat = 0 car les users sont dans les sous-dossiers).
- **SearchScope 'Subtree'** : Cherche dans le dossier ET les sous-dossiers (Résultat correct).
```powershell
# Compter tous les utilisateurs de l'IT (y compris Développement et Infra)
(Get-ADUser -Filter * -SearchBase $pathIT -SearchScope Subtree).Count
```

![Audit avec SearchScope](screenshots/14-searchscope-audit.png)

---

## 📚 Ressources

- [Documentation Microsoft : Module ActiveDirectory](https://docs.microsoft.com/powershell/module/activedirectory/)
- [Get-ADUser cmdlet reference](https://docs.microsoft.com/powershell/module/activedirectory/get-aduser)
- [New-ADGroup cmdlet reference](https://docs.microsoft.com/powershell/module/activedirectory/new-adgroup)
- [New-ADOrganizationalUnit cmdlet reference](https://docs.microsoft.com/powershell/module/activedirectory/new-adorganizationalunit)
