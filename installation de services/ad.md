Voici une version ultra-détaillée en Markdown, prête à être copiée/collée dans un fichier `.md` ou un wiki. Chaque section est structurée pour une mise en œuvre pas à pas, avec des commandes, des captures d’écran suggérées, et des astuces de dépannage.

```markdown
# Guide Complet : Installation et Configuration d’Active Directory sur Proxmox avec pfSense

---

## 📌 Table des Matières
1. [Présentation et Prérequis](#1-présentation-et-prérequis)
2. [Installation de la VM Windows Server sur Proxmox](#2-installation-de-la-vm-windows-server-sur-proxmox)
3. [Configuration d’Active Directory et DNS](#3-configuration-dactive-directory-et-dns)
4. [Intégration avec Proxmox/pfSense](#4-intégration-avec-proxmoxpfSense)
5. [Ajout de Postes au Domaine](#5-ajout-de-postes-au-domaine)
6. [Création et Application de Stratégies de Groupe (GPO)](#6-création-et-application-de-stratégies-de-groupe-gpo)
7. [Dépannage et Logs](#7-dépannage-et-logs)
8. [Bonnes Pratiques et Sécurité](#8-bonnes-pratiques-et-sécurité)
9. [Annexes : Commandes Utiles](#9-annexes-commandes-utiles)

---

## 1. Présentation et Prérequis

### 🔹 Fonctionnalités Clés
- **Authentification centralisée** : Gestion des utilisateurs et des postes depuis un seul point.
- **Stratégies de Groupe (GPO)** : Application de règles (ex : restrictions USB, mots de passe, logiciels).
- **DNS Interne** : Résolution des noms pour les machines du réseau local.

### 🔹 Prérequis Matériels et Logiciels
| Ressource          | Exigence Minimum       |
|--------------------|------------------------|
| RAM                | 4 Go                   |
| vCPU               | 2 cœurs                |
| Stockage           | 20 Go SSD              |
| OS                 | Windows Server 2022    |
| Licence            | Évaluation (180 jours) ou permanente |

### 🔹 Schéma Réseau Recommandé
```
[Postes Clients] ←→ [pfSense (DHCP/DNS)] ←→ [VM Active Directory (192.168.1.10)]
```

---

## 2. Installation de la VM Windows Server sur Proxmox

### 🔹 Étape 1 : Télécharger l’ISO
- Télécharger **Windows Server 2022** depuis le [Centre d’Évaluation Microsoft](https://www.microsoft.com/fr-fr/evalcenter/evaluate-windows-server-2022).
- Placer l’ISO dans le stockage local de Proxmox (ex : `/var/lib/vz/template/iso/`).

### 🔹 Étape 2 : Créer la VM
```bash
# Créer une VM avec 4 Go de RAM, 2 vCPU, et 20 Go de stockage
qm create 100 --name "Windows-AD" --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0

# Attacher un disque de 20 Go et monter l’ISO
qm set 100 --scsi0 local-lvm:20G
qm set 100 --ide2 local:iso/Windows_Server_2022.iso,media=cdrom

# Définir l’ordre de boot
qm set 100 --boot order=scsi0;ide2

