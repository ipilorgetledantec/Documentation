Voici un **guide complet et détaillé** pour installer **Jellyfin** et **Mopidy + Iris** sur **Proxmox en mode graphique**, avec :
✅ **Deux conteneurs LXC Debian 12** (un pour Jellyfin, un pour Mopidy).
✅ **Multi-room audio** avec **Snapcast** (intégré à Mopidy).
✅ **Accès à distance sécurisé en HTTPS** (via **Nginx Proxy Manager**).
✅ **Pas de ligne de commande** pour la création des conteneurs (tout en interface graphique Proxmox).
✅ **Stockage partagé** entre les conteneurs (pour les médias).

---

---

---

## **📌 Prérequis**
1. **Proxmox VE** installé et à jour (version 7.x ou 8.x).
2. **Un pool de stockage** configuré (ex: `local-lvm`, `ZFS`, ou un NAS monté en NFS/SMB).
3. **Un nom de domaine** (ex: `ton-domaine.com`) avec un **enregistrement DNS** pointant vers ton IP publique.
   - Si tu n’as pas d’IP fixe, utilise un service comme **DuckDNS** ou **No-IP**.
4. **Un certificat SSL** (nous utiliserons **Let’s Encrypt** via Nginx Proxy Manager).
5. **Matériel audio** :
   - Pour le **multi-room**, tu auras besoin d’au moins **2 appareils** (ex: Raspberry Pi, PC, ou enceintes connectées) pour tester Snapcast.

---

---

---

## **🛠️ Étape 1 : Préparer l’environnement Proxmox**
### **1.1. Créer un dossier pour les médias (si ce n’est pas déjà fait)**
- **Objectif** : Centraliser tes fichiers (films, musique) dans un **dossier accessible par les deux conteneurs**.
- **Où ?** :
  - Soit sur le **stockage local de Proxmox** (ex: `/mnt/pve/nas`).
  - Soit sur un **NAS externe** monté en NFS/SMB (recommandé pour la flexibilité).

#### **Si tu utilises un stockage local :**
1. Dans l’interface Proxmox, va dans **Datacenter > Storage**.
2. Clique sur **Add > Directory**.
   - **ID** : `nas` (ou un nom de ton choix).
   - **Directory** : `/mnt/pve/nas` (ou `/srv/nas` si tu préfères).
   - **Content** : Sélectionne **Disk image, Container, Backup** (pour permettre les bind mounts).
3. Clique sur **OK**.

#### **Si tu utilises un NAS externe (ex: Synology, TrueNAS) :**
1. Monte ton partage NFS/SMB dans Proxmox :
   - Va dans **Datacenter > Storage > Add > NFS** ou **SMB/CIFS**.
   - Renseigne l’**IP du NAS**, le **chemin du partage** (ex: `/volume1/media`), et les identifiants si nécessaire.
   - **ID** : `nas` (pour simplifier).
2. Clique sur **OK**.

---
### **1.2. Créer les dossiers pour les médias et les configs**
1. **Sur le serveur Proxmox (en SSH ou via l’interface web)** :
   - Crée les dossiers suivants dans ton stockage `nas` :
     ```bash
     mkdir -p /mnt/pve/nas/media/{films,musique}
     mkdir -p /mnt/pve/nas/config/{jellyfin,mopidy,nginx}
     ```
   - **Explications** :
     - `/mnt/pve/nas/media/films` : Pour tes films (Jellyfin).
     - `/mnt/pve/nas/media/musique` : Pour ta musique (Mopidy et Jellyfin).
     - `/mnt/pve/nas/config/jellyfin` : Configuration de Jellyfin.
     - `/mnt/pve/nas/config/mopidy` : Configuration de Mopidy.
     - `/mnt/pve/nas/config/nginx` : Configuration de Nginx Proxy Manager.

2. **Donne les permissions** (en SSH) :
   ```bash
   chmod -R 775 /mnt/pve/nas/media
   chmod -R 775 /mnt/pve/nas/config
   ```

---
---
---

