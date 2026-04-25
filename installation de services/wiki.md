# DokuWiki - Guide d'installation et d'utilisation

---

## 1. Présentation

**Fonction** :

- Wiki léger pour documenter ton infrastructure (procédures, schémas, etc.).
- Pas de base de données (fichiers plats).

**Prérequis** :

- **Ressources** : 512 Mo RAM, 1 vCPU, 5 Go SSD.
- **Dépendances** : Apache/PHP.

---

## 2. Installation

### 2.1. Créer un conteneur LXC

```bash
pct create 201 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.gz \
  --rootfs local-lvm:5G \
  --memory 512 \
  --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --hostname dokuwiki \
  --password motdepasse
```

### 2.2. Installer DokuWiki

1. **Installer Apache/PHP** :
  ```bash
   apt update && apt install -y apache2 php php-cli php-gd php-xml php-json
  ```
2. **Télécharger DokuWiki** :
  ```bash
   wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
   tar -xzf dokuwiki-stable.tgz -C /var/www/html/
   mv /var/www/html/dokuwiki-* /var/www/html/dokuwiki
   chown -R www-data:www-data /var/www/html/dokuwiki
  ```
3. **Configurer Apache** :
  ```bash
   a2enmod rewrite
   systemctl restart apache2
  ```
4. **Finaliser l’installation** :
  - Accéder à `http://<IP_DOKUWIKI>/dokuwiki/install.php` et suivre les étapes.

---

## 3. Intégration avec Proxmox/pfSense

- **Règles pfSense** :
  - Autoriser le trafic vers DokuWiki (`192.168.1.21`) sur le port **80**.
  - Si tu utilises HAProxy, ajouter une règle pour `/dokuwiki`.
- **Sauvegardes** :
  - Sauvegarder `/var/www/html/dokuwiki` avec PBS.

---

## 4. Utilisation courante

### 4.1. Créer une page

1. **Depuis l’interface** :
  - Cliquer sur "Créer cette page" après avoir saisi son nom dans la barre de recherche.
2. **Syntaxes utiles** :
  - Lien interne : `[[page]]`.
  - Image : `{{:namespace:fichier.jpg?200}}`.

### 4.2. Configurer les ACL (contrôle d’accès)

1. **Éditer `conf/acl.auth.php**` :
  ```php
   '*':['@ALL': 1],  // Tout le monde peut lire
   '*':['@admin': 7], // Seuls les admins peuvent éditer
  ```
2. **Créer un utilisateur admin** :
  - `Admin > User Manager` > Ajouter un utilisateur.

---

## 5. Dépannage

### 5.1. Erreurs fréquentes

- **Erreur 403 lors de l’accès** :
  - Vérifier les permissions :
    ```bash
    chown -R www-data:www-data /var/www/html/dokuwiki
    chmod -R 755 /var/www/html/dokuwiki
    ```
- **Les modifications ne sont pas enregistrées** :
  - Vérifier que le dossier `data/` est writable :
    ```bash
    chmod -R 775 /var/www/html/dokuwiki/data
    ```

### 5.2. Logs utiles

- **Logs Apache** :
  ```bash
  tail -f /var/log/apache2/error.log
  ```

---

## 6. Bonnes pratiques

- **Sécurité** :
  - Désactiver l’installateur après la première utilisation :
    ```bash
    rm /var/www/html/dokuwiki/install.php
    ```
  - Activer HTTPS avec Let’s Encrypt (comme pour GLPI).
- **Sauvegardes** :
  - Sauvegarder régulièrement `/var/www/html/dokuwiki/data`.

---

## 7. Références officielles

- [Documentation DokuWiki](https://www.dokuwiki.org/dokuwiki)
- [Guide ACL](https://www.dokuwiki.org/acl)