# Démarrer la VM
qm start 100
```

### 🔹 Étape 3 : Installer Windows Server
1. **Lancer l’installation** :
   - Sélectionner la langue et le clavier.
   - Choisir **"Windows Server 2022 Standard (Desktop Experience)"** pour une interface graphique.
2. **Partitionnement** :
   - Laisser les options par défaut (le disque sera formaté en NTFS).
3. **Mot de passe Administrateur** :
   - Définir un mot de passe complexe (ex : `P@ssw0rd2026!`).

---

## 3. Configuration d’Active Directory et DNS

### 🔹 Étape 1 : Installer le Rôle AD
1. **Ouvrir PowerShell en tant qu’administrateur** :
   ```powershell
   Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
   ```
2. **Promouvoir le serveur en contrôleur de domaine** :
   ```powershell
   Install-ADDSForest -DomainName "mon-domaine.local" -SafeModeAdministratorPassword (ConvertTo-SecureString "VotreMotDePasse" -AsPlainText -Force)
   ```
   - **Options** :
     - `DomainName` : Nom de votre domaine (ex : `entreprise.local`).
     - `SafeModeAdministratorPassword` : Mot de passe pour le mode restauration.

3. **Redémarrer automatiquement** :
   ```powershell
   Restart-Computer
   ```

### 🔹 Étape 2 : Configurer le DNS
1. **Vérifier l’installation du rôle DNS** :
   - Ouvrir **Gestionnaire de serveur** > **Ajouter des rôles et fonctionnalités** > Sélectionner **Serveur DNS**.
2. **Créer une zone de recherche directe** :
   - Ouvrir **Gestionnaire DNS** (`dnsmgmt.msc`).
   - Clic droit sur **Zones de recherche directe** > **Nouvelle zone** :
     - Type : **Zone principale**.
     - Nom : `mon-domaine.local`.

---

## 4. Intégration avec Proxmox/pfSense

### 🔹 Étape 1 : Configurer les Règles pfSense
1. **Autoriser les ports nécessaires** :
   - **TCP 389** (LDAP), **TCP 445** (SMB), **UDP 53** (DNS).
   - Aller dans **Firewall > Rules** et ajouter une règle pour le LAN :
     ```
     Action: Pass
     Interface: LAN
     Protocol: TCP/UDP
     Source: Any
     Destination: 192.168.1.10 (IP de l’AD)
     Ports: 389, 445, 53
     ```

### 🔹 Étape 2 : Configurer le DHCP
1. **Modifier les options DHCP** :
   - Aller dans **Services > DHCP Server**.
   - Dans **Options DHCP**, ajouter :
     ```
     Option 006 (DNS Servers): 192.168.1.10
     ```

---

## 5. Ajout de Postes au Domaine

### 🔹 Étape 1 : Préparer le Poste Client
1. **Vérifier la connectivité** :
   ```powershell
   ping mon-domaine.local
   ```
   - Si échec : Vérifier le DNS (`ipconfig /all`).

2. **Rejoindre le domaine** :
   - `Paramètres > Système > À propos > Renommer ce PC > Domaine`.
   - Saisir `mon-domaine.local` et les identifiants administrateur.

---

## 6. Création et Application de Stratégies de Groupe (GPO)

### 🔹 Exemple : Désactiver l’Accès USB
1. **Ouvrir la console GPO** :
   - `Outils > Gestion des stratégies de groupe`.
2. **Créer une nouvelle GPO** :
   - Clic droit sur **Objets de stratégie de groupe** > **Nouveau** > Nommer la stratégie (ex : `Restriction_USB`).
3. **Configurer la stratégie** :
   - Naviguer vers :
     ```
     Configuration utilisateur > Stratégies > Modèles d’administration > Système > Accès au stockage amovible
     ```
   - Activer **"Désactiver l’accès en écriture aux périphériques de stockage amovibles"**.

4. **Lier la GPO à une Unité d’Organisation (OU)** :
   - Clic droit sur l’OU cible > **Lier un objet de stratégie de groupe existant**.

---

## 7. Dépannage et Logs

### 🔹 Erreurs Courantes
| Problème                          | Solution                                                                 |
|-----------------------------------|--------------------------------------------------------------------------|
| Échec de jonction au domaine      | Vérifier `ipconfig /all` (DNS doit pointer vers l’AD).                  |
| Réplication AD lente              | Vérifier les logs DNS (`Observateur d’événements > DNS Server`).        |
| GPO non appliquée                 | Forcer la mise à jour : `gpupdate /force`.                             |

### 🔹 Commandes de Diagnostic
```powershell
# Vérifier les logs AD
Get-EventLog -LogName "Directory Service" -Newest 20

# Tester la résolution DNS
Resolve-DnsName mon-domaine.local

# Forcer la synchronisation GPO
gpupdate /force
```

---

## 8. Bonnes Pratiques et Sécurité

### 🔹 Sécurité
- **Désactiver le compte administrateur par défaut** :
  ```powershell
  Disable-LocalUser -Name "Administrator"
  ```
- **Mots de passe** :
  - Longueur minimale : 12 caractères.
  - Complexité : Majuscules, minuscules, chiffres, symboles.

### 🔹 Sauvegardes
- **Sauvegarder l’état système** :
  ```powershell
  wbadmin start backup -backuptarget:E: -include:C: -allCritical -quiet
  ```
- **Automatiser avec une tâche planifiée** :
  - Utiliser `Task Scheduler` pour des sauvegardes quotidiennes.

---

## 9. Annexes : Commandes Utiles

| Besoin                          | Commande                                                                 |
|----------------------------------|--------------------------------------------------------------------------|
| Lister les utilisateurs AD       | `Get-ADUser -Filter *`                                                  |
| Réinitialiser un mot de passe    | `Set-ADAccountPassword -Identity "utilisateur" -NewPassword (ConvertTo-SecureString "NouveauMotDePasse" -AsPlainText -Force)` |
| Vérifier la santé de l’AD        | `dcdiag /test:dns`                                                      |

---

## 📌 Notes Finales
- **Tester** : Toujours valider les GPO sur un poste test avant déploiement.
- **Documenter** : Noter les mots de passe, IP, et configurations dans un fichier sécurisé.
- **Mettre à jour** : Appliquer les correctifs Windows régulièrement.

---
**Besoin d’approfondir une section ou d’exemples supplémentaires ?** Dis-moi ce que tu veux explorer en détail ! 🚀
```

---
### Points forts de ce guide :
- **Pas à pas** : Chaque étape est détaillée avec des commandes et des captures suggérées.
- **Dépannage** : Tableau des erreurs courantes et solutions.
- **Sécurité** : Bonnes pratiques pour protéger l’AD.
- **Annexes** : Commandes PowerShell utiles pour l’administration quotidienne.

Tu peux copier ce Markdown dans un fichier `.md` ou un outil comme **Confluence** ou **Notion** pour une référence facile. 😊
