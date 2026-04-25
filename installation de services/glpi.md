# GLPI - Guide d'installation et d'utilisation

---

## 1. Présentation

**Fonction** :

- Gestion des tickets (helpdesk).
- Inventaire automatique du parc (via FusionInventory).
- Gestion des licences, contrats, et documents.

**Prérequis** :

- **Ressources** : 1 Go RAM, 1 vCPU, 10 Go SSD.
- **Dépendances** : Apache/MySQL/MariaDB, PHP 8.0+.

---

## 2. Installation

### 2.1. Créer un conteneur LXC sur Proxmox

```bash
pct create 200 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.gz \
  --rootfs local-lvm:10G \
  --memory 1024 \
  --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --hostname glpi \
  --password motdepasse
```

### 2.2. Installer GLPI

1. **Mettre à jour le système** :
  ```bash
   apt update && apt upgrade -y
   apt install -y apache2 mariadb-server php php-mysql php-gd php-curl php-xml php-mbstring php-intl php-apcu php-ldap
  ```
2. **Télécharger GLPI** :
  ```bash
   wget https://github.com/glpi-project/glpi/releases/download/10.0.10/glpi-10.0.10.tgz
   tar -xzf glpi-*.tgz -C /var/www/html/
   chown -R www-data:www-data /var/www/html/glpi
  ```
3. **Configurer Apache** :
  ```bash
   a2enmod rewrite
   systemctl restart apache2
  ```
4. **Créer la base de données** :
  ```bash
   mysql -u root -p
   CREATE DATABASE glpi;
   CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'motdepasse';
   GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost';
   FLUSH PRIVILEGES;
  ```
5. **Finaliser l’installation** :
  - Accéder à `http://<IP_GLPI>/glpi` et suivre l’assistant.

---

## 3. Intégration avec Proxmox/pfSense

- **Règles pfSense** :
  - Autoriser le trafic vers GLPI (`192.168.1.20`) sur le port **80/443**.
  - Si tu utilises HAProxy, ajouter une règle pour rediriger `/glpi` vers `192.168.1.20:80`.
- **Sauvegardes** :
  - Sauvegarder `/var/www/html/glpi` et la base de données avec PBS.

---

## 4. Utilisation courante

### 4.1. Configurer FusionInventory

1. **Installer le plugin** :
  - Dans GLPI : `Configuration > Plugins` > Installer FusionInventory.
2. **Déployer l’agent sur les postes** :
  - Télécharger l’agent depuis `FusionInventory > Agents`.
  - Déployer via WAPT ou script PowerShell.

### 4.2. Créer un ticket

1. **Depuis l’interface** :
  - `Assistance > Tickets` > `Nouveau ticket`.
2. **Automatiser via email** :
  - Configurer un compte IMAP dans `Configuration > Notifications`.

---

## 5. Dépannage

### 5.1. Erreurs fréquentes

- **Erreur de connexion à la base de données** :
  - Vérifier les identifiants dans `/var/www/html/glpi/config/db.config.php`.
  - Redémarrer Apache :
    ```bash
    systemctl restart apache2
    ```
- **FusionInventory ne remonte pas les inventaires** :
  - Vérifier que l’agent est installé et que le serveur GLPI est accessible (`telnet <IP_GLPI> 80`).

### 5.2. Logs utiles

- **Logs Apache** :
  ```bash
  tail -f /var/log/apache2/error.log
  ```
- **Logs GLPI** :
  ```bash
  tail -f /var/www/html/glpi/files/_logs/php-error.log
  ```

---

## 6. Bonnes pratiques

- **Sécurité** :
  - Activer HTTPS avec Let’s Encrypt :
    ```bash
    apt install certbot python3-certbot-apache
    certbot --apache -d glpi.mon-domaine.local
    ```
  - Restreindre l’accès à GLPI via pfSense (ex: seulement le LAN).
- **Optimisation** :
  - Désactiver les plugins inutilisés (`Configuration > Plugins`).

---

## 7. Références officielles

- [Documentation GLPI](https://glpi-install.readthedocs.io/)
- [Plugin FusionInventory](https://github.com/fusioninventory/fusioninventory-for-glpi)
