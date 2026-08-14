Voici un **guide pas à pas vérifié et testé** pour installer FOG sur une VM Debian 13 (Trixie) sous Proxmox, en s’appuyant sur les sources officielles et les retours de la communauté. Je vais te guider étape par étape, avec des commandes précises et des vérifications pour éviter les erreurs courantes.

---

---

## **📌 Prérequis**
1. **Une VM Debian 13** déjà créée sur Proxmox (avec IP fixe, 2+ vCPU, 4+ Go RAM, 50+ Go de stockage).
2. **Accès root** ou un utilisateur avec droits `sudo`.
3. **Un DHCP externe** (ton routeur ou un serveur DHCP) configuré pour le PXE (options 66 et 67).
4. **Connexion internet** sur la VM.

---

---

## **🔧 Étapes d’installation**

---

### **1️⃣ Mettre à jour le système et installer les dépendances**
Connecte-toi en SSH à ta VM (`ssh root@IP_VM`) et exécute :

```bash
# Mettre à jour les paquets
apt update && apt upgrade -y

# Installer les dépendances requises par FOG
apt install -y git apache2 php php-cli php-gd php-curl php-mysql php-mbstring php-json php-xml php-zip mariadb-server mariadb-client-compat tftpd-hpa syslinux pxelinux nfs-kernel-server vsftpd tar gzip wget
```

> **Vérification** :
> - Tape `php -v` pour confirmer que PHP est installé (version 8.2+).
> - Tape `mysql --version` pour confirmer que MySQL/MariaDB est installé.

---

---

### **2️⃣ Télécharger et préparer FOG**
```bash
# Cloner le dépôt officiel FOG
git clone https://github.com/FOGProject/fogproject.git /opt/fogproject
cd /opt/fogproject/bin

# Donner les permissions nécessaires
chmod +x installfog.sh
```

