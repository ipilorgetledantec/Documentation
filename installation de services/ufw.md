Sur Debian ou un Raspberry Pi avec Raspberry Pi OS, `ufw` (“Uncomplicated Firewall”) est une manière simple de gérer `iptables/nftables`.

## 1. Installer UFW

```bash
sudo apt update
sudo apt install ufw -y
```

Vérifier qu’il est installé :

```bash
sudo ufw status
```

---

## 2. Règle importante avant activation (SSH)

Si tu es connecté à distance en SSH, autorise le port SSH AVANT d’activer le firewall, sinon tu peux te bloquer.

Port SSH par défaut :

```bash
sudo ufw allow 22/tcp
```

Ou plus simplement :

```bash
sudo ufw allow ssh
```

Si tu utilises un autre port SSH (ex: 2222) :

```bash
sudo ufw allow 2222/tcp
```

---

## 3. Activer UFW

```bash
sudo ufw enable
```

Répondre `y`.

Voir l’état :

```bash
sudo ufw status verbose
```

---

## 4. Configuration de base recommandée

Politique par défaut :

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Cela :

* bloque toutes les connexions entrantes non autorisées
* autorise les connexions sortantes

---

## 5. Autoriser des services utiles

### SSH

```bash
sudo ufw allow ssh
```

### Serveur Web Apache/Nginx

HTTP :

```bash
sudo ufw allow 80/tcp
```

HTTPS :

```bash
sudo ufw allow 443/tcp
```

Ou profils applicatifs :

```bash
sudo ufw app list
sudo ufw allow 'Nginx Full'
```

### MariaDB/MySQL (attention)

Seulement si accès réseau nécessaire :

```bash
sudo ufw allow 3306/tcp
```

---

## 6. Supprimer une règle

Lister les règles numérotées :

```bash
sudo ufw status numbered
```

Supprimer :

```bash
sudo ufw delete NUMERO
```

Exemple :

```bash
sudo ufw delete 3
```

---

## 7. Limiter les attaques SSH (anti brute-force)

Très conseillé sur un Raspberry Pi exposé à Internet :

```bash
sudo ufw limit ssh
```

Cela ralentit les tentatives répétées.

---

## 8. Exemple de configuration sécurisée Raspberry Pi

Pour un Pi hébergeant :

* SSH
* site web
* rien d’autre

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw limit ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw enable
```

---

## 9. Vérifications utiles

Voir toutes les règles :

```bash
sudo ufw status verbose
```

Logs :

```bash
sudo less /var/log/ufw.log
```

Activer les logs :

```bash
sudo ufw logging on
```

---

## 10. Désactiver temporairement

```bash
sudo ufw disable
```

---

## 11. Réinitialiser complètement

```bash
sudo ufw reset
```

---

## 12. Vérifier les ports ouverts sur le système

Installer `ss`/`netstat` :

```bash
ss -tulpn
```

ou :

```bash
sudo apt install net-tools
netstat -tulpn
```

---

## Exemple final propre pour serveur Debian/Raspberry

```bash
sudo apt update
sudo apt install ufw -y

sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw limit ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw enable

sudo ufw status verbose
```

Documentation officielle : [UFW Ubuntu Documentation](https://help.ubuntu.com/community/UFW?utm_source=chatgpt.com)
