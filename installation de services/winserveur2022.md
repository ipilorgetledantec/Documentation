


# DNS sec

### 1

``` bash 
nano /etc/bind/named.conf.options
```

changer le nom de la machine

srv-win1


![alt text](image.png)


puis dans la page de gestion gere>ajouter role et fonction

![alt text](image-1.png)

sous win 2025 il faur

puis suivant j usqu à la fin.


sur le 2 emme

C:\Windows\System32\Sysprep

clique droit ouvrir en tant qu admin

puis mettre generaliser et redemarer

![alt text](image-2.png)


appuyer sur promouvoir

![alt text](image-3.png)

mettre le mdp 
puis suivant partout


sur le serv 2 

change l ip et le nom

172.16.0.2
/24
172.16.0.254

172.16.0.1
172.16.0.2

redemarrage



sur win serv 2 

parametre avance renommage 

![alt text](image-4.png)


![alt text](image-5.png)

sur win serv 1
terminer le dhcp 
![alt text](image-6.png)

faire ok ok terminer la conf


cliquer dhcp faire une étendue 

![alt text](image-7.png)

![alt text](image-8.png)


![alt text](image-9.png)
suivant
mettre un 1 pour les jours
oui
![alt text](image-10.png)
![alt text](image-11.png)

puis suivant suivant

dans l annuiaire

faire une uo


![alt text](image-12.png)

prendre une machine client
![alt text](image-13.png)



win serv 2

![alt text](image-14.png)

et changer le nom et mdp par celui du domaine
puis suivant et installer
redemare

puis dhcp 
tout ok 

win 1 

confi dhcp

clic droit dhcp

![alt text](image-15.png)

gerer les servs

puis ajouter si le serv n apparait pas

![alt text](image-16.png)

![alt text](image-17.png)

![alt text](image-18.png)
mdp rootsio201
puis ok ok ok 


![alt text](image-19.png)
![alt text](image-20.png)