## **🖥️ Étape 2 : Créer les conteneurs LXC (Debian 12) en mode graphique**
### **2.1. Télécharger le template Debian 12**
1. Dans l’interface Proxmox, va dans **Datacenter > Storage > local (ou ton stockage)**.
2. Clique sur **Content**.
3. Si le template **Debian 12 (Bookworm)** n’est pas présent, clique sur **Templates** et télécharge-le :
   - Sélectionne **debian-12-standard_12.0-1_amd64.tar.gz** (ou la dernière version).
   - Clique sur **Download**.

---
### **2.2. Créer le conteneur LXC pour Jellyfin**
1. Dans l’interface Proxmox, clique sur **Create CT** (en haut à droite).
2. **Paramètres de base** :
   - **Node** : Sélectionne ton nœud Proxmox.
   - **CT ID** : `100` (ou un autre numéro disponible).
   - **Hostname** : `jellyfin`.
   - **Password** : Définis un mot de passe root (ou utilise une clé SSH).
   - **Unprivileged container** : **Désactivé** (pour permettre les bind mounts).
3. **Template** :
   - Sélectionne **debian-12-standard_12.0-1_amd64**.
4. **Disk** :
   - **Storage** : `local-lvm` (ou ton stockage principal).
   - **Size** : `8 GiB` (suffisant pour le système, les médias seront montés depuis `nas`).
5. **CPU** :
   - **Cores** : `4` (pour le transcodage).
   - **Type** : `host` (pour de meilleures performances).
6. **Memory** :
   - **RAM** : `4096 MiB` (4 Go, tu peux augmenter si tu as beaucoup de flux simultanés).
   - **Swap** : `1024 MiB` (1 Go).
7. **Network** :
   - **Bridge** : `vmbr0` (par défaut).
   - **IPv4** : **DHCP** ou **Statique** (ex: `192.168.1.100`).
     - Si tu choisis **Statique**, renseigne :
       - **IP** : `192.168.1.100` (à adapter à ton réseau).
       - **Gateway** : `192.168.1.1` (ta box).
       - **DNS** : `8.8.8.8,8.8.4.4` (Google DNS) ou `1.1.1.1` (Cloudflare).
8. **DNS** :
   - **Domain** : `local` (ou ton domaine local si tu en as un).
9. **Confirmation** :
   - Clique sur **Finish**.

---
### **2.3. Créer le conteneur LXC pour Mopidy + Iris + Snapcast**
1. Clique à nouveau sur **Create CT**.
2. **Paramètres de base** :
   - **CT ID** : `101`.
   - **Hostname** : `mopidy`.
   - **Password** : Même mot de passe que pour Jellyfin (ou un autre).
   - **Unprivileged container** : **Désactivé**.
3. **Template** :
   - Sélectionne **debian-12-standard_12.0-1_amd64**.
4. **Disk** :
   - **Storage** : `local-lvm`.
   - **Size** : `4 GiB` (Mopidy est léger).
5. **CPU** :
   - **Cores** : `2`.
6. **Memory** :
   - **RAM** : `1024 MiB` (1 Go, suffisant pour Mopidy + Snapcast).
   - **Swap** : `512 MiB`.
7. **Network** :
   - **Bridge** : `vmbr0`.
   - **IPv4** : **Statique** (ex: `192.168.1.101`).
     - **IP** : `192.168.1.101`.
     - **Gateway** : `192.168.1.1`.
     - **DNS** : `8.8.8.8,8.8.4.4`.
8. **DNS** :
   - **Domain** : `local`.
9. **Confirmation** :
   - Clique sur **Finish**.

---
### **2.4. Créer le conteneur LXC pour Nginx Proxy Manager**
1. Clique sur **Create CT**.
2. **Paramètres de base** :
   - **CT ID** : `102`.
   - **Hostname** : `nginx-proxy`.
   - **Password** : Définis un mot de passe.
   - **Unprivileged container** : **Désactivé**.
3. **Template** :
   - Sélectionne **debian-12-standard_12.0-1_amd64**.
4. **Disk** :
   - **Storage** : `local-lvm`.
   - **Size** : `4 GiB`.
5. **CPU** :
   - **Cores** : `1`.
