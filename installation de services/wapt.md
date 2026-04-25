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
