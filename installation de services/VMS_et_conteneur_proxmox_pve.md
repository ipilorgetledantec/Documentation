installation vms proxmox pve






vms windows :

Prérequis :
iso de win 11
iso de supplément : virtio-win.iso


sur proxmox :
- créé vm dans proxmox puis
- dans l'onglet general mettre un id, un nom, un pool
- dans l'onglet os : mettre iso win 11, type = microsoft windows puis add aditional visio driver (allez la télécharger avant la configuration de la machine), dans iso image mettre le virtio-win.iso 
- dans l'onglet system : mettre les storages et cocher qemu agent
- dans l'onglet disks : size = 64g pour ne pas etre embété, cocher discard
- dans l'onglet cpu : 2 coeurs et type hosts
- dans l'onglet memory : 4g
puis next et finish

lancer la vm.

faire suivant pour la configuration de la langue sauf si ce n est pas fr.
suivant et mettre je n ai pas la clé.
prendre edition 

partie disk : 
- click sur load driver
- parcourir
- click sur le premier lecteur
- click vioscsi
- click win11
- click amd64
- fairre ok

selectionner le disk et faire installer.

Une fois tout installer vous allez arriver sur la partie pilote :
- selectionner le lecteur cd sans rentrer dedans
- puis clicker sur selectionner le dossier

suivant mettre non à tout.

Une fois sur le bureau :
- aller sur le lecteur virtio
- descendre et selectionner virtio-win-gtX64 et  executer
- pareil pour win guest tool




vm linux :



Prérequis :
iso deb 12 ou autre


sur proxmox :
- créé vm dans proxmox puis
- dans l'onglet general mettre un id, un nom, un pool
- dans l'onglet os : mettre iso deb 12, type = linux
- dans l'onglet system : mettre les storages et cocher qemu agent 
- dans l'onglet disks : size = à voir en foncton des recommandations
- dans l'onglet cpu : à voir en foncton des recommandations
- dans l'onglet memory : à voir en foncton des recommandations
puis next et finish

lancer la vm.





conteneur :

Que des conteneurs avec un noyau linux.


prérequis :

allez sur datacenter>le_nom_pve_du_serv>local (pas le lvlm)>ct_templates>templates
choisir votre os.
Les turquey sont des ct près à l'emploi.


sur proxmox : 

créé ct :
- dans l'onglet general mettre un id, un nom, un pool,mdp
- dans l'onglet template : mettre le conteneur précédement installer
- dans l'onglet disks : size = à voir en foncton des recommandations
- dans l'onglet cpu : à voir en foncton des recommandations
- dans l'onglet memory : à voir en foncton des recommandations
puis next et finish










