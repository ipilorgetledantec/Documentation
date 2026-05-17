## Installation complète avec `sudo`

Pour un serveur musique type Spotify avec Navidrome sur Debian / Proxmox VE / Raspberry Pi

---

# 1. Mise à jour système

```bash
sudo apt update && sudo apt upgrade -y
```

---

# 2. Installer dépendances Docker

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

---

# 3. Ajouter clé Docker

```bash
sudo mkdir -p /etc/apt/keyrings
```

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

---

# 4. Ajouter dépôt Docker

```bash
echo \
"deb [arch=$(dpkg --print-architecture) \
signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/debian \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

# 5. Installer Docker

```bash
sudo apt update
```

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

---

# 6. Vérifier Docker

```bash
sudo docker --version
```

---

# 7. Créer dossiers musique

```bash
sudo mkdir -p /srv/music
```

```bash
sudo mkdir -p /srv/navidrome/data
```

---

# 8. Donner permissions utilisateur

Remplace `tonuser` par ton utilisateur Debian :

```bash
sudo chown -R tonuser:tonuser /srv/music
```

```bash
sudo chown -R tonuser:tonuser /srv/navidrome
```

---

# 9. Créer dossier projet

```bash
sudo mkdir -p /opt/navidrome
```

```bash
cd /opt/navidrome
```

---

# 10. Créer docker-compose.yml

```bash
sudo nano docker-compose.yml
```

Colle :

```yaml
services:
  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    user: 1000:1000
    ports:
      - "4533:4533"
    restart: unless-stopped

    environment:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info
      ND_SESSIONTIMEOUT: 24h
      ND_BASEURL: ""

    volumes:
      - /srv/navidrome/data:/data
      - /srv/music:/music:ro
