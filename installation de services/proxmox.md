# Proxmox VE - Guide d'installation et d'utilisation

---

## 1. Présentation

**Fonction** :

- Hyperviseur pour héberger VM/LXC (AD, GLPI, etc.).
- **PBS (Proxmox Backup Server)** : Sauvegarde centralisée des VM/LXC.

**Prérequis** :

- Matériel : 32 Go RAM, CPU 64-bit (Intel VT-x/AMD-V activé), stockage SSD/HDD.
- Réseau : 1 interface physique minimum (2+ recommandé pour séparer LAN/DMZ).

---

## 2. Installation

### 2.1. Installation de Proxmox VE

1. **Télécharger l’ISO** :
  [Lien officiel](https://www.proxmox.com/en/downloads).
2. **Créer une clé USB bootable** :
  ```bash
   dd if=proxmox-ve_8.1-1.iso of=/dev/sdX bs=1M status=progress
  ```
3. **Installer Proxmox** :
  - Boot sur la clé USB, suivre l’assistant.
  - Partitionnement : Utiliser **ZFS** (recommandé) ou LVM.
  - Définir un mot de passe root et une IP statique.
4. **Post-installation** :
  - Mettre à jour :
  - Supprimer l’abonnement enterprise (si pas de licence) :
    ```bash
    sed -i "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
    systemctl restart pveproxy
    ```

### 2.2. Configuration réseau

- Éditer `/etc/network/interfaces` :
  ```bash
  auto vmbr0
  iface vmbr0 inet static
      address 192.168.1.254/24
      bridge-ports enp3s0
      bridge-stp off
  ```
- Redémarrer le réseau :
  ```bash
  systemctl restart networking
  ```

### 2.3. Installation de PBS (Proxmox Backup Server)

1. **Télécharger l’ISO** :
  [Lien officiel](https://www.proxmox.com/en/downloads#proxmox-backup-server).
2. **Installer sur une VM ou un serveur dédié** :
  - Suivre l’assistant (similaire à Proxmox VE).
3. **Ajouter PBS à Proxmox** :
  - Dans l’interface Proxmox : `Datacenter > Storage > Add > Proxmox Backup Server`.
  - Renseigner l’IP de PBS et les identifiants.

---

## 3. Intégration avec pfSense

- **Règles pfSense** :
  - Autoriser le trafic entre Proxmox (IP `192.168.1.254`) et PBS (IP `192.168.1.253`) sur le port **8007** (backup).
- **VLANs** :
  - Si tu utilises des VLANs, configure-les dans Proxmox (`Datacenter > Network`).

---

## 4. Utilisation courante

### 4.1. Créer une VM/LXC

- **Via l’interface web** :
  - `Create VM` > Choisir ISO (ex: Windows Server pour AD).
  - Allouer RAM/CPU (ex: 4 Go RAM, 2 vCPUs pour AD).
- **Via CLI** :
  ```bash
  qm create 100 --name "Windows-AD" --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0
  ```

### 4.2. Sauvegarder avec PBS

- **Planifier une sauvegarde** :
  - Dans l’interface Proxmox : Sélectionner une VM > `Backup` > Choisir PBS comme cible.
  - Exemple de commande CLI :
    ```bash
    pvesm backup vm 100 --storage pbs --mode snapshot
    ```

---

## 5. Dépannage

### 5.1. Erreurs fréquentes

- **Erreur "Permission denied" lors des sauvegardes** :
  - Vérifier que l’utilisateur PBS a les droits sur le datastore.
  - Commande pour tester la connexion :
    ```bash
    pvesm auth --storage pbs
    ```
- **Proxmox ne détecte pas le stockage** :
  - Vérifier `/etc/pve/storage.cfg` et redémarrer `pve-storage` :
    ```bash
    systemctl restart pve-storage
    ```

### 5.2. Logs utiles

- **Logs Proxmox** :
  ```bash
  journalctl -u pveproxy -u pvedaemon
  ```
- **Logs PBS** :
  ```bash
  journalctl -u proxmox-backup
  ```

---

## 6. Bonnes pratiques

- **Sécurité** :
  - Désactiver l’accès root en SSH (utiliser un utilisateur dédié) :
    ```bash
    sed -i 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
    systemctl restart sshd
    ```
  - Activer le pare-feu Proxmox :
    ```bash
    ufw allow 8006  # Port de l'interface web
    ufw enable
    ```
- **Optimisation** :
  - Limiter la RAM des LXC (éviter le swapping) :
    ```bash
    pct set 100 --memory 1024 --swap 512
    ```

---

## 7. Références officielles

- [Documentation Proxmox VE](https://pve.proxmox.com/wiki/Main_Page)
- [Documentation PBS](https://pbs.proxmox.com/docs/)
- [Forum Proxmox](https://forum.proxmox.com/)
