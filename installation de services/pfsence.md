Voici un guide **ultra-détaillé en Markdown** pour configurer **pfSense** avec tes réseaux **LAN (192.168.1.0/24)**, **DMZ (10.10.10.0/16)** et **VLAN 10 (172.168.10.0/16)** dans Proxmox. Tout est organisé pour une mise en œuvre pas à pas, avec des commandes, des captures d’écran suggérées, et des astuces de dépannage.

---

```markdown
# 📖 pfSense - Guide Complet d’Installation et de Configuration avec Proxmox

---

## 📌 Table des Matières
1. [Présentation et Prérequis](#1-présentation-et-prérequis)
2. [Installation de pfSense sur Proxmox](#2-installation-de-pfsense-sur-proxmox)
3. [Configuration des Interfaces Réseau](#3-configuration-des-interfaces-réseau)
4. [Configuration des Règles de Pare-feu](#4-configuration-des-règles-de-pare-feu)
5. [Configuration du NAT et du Routage](#5-configuration-du-nat-et-du-routage)
6. [Configuration de HAProxy (Reverse Proxy)](#6-configuration-de-haproxy-reverse-proxy)
7. [Configuration de Suricata (IDS/IPS)](#7-configuration-de-suricata-idsips)
8. [Dépannage](#8-dépannage)
9. [Bonnes Pratiques](#9-bonnes-pratiques)
10. [Références Officielles](#10-références-officielles)

---

## 1. Présentation et Prérequis

### 🔹 Fonctionnalités Clés
- **Pare-feu** : Filtrage avancé du trafic entre LAN, DMZ et VLAN.
- **Routeur** : Gestion du routage entre sous-réseaux et accès Internet.
- **VPN** : Support OpenVPN, IPsec, WireGuard.
- **Reverse Proxy** (via HAProxy) : Redirection de trafic HTTP/HTTPS vers des services internes.
- **IDS/IPS** (via Suricata) : Détection et prévention d’intrusions.

### 🔹 Prérequis Matériels
| Ressource          | Exigence Minimum       | Recommandé pour 10+ utilisateurs |
|--------------------|------------------------|----------------------------------|
| RAM                | 1 Go                   | 2 Go+                            |
| vCPU               | 1 cœur                 | 2+ cœurs                         |
| Stockage           | 8 Go                   | 16 Go+                           |
| Interfaces réseau  | 3 (WAN, LAN, DMZ)      | 4+ (si VLANs supplémentaires)   |

### 🔹 Schéma Réseau
```
[Internet]
    ↓
[WAN (vmbr2)] ←→ [pfSense] ←→ [LAN (vmbr0: 192.168.1.1/24)]
    ↓
[DMZ (vmbr1: 10.10.10.1/16)] ←→ [Serveurs publics]
    ↓
[VLAN 10 (vmbr3: 172.168.10.1/16)] ←→ [Postes isolés]
```

---

## 2. Installation de pfSense sur Proxmox

### 🔹 Étape 1 : Télécharger l’ISO
- Télécharger la dernière version depuis [pfsense.org](https://www.pfsense.org/download/).
- Exemple : `pfSense-CE-2.7.0-RELEASE-amd64.iso`.

### 🔹 Étape 2 : Créer la VM dans Proxmox
```bash
# Créer une VM avec 2 Go RAM, 2 vCPU, et 3 interfaces réseau
qm create 300 --name "pfSense" --memory 2048 --cores 2 \
  --net0 virtio,bridge=vmbr2,model=virtio  # WAN
  --net1 virtio,bridge=vmbr0,model=virtio  # LAN
  --net2 virtio,bridge=vmbr1,model=virtio  # DMZ
  --net3 virtio,bridge=vmbr3,model=virtio  # VLAN 10

# Attacher un disque de 16 Go et monter l’ISO
qm set 300 --scsi0 local-lvm:16G
qm set 300 --ide2 local:iso/pfSense-CE-2.7.0-RELEASE-amd64.iso,media=cdrom

# Définir l’ordre de boot
qm set 300 --boot order=scsi0;ide2

