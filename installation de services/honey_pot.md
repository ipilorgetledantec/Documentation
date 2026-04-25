# Honeypot (Cowrie) - Guide d'installation et d'utilisation

---

## 1. Présentation

**Fonction** :

- Simuler un système vulnérable (SSH/Telnet) pour attirer et analyser les attaques.
- **Cowrie** : Honeypot SSH/Telnet léger, idéal pour la DMZ.

**Prérequis** :

- **Ressources** : 1 Go RAM, 1 vCPU, 5 Go SSD.
- **Réseau** : Placer le conteneur dans la DMZ (`vmbr1`).

---

## 2. Installation

### 2.1. Créer un conteneur LXC dans la DMZ

```bash
pct create 204 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.gz \
  --rootfs local-lvm:5G \
  --memory 1024 \
  --cores 1 \
  --net0 name=eth0,bridge=vmbr1,ip=dhcp \  # Réseau DMZ
  --hostname cowrie \
  --password motdepasse
```

### 2.2. Installer Cowrie

1. **Installer les dépendances** :
  ```bash
   apt update && apt install -y git python3 python3-pip authbind
  ```
2. **Cloner Cowrie** :
  ```bash
   git clone https://github.com/cowrie/cowrie /opt/cowrie
   cd /opt/cowrie
   pip3 install -r requirements.txt
  ```
3. **Configurer Cowrie** :
  - Éditer `/opt/cowrie/etc/cowrie.cfg` :
4. **Démarrer Cowrie** :
  ```bash
   authbind --deep /opt/cowrie/bin/cowrie start
  ```
5. **Rendre Cowrie persistant** :
  - Créer un service systemd (`/etc/systemd/system/cowrie.service`) :
  - Activer le service :
    ```bash
    systemctl enable --now cowrie
    ```

---

## 3. Intégration avec Proxmox/pfSense

- **Règles pfSense** :
  - **Rediriger le port 22 (SSH) vers Cowrie** :
    - `Firewall > NAT > Port Forward` :
      - Interface : WAN.
      - Port : 22 (ext) → 2222 (int, IP de Cowrie `192.168.2.10`).
  - **Bloquer tout trafic depuis Cowrie vers le LAN**.

---

## 4. Utilisation courante

### 4.1. Consulter les logs

- **Logs des connexions** :
  ```bash
  tail -f /opt/cowrie/var/log/cowrie.log
  ```
- **Fichiers de téléchargement (malwares)** :
  - Dossier : `/opt/cowrie/var/lib/cowrie/downloads/`.

### 4.2. Analyser les attaques

- **Outils utiles** :
  - **Cowrie-shell** : Pour interagir avec les attaquants en temps réel.
  - **ELK Stack** : Pour centraliser et analyser les logs (optionnel).

---

## 5. Dépannage

### 5.1. Erreurs fréquentes

- **Cowrie ne démarre pas** :
  - Vérifier les ports utilisés (`netstat -tulnp | grep 2222`).
  - Vérifier les logs :
    ```bash
    journalctl -u cowrie -f
    ```
- **Pas de connexions entrantes** :
  - Vérifier la redirection de port dans pfSense.
  - Tester avec `nc` :
    ```bash
    nc -zv 192.168.2.10 2222
    ```

---

## 6. Bonnes pratiques

- **Sécurité** :
  - Ne **jamais** exposer Cowrie sans redirection de port (risque de scan direct).
  - Isoler complètement le conteneur (pas d’accès au LAN).
- **Analyse** :
  - Automatiser l’envoi des logs vers un SIEM (ex: Graylog) pour une analyse centralisée.

---

## 7. Références officielles

- [Documentation Cowrie](https://github.com/cowrie/cowrie/wiki)
- [Guide d’analyse des logs](https://github.com/cowrie/cowrie/wiki/LogAnalysis)