```

Sauvegarder :

* `CTRL + O`
* Entrée
* `CTRL + X`

---

# 11. Lancer Navidrome

```bash
sudo docker compose up -d
```

---

# 12. Vérifier conteneur

```bash
sudo docker ps
```

Tu dois voir :

* `navidrome`
* port `4533`

---

# 13. Accéder au serveur musique

Depuis navigateur :

```text
http://IP_DE_TON_SERVEUR:4533
```

Exemple :

```text
http://192.168.1.50:4533
```

---

# 14. Ajouter tes musiques

Copie tes MP3/FLAC ici :

```bash
/srv/music
```

Exemple :

```bash
sudo cp -r ~/Musique/* /srv/music/
```

Puis rescanner :

```bash
sudo docker restart navidrome
```

---

# 15. Applications téléphone

## Android

* [Symfonium](https://symfonium.app/?utm_source=chatgpt.com)
* [Tempo Subsonic](https://play.google.com/store/apps/details?id=com.cappielloantonio.notquitemy.tempo&utm_source=chatgpt.com)

## iPhone

* [Amperfy](https://github.com/BLeeEZ/amperfy?utm_source=chatgpt.com)

---

# OPTION : téléchargement automatique musique

## Installer Lidarr + slskd

Créer compose avancé :

```bash
sudo nano docker-compose.yml
```

Remplace par :

```yaml
services:

  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    ports:
      - "4533:4533"
    restart: unless-stopped
    volumes:
      - /srv/navidrome/data:/data
      - /srv/music:/music

  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    container_name: lidarr
    ports:
      - "8686:8686"
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /srv/lidarr/config:/config
      - /srv/music:/music
      - /srv/downloads:/downloads
    restart: unless-stopped

  slskd:
    image: slskd/slskd:latest
    container_name: slskd
    ports:
      - "5030:5030"
    volumes:
      - /srv/slskd:/app
      - /srv/downloads:/downloads
    restart: unless-stopped
```

---

# Redémarrer stack

```bash
sudo docker compose down
```

```bash
sudo docker compose up -d
```

---

# Interfaces

| Service   | URL              |
| --------- | ---------------- |
| Navidrome | `http://IP:4533` |
| Lidarr    | `http://IP:8686` |
| slskd     | `http://IP:5030` |

---

# Commandes utiles

## Voir logs

```bash
sudo docker logs navidrome
```

---

## Redémarrer

```bash
sudo docker restart navidrome
```

---

## Stopper

```bash
sudo docker stop navidrome
```

---

## Mettre à jour

```bash
sudo docker compose pull
```

```bash
sudo docker compose up -d
```

---

# Bonus recommandé

## Ajouter interface HTTPS

Installer :

* [Nginx Proxy Manager](https://nginxproxymanager.com/?utm_source=chatgpt.com)

Tu pourras avoir :

```text
https://music.tondomaine.com
```

avec certificat SSL automatique.
# 1. Vérifier le disque USB branché

Voir les disques :

```bash
sudo lsblk
```

Tu verras un truc du genre :

```text
sda
├─sda1
sdb
└─sdb1
```

Le HDD USB sera souvent :

* `sdb`
* `sdb1`

---

# 2. Voir le système de fichiers

```bash
sudo blkid
```

Exemple :

```text
/dev/sdb1: UUID="XXXX" TYPE="ext4"
```

---

# 3. Créer point de montage

Exemple :

```bash
sudo mkdir -p /mnt/musicdisk
```

---

# 4. Monter le disque

## Si EXT4

```bash
sudo mount /dev/sdb1 /mnt/musicdisk
```

---

## Si NTFS (disque Windows)

Installer support :

```bash
sudo apt install -y ntfs-3g
```

Puis :

```bash
sudo mount -t ntfs-3g /dev/sdb1 /mnt/musicdisk
```

---

# 5. Vérifier

```bash
df -h
```

Tu dois voir :

```text
/dev/sdb1   xxxG   ...   /mnt/musicdisk
```

---

# 6. Permissions

Donner accès à ton utilisateur :

Remplace `tonuser`

```bash
sudo chown -R tonuser:tonuser /mnt/musicdisk
```

---

# 7. Utiliser le HDD avec Navidrome

Tu peux soit :

* déplacer ta musique
* soit monter directement le disque dans Docker.

---

# Solution recommandée

## Modifier docker-compose.yml

```bash
cd /opt/navidrome
sudo nano docker-compose.yml
```

Remplace :

```yaml
- /srv/music:/music
```

par :

```yaml
- /mnt/musicdisk:/music
```

---

# Exemple complet

```yaml
services:

  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    ports:
      - "4533:4533"
    restart: unless-stopped

    volumes:
      - /srv/navidrome/data:/data
      - /mnt/musicdisk:/music
```

---

# 8. Redémarrer Docker

```bash
sudo docker compose down
```

```bash
sudo docker compose up -d
```

---

# 9. Rescanner bibliothèque

```bash
sudo docker restart navidrome
```

---

# Montage automatique au démarrage

Très important sinon le disque disparaît après reboot.

## Récupérer UUID

```bash
sudo blkid
```

Exemple :

```text
/dev/sdb1: UUID="1234-ABCD"
```

---

## Modifier fstab

```bash
sudo nano /etc/fstab
```

Ajouter :

## EXT4

```text
UUID=1234-ABCD /mnt/musicdisk ext4 defaults,nofail 0 2
```

---

## NTFS

```text
UUID=1234-ABCD /mnt/musicdisk ntfs-3g defaults,nofail,uid=1000,gid=1000 0 0
```

---

# Tester sans reboot

```bash
sudo mount -a
```

Si aucune erreur :
OK.

---

# SLSKD : comment se connecter

## Interface web

Depuis navigateur :

```text
http://IP_DU_SERVEUR:5030
```

Exemple :

```text
http://192.168.1.50:5030
```

---

# Identifiants par défaut

Selon version :

| Login | Password |
| ----- | -------- |
| slskd | slskd    |

ou parfois :

* admin/admin

---

# Si ça ne marche pas

Voir les logs :

```bash
sudo docker logs slskd
```

Tu verras les credentials générés.

---

# Changer mot de passe SLSKD

Créer config persistante :

```bash
sudo mkdir -p /srv/slskd
```

Puis éditer :

```bash
sudo nano /srv/slskd/slskd.yml
```

Ajouter :

```yaml
web:
  username: admin
  password: motdepassefort
```

---

# Redémarrer

```bash
sudo docker restart slskd
```

---

# Ajouter dossier téléchargement

Dans slskd :

* Settings
* Downloads

Mettre :

```text
/downloads
```

Comme dans ton compose Docker.

---

# Ajouter compte Soulseek

Tu dois avoir un compte :

* [Soulseek](https://www.slsknet.org/?utm_source=chatgpt.com)

Dans slskd :

* Settings
* Soulseek
* Username/password

---

# Flux complet recommandé

```text
slskd
↓
downloads
↓
Lidarr trie automatiquement
↓
Navidrome détecte
↓
Spotify maison
```

---

# Structure idéale

```text
/mnt/musicdisk/
├── Music/
├── Downloads/
├── Lidarr/
└── Navidrome/
```

---

# Conseil important

Pour Raspberry Pi :

* utiliser alimentation correcte
* préférer HDD avec alimentation externe
* idéalement SSD USB 3.0.