6. **Memory** :
   - **RAM** : `512 MiB`.
   - **Swap** : `256 MiB`.
7. **Network** :
   - **Bridge** : `vmbr0`.
   - **IPv4** : **Statique** (ex: `192.168.1.102`).
     - **IP** : `192.168.1.102`.
     - **Gateway** : `192.168.1.1`.
     - **DNS** : `8.8.8.8,8.8.4.4`.
8. **DNS** :
   - **Domain** : `local`.
9. **Confirmation** :
   - Clique sur **Finish**.

---
---
---

## **📁 Étape 3 : Configurer les bind mounts (montages des dossiers)**
### **3.1. Monter les dossiers dans le conteneur Jellyfin (CT 100)**
1. Dans l’interface Proxmox, sélectionne le conteneur **jellyfin (100)**.
2. Clique sur **Options >ressources**.
3. Clique sur **Add**.
   - **Storage** : Sélectionne `nas` (ou ton stockage).
   - **Source** : `/mnt/pve/nas/media` (dossier source sur Proxmox).
   - **Destination** : `/srv/media` (dossier de destination dans le conteneur).
   - **Options** : `rw,noatime` (pour optimiser les performances).
4. Clique sur **OK**.
5. **Répète l’opération** pour monter :
   - **Source** : `/mnt/pve/nas/config/jellyfin` → **Destination** : `/etc/jellyfin`.

---
### **3.2. Monter les dossiers dans le conteneur Mopidy (CT 101)**
1. Sélectionne le conteneur **mopidy (101)**.
2. Clique sur **Options > Mount Points**.
3. Ajoute les montages suivants :
   - **Source** : `/mnt/pve/nas/media/musique` → **Destination** : `/srv/musique`.
   - **Source** : `/mnt/pve/nas/config/mopidy` → **Destination** : `/etc/mopidy`.
4. Clique sur **OK** pour chaque montage.

---
### **3.3. Monter les dossiers dans le conteneur Nginx Proxy Manager (CT 102)**
1. Sélectionne le conteneur **nginx-proxy (102)**.
2. Clique sur **Options > Mount Points**.
3. Ajoute le montage :
   - **Source** : `/mnt/pve/nas/config/nginx` → **Destination** : `/etc/nginx/proxy`.
4. Clique sur **OK**.

---
### **3.4. Redémarrer les conteneurs**
1. Sélectionne chaque conteneur (100, 101, 102).
2. Clique sur **More > Restart**.

---
---
---

## **🚀 Étape 4 : Installer et configurer Jellyfin (CT 100)**
### **4.1. Accéder au conteneur Jellyfin en console**
1. Sélectionne le conteneur **jellyfin (100)**.
2. Clique sur **Console** (en haut).
3. Connecte-toi avec :
   - **Login** : `root`.
   - **Password** : celui que tu as défini à la création.

---
### **4.2. Mettre à jour le système et installer Jellyfin**
Exécute les commandes suivantes **une par une** :
```bash
apt update && apt upgrade -y
```
```bash
apt install -y curl gnupg2
```
```bash
curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg
```
```bash
echo "deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/debian bookworm main" | tee /etc/apt/sources.list.d/jellyfin.list
```
```bash
apt update
```
```bash
apt install -y jellyfin
```
```bash
systemctl enable --now jellyfin
```

---
### **4.3. Configurer Jellyfin**
1. **Accéder à l’interface web** :
   - Ouvre un navigateur et va sur `http://192.168.1.100:8096`.
   - **Langue** : Sélectionne **Français**.
   - **Utilisateur** : Crée un compte admin (ex: `ivan`, mot de passe sécurisé).
2. **Ajouter une bibliothèque** :
   - Clique sur **+ Ajouter une bibliothèque**.
   - **Type** : Sélectionne **Films** ou **Musique**.
   - **Dossier** : `/srv/media/films` (ou `/srv/media/musique`).
   - **Agent de métadonnées** : **The Movie Database** (pour les films) ou **MusicBrainz** (pour la musique).
   - Clique sur **OK**.
