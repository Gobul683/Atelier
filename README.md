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

## 📚 Ressources

- [Documentation Microsoft : Module ActiveDirectory](https://docs.microsoft.com/powershell/module/activedirectory/)
- [Get-ADUser cmdlet reference](https://docs.microsoft.com/powershell/module/activedirectory/get-aduser)

## 🎯 Prochaines étapes

- Automatisation de la création en masse (import CSV)
- Gestion des groupes de sécurité
- Scripts de reporting et d'audit
