# FOG Project - Guide d'installation et d'utilisation

---

## 1. Présentation

**Fonction** :

- Déploiement et sauvegarde d’images disque pour tes 10 postes.
- Support du multicast (déploiement simultané sur plusieurs postes).

**Prérequis** :

- **Ressources** : 3 Go RAM, 2 vCPUs, 50 Go+ HDD (pour stocker les images).
- **Réseau** : PXE activé sur pfSense (port UDP 69).

---

## 2. Installation

### 2.1. Créer une VM sur Proxmox

```bash
qm create 202 --name "FOG-Server" --memory 3072 --cores 2 --net0 virtio,bridge=vmbr0
qm set 202 --scsi0 local-lvm:50G
qm set 202 --ide2 local:iso/ubuntu-22.04-live-server-amd64.iso,media=cdrom
```

### 2.2. Installer FOG sur Ubuntu

1. **Installer Ubuntu Server** :
  - Suivre l’assistant (choisir "OpenSSH server" pendant l’installation).
2. **Mettre à jour le système** :
  ```bash
   apt update && apt upgrade -y
  ```
3. **Installer FOG** :
  ```bash
   wget https://github.com/FOGProject/fogproject/archive/refs/tags/1.5.10.tar.gz
   tar -xzf 1.5.10.tar.gz
   cd fogproject-1.5.10/bin
   ./installfog.sh
  ```
  - Choisir :
    - **Type d’installation** : "Normal Server".
    - **Interface réseau** : `eth0` (ou `ens18` selon Proxmox).
    - **IP du serveur DHCP** : Laisser vide (pfSense gère le DHCP).
    - **Mot de passe MySQL** : Définir un mot de passe sécurisé.
4. **Configurer le stockage** :
  - Les images seront stockées dans `/images/` (monter un disque supplémentaire si besoin).

---

## 3. Intégration avec Proxmox/pfSense

- **Règles pfSense** :
  - Autoriser le trafic **UDP 69** (TFTP) et **TCP 80** (interface web FOG) vers `192.168.1.30`.
  - **DHCP** :
    - Dans pfSense, configurer l’option **66 (Boot Server)** avec l’IP de FOG (`192.168.1.30`).
    - Option **67 (Bootfile)** : `undionly.kpxe`.
- **Sauvegardes** :
  - Sauvegarder `/images/` et la base de données FOG avec PBS.

---

## 4. Utilisation courante

### 4.1. Capturer une image

1. **Depuis l’interface FOG** (`http://192.168.1.30/fog`) :
  - `Hosts > Create New Host` :
    - Nom : `Poste-01`, MAC address : `00:11:22:33:44:55`.
  - `Images > Create New Image` :
    - Nom : `Windows10-Master`, type : "Single Disk - Resizable".
2. **Démarrer le poste en PXE** :
  - Dans le BIOS, activer le boot PXE.
  - Sélectionner "FOG Capture Image" dans le menu.

### 4.2. Déployer une image

1. **Depuis l’interface FOG** :
  - Sélectionner le poste et l’image à déployer.
  - Choisir "Deploy Image".
2. **Démarrer le poste en PXE** :
  - Sélectionner "FOG Deploy Image".

---

## 5. Dépannage

### 5.1. Erreurs fréquentes

- **Le poste ne boot pas en PXE** :
  - Vérifier que le DHCP de pfSense renvoie bien l’IP de FOG (option 66/67).
  - Tester avec `tcpdump` sur FOG :
    ```bash
    tcpdump -i eth0 port 69
    ```
- **Erreur "Unable to mount filesystem"** :
  - Vérifier que `/images/` a assez d’espace (`df -h`).
  - Vérifier les permissions :
    ```bash
    chmod -R 777 /images
    ```

### 5.2. Logs utiles

- **Logs FOG** :
  ```bash
  tail -f /var/log/fog/fog.log
  ```
- **Logs Apache** :
  ```bash
  tail -f /var/log/apache2/error.log
  ```

---

## 6. Bonnes pratiques

- **Optimisation** :
  - Utiliser le **multicast** pour déployer sur plusieurs postes simultanément.
  - Compresser les images (`Partclone gzip` dans les paramètres de l’image).
- **Sécurité** :
  - Restreindre l’accès à l’interface web FOG (pfSense ou `.htaccess`).

---

## 7. Références officielles

- [Documentation FOG](https://wiki.fogproject.org/)
- [Guide PXE avec pfSense](https://docs.netgate.com/pfsense/en/latest/services/dhcp-relay/bootp-dhcp-tftp.html)
