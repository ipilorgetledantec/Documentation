Voici un **guide ultra-détaillé** pour installer et configurer le **FOG Project** sur **Debian** (plutôt qu’Ubuntu), avec des étapes claires pour chaque clic, chaque fichier, et chaque commande. Je vais aussi préciser les chemins exacts et les captures d’écran attendues (même si je ne peux pas en générer ici, je te décris ce que tu dois voir).

---

# **Guide Complet : Installation et Configuration de FOG sur Debian**

---

## **1. Préparation de la VM sur Proxmox**

### **1.1. Créer la VM**
```bash
qm create 202 --name "FOG-Server" --memory 3072 --cores 2 --net0 virtio,bridge=vmbr0
qm set 202 --scsi0 local-lvm:50Go  # 50 Go minimum pour les images
qm set 202 --ide2 local:iso/debian-12.0.0-amd64-netinst.iso,media=cdrom
qm set 202 --boot order=scsi0;net0
qm start 202
```

### **1.2. Installer Debian 12 (Bookworm)**
- **Étape 1** : Démarrer la VM et sélectionner **"Install"**.
- **Étape 2** : Choisir la langue (Français), le pays (France), et la disposition du clavier (Français).
- **Étape 3** : Nom de la machine : `fog-server`.
- **Étape 4** : Ne pas configurer de domaine.
- **Étape 5** : Mot de passe root : **à définir** (ex: `FogAdmin2026!`).
- **Étape 6** : Créer un utilisateur : `foguser` (mot de passe : `FogUser2026!`).
- **Étape 7** : Partitionnement : **"Guided - use entire disk"**, puis **"All files in one partition"**.
- **Étape 8** : Valider et attendre la fin de l’installation.
- **Étape 9** : **Ne pas installer d’environnement graphique**, mais cocher **"SSH server"** et **"system utilities"**.
- **Étape 10** : Redémarrer la VM.

---

## **2. Configuration initiale de Debian**

### **2.1. Se connecter en SSH**
```bash
ssh root@<IP_DE_LA_VM>
```
(ex: `ssh root@192.168.1.30`)

### **2.2. Mettre à jour le système**
```bash
apt update && apt upgrade -y
apt install -y sudo wget gnupg2 ca-certificates lsb-release debian-archive-keyring
```

### **2.3. Ajouter l’utilisateur `foguser` au sudoers**
```bash
usermod -aG sudo foguser
```

### **2.4. Redémarrer**
```bash
reboot
```

---

## **3. Installation de FOG sur Debian**

### **3.1. Installer les dépendances**
```bash
sudo apt install -y apache2 mariadb-server php php-cli php-gd php-curl php-mysql php-json php-zip php-mbstring php-xml php-bcmath tftpd-hpa nfs-kernel-server vsftpd syslinux pxelinux net-tools
```

### **3.2. Configurer MariaDB**
```bash
sudo mysql_secure_installation
```
- Répondre **Y** à toutes les questions (sauf "Change the root password?" si tu as déjà défini un mot de passe).

### **3.3. Créer la base de données FOG**
```bash
sudo mysql -u root -p
```
Puis exécuter ces commandes SQL :
```sql
CREATE DATABASE fog;
GRANT ALL PRIVILEGES ON fog.* TO 'fog'@'localhost' IDENTIFIED BY 'TonMotDePasseFOG2026!';
FLUSH PRIVILEGES;
EXIT;
```

### **3.4. Télécharger et installer FOG**
```bash
cd /opt
sudo wget https://github.com/FOGProject/fogproject/archive/refs/tags/1.5.10.tar.gz
sudo tar -xzf 1.5.10.tar.gz
cd fogproject-1.5.10/bin
sudo ./installfog.sh
```

### **3.5. Répondre aux questions de l’installateur**
| Question | Réponse |
|----------|---------|
| **Type d’installation** | `N` (Normal Server) |
| **Interface réseau** | `ens18` (ou `eth0` selon ta VM) |
| **IP du serveur DHCP** | **Laisser vide** (pfSense gère le DHCP) |
| **Mot de passe MySQL** | `TonMotDePasseFOG2026!` |
| **Confirmer l’installation** | `Y` |

### **3.6. Vérifier les services**
```bash
sudo systemctl status apache2
sudo systemctl status mariadb
sudo systemctl status tftpd-hpa
```

---

## **4. Configuration de FOG**

### **4.1. Accéder à l’interface web**
- Ouvrir un navigateur et aller sur :
  **`http://192.168.1.30/fog/management`**
- Identifiants par défaut :
  - **Utilisateur** : `fog`
  - **Mot de passe** : `password` (à changer immédiatement !)

### **4.2. Changer le mot de passe admin**
1. Cliquer sur **"FOG Configuration"** (en haut à droite).
2. Aller dans **"FOG Settings"** > **"Web Server"** > **"FOG Password"**.
3. Définir un nouveau mot de passe (ex: `FogAdmin2026!`).