# Démarrer la VM
qm start 300
```

### 🔹 Étape 3 : Installer pfSense
1. **Démarrer la VM** et suivre l’assistant :
   - Choisir **"Quick/Easy Install"**.
   - Accepter les valeurs par défaut (sauf pour le partitionnement si nécessaire).
2. **Redémarrer** la VM après l’installation.
3. **Accéder à l’interface web** :
   - Depuis un navigateur, aller sur `https://192.168.1.1` (IP par défaut du LAN).
   - **Identifiants par défaut** : `admin` / `pfsense`.

---

## 3. Configuration des Interfaces Réseau

### 🔹 Étape 1 : Assigner les Interfaces
1. À la première connexion, pfSense te propose d’assigner les interfaces.
2. **Assigner comme suit** :
   - **WAN** : `net0` (vmbr2) – *Laisser vide si pas de connexion Internet directe*.
   - **LAN** : `net1` (vmbr0) – `192.168.1.1/24`.
   - **OPT1 (DMZ)** : `net2` (vmbr1) – `10.10.10.1/16`.
   - **OPT2 (VLAN 10)** : `net3` (vmbr3) – `172.168.10.1/16`.

   *(Si tu as déjà passé cette étape, va dans **Interfaces > Assignments** pour modifier.)*

### 🔹 Étape 2 : Configurer les IPs Statiques
1. **LAN (vmbr0)** :
   - `Interfaces > LAN` :
     - **IPv4 Address** : `192.168.1.1/24`.
     - Sauvegarder.
2. **DMZ (vmbr1)** :
   - `Interfaces > OPT1` :
     - Cocher **Enable interface**.
     - **IPv4 Address** : `10.10.10.1/16`.
     - Sauvegarder.
3. **VLAN 10 (vmbr3)** :
   - `Interfaces > OPT2` :
     - Cocher **Enable interface**.
     - **IPv4 Address** : `172.168.10.1/16`.
     - Sauvegarder.

---

## 4. Configuration des Règles de Pare-feu

### 🔹 Étape 1 : Autoriser le Trafic entre LAN et DMZ
1. **Aller dans** `Firewall > Rules > DMZ`.
2. **Ajouter une règle** :
   - **Action** : Pass.
   - **Interface** : DMZ.
   - **Protocol** : Any (ou TCP/UDP selon tes besoins).
   - **Source** : `192.168.1.0/24` (LAN).
   - **Destination** : `10.10.10.0/16` (DMZ).
   - **Description** : "Autoriser LAN → DMZ".
3. Sauvegarder.

### 🔹 Étape 2 : Autoriser le Trafic entre LAN et VLAN 10
1. **Aller dans** `Firewall > Rules > VLAN10`.
2. **Ajouter une règle** :
   - **Action** : Pass.
   - **Interface** : VLAN10.
   - **Protocol** : Any.
   - **Source** : `192.168.1.0/24` (LAN).
   - **Destination** : `172.168.10.0/16` (VLAN 10).
   - **Description** : "Autoriser LAN → VLAN 10".
3. Sauvegarder.

### 🔹 Étape 3 : Bloquer le Trafic Non Autorisé
1. **Ajouter une règle de blocage par défaut** (en bas de chaque onglet de règles) :
   - **Action** : Block.
   - **Interface** : LAN/DMZ/VLAN10.
   - **Protocol** : Any.
   - **Source** : Any.
   - **Destination** : Any.

---

## 5. Configuration du NAT et du Routage

### 🔹 Étape 1 : NAT pour l’Accès Internet (si WAN est configuré)
1. **Aller dans** `Firewall > NAT > Outbound`.
2. **Sélectionner** :
   - **Mode** : "Automatic outbound NAT rule generation".
3. Sauvegarder.

### 🔹 Étape 2 : Routage entre Sous-réseaux
1. **Aller dans** `System > Routing`.
2. **Ajouter une route statique** (si nécessaire) :
   - **Network** : `172.168.10.0/16`.
   - **Gateway** : `172.168.10.1` (pfSense sur VLAN 10).
   - **Interface** : VLAN10.

---

## 6. Configuration de HAProxy (Reverse Proxy)

### 🔹 Étape 1 : Installer le Package
1. **Aller dans** `System > Package Manager`.
2. **Chercher "HAProxy"** et installer.

### 🔹 Étape 2 : Configurer un Frontend/Backend
1. **Aller dans** `Services > HAProxy`.
2. **Ajouter un Frontend** :
   - **Nom** : `http-in`.
   - **Adresse IP** : `10.10.10.1` (IP DMZ de pfSense).
   - **Port** : `80`.