3. **Activer le transcodage matériel (si GPU disponible)** :
   - Va dans **Tableau de bord > Paramètres > Lecture**.
   - **Transcodage matériel** : Sélectionne **VA-API** (Intel) ou **AMF** (AMD).
   - **Dossier temporaire** : `/tmp` (ou `/srv/media/transcode` si tu veux un dossier dédié).
   - Clique sur **Enregistrer**.

---
---
---

## **🎵 Étape 5 : Installer et configurer Mopidy + Iris + Snapcast (CT 101)**
### **5.1. Accéder au conteneur Mopidy en console**
1. Sélectionne le conteneur **mopidy (101)**.
2. Clique sur **Console**.
3. Connecte-toi avec `root` et ton mot de passe.

---
### **5.2. Installer les dépendances et Mopidy**
Exécute les commandes suivantes :
```bash
apt update && apt upgrade -y
```
```bash
apt install -y python3-pip python3-dev libssl-dev libfftw3-dev libasound2-dev
```
```bash
pip install --upgrade pip
```
```bash
pip install mopidy mopidy-iris mopidy-spotify mopidy-deezer mopidy-snapcast
```
```bash
# Installer les dépendances pour Snapcast
apt install -y snapserver snapclient
```

---
### **5.3. Configurer Mopidy**
1. **Créer le fichier de configuration** :
   ```bash
   nano /etc/mopidy/mopidy.conf
   ```
2. **Coller la configuration suivante** (adapte les valeurs entre `< >`) :
   ```ini
   [core]
   config_dir = /etc/mopidy
   data_dir = /var/lib/mopidy

   [audio]
   mixer = software
   output = alsasink device=default  # Pour une sortie audio standard
   # Pour une sortie bit-perfect (si tu as un DAC USB) :
   # output = alsasink device=hw:1,0  # À adapter selon ton périphérique (voir `aplay -l`)

   [mpd]
   enabled = true
   hostname = 0.0.0.0
   port = 6600

   [http]
   enabled = true
   hostname = 0.0.0.0
   port = 6680
   static_dir = /etc/mopidy/iris

   [iris]
   enabled = true
   country = FR
   locale = fr_FR

   [spotify]
   enabled = true
   username = <ton_email_spotify>
   password = <ton_mot_de_passe_spotify>
   client_id = <ton_client_id_spotify>  # À créer sur https://developer.spotify.com/dashboard/applications
   client_secret = <ton_client_secret_spotify>

   [deezer]
   enabled = true
   auth_token = <ton_token_deezer>  # À générer via l'API Deezer (voir ci-dessous)

   [snapcast]
   enabled = true
   host = 0.0.0.0
   port = 1704
   soundcard = default
   ```
   - **Pour obtenir un `client_id` et `client_secret` Spotify** :
     1. Va sur [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/applications).
     2. Crée une nouvelle application.
     3. Dans **Redirect URIs**, ajoute `http://localhost:6680/iris/callback`.
     4. Copie le `Client ID` et `Client Secret` dans la config.

   - **Pour obtenir un `auth_token` Deezer** :
     1. Installe `deezer-python` :
        ```bash
        pip install deezer-python
        ```
     2. Génère un token (en console) :
        ```bash
        python3 -c "from deezer import Client; c = Client(); print(c.get_user('me')['id'])"  # Se connecte via le navigateur
        ```
        (Le token sera stocké dans `~/.config/deezer-python/token.json`. Copie-le dans la config.)

3. **Enregistrer et quitter** (`Ctrl + X`, puis `Y`, puis `Entrée`).

---
### **5.4. Configurer Snapcast**
1. **Éditer le fichier de config de Snapserver** :
   ```bash
   nano /etc/default/snapserver
   ```
2. **Ajouter/modifier** :
   ```ini
   SNAPSERVER_OPTS="--host 0.0.0.0 --port 1704 --soundcard default"
   ```
   - Pour lister tes cartes audio : `aplay -l` (et adapte `soundcard` si nécessaire).

3. **Redémarrer Snapserver** :
   ```bash
   systemctl enable --now snapserver
   ```

