# config de base d'une machine linux 
## 1.1 

dans /etc/interfaces

``` bash 

iface ens33 inet static
        address 172.16.0.10/24
        gateway 172.16.0.254
        dns-nameservers 1.1.1.1
```

dans /etc/hostname

``` bash 

srv-web1

```


dans /etc/hosts

``` bash 


127.0.0.1       localhost
127.0.1.1       srv-web1

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```


dans /etc/resolv.conf

``` bash 

nameserver 1.1.1.1


```





