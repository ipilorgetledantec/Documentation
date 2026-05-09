# Installation de VMs et CT sur Proxmox VE

---

## 🖥️ **Installation d'une VM Windows**

### **Prérequis**
- ISO de **Windows 11**
- ISO supplémentaire : **`virtio-win.iso`** (à télécharger avant la configuration)

---

### **Configuration dans Proxmox**
1. **Créer une nouvelle VM**
2. **Onglet Général**
   - Définir un **ID**, un **nom**, et un **pool**
3. **Onglet OS**
   - Sélectionner l'ISO de **Windows 11**
   - Type : **Microsoft Windows**
   - Ajouter un **pilote virtio** :
     - Cliquer sur **"Add additional VirtIO driver"**
     - Sélectionner **`virtio-win.iso`** dans **ISO Image**
4. **Onglet System**
   - Configurer les **storages**
   - Cocher **QEMU Guest Agent**
5. **Onglet Disks**
   - Taille : **64 Go** (pour éviter les problèmes)
   - Cocher **Discard**
6. **Onglet CPU**
   - **2 cœurs**
   - Type : **Host**
7. **Onglet Memory**
   - **4 Go** de RAM
8. **Valider** avec **Next** puis **Finish**

---

### **Installation de Windows 11**
1. **Lancer la VM**
2. **Configuration initiale**
   - Suivre les étapes jusqu'à la sélection de la **langue** (choisir **Français** si nécessaire)
   - Sélectionner **"Je n'ai pas de clé de produit"**
   - Choisir l'**édition** souhaitée
3. **Partitionnement du disque**
   - Cliquer sur **"Charger le pilote"**
   - Parcourir → Sélectionner le **premier lecteur CD**
   - Aller dans :
     - **`vioscsi`** → **`win11`** → **`amd64`**
   - Valider avec **OK**
   - Sélectionner le disque et lancer l'**installation**
4. **Installation des pilotes**
   - Une fois l'installation terminée, sur l'écran des **pilotes** :
     - Sélectionner le **lecteur CD** (sans y entrer)
     - Cliquer sur **"Sélectionner le dossier"**
     - Répondre **"Non"** à toutes les questions suivantes
5. **Finalisation**
   - Sur le **bureau** :
     - Ouvrir le lecteur **VirtIO**
     - Descendre et sélectionner **`virtio-win-guest-tools`** (version **GTx64**)
     - Exécuter l'installation
     - Répéter pour **Win Guest Tools**

---

---

## 🐧 **Installation d'une VM Linux**

### **Prérequis**
- ISO de **Debian 12** (ou autre distribution Linux)

---

### **Configuration dans Proxmox**
1. **Créer une nouvelle VM**
2. **Onglet Général**
   - Définir un **ID**, un **nom**, et un **pool**
3. **Onglet OS**
   - Sélectionner l'ISO de **Debian 12** (ou autre)
   - Type : **Linux**
4. **Onglet System**
   - Configurer les **storages**
   - Cocher **QEMU Guest Agent**
5. **Onglet Disks**
   - Taille : **À adapter selon les recommandations** de la distribution
6. **Onglet CPU**
   - Nombre de cœurs : **À adapter selon les recommandations**
7. **Onglet Memory**
   - RAM : **À adapter selon les recommandations**
8. **Valider** avec **Next** puis **Finish**

---
### **Installation de Linux**
1. **Lancer la VM**
2. Suivre les étapes d'installation standard de la distribution choisie.

---

---

## 📦 **Création d'un conteneur (CT) Linux**

### **Prérequis**
- **Template de conteneur** :
  - Aller dans **Datacenter** > **Nom du serveur Proxmox** > **local** (pas **lvm**) > **ct_templates** > **templates**
  - Choisir le **template** correspondant à votre OS (ex: Debian, Ubuntu, etc.)
  - *Les templates "TurnKey" sont des CT prêts à l'emploi.*

---

### **Configuration dans Proxmox**
1. **Créer un nouveau CT**
2. **Onglet Général**
   - Définir un **ID**, un **nom**, un **pool**, et un **mot de passe**
3. **Onglet Template**
   - Sélectionner le **template** téléchargé précédemment
4. **Onglet Disks**
   - Taille : **À adapter selon les besoins**
5. **Onglet CPU**
   - Nombre de cœurs : **À adapter selon les besoins**
6. **Onglet Memory**
   - RAM : **À adapter selon les besoins**
7. **Valider** avec **Next** puis **Finish**

---


### **Une fois lancé, il peut être un peu long à démarrer**