---
### **5.5. Lancer Mopidy et Iris**
1. **Lancer Mopidy** :
   ```bash
   mopidy --config /etc/mopidy/mopidy.conf
   ```
   - Pour le lancer en arrière-plan (et au démarrage) :
     ```bash
     nano /etc/systemd/system/mopidy.service
     ```
     Coller :
     ```ini
     [Unit]
     Description=Mopidy music server
     After=network.target

     [Service]
     ExecStart=/usr/local/bin/mopidy --config /etc/mopidy/mopidy.conf
     User=root
     Group=root
     Restart=on-failure

     [Install]
     WantedBy=multi-user.target
     ```
     Puis :
     ```bash
     systemctl enable --now mopidy
     ```

2. **Accéder à Iris** :
   - Ouvre un navigateur et va sur `http://192.168.1.101:6680/iris`.
   - **Premier lancement** : Iris va te guider pour configurer Spotify/Deezer.

---
### **5.6. Configurer le multi-room avec Snapcast**
1. **Sur un autre appareil (ex: Raspberry Pi)** :
   - Installe **Snapclient** :
     ```bash
     sudo apt update && sudo apt install -y snapclient
     ```
   - Lance le client :
     ```bash
     snapclient --host 192.168.1.101 --port 1704 --soundcard default
     ```
     - Remplace `soundcard` par le nom de ta carte audio (`aplay -l`).

2. **Dans l’interface Iris** :
   - Va dans **Settings > Snapcast** pour gérer les zones audio.

---
---
---

## **🌐 Étape 6 : Installer et configurer Nginx Proxy Manager (CT 102)**
### **6.1. Accéder au conteneur Nginx Proxy Manager en console**
1. Sélectionne le conteneur **nginx-proxy (102)**.
2. Clique sur **Console**.
3. Connecte-toi avec `root` et ton mot de passe.

---
### **6.2. Installer Docker (nécessaire pour Nginx Proxy Manager)**
Exécute les commandes suivantes :
```bash
apt update && apt install -y curl gnupg2 ca-certificates
```
```bash
install -m 0755 -d /etc/apt/keyrings
```
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.asc
```
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian bookworm stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```bash
apt update
```
```bash
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
```bash
systemctl enable --now docker
```

---
### **6.3. Installer Nginx Proxy Manager**
```bash
docker run -d \
  --name=nginx-proxy-manager \
  -p 80:80 \
  -p 443:443 \
  -p 81:81 \
  -v /etc/nginx/proxy/data:/data \
  -v /etc/nginx/proxy/letsencrypt:/etc/letsencrypt \
  --restart=unless-stopped \
  jc21/nginx-proxy-manager:latest
```

---
### **6.4. Configurer Nginx Proxy Manager**
1. **Accéder à l’interface web** :
   - Ouvre un navigateur et va sur `http://192.168.1.102:81`.
   - **Identifiants par défaut** :
     - **Email** : `admin@example.com`
     - **Mot de passe** : `changeme`
   - **Change le mot de passe** à la première connexion.

2. **Ajouter un proxy pour Jellyfin** :
   - Clique sur **Hosts > Add Proxy Host**.
   - **Domain Names** : `jellyfin.tondomaine.com` (ou `jellyfin.ton-ip-dynamique.duckdns.org` si tu utilises DuckDNS).
   - **Scheme** : `http`.
   - **Forward Hostname/IP** : `192.168.1.100` (IP du conteneur Jellyfin).
   - **Forward Port** : `8096`.
   - **SSL** : Active **Request a new SSL Certificate**.
     - **Email** : Ton email (pour Let’s Encrypt).
     - **Agree to Let’s Encrypt Terms** : ✅ Coché.
   - Clique sur **Save**.

3. **Ajouter un proxy pour Mopidy (Iris)** :
   - Clique sur **Hosts > Add Proxy Host**.
   - **Domain Names** : `mopidy.tondomaine.com`.
   - **Scheme** : `http`.
   - **Forward Hostname/IP** : `192.168.1.101`.
   - **Forward Port** : `6680`.
   - **SSL** : Active **Request a new SSL Certificate** (même email).
   - Clique sur **Save**.