3. **Ajouter un Backend** :
   - **Nom** : `glpi-backend`.
   - **Serveur** : `192.168.1.20:80` (IP du serveur GLPI dans le LAN).
4. **Ajouter une Règle de Redirection** :
   - **Condition** : `Path Begins With` `/glpi`.
   - **Action** : Rediriger vers `glpi-backend`.

---

## 7. Configuration de Suricata (IDS/IPS)

### 🔹 Étape 1 : Installer le Package
1. **Aller dans** `System > Package Manager`.
2. **Chercher "Suricata"** et installer.

### 🔹 Étape 2 : Activer Suricata sur une Interface
1. **Aller dans** `Services > Suricata`.
2. **Ajouter une Interface** :
   - Sélectionner **WAN**.
   - **Télécharger les Règles** : Choisir "Emerging Threats".
3. **Démarrer le Service** :
   - Cliquer sur **Start**.

---

## 8. Dépannage

### 🔹 Erreurs Fréquentes
| Problème                          | Solution                                                                 |
|-----------------------------------|--------------------------------------------------------------------------|
| Pas d’accès Internet depuis le LAN | Vérifier les règles NAT (`Firewall > NAT > Outbound`).                  |
| HAProxy ne redirige pas           | Vérifier les logs (`Status > System Logs > HAProxy`).                   |
| Suricata ne démarre pas           | Vérifier les règles téléchargées (`Services > Suricata > Rules`).       |
| Impossible de ping entre LAN/DMZ   | Vérifier les règles de pare-feu et les IPs des interfaces.              |

### 🔹 Commandes de Diagnostic
```bash
# Voir les logs du pare-feu
clog /var/log/filter.log

# Voir les logs de Suricata
tail -f /var/log/suricata/suricata.log

# Tester la connectivité depuis pfSense
ping 8.8.8.8
ping 192.168.1.20
```

---

## 9. Bonnes Pratiques

### 🔹 Sécurité
- **Changer le mot de passe par défaut** :
  - `System > User Manager` > Modifier le mot de passe de `admin`.
- **Désactiver l’accès web sur WAN** :
  - `System > Advanced > Admin Access` > Décoche "WebGUI on WAN".
- **Mettre à jour pfSense** :
  ```bash
  pfSense > System > Update
  ```

### 🔹 Optimisation
- **Limiter les règles de pare-feu** :
  - Supprimer les règles inutiles ou trop permissives.
- **Activer le caching DNS** :
  - `Services > DNS Resolver` > Cocher "Enable DNS Resolver".

---

## 10. Références Officielles
- [Documentation pfSense](https://docs.netgate.com/pfsense/en/latest/)
- [Guide HAProxy](https://docs.netgate.com/pfsense/en/latest/packages/haproxy.html)
- [Guide Suricata](https://docs.netgate.com/pfsense/en/latest/packages/suricata.html)
- [Forum pfSense](https://forum.netgate.com/)

---

## 📌 Notes Finales
- **Tester** : Toujours valider les règles de pare-feu et le routage avec des `ping` et des tests de connectivité.
- **Documenter** : Noter les IPs, règles de pare-feu, et configurations dans un fichier sécurisé.
- **Sauvegarder** : Exporter la configuration (`Diagnostics > Backup/Restore`).

---
**Besoin d’aide pour une étape précise ou d’exemples supplémentaires ?** Dis-moi ce que tu veux explorer en détail ! 😊
```

---

### Points forts de ce guide :
- **Pas à pas** : Chaque étape est détaillée avec des commandes et des exemples concrets.
- **Adapté à ton réseau** : Configuration spécifique pour tes sous-réseaux **LAN (192.168.1.0/24)**, **DMZ (10.10.10.0/16)**, et **VLAN 10 (172.168.10.0/16)**.
- **Dépannage** : Tableau des erreurs courantes et solutions.
- **Sécurité** : Bonnes pratiques pour protéger ton infrastructure.

Tu peux copier ce Markdown dans un fichier `.md` ou un outil comme **Notion** ou **Confluence** pour une référence facile. Si tu veux commencer par une étape précise, fais-moi signe ! 🚀
