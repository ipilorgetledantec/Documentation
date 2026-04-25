# Active Directory - Guide d'installation et d'utilisation

---

## 1. Présentation

**Fonction** :

- Annuaire centralisé pour l’authentification des 10 postes.
- Gestion des stratégies de groupe (GPO), DNS interne.

**Prérequis** :

- **Ressources** : 4 Go RAM, 2 vCPUs, 20 Go SSD.
- **Licence** : Windows Server (évaluation 180 jours ou licence permanente).

---

## 2. Installation

### 2.1. Créer une VM Windows Server sur Proxmox

1. **Télécharger l’ISO** :
  [Windows Server 2022](https://www.microsoft.com/fr-fr/evalcenter/evaluate-windows-server-2022).
2. **Créer la VM** :
  ```bash
   qm create 100 --name "Windows-AD" --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0
   qm set 100 --scsi0 local-lvm:20G
   qm set 100 --ide2 local:iso/Windows_Server_2022.iso,media=cdrom
   qm set 100 --boot order=scsi0;ide2
  ```
3. **Installer Windows Server** :
  - Suivre l’assistant (choisir "Desktop Experience" ou "Core").
  - Définir un mot de passe administrateur.

### 2.2. Configurer Active Directory

1. **Installer le rôle AD** :
  - Ouvrir PowerShell en admin :
  - Redémarrer le serveur.
2. **Configurer DNS** :
  - Vérifier que le rôle DNS est installé (`Server Manager > Add Roles`).
  - Créer une zone de recherche directe pour `mon-domaine.local`.

---

## 3. Intégration avec Proxmox/pfSense

- **Règles pfSense** :
  - Autoriser le trafic entre les postes clients et l’AD (`192.168.1.10`) sur les ports :
    - **TCP 389** (LDAP), **TCP 445** (SMB), **UDP 53** (DNS).
- **DHCP** :
  - Configurer pfSense pour distribuer l’IP de l’AD comme DNS (`Services > DHCP Server`).

---

## 4. Utilisation courante

### 4.1. Ajouter un poste au domaine

1. **Sur le poste client (Windows 10/11)** :
  - `Paramètres > Système > À propos > Renommer ce PC > Domaine`.
  - Saisir `mon-domaine.local` et les identifiants admin.

### 4.2. Créer une GPO

1. **Ouvrir "Gestion des stratégies de groupe"** :
  - `Outils > Gestion des stratégies de groupe`.
2. **Créer une nouvelle stratégie** :
  - Clic droit sur `Objets de stratégie de groupe` > `Nouveau`.
  - Exemple : Désactiver l’accès USB :
    - `Configuration utilisateur > Stratégies > Modèles d’administration > Système > Accès au stockage amovible`.

---

## 5. Dépannage

### 5.1. Erreurs fréquentes

- **Les postes ne rejoignent pas le domaine** :
  - Vérifier la connectivité réseau (`ping mon-domaine.local`).
  - Vérifier que le DNS du poste pointe vers l’AD (`ipconfig /all`).
- **Réplication AD lente** :
  - Vérifier les logs DNS (`Evenement Viewer > DNS Server`).

### 5.2. Logs utiles

- **Logs AD** :
  ```powershell
  Get-EventLog -LogName "Directory Service" -Newest 20
  ```
- **Logs DNS** :
  ```powershell
  Get-DnsServerDiagnostics
  ```

---

## 6. Bonnes pratiques

- **Sécurité** :
  - Désactiver le compte administrateur par défaut (`Disable-LocalUser -Name "Administrator"`).
  - Utiliser des mots de passe complexes pour les comptes admin.
- **Sauvegardes** :
  - Sauvegarder l’état système avec `wbadmin` :
    ```powershell
    wbadmin start backup -backuptarget:E: -include:C: -allCritical -quiet
    ```

---

## 7. Références officielles

- [Documentation AD Microsoft](https://learn.microsoft.com/fr-fr/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Guide PowerShell pour AD](https://learn.microsoft.com/fr-fr/powershell/module/activedirectory/?view=windowsserver2022-ps)
