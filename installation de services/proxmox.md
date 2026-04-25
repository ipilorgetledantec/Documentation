Voici une version **ultra-détaillée en Markdown** pour Proxmox VE, incluant l’installation, la configuration, l’intégration avec pfSense, et les bonnes pratiques. Tout est organisé pour une mise en œuvre pas à pas, avec des commandes, des exemples et des astuces de dépannage.

---

```markdown
# 📖 Proxmox VE - Guide Complet d’Installation et d’Utilisation

---

## 📌 Table des Matières
1. [Présentation et Prérequis](#1-présentation-et-prérequis)
2. [Installation de Proxmox VE](#2-installation-de-proxmox-ve)
3. [Configuration Réseau](#3-configuration-réseau)
4. [Installation de PBS (Proxmox Backup Server)](#4-installation-de-pbs-proxmox-backup-server)
5. [Intégration avec pfSense](#5-intégration-avec-pfsense)
6. [Utilisation Courante](#6-utilisation-courante)
7. [Dépannage](#7-dépannage)
8. [Bonnes Pratiques](#8-bonnes-pratiques)
9. [Références Officielles](#9-références-officielles)

---

## 1. Présentation et Prérequis

### 🔹 Fonctionnalités Clés
- **Hyperviseur** : Hébergement de machines virtuelles (VM) et conteneurs LXC (ex : Active Directory, GLPI, Nextcloud).
- **PBS (Proxmox Backup Server)** : Sauvegarde centralisée et incrémentielle des VM/LXC.
- **Interface Web** : Gestion complète via une interface intuitive.

### 🔹 Prérequis Matériels
| Ressource          | Exigence Minimum       | Recommandé pour 10 VMs/LXC |
|--------------------|------------------------|-----------------------------|
| RAM                | 8 Go                   | 32 Go                       |
| CPU                | 64-bit (VT-x/AMD-V)    | 4+ cœurs                    |
| Stockage           | 128 Go SSD/HDD         | 500 Go+ SSD (ZFS)           |
| Réseau             | 1 interface            | 2+ interfaces (LAN/DMZ)     |

### 🔹 Schéma Réseau Recommandé
```
[Internet] ←→ [pfSense] ←→ [Proxmox VE (192.168.1.254)] ←→ [PBS (192.168.1.253)]
                          ↓
                     [VM/LXC (ex: AD, GLPI)]
