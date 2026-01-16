```markdown
# Sécurisation d’un LAN : Guide Pratique

---

## **1. Introduction**
Ce guide explique comment sécuriser un LAN contre les attaques courantes (ARP spoofing, DHCP spoofing, DHCP starvation) sur deux environnements :
- **Switch Cisco 2960**
- **Linux (Debian/Kali)**

---

## **2. Atténuation des attaques sur un Switch Cisco 2960**

### **2.1. Commandes de base pour la surveillance**
| Commande | Description |
|----------|-------------|
| `show mac address-table` | Affiche la table CAM (adresses MAC apprises par le switch). |
| `shutdown` | Désactive complètement un port. |

### **2.2. Sécurisation contre le DHCP Spoofing**
Pour activer le **DHCP Snooping** et limiter les attaques :
```bash
enable
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 1
interface GigabitEthernet0/1
 ip dhcp snooping trust
exit
ip dhcp snooping information option
interface range FastEthernet0/1-24
 ip dhcp snooping limit rate 4
exit
```
- **`ip dhcp snooping trust`** : À appliquer uniquement sur le port connecté au serveur DHCP légitime.
- **`ip dhcp snooping limit rate 4`** : Limite le nombre de requêtes DHCP par seconde (4 par défaut).

### **2.3. Sécurisation des ports (Port Security)**
```bash
configure terminal
interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 1
 switchport port-security violation shutdown
 switchport port-security maximum 4
 switchport port-security mac-address sticky
exit
```
- **`switchport port-security violation shutdown`** : Désactive le port en cas de violation.
- **`switchport port-security mac-address sticky`** : Apprend dynamiquement les adresses MAC et les associe au port.

### **2.4. Vérification de la configuration**
| Commande | Description |
|----------|-------------|
| `show mac address-table count` | Affiche le nombre d’adresses MAC apprises. |
| `show port-security` | Affiche l’état de la sécurité des ports. |

---

## **3. Configuration sur Linux (Debian/Kali)**

### **3.1. Configuration du serveur DHCP (Debian)**
1. **Modifier le fichier de configuration** :
   ```bash
   su -
   nano /etc/dhcp/dhcpd.conf
   ```
   - Remplacer `subnet 0.0.0.0 255.255.255.0` par le sous-réseau souhaité, par exemple :
     ```plaintext
     subnet 192.168.1.0 netmask 255.255.255.0 {
       range 192.168.1.100 192.168.1.200;
     }
     ```
   - Pour un sous-réseau "rogue" :
     ```plaintext
     subnet 10.10.10.0 netmask 255.255.255.0 {
       range 10.10.10.100 10.10.10.200;
     }
     ```

2. **Redémarrer l’interface réseau** :
   ```bash
   ifdown enp0s3
   ifup enp0s3
   ```

3. **Vérifier le statut du serveur DHCP** :
   ```bash
   systemctl status isc-dhcp-server
   ```

4. **Installer le serveur DHCP (si nécessaire)** :
   ```bash
   apt install isc-dhcp-server
   ```

5. **Voir les baux DHCP attribués** :
   ```bash
   sudo cat /var/lib/dhcp/dhcpd.leases
   ```

### **3.2. Simulation d’attaques (Kali Linux)**
1. **Configurer le clavier en français** :
   ```bash
   setxkbmap fr
   ```

2. **Installer Yersinia (outil d’attaque)** :
   ```bash
   sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys ED65462EC8D5E4C5
   apt update
   apt install yersinia libgtk-4-dev
   ```

3. **Lancer une attaque DHCP Starvation** :
   ```bash
   yersinia -I
   yersinia dhcp -attack 1 -interface eth0
   ```
   - **Surveiller le trafic DHCP** :
     ```bash
     sudo tcpdump -i eth0 port 67 or port 68
     ```

4. **Surveiller les logs du serveur DHCP (Debian)** :
   ```bash
   journalctl -u isc-dhcp-server -f
   ```

5. **Lancer une attaque ARP Spoofing (macof)** :
   ```bash
   macof -i eth0
   ```

---

## **4. Schéma de l’infrastructure**
```
+----------------+       +----------------+       +----------------+
|   Attaquant    |-------|    Switch      |-------|     Serveur    |
| (Kali Linux)   |       |  (Cisco 2960)  |       |    (Debian)    |
+----------------+       +----------------+       +----------------+
                                      |
                              +----------------+
                              |   Laptop       |
                              +----------------+
```

---

## **5. Résumé des bonnes pratiques**
- **Sur le switch** :
  - Activer le **DHCP Snooping** et la **Port Security**.
  - Limiter le nombre de requêtes DHCP par port.
  - Désactiver les ports non utilisés.

- **Sur le serveur DHCP (Debian)** :
  - Configurer correctement les sous-réseaux.
  - Surveiller les logs et les baux DHCP.

- **Sur Kali Linux** :
  - Utiliser Yersinia pour tester les vulnérabilités.
  - Surveiller le trafic avec `tcpdump`.

---