4. **Ajouter un proxy pour Snapcast (optionnel)** :
   - Si tu veux accéder à l’interface de Snapserver :
     - **Domain Names** : `snapcast.tondomaine.com`.
     - **Forward Hostname/IP** : `192.168.1.101`.
     - **Forward Port** : `1780` (port web de Snapserver).
     - **SSL** : Active le certificat.

---
### **6.5. Configurer le DNS (si tu utilises un nom de domaine)**
1. **Sur ton registrar (OVH, Gandi, Cloudflare, etc.)** :
   - Ajoute un **enregistrement A** pour :
     - `jellyfin.tondomaine.com` → `192.168.1.102` (IP de ton conteneur Nginx Proxy Manager).
     - `mopidy.tondomaine.com` → `192.168.1.102`.
     - `snapcast.tondomaine.com` → `192.168.1.102`.
   - Si tu utilises **DuckDNS** :
     - Va sur [DuckDNS](https://www.duckdns.org/) et ajoute les sous-domaines `jellyfin`, `mopidy`, `snapcast`.

2. **Si tu n’as pas d’IP fixe** :
   - Installe **DuckDNS** sur Proxmox (ou sur un Raspberry Pi) :
     ```bash
     docker run -d --name=duckdns \
       -e PUID=0 -e PGID=0 \
       -e TZ=Europe/Paris \
       -e LOG_FILE=false \
       -e TOKEN=ton_token_duckdns \
       -e DOMAINS=jellyfin,mopidy,snapcast \
       linuxserver/duckdns
     ```
     - Récupère ton **token** sur [DuckDNS](https://www.duckdns.org/).

---
### **6.6. Tester l’accès en HTTPS**
1. Attends **5-10 minutes** pour que Let’s Encrypt génère les certificats.
2. Ouvre un navigateur et va sur :
   - `https://jellyfin.tondomaine.com`
   - `https://mopidy.tondomaine.com/iris`
   - `https://snapcast.tondomaine.com` (si configuré).
3. **Vérifie** :
   - Le cadenas 🔒 doit apparaître dans la barre d’adresse.
   - Pas d’avertissement de sécurité.

---
---
---

## **🔒 Étape 7 : Sécuriser l’accès (optionnel mais recommandé)**
### **7.1. Configurer un pare-feu (UFW) sur Proxmox**
1. **Sur le nœud Proxmox (en SSH)** :
   ```bash
   apt install -y ufw
   ufw default deny incoming
   ufw default allow outgoing
   ufw allow 22/tcp  # SSH
   ufw allow 80/tcp  # HTTP (redirigé vers HTTPS)
   ufw allow 443/tcp # HTTPS
   ufw enable
   ```

---
### **7.2. Configurer un VPN (WireGuard) pour un accès sécurisé**
Si tu veux **éviter d’exposer tes services sur Internet**, tu peux utiliser **WireGuard** pour accéder à ton réseau local depuis l’extérieur.

1. **Installer WireGuard sur Proxmox** :
   ```bash
   apt install -y wireguard
   ```
2. **Générer les clés** :
   ```bash
   wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
   ```
3. **Créer le fichier de config** (`/etc/wireguard/wg0.conf`) :
   ```ini
   [Interface]
   Address = 10.0.0.1/24
   ListenPort = 51820
   PrivateKey = <contenu_de_/etc/wireguard/privatekey>

   [Peer]
   PublicKey = <clé_publique_du_client>
   AllowedIPs = 10.0.0.2/32
   ```
4. **Démarrer WireGuard** :
   ```bash
   systemctl enable --now wg-quick@wg0
   ```
5. **Configurer un client (ex: smartphone)** :
   - Utilise l’app **WireGuard** et importe la config suivante :
     ```ini
     [Interface]
     PrivateKey = <clé_privée_du_client>
     Address = 10.0.0.2/24
     DNS = 8.8.8.8

     [Peer]
     PublicKey = <contenu_de_/etc/wireguard/publickey>
     Endpoint = ton-ip-publique:51820
     AllowedIPs = 192.168.1.0/24, 10.0.0.0/24
     PersistentKeepalive = 25
     ```
6. **Ouvrir le port WireGuard** :
   ```bash
   ufw allow 51820/udp
   ```

---
---
---

## **🎉 Étape 8 : Tester et finaliser**
### **8.1. Tester Jellyfin**
1. Va sur `https://jellyfin.tondomaine.com`.
2. **Ajoute des médias** :
   - Place des films dans `/mnt/pve/nas/media/films` (via SMB/NFS ou en SSH).
   - Jellyfin les détectera automatiquement.
3. **Teste la lecture** :
   - Sur un PC, une tablette, ou un smartphone.

---
### **8.2. Tester Mopidy + Iris**
1. Va sur `https://mopidy.tondomaine.com/iris`.
2. **Connecte-toi à Spotify/Deezer** :
   - Suis les instructions pour autoriser l’accès.
3. **Teste la lecture** :
   - Lance une playlist et vérifie que le son sort bien.
4. **Teste le multi-room** :
   - Lance Snapcast sur un autre appareil et vérifie que le son est synchronisé.

---
### **8.3. Tester Snapcast**
1. Va sur `https://snapcast.tondomaine.com` (si configuré).
2. **Vérifie les zones audio** :
   - Tu devrais voir tes appareils connectés (ex: `Mopidy`, `Raspberry Pi`).

---
---
---

## **📝 Résumé des URLs et accès**
| **Service**       | **URL**                          | **Port interne** | **Conteneur** | **IP locale**     |
|-------------------|----------------------------------|------------------|---------------|-------------------|
| Jellyfin          | `https://jellyfin.tondomaine.com` | 8096             | 100           | 192.168.1.100     |
| Mopidy (Iris)     | `https://mopidy.tondomaine.com`  | 6680             | 101           | 192.168.1.101     |
| Snapcast          | `https://snapcast.tondomaine.com`| 1780             | 101           | 192.168.1.101     |
| Nginx Proxy Manager | `http://192.168.1.102:81`       | 81               | 102           | 192.168.1.102     |

---
---
---

## **🔧 Dépannage**
### **Problème : Jellyfin ne détecte pas les médias**
- **Vérifie les permissions** :
  ```bash
  chmod -R 775 /mnt/pve/nas/media
  chown -R 1000:1000 /mnt/pve/nas/media  # Si Jellyfin tourne avec l'utilisateur `jellyfin` (UID 1000)
  ```
- **Vérifie le montage** :
  Dans le conteneur Jellyfin, exécute :
  ```bash
  ls /srv/media
  ```
  (Doit lister tes fichiers.)

---
### **Problème : Mopidy ne se connecte pas à Spotify/Deezer**
- **Vérifie les logs** :
  ```bash
  journalctl -u mopidy -f
  ```
- **Problème de token** :
  - Pour Spotify, assure-toi que le `client_id` et `client_secret` sont corrects.
  - Pour Deezer, régénère le token avec :
    ```bash
    python3 -c "from deezer import Client; c = Client(); print(c.get_user('me')['id'])"
    ```

---
### **Problème : Snapcast ne fonctionne pas**
- **Vérifie que Snapserver tourne** :
  ```bash
  systemctl status snapserver
  ```
- **Vérifie les logs** :
  ```bash
  journalctl -u snapserver -f
  ```
- **Teste la connexion** :
  ```bash
  snapclient --host 192.168.1.101 --soundcard default
  ```

---
### **Problème : HTTPS ne fonctionne pas**
- **Vérifie que les ports 80 et 443 sont ouverts** :
  ```bash
  ufw status
  ```
- **Vérifie que Nginx Proxy Manager a bien généré les certificats** :
  - Dans l’interface Nginx Proxy Manager, va dans **SSL Certificates**.
  - Vérifie que les certificats pour `jellyfin.tondomaine.com` et `mopidy.tondomaine.com` sont **valides**.
- **Teste avec `curl`** :
  ```bash
  curl -v https://jellyfin.tondomaine.com
  ```
  (Doit retourner le code HTML de Jellyfin sans erreur SSL.)

---
---
---

## **📌 Conseils supplémentaires**
1. **Sauvegardes** :
   - Sauvegarde régulièrement :
     - `/mnt/pve/nas/config/jellyfin` (config Jellyfin).
     - `/mnt/pve/nas/config/mopidy` (config Mopidy).
     - `/etc/nginx/proxy` (config Nginx Proxy Manager).
   - Utilise **rsync** ou **BorgBackup** pour automatiser les sauvegardes.

2. **Mises à jour** :
   - **Jellyfin** :
     ```bash
     apt update && apt upgrade -y
     ```
   - **Mopidy** :
     ```bash
     pip install --upgrade mopidy mopidy-iris mopidy-spotify mopidy-deezer
     ```
   - **Nginx Proxy Manager** :
     ```bash
     docker pull jc21/nginx-proxy-manager:latest
     docker stop nginx-proxy-manager
     docker rm nginx-proxy-manager
     docker run -d ...  # (même commande que précédemment)
     ```

3. **Optimisations** :
   - **Pour Jellyfin** :
     - Active le **transcodage matériel** (VA-API/AMF) si ton CPU le supporte.
     - Désactive le transcodage pour les appareils compatibles (ex: TV 4K).
   - **Pour Mopidy** :
     - Utilise un **DAC USB** pour une qualité audio optimale.
     - Configure **Snapcast** pour synchroniser plusieurs enceintes.

4. **Multi-room avancé** :
   - Ajoute des **Raspberry Pi** avec **Snapclient** dans chaque pièce.
   - Utilise des **enceintes USB** ou **HDMI** pour la sortie audio.

---
---
---
## **🎯 Résumé des commandes utiles**
| **Action**                          | **Commande**                                                                 |
|-------------------------------------|------------------------------------------------------------------------------|
| Redémarrer Jellyfin                 | `systemctl restart jellyfin` (dans le conteneur 100)                       |
| Redémarrer Mopidy                   | `systemctl restart mopidy` (dans le conteneur 101)                         |
| Redémarrer Nginx Proxy Manager      | `docker restart nginx-proxy-manager` (dans le conteneur 102)               |
| Voir les logs Jellyfin              | `journalctl -u jellyfin -f` (dans le conteneur 100)                        |
| Voir les logs Mopidy                | `journalctl -u mopidy -f` (dans le conteneur 101)                          |
| Voir les logs Nginx Proxy Manager   | `docker logs -f nginx-proxy-manager` (dans le conteneur 102)               |
| Lister les conteneurs LXC           | `pct list` (sur le nœud Proxmox)                                             |
| Accéder à un conteneur LXC          | `pct enter 100` (pour Jellyfin)                                             |
| Vérifier les montages               | `mount` (dans un conteneur)                                                 |
| Vérifier l’espace disque            | `df -h` (dans un conteneur)                                                 |

---
---
---
## **🔥 Prochaines étapes (optionnelles)**
1. **Ajouter un stockage NAS** :
   - Monte un partage NFS/SMB depuis ton NAS dans Proxmox (`Datacenter > Storage > Add > NFS/SMB`).
   - Modifie les bind mounts pour pointer vers ce stockage.

2. **Configurer un backup automatique** :
   - Utilise **rsync** pour sauvegarder `/mnt/pve/nas` vers un autre disque ou un cloud (ex: Backblaze B2).

3. **Ajouter un client Kodi** :
   - Installe Kodi sur une TV ou un Raspberry Pi.
   - Ajoute le **plugin Jellyfin pour Kodi** pour connecter Kodi à ton serveur Jellyfin.

4. **Configurer un DNS dynamique** :
   - Si tu n’as pas d’IP fixe, utilise **DuckDNS** ou **No-IP** avec un script de mise à jour.

5. **Optimiser le réseau** :
   - Configure **QoS** sur ton routeur pour prioriser le trafic audio/vidéo.

---
---
---
## **❓ Besoin d’aide ?**
Si tu bloques sur une étape, dis-moi :
- **Quel conteneur pose problème** (Jellyfin, Mopidy, Nginx Proxy Manager) ?
- **Quel est le message d’erreur** (si applicable) ?
- **As-tu accès à l’interface web** de Nginx Proxy Manager/Jellyfin/Iris ?

**Je peux te guider pas à pas pour résoudre le problème !** 🚀