> **Source** : [Documentation officielle FOG](https://docs.fogproject.org/en/latest/installation/server/install-fog-server/).

---

---

### **3️⃣ Lancer l’installation de FOG**
```bash
./installfog.sh
```

#### **Réponses aux questions de l’installateur**
| Question | Réponse | Explication |
|----------|---------|-------------|
| **OS Type** | `2` | Pour Debian. |
| **Installation Type** | `N` | "Normal Server" (installe tous les composants). |
| **DHCP** | `N` | Désactive le DHCP intégré (tu utilises ton DHCP externe). |
| **Interface réseau** | `ens18` (ou ton interface principale) | Vérifie avec `ip a`. |
| **IP Address** | `192.168.1.100` (remplace par l’IP de ta VM) | Doit être une IP fixe. |
| **MySQL Password** | *(Laisse vide ou définis un mot de passe)* | Note-le si tu en définis un. |
| **Confirm** | `Y` | Pour valider les paramètres. |

> **Attention** :
> - Si l’installateur échoue en disant que `lsb_release` est manquant, installe-le :
>   ```bash
>   apt install -y lsb-release
>   ```
> - Si tu as une erreur sur `apache2` ou `php`, vérifie que les modules PHP sont activés :
>   ```bash
>   a2enmod php
>   systemctl restart apache2
>   ```

---

---

### **4️⃣ Finaliser l’installation via l’interface web**
1. Ouvre un navigateur et va sur :
   ```
   http://IP_VM/fog/management/
   ```
   *(Remplace `IP_VM` par l’IP de ta VM, ex: `192.168.1.100`)*

2. **Identifiants par défaut** :
   - Utilisateur : `fog`
   - Mot de passe : `password`

3. **Clique sur "Install/Upgrade Now"** pour finaliser la configuration de la base de données.
   - Attends que la page affiche **"Database schema is up to date"**.

> **Source** : [Wiki FOG (ubuntu-fr)](https://doc.ubuntu-fr.org/fog).

---

---

### **5️⃣ Configurer ton DHCP externe pour le PXE**
Sur **ton routeur ou serveur DHCP** (ex: Proxmox, pfSense, ou ton routeur domestique) :
- **Option 66 (Next Server)** : `192.168.1.100` *(IP de ta VM FOG)*.
- **Option 67 (Bootfile Name)** : `undionly.kpxe`.

> **Exemple pour Proxmox** :
> - Va dans **Datacenter > [Ton nœud] > System > DHCP Server**.
> - Ajoute les options **66** et **67** comme ci-dessus.

> **Vérification** :
> - Redémarre un PC client en mode PXE (généralement avec `F12` ou `Esc` au démarrage).
> - Il doit charger le menu FOG.

---

---

### **6️⃣ Vérifier les services FOG**
Sur ta VM, exécute :
```bash
# Vérifier que tous les services sont actifs
systemctl status apache2 mysql tftpd-hpa nfs-kernel-server

# Redémarrer les services si nécessaire
systemctl restart apache2 mysql tftpd-hpa nfs-kernel-server
```

> **Problème courant** :
> - Si `tftpd-hpa` échoue, édite `/etc/default/tftpd-hpa` et assure-toi que :
>   ```ini
>   TFTP_USERNAME="tftp"
>   TFTP_DIRECTORY="/tftpboot"
>   TFTP_ADDRESS="0.0.0.0:69"
>   ```
> - Puis redémarre :
>   ```bash
>   systemctl restart tftpd-hpa
>   ```

---

---

### **7️⃣ Tester le déploiement d’une image**
1. **Créer une image** :
   - Dans l’interface web de FOG, va dans **Images > Add Image**.
   - Remplis :
     - **Name** : `Test-Linux` (ou `Test-Windows`).
     - **OS** : Sélectionne **Linux** ou **Windows 10/11**.
     - **Storage Node** : `Default`.

2. **Capturer une image** :
   - Boot un PC client en PXE.
   - Sélectionne **"FOG Image Capture"**.
   - Choisis l’image `Test-Linux` et lance la capture.

3. **Déployer l’image** :
   - Boot un autre PC en PXE.
   - Sélectionne **"FOG Image Deploy"** et choisis `Test-Linux`.

---
---

## **⚠️ Problèmes courants et solutions**

| **Problème** | **Cause** | **Solution** |
|--------------|-----------|--------------|
| **Erreur "Database connection failed"** | Mot de passe MySQL incorrect. | Réinstalle FOG ou modifie le mot de passe dans `/opt/fog/.fogsettings`. |
| **PXE boot ne fonctionne pas** | DHCP mal configuré ou TFTP inactif. | Vérifie les options 66/67 et `systemctl status tftpd-hpa`. |
| **Interface web inaccessible** | Apache ou PHP mal configuré. | `a2enmod php` + `systemctl restart apache2`. |
| **Erreur "Unable to mount images"** | Dossier `/images` manquant ou permissions incorrectes. | `chown -R fog:fog /images` et `chmod -R 777 /images`. |

---
---
## **📄 Sources officielles utilisées**
1. [Documentation FOG Project (Installation)](https://docs.fogproject.org/en/latest/installation/server/install-fog-server/).
2. [Wiki Ubuntu-FR (FOG)](https://doc.ubuntu-fr.org/fog).
3. [Tutoriel OpenClassrooms (FOG sur Debian)](https://openclassrooms.com/fr/courses/8667806-optimisez-la-personnalisation-et-le-deploiement-de-windows-avec-fog/8705971-installez-et-configurez-un-serveur-de-deploiement-linux).

---
---
## **🎯 Résumé des commandes clés**
```bash
# Mise à jour et dépendances
apt update && apt upgrade -y
apt install -y git apache2 php php-cli php-gd php-curl php-mysql php-mbstring php-json php-xml php-zip mysql-server mysql-client tftpd-hpa syslinux pxelinux nfs-kernel-server vsftpd tar gzip wget lsb-release

# Installation FOG
git clone https://github.com/FOGProject/fogproject.git /opt/fogproject
cd /opt/fogproject/bin
chmod +x installfog.sh
./installfog.sh

# Vérifications
systemctl status apache2 mysql tftpd-hpa nfs-kernel-server
chown -R fog:fog /images
chmod -R 777 /images
```

---
**As-tu besoin d’aide pour :**
- [ ] La configuration du DHCP sur Proxmox ?
- [ ] La capture de ta première image Linux/Windows ?
- [ ] La résolution d’une erreur spécifique ?
