Voici un **guide Markdown clair et complet** pour une installation propre et sécurisée de MariaDB sur Debian 13, avec les commandes à exécuter, leur utilité, et les réponses recommandées pour `mysql_secure_installation`.
Tu peux copier-coller ce bloc dans un fichier `.md` ou l’utiliser directement dans un README, un wiki, ou un script de déploiement.

---

```markdown
# 📌 Guide d'installation et sécurisation de MariaDB sur Debian 13

---

## 🔹 1. Installation de MariaDB

### **Commande**
```bash
sudo apt update && sudo apt install mariadb-server mariadb-client -y
```

### **À quoi ça sert ?**
- Met à jour la liste des paquets disponibles.
- Installe le serveur MariaDB (`mariadb-server`) et le client (`mariadb-client`).
- Le flag `-y` évite la confirmation interactive.

---

## 🔹 2. Démarrage et activation du service

### **Commandes**
```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
```

### **À quoi ça sert ?**
- `start` : Démarre le service MariaDB immédiatement.
- `enable` : Active le démarrage automatique au boot.
- `status` : Vérifie que le service est actif (`active (running)`).

---

## 🔹 3. Sécurisation de MariaDB avec `mysql_secure_installation`

### **Lancer la sécurisation**
```bash
sudo mysql_secure_installation
```

### **Réponses recommandées et explications**

| Question | Réponse | Explication |
|----------|---------|-------------|
| **Enter current password for root (enter for none):** | `Entrée` | Par défaut, MariaDB n'a pas de mot de passe root sur Debian. Appuie sur `Entrée`. |
| **Switch to unix_socket authentication [Y/n]** | `n` | Garde l'authentification par mot de passe (plus compatible avec la plupart des outils). |
| **Change the root password? [Y/n]** | `n` | Si tu veux garder l'authentification par socket Unix (recommandé pour la sécurité locale). Sinon, choisis `Y` et définis un mot de passe fort. |
| **Remove anonymous users? [Y/n]** | `Y` | Supprime les utilisateurs anonymes (sécurité). |
| **Disallow root login remotely? [Y/n]** | `Y` | Empêche la connexion root à distance (sécurité). |
| **Remove test database and access to it? [Y/n]** | `Y` | Supprime la base de test accessible à tous (sécurité). |
| **Reload privilege tables now? [Y/n]** | `Y` | Recharge les tables de privilèges pour appliquer les changements. |

> ⚠️ **Note** : Si tu choisis de définir un mot de passe root (`Y` à "Change the root password?"), note-le bien. Sinon, l'accès root se fera via `sudo mysql -u root` (authentification Unix).

---

## 🔹 4. Vérification de l'installation

### **Se connecter à MariaDB**
```bash
sudo mysql -u root
```

### **Vérifier les utilisateurs existants**
```sql
SELECT User, Host FROM mysql.user;
```

### **Résultat attendu**
```
+-------------+-----------+
| User        | Host      |
+-------------+-----------+
| mariadb.sys | localhost |
| mysql       | localhost |
| root        | localhost |
+-------------+-----------+
```
- Seuls `root@localhost`, `mysql@localhost` et `mariadb.sys@localhost` doivent apparaître.
- Pas d'utilisateurs anonymes (`User = ''`).

---

## 🔹 5. (Optionnel) Créer un utilisateur dédié

### **Créer un utilisateur et une base de données**
```sql
CREATE DATABASE ma_base;
CREATE USER 'mon_user'@'localhost' IDENTIFIED BY 'mon_mot_de_passe_fort';
GRANT ALL PRIVILEGES ON ma_base.* TO 'mon_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### **À quoi ça sert ?**
- Évite d'utiliser `root` pour les applications.
- Isole les permissions par projet.

---

## 🔹 6. (Optionnel) Activer le pare-feu pour MariaDB

### **Autoriser le port 3306 (si accès distant nécessaire)**
```bash
sudo ufw allow 3306/tcp
sudo ufw enable
```

> ⚠️ **Attention** : N'ouvre le port 3306 que si tu as **absolument besoin** d'un accès distant, et seulement après avoir sécurisé MariaDB (mot de passe fort, pas d'utilisateurs anonymes, etc.).

---

## 🔹 7. Redémarrer MariaDB (si nécessaire)
```bash
sudo systemctl restart mariadb
```

---

## 📝 Résumé des bonnes pratiques
✅ Toujours exécuter `mysql_secure_installation` après l'installation.
✅ Supprimer les utilisateurs anonymes et la base `test`.
✅ Désactiver l'accès root à distance.
✅ Utiliser des utilisateurs dédiés pour les applications.
✅ Ne jamais exposer MariaDB sur Internet sans protection (pare-feu, VPN, etc.).

---
## 🚀 Prochaines étapes
- [ ] Tester la connexion avec ton utilisateur dédié.
- [ ] Sauvegarder régulièrement tes bases de données (`mysqldump`).
- [ ] Configurer des sauvegardes automatiques (cron + `mysqldump`).
```

---

### **Comment utiliser ce guide ?**
- Copie tout le bloc Markdown ci-dessus.
- Colle-le dans un fichier `install_mariadb_debian13.md`.
- Adapte les noms de bases/utilisateurs/mots de passe selon tes besoins.

---
**Besoin d’aide pour une étape précise ?** Dis-moi laquelle et je te guide en détail !