```

---

## 2. Installation de Proxmox VE

### 🔹 Étape 1 : Télécharger l’ISO
- Télécharger la dernière version depuis [proxmox.com](https://www.proxmox.com/en/downloads).
- Exemple : `proxmox-ve_8.1-1.iso`.

### 🔹 Étape 2 : Créer une Clé USB Bootable
```bash
# Remplacer /dev/sdX par ton périphérique USB (ex: /dev/sdb)
dd if=proxmox-ve_8.1-1.iso of=/dev/sdX bs=1M status=progress
sync
```

### 🔹 Étape 3 : Installer Proxmox VE
1. **Démarrer sur la clé USB** :
   - Sélectionner **Install Proxmox VE**.
2. **Partitionnement** :
   - **Option recommandée** : **ZFS** (RAID1 si 2 disques).
   - Sinon : **LVM** (pour les débutants).
3. **Configuration réseau** :
   - **IP statique** : Ex : `192.168.1.254/24`.
   - **Passerelle** : IP de ton routeur (ex : `192.168.1.1`).
   - **DNS** : `8.8.8.8` (Google) ou `1.1.1.1` (Cloudflare).
4. **Mot de passe root** :
   - Définir un mot de passe complexe (ex : `P@ssw0rd2026!`).

### 🔹 Étape 4 : Post-Installation
1. **Mettre à jour Proxmox** :
   ```bash
   apt update && apt upgrade -y
   ```
2. **Supprimer le message d’abonnement** (si pas de licence) :
   ```bash
   sed -i "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
   systemctl restart pveproxy
   ```

---

## 3. Configuration Réseau

### 🔹 Étape 1 : Configurer les Interfaces
- Éditer `/etc/network/interfaces` :
  ```bash
  nano /etc/network/interfaces
  ```
- Exemple pour un bridge (`vmbr0`) :
  ```bash
  auto vmbr0
  iface vmbr0 inet static
      address 192.168.1.254/24
      gateway 192.168.1.1
      bridge-ports enp3s0
      bridge-stp off
  ```
- **Redémarrer le réseau** :
  ```bash
  systemctl restart networking
  ```

### 🔹 Étape 2 : Vérifier la Connectivité
```bash
ping 8.8.8.8
ping google.com
```

---

## 4. Installation de PBS (Proxmox Backup Server)

### 🔹 Étape 1 : Télécharger l’ISO
- [Lien officiel PBS](https://www.proxmox.com/en/downloads#proxmox-backup-server).

### 🔹 Étape 2 : Installer PBS
1. **Créer une VM ou utiliser un serveur dédié** :
   - 2 vCPU, 4 Go RAM, 500 Go stockage.
2. **Suivre l’assistant d’installation** (similaire à Proxmox VE).
3. **Configurer le stockage** :
   - Utiliser un disque dédié pour les sauvegardes (ex : `/mnt/backup`).

### 🔹 Étape 3 : Intégrer PBS à Proxmox
1. **Dans l’interface Proxmox** :
   - Aller dans `Datacenter > Storage > Add > Proxmox Backup Server`.
2. **Renseigner** :
   - **ID** : `pbs`
   - **Serveur** : IP de PBS (ex : `192.168.1.253`).
   - **Fingerprint** : Récupéré via `pbsconfig fingerprint` sur le serveur PBS.
   - **Utilisateur** : `root@pam` (ou un utilisateur dédié).

---

## 5. Intégration avec pfSense

### 🔹 Étape 1 : Autoriser le Trafic
- **Règles pfSense** :
  - Autoriser le trafic entre Proxmox (`192.168.1.254`) et PBS (`192.168.1.253`) sur le port **8007** (backup).
  - Exemple :
    ```
    Interface: LAN
    Protocol: TCP
    Source: 192.168.1.254
    Destination: 192.168.1.253
    Port: 8007
    Action: Pass
    ```

### 🔹 Étape 2 : Configurer les VLANs (Optionnel)
- **Dans Proxmox** :
  - `Datacenter > Network > Create > Linux VLAN`.
  - Exemple pour un VLAN 10 :
    ```bash
    auto vmbr0.10
    iface vmbr0.10 inet static
        address 192.168.10.1/24
        vlan-raw-device vmbr0
    ```

---

## 6. Utilisation Courante

### 🔹 Créer une VM
- **Via l’interface web** :
  - `Create VM` > Sélectionner un ISO (ex : Windows Server pour AD).
  - Allouer **4 Go RAM, 2 vCPU, 20 Go SSD**.
- **Via CLI** :
  ```bash
  qm create 100 --name "Windows-AD" --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0
  qm set 100 --scsi0 local-lvm:20G
  qm set 100 --ide2 local:iso/Windows_Server_2022.iso,media=cdrom
  ```

### 🔹 Sauvegarder une VM avec PBS
- **Via l’interface** :
  - Sélectionner la VM > `Backup` > Choisir `pbs` comme cible.
- **Via CLI** :
  ```bash
  vzdump 100 --storage pbs --mode snapshot --compress lzo
  ```

---

## 7. Dépannage

### 🔹 Erreurs Fréquentes
| Problème                          | Solution                                                                 |
|-----------------------------------|--------------------------------------------------------------------------|
| "Permission denied" lors des sauvegardes | Vérifier les droits sur le datastore PBS : `pvesm auth --storage pbs` |
| Proxmox ne détecte pas le stockage | Vérifier `/etc/pve/storage.cfg` et redémarrer : `systemctl restart pve-storage` |
| Impossible de démarrer une VM      | Vérifier les logs : `journalctl -u pve-qemu`                           |

### 🔹 Commandes de Diagnostic
```bash
# Logs Proxmox
journalctl -u pveproxy -u pvedaemon -f

# Logs PBS
journalctl -u proxmox-backup -f

# Tester la connexion à PBS
pvesm auth --storage pbs
```

---

## 8. Bonnes Pratiques

### 🔹 Sécurité
- **Désactiver l’accès root en SSH** :
  ```bash
  sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
  systemctl restart sshd
  ```
- **Activer le pare-feu** :
  ```bash
  ufw allow 8006/tcp  # Interface web
  ufw enable
  ```

### 🔹 Optimisation
- **Limiter la RAM des LXC** :
  ```bash
  pct set 100 --memory 1024 --swap 512
  ```
- **Utiliser ZFS pour les sauvegardes** :
  ```bash
  zfs create rpool/backup
  ```

---

## 9. Références Officielles
- [Documentation Proxmox VE](https://pve.proxmox.com/wiki/Main_Page)
- [Documentation PBS](https://pbs.proxmox.com/docs/)
- [Forum Proxmox](https://forum.proxmox.com/)

---

## 📌 Notes Finales
- **Tester** : Toujours valider les configurations sur une VM de test avant de déployer en production.
- **Documenter** : Noter les IP, mots de passe, et configurations dans un fichier sécurisé.
- **Mettre à jour** : Appliquer les correctifs Proxmox régulièrement (`apt update && apt upgrade`).

---
**Besoin d’approfondir une section ou d’exemples supplémentaires ?** Dis-moi ce que tu veux explorer en détail ! 🚀
```

---
### Points forts de ce guide :
- **Pas à pas** : Chaque étape est détaillée avec des commandes et des exemples concrets.
- **Dépannage** : Tableau des erreurs courantes et solutions.
- **Sécurité** : Bonnes pratiques pour protéger ton infrastructure.
- **Intégration** : Configuration réseau et règles pfSense incluses.

Tu peux copier ce Markdown dans un fichier `.md` ou un outil comme **Notion** ou **Confluence** pour une référence facile. 😊
Si tu veux commencer par une étape précise, dis-le-moi !
