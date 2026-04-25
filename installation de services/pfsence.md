Avec ta configuration Proxmox actuelle, voici comment configurer **pfSense** pour gérer tes trois réseaux : **LAN (vmbr0)**, **DMZ (vmbr1)**, et **VLAN 10 (vmbr2)**.

---

## **1. Prérequis**
- pfSense est installé en tant que VM dans Proxmox.
- Les interfaces réseau de la VM pfSense sont assignées comme suit :
  - **net0** → `vmbr0` (LAN : 192.168.1.113/24)
  - **net1** → `vmbr1` (DMZ : 10.10.10.1/24)
  - **net2** → `vmbr2` (VLAN 10 : 172.168.10.1/24)

---

## **2. Configuration initiale de pfSense**

### **A. Accéder à l'interface web de pfSense**
1. Démarre la VM pfSense.
2. Depuis un navigateur, connecte-toi à l’IP du LAN de pfSense (par défaut, c’est généralement `192.168.1.1` si tu n’as pas changé la configuration initiale).
   - Si tu as déjà configuré pfSense, utilise l’IP que tu as définie pour le LAN.
   - le Nom d’utilisateur : admin
            Mot de passe : pfsense
     sont par default.


---

### **B. Assigner les interfaces réseau**
1. À la première connexion, pfSense te demandera d’assigner les interfaces.
2. Assigne :
   - **WAN** : `net0` (si tu veux que pfSense gère aussi la connexion Internet, sinon laisse vide).
   - **LAN** : `net0` (si WAN est vide) ou `net1` (selon ton choix).
   - **OPT1** : `net1` (DMZ).
   - **OPT2** : `net2` (VLAN 10).

   *(Si tu as déjà passé cette étape, va dans **Interfaces > Assignments** pour modifier.)*

---

### **C. Configurer les interfaces**

#### **LAN (vmbr0)**
- Va dans **Interfaces > LAN**.
- **IPv4 Address** : `192.168.1.113/24` (ou une autre IP libre dans ton sous-réseau LAN).
- **Gateway** : `192.168.1.1` (si pfSense doit router vers Internet via ton réseau existant).
- Sauvegarde.

#### **DMZ (vmbr1)**
- Va dans **Interfaces > OPT1**.
- Coche **Enable interface**.
- **IPv4 Address** : `10.10.10.1/24`.
- Sauvegarde.

#### **VLAN 10 (vmbr2)**
- Va dans **Interfaces > OPT2**.
- Coche **Enable interface**.
- **IPv4 Address** : `172.168.10.1/24`.
- Sauvegarde.

---

## **3. Configurer les règles de pare-feu**

### **A. Règles pour la DMZ (OPT1)**
1. Va dans **Firewall > Rules > DMZ**.
2. Ajoute une règle pour autoriser le trafic depuis le LAN vers la DMZ (si nécessaire) :
   - **Action** : Pass.
   - **Interface** : DMZ.
   - **Protocol** : Any (ou spécifie TCP/UDP selon tes besoins).
   - **Source** : `192.168.1.0/24` (LAN).
   - **Destination** : `10.10.10.0/24` (DMZ).
3. Sauvegarde.

### **B. Règles pour le VLAN 10 (OPT2)**
1. Va dans **Firewall > Rules > VLAN10**.
2. Ajoute une règle pour autoriser le trafic entre le LAN et le VLAN 10 :
   - **Action** : Pass.
   - **Interface** : VLAN10.
   - **Protocol** : Any.
   - **Source** : `192.168.1.0/24` (LAN).
   - **Destination** : `172.168.10.0/24` (VLAN 10).
3. Sauvegarde.

---

## **4. Configurer le NAT et le routage (si nécessaire)**

### **A. NAT pour la DMZ ou le VLAN**
Si tu veux que les machines dans la DMZ ou le VLAN accèdent à Internet :
1. Va dans **Firewall > NAT > Outbound**.
2. Ajoute une règle de NAT pour la DMZ :
   - **Interface** : WAN.
   - **Source** : `10.10.10.0/24` (DMZ).
   - **Translation** : Interface Address.
3. Fais de même pour le VLAN 10 (`172.168.10.0/24`).

---

## **5. Vérifier la connectivité**

### **A. Depuis une VM dans le LAN**
- Ping `10.10.10.1` (pfSense sur la DMZ).
- Ping `172.168.10.1` (pfSense sur le VLAN 10).

### **B. Depuis une VM dans la DMZ**
- Ping `192.168.1.113` (pfSense sur le LAN).
- Vérifie l’accès Internet si le NAT est configuré.

### **C. Depuis une VM dans le VLAN 10**
- Ping `192.168.1.113` (pfSense sur le LAN).
- Vérifie l’accès Internet si le NAT est configuré.

---

## **6. Dépannage**
- Si une interface ne répond pas :
  - Vérifie que la VM pfSense a bien ses interfaces assignées aux bons bridges (`vmbr0`, `vmbr1`, `vmbr2`).
  - Vérifie les règles de pare-feu dans pfSense.
  - Exécute `ping` depuis pfSense (via **Diagnostics > Ping**) pour tester la connectivité.

---

### **Résumé des IPs à utiliser**
| Réseau       | IP pfSense       | Exemple IP VM       |
|--------------|------------------|---------------------|
| LAN (vmbr0)  | 192.168.1.113/24 | 192.168.1.100/24    |
| DMZ (vmbr1)  | 10.10.10.1/24    | 10.10.10.100/24     |
| VLAN 10 (vmbr2) | 172.168.10.1/24 | 172.168.10.100/24  |

---
### **Besoin d’aide pour une étape précise ?**
- Veux-tu que je détaille la configuration du NAT, des règles de pare-feu, ou la vérification de la connectivité ?
- As-tu des erreurs spécifiques ?

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
