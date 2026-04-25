# pfSense - Guide d'installation et d'utilisation

---

## 1. Présentation

**Fonction** :

- Pare-feu, routeur, VPN, IDS/IPS (via Suricata), gestion de VLANs/DMZ.
- **Reverse Proxy** (via le package HAProxy).

**Prérequis** :

- **Ressources** : 2 vCPUs, 2 Go RAM, 16 Go SSD.
- **Réseau** : 3 interfaces (WAN, LAN, DMZ).

---

## 2. Installation

### 2.1. Installation sur Proxmox

1. **Télécharger l’ISO** :
  [Lien officiel](https://www.pfsense.org/download/).
2. **Créer une VM dans Proxmox** :
  ```bash
   qm create 300 --name "pfSense" --memory 2048 --cores 2 \
   --net0 virtio,bridge=vmbr2,model=virtio # WAN
   --net1 virtio,bridge=vmbr0,model=virtio # LAN
   --net2 virtio,bridge=vmbr1,model=virtio # DMZ
   qm set 300 --scsi0 local-lvm:16G
   qm set 300 --ide2 local:iso/pfSense-CE-2.7.0-RELEASE-amd64.iso,media=cdrom
   qm set 300 --boot order=scsi0;ide2
  ```
3. **Démarrer la VM et installer pfSense** :
  - Suivre l’assistant (choisir "Quick/Easy Install").
  - Redémarrer et accéder à l’interface web : `https://<IP_LAN_pfSense>`.

### 2.2. Configuration initiale

1. **Assigner les interfaces** :
  - Dans l’interface web : `Interfaces > Assign` :
    - WAN : `vmbr2` (ex: DHCP ou IP fixe).
    - LAN : `vmbr0` (ex: `192.168.1.1/24`).
    - DMZ : `vmbr1` (ex: `192.168.2.1/24`).
2. **Activer SSH** (optionnel) :
  - `System > Advanced > Admin Access` > Cocher "Enable Secure Shell".

---

## 3. Intégration avec Proxmox

- **Règles de pare-feu** :
  - Autoriser le trafic entre Proxmox (`192.168.1.254`) et pfSense (`192.168.1.1`) sur les ports nécessaires (ex: 8006 pour l’interface Proxmox).
- **VLANs** :
  - Si tu utilises des VLANs, les configurer dans `Interfaces > Assignments > VLANs`.

---

## 4. Utilisation courante

### 4.1. Configurer HAProxy (Reverse Proxy)

1. **Installer le package** :
  - `System > Package Manager` > Chercher "HAProxy" et installer.
2. **Configurer un frontend/backend** :
  - `Services > HAProxy` :
    - **Frontend** : Écouter sur `192.168.2.1:80` (DMZ).
    - **Backend** : Rediriger vers `192.168.1.20:80` (GLPI).
  - Exemple de règle :
    ```
    frontend http-in
        bind 192.168.2.1:80
        acl is_glpi path_beg /glpi
        use_backend glpi if is_glpi

    backend glpi
        server glpi-server 192.168.1.20:80
    ```

### 4.2. Configurer Suricata (IDS/IPS)

1. **Installer le package** :
  - `System > Package Manager` > Chercher "Suricata" et installer.
2. **Activer Suricata** :
  - `Services > Suricata` > Sélectionner l’interface WAN.
  - Télécharger les règles (ex: Emerging Threats) :
    - `Rules > Download Rules`.

---

## 5. Dépannage

### 5.1. Erreurs fréquentes

- **Pas d’accès internet depuis le LAN** :
  - Vérifier les règles NAT (`Firewall > NAT > Outbound`).
  - Solution : Créer une règle NAT automatique :
    - `Firewall > NAT > Outbound` > Mode "Automatic".
- **HAProxy ne redirige pas** :
  - Vérifier les logs : `Status > System Logs` > Onglet "HAProxy".
  - Solution : Vérifier que le backend est "UP" (`Services > HAProxy > Status`).

### 5.2. Logs utiles

- **Logs pare-feu** :
  ```bash
  clog /var/log/filter.log
  ```
- **Logs Suricata** :
  ```bash
  tail -f /var/log/suricata/suricata.log
  ```

---

## 6. Bonnes pratiques

- **Sécurité** :
  - Changer le mot de passe par défaut (`admin/pfsense`).
  - Désactiver l’accès web sur WAN (`System > Advanced > Admin Access`).
- **Optimisation** :
  - Limiter les règles de pare-feu inutiles.
  - Désactiver les services inutilisés (ex: DNS Forwarder si tu utilises l’AD).

---

## 7. Références officielles

- [Documentation pfSense](https://docs.netgate.com/pfsense/en/latest/)
- [Guide HAProxy](https://docs.netgate.com/pfsense/en/latest/packages/haproxy.html)
- [Forum pfSense](https://forum.netgate.com/)
