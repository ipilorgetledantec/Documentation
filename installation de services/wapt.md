# WAPT - Guide d'installation et d'utilisation

---

## 1. Présentation

**Fonction** :

- Déploiement centralisé de logiciels et mises à jour sur tes 10 postes.
- Gestion des paquets (ex: Firefox, LibreOffice) et scripts PowerShell.

**Prérequis** :

- **Ressources** : 2 Go RAM, 1 vCPU, 10 Go SSD.
- **Dépendances** : Python 3, MongoDB.

---

## 2. Installation

### 2.1. Créer un conteneur LXC

```bash
pct create 203 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.gz \
  --rootfs local-lvm:10G \
  --memory 2048 \
  --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --hostname wapt \
  --password motdepasse
```

### 2.2. Installer WAPT

1. **Installer les dépendances** :
  ```bash
   apt update && apt install -y python3 python3-pip mongodb
   pip3 install waptserver
  ```
si mongodb ne s'installe pas :
C’est normal sur Debian 12 : le paquet `mongodb` n’existe plus dans les dépôts officiels Debian à cause des problèmes de licence MongoDB.

Pour WAPT Server sur Debian 12, il faut généralement installer MongoDB depuis le dépôt officiel MongoDB ou utiliser la version supportée par WAPT.

Vérifie d’abord quelle version de WAPT tu installes, car les versions récentes utilisent parfois PostgreSQL au lieu de MongoDB.

Tu peux vérifier la doc officielle :
[Documentation WAPT](https://www.wapt.fr/fr/doc/?utm_source=chatgpt.com)

Si ton WAPT nécessite MongoDB, voici la méthode correcte pour Debian 12.

### 1. Ajouter la clé MongoDB

```bash
apt install -y gnupg curl

curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
--dearmor
```

### 2. Ajouter le dépôt MongoDB 7

```bash
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/7.0 main" \
> /etc/apt/sources.list.d/mongodb-org-7.0.list
```

### 3. Mettre à jour les dépôts

```bash
apt update
```

### 4. Installer MongoDB

```bash
apt install -y mongodb-org
```

### 5. Démarrer MongoDB

```bash
systemctl enable mongod
systemctl start mongod
```

### 6. Vérifier

```bash
systemctl status mongod
```

et :

```bash
mongosh
```

Ensuite tu pourras poursuivre l’installation de WAPT Server.

Le problème venait donc simplement du fait que :

```bash
apt install mongodb
```




2. **Configurer WAPT** :
  ```bash
   wapt-get setup --server-url=http://192.168.1.40/wapt
  ```
3. **Démarrer le service** :
  ```bash
   systemctl enable --now waptserver
  ```
4. **Accéder à l’interface** :
  - `http://192.168.1.40/wapt` (identifiants par défaut : `admin/wapt`).

---

## 3. Intégration avec Proxmox/pfSense/AD

- **Règles pfSense** :
  - Autoriser le trafic vers WAPT (`192.168.1.40`) sur les ports **80/443**.
- **Intégration avec AD** :
  - Dans WAPT : `Configuration > Active Directory` :
    - Domaine : `mon-domaine.local`.
    - Utilisateur : `admin@mon-domaine.local`.
    - Mot de passe : `MotDePasseAD`.

---

## 4. Utilisation courante

### 4.1. Ajouter un paquet

1. **Créer un paquet** :
  - `Packages > New` :
    - Nom : `firefox`, version : `115.0`.
    - Télécharger le `.msi` de Firefox et l’uploader.
2. **Déployer sur un poste** :
  - `Hosts > Sélectionner un poste > Deploy > firefox`.

### 4.2. Créer un groupe de postes

1. **Dans WAPT** :
  - `Hosts > Groups > New Group` (ex: "Comptabilité").
  - Ajouter les postes concernés.

---

## 5. Dépannage

### 5.1. Erreurs fréquentes

- **Les postes ne se connectent pas à WAPT** :
  - Vérifier que le service `wapt-service` est démarré sur les postes clients.
  - Vérifier la connectivité (`telnet 192.168.1.40 80`).
- **Erreur "Package not found"** :
  - Vérifier que le paquet est bien uploadé dans `/var/www/wapt`.

### 5.2. Logs utiles

- **Logs WAPT Server** :
  ```bash
  tail -f /var/log/wapt/waptserver.log
  ```
- **Logs client (sur un poste Windows)** :
  - `C:\Program Files (x86)\wapt\logs\waptconsole.log`.

---

## 6. Bonnes pratiques

- **Sécurité** :
  - Activer HTTPS pour l’interface WAPT (avec Let’s Encrypt).
  - Restreindre l’accès à l’interface via pfSense.
- **Optimisation** :
  - Utiliser les **stratégies de déploiement** pour étaler les mises à jour.

---

## 7. Références officielles

- [Documentation WAPT](https://wapt.fr/doc)
- [Guide d’intégration AD](https://wapt.fr/doc/ad-integration.html)