### **4.3. Configurer le stockage des images**
1. Dans l’interface FOG, aller dans **"Storage Management"** > **"New Storage Node"**.
2. Remplir comme suit :
   - **Name** : `FOG-Storage`
   - **IP Address** : `192.168.1.30`
   - **Path** : `/images/`
   - **Username** : `fog`
   - **Password** : `TonMotDePasseFOG2026!`
3. Cliquer sur **"Add"**.

### **4.4. Vérifier les permissions**
```bash
sudo chown -R fog:www-data /images/
sudo chmod -R 775 /images/
```

---

## **5. Configuration de pfSense pour le PXE**

### **5.1. Activer le DHCP PXE**
1. Dans pfSense, aller dans **"Services"** > **"DHCP Server"**.
2. Dans l’onglet **"LAN"**, ajouter :
   - **Option 66 (Boot Server)** : `192.168.1.30`
   - **Option 67 (Bootfile)** : `undionly.kpxe`
3. Sauvegarder et redémarrer le service DHCP.

### **5.2. Autoriser le trafic nécessaire**
1. Aller dans **"Firewall"** > **"Rules"** > **"LAN"**.
2. Ajouter une règle pour autoriser :
   - **Protocole** : UDP
   - **Port** : 69 (TFTP)
   - **Destination** : `192.168.1.30`
3. Ajouter une règle pour le **port 80 (HTTP)** vers `192.168.1.30`.

---

## **6. Capturer une image avec FOG**

### **6.1. Créer un hôte dans FOG**
1. Dans l’interface FOG, aller dans **"Hosts"** > **"New Host"**.
2. Remplir :
   - **Host Name** : `Poste-01`
   - **MAC Address** : `00:11:22:33:44:55` (remplacer par la vraie MAC du poste)
   - **Host Image** : `Windows10-Master` (à créer ensuite)
3. Cliquer sur **"Add"**.

### **6.2. Créer une image**
1. Aller dans **"Images"** > **"New Image"**.
2. Remplir :
   - **Name** : `Windows10-Master`
   - **Image Type** : `Single Disk - Resizable`
   - **Storage Group** : `Default`
3. Cliquer sur **"Add"**.

### **6.3. Lancer la capture**
1. Démarrer le poste client en **PXE** (F12 ou option dans le BIOS).
2. Sélectionner **"FOG Capture Image"**.
3. Choisir l’image `Windows10-Master` et valider.
4. Attendre la fin de la capture (les logs s’affichent à l’écran).

---

## **7. Déployer une image avec FOG**

### **7.1. Associer l’image à un hôte**
1. Dans FOG, aller dans **"Hosts"**, sélectionner `Poste-01`.
2. Dans **"Host Image"**, choisir `Windows10-Master`.
3. Cliquer sur **"Update"**.

### **7.2. Lancer le déploiement**
1. Démarrer le poste en **PXE**.
2. Sélectionner **"FOG Deploy Image"**.
3. Choisir l’image et valider.

---

## **8. Dépannage**

### **8.1. Le poste ne boot pas en PXE**
- **Vérifier** :
  - Que le DHCP de pfSense renvoie bien l’IP de FOG (options 66/67).
  - Que le service `tftpd-hpa` est actif :
    ```bash
    sudo systemctl restart tftpd-hpa
    ```
  - Les logs TFTP :
    ```bash
    sudo tail -f /var/log/syslog | grep tftp
    ```

### **8.2. Erreur "Unable to mount filesystem"**
- **Vérifier** :
  - L’espace disque sur `/images/` :
    ```bash
    df -h /images/
    ```
  - Les permissions :
    ```bash
    sudo chmod -R 777 /images/
    ```

### **8.3. Logs utiles**
- **Logs FOG** :
  ```bash
  sudo tail -f /var/log/fog/fog.log
  ```
- **Logs Apache** :
  ```bash
  sudo tail -f /var/log/apache2/error.log
  ```

---

## **9. Bonnes pratiques**

### **9.1. Optimisation**
- **Multicast** : Pour déployer sur plusieurs postes en même temps, activer le multicast dans les paramètres de l’image.
- **Compression** : Dans les paramètres de l’image, choisir `"Partclone gzip"` pour réduire la taille.

### **9.2. Sécurité**
- **Restriction d’accès** :
  - Limiter l’accès à l’interface FOG via pfSense (règles firewall).
  - Changer les mots de passe par défaut (MySQL, FOG, Debian).

---

## **10. Sauvegardes**
- **Sauvegarder `/images/`** :
  ```bash
  sudo tar -czvf fog_images_backup_$(date +%Y%m%d).tar.gz /images/
  ```
- **Sauvegarder la base de données** :
  ```bash
  sudo mysqldump -u fog -p fog > fog_db_backup_$(date +%Y%m%d).sql
  ```

---

### **Résumé des chemins importants**
| Dossier/Fichier | Description |
|-----------------|-------------|
| `/images/` | Stockage des images FOG |
| `/var/www/html/fog/` | Interface web FOG |
| `/tftpboot/` | Fichiers de boot PXE |
| `/var/log/fog/fog.log` | Logs FOG |

---
**Prochaine étape** : Tu veux que je détaille un point spécifique (ex: configuration multicast, dépannage avancé) ? Ou as-tu des questions sur une étape en particulier ?
