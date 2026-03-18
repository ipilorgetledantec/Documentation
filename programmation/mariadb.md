# 📘 Fiche 1 — Introduction à MariaDB

## 🔹 Qu’est-ce que MariaDB ?

* Système de gestion de base de données relationnelle (SGBDR)
* Fork de MySQL (créé en 2009)
* Open source et maintenu par la communauté

## 🔹 Pourquoi utiliser MariaDB ?

* Compatible avec MySQL
* Plus performant sur certains cas
* Plus de fonctionnalités open source
* Pas contrôlé par Oracle

## 🔹 Cas d’usage

* Applications web (PHP, Node.js…)
* CMS (WordPress, Drupal)
* Data analytics léger

---

# 📘 Fiche 2 — Installation & Connexion

## 🔹 Installation (Linux - exemple)

```bash
sudo apt install mariadb-server
```

## 🔹 Démarrer le service

```bash
sudo systemctl start mariadb
```

## 🔹 Connexion

```bash
mysql -u root -p
```

## 🔹 Sécurisation

```bash
mysql_secure_installation
```

---

# 📘 Fiche 3 — Commandes de base SQL

## 🔹 Bases de données

```sql
CREATE DATABASE ma_base;
SHOW DATABASES;
USE ma_base;
DROP DATABASE ma_base;
```

## 🔹 Tables

```sql
CREATE TABLE utilisateurs (
  id INT PRIMARY KEY AUTO_INCREMENT,
  nom VARCHAR(100),
  email VARCHAR(100)
);

SHOW TABLES;
DROP TABLE utilisateurs;
```

---

# 📘 Fiche 4 — CRUD (opérations principales)

## 🔹 INSERT

```sql
INSERT INTO utilisateurs (nom, email)
VALUES ('Alice', 'alice@mail.com');
```

## 🔹 SELECT

```sql
SELECT * FROM utilisateurs;
SELECT nom FROM utilisateurs WHERE id = 1;
```

## 🔹 UPDATE

```sql
UPDATE utilisateurs
SET nom = 'Bob'
WHERE id = 1;
```

## 🔹 DELETE

```sql
DELETE FROM utilisateurs WHERE id = 1;
```

---

# 📘 Fiche 5 — Types de données

## 🔹 Numériques

* INT
* FLOAT
* DECIMAL

## 🔹 Texte

* VARCHAR(n)
* TEXT

## 🔹 Date

* DATE
* DATETIME
* TIMESTAMP

## 🔹 Booléen

* BOOLEAN (alias de TINYINT)

---

# 📘 Fiche 6 — Clés & Index

## 🔹 Clé primaire

```sql
id INT PRIMARY KEY
```

## 🔹 Clé étrangère

```sql
FOREIGN KEY (user_id) REFERENCES utilisateurs(id)
```

## 🔹 Index

```sql
CREATE INDEX idx_nom ON utilisateurs(nom);
```

## 🔹 Pourquoi ?

* Accélérer les recherches
* Optimiser les requêtes

---

# 📘 Fiche 7 — Jointures

## 🔹 INNER JOIN

```sql
SELECT *
FROM commandes
INNER JOIN utilisateurs
ON commandes.user_id = utilisateurs.id;
```

## 🔹 LEFT JOIN

```sql
SELECT *
FROM utilisateurs
LEFT JOIN commandes
ON utilisateurs.id = commandes.user_id;
```

---

# 📘 Fiche 8 — Fonctions utiles

## 🔹 Agrégation

```sql
SELECT COUNT(*) FROM utilisateurs;
SELECT AVG(age) FROM utilisateurs;
SELECT MAX(age) FROM utilisateurs;
```

## 🔹 Groupement

```sql
SELECT age, COUNT(*)
FROM utilisateurs
GROUP BY age;
```

---

# 📘 Fiche 9 — Gestion des utilisateurs

## 🔹 Créer un utilisateur

```sql
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
```

## 🔹 Donner des droits

```sql
GRANT ALL PRIVILEGES ON ma_base.* TO 'user'@'localhost';
```

## 🔹 Appliquer

```sql
FLUSH PRIVILEGES;
```

---

# 📘 Fiche 10 — Sauvegarde & restauration

## 🔹 Sauvegarde

```bash
mysqldump -u root -p ma_base > backup.sql
```

## 🔹 Restauration

```bash
mysql -u root -p ma_base < backup.sql
```

---

# 📘 Fiche 11 — Bonnes pratiques

## ✅ À faire

* Utiliser des index
* Sauvegarder régulièrement
* Normaliser les données

## ❌ À éviter

* SELECT *
* Trop de jointures complexes
* Absence de clés primaires

---

# 📘 Fiche 12 — Différences MariaDB vs MySQL

| MariaDB                       | MySQL                      |
| ----------------------------- | -------------------------- |
| Open source complet           | Partiellement propriétaire |
| Plus de moteurs de stockage   | Moins                      |
| Optimisations supplémentaires | Standard                   |


