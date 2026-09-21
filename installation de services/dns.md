



# DNS sec

### 1

``` bash 
nano /etc/bind/named.conf.options
```

``` bash 

options {
        directory "/var/cache/bind";

        forwarders {
                1.1.1.1;
        };

        dnssec-validation auto;

};


 cd /etc/bind/

 mkdir keys
cd keys/

dnssec-keygen -a rsasha256 -b 1024 -n zone sodecaf.fr

prnendre le nom de la clé exemple :Ksodecaf.fr.+008+18471

ls -l

-rw-r--r-- 1 root bind  429 21 sept. 10:46 Ksodecaf.fr.+008+18471.key
-rw------- 1 root bind 1012 21 sept. 10:46 Ksodecaf.fr.+008+18471.private
47587.
 dnssec-keygen -a rsasha256 -b 1024 -f KSK -n zone sodecaf.fr

```

nano  nano /etc/bind/named.conf.options  

; KSK
$include "/etc/bind/keys/Ksodecaf.fr.+008+47587.key";
; ZSK
$include "/etc/bind/keys/Ksodecaf.fr.+008+18471.key";


  dnssec-signzone -o sodecaf.fr -t -k /etc/bind/keys/Ksodecaf.fr.+008+47587. /var/cache/bind/db.sodecaf.fr /etc/bind/keys/Ksodecaf.fr.+008+18471

 nano /etc/bind/named.conf.local
ajouter le .signed
//
// Do any local configuration here
//
zone "sodecaf.fr" {
        type master;
        file "db.sodecaf.fr.signed";
        allow-transfer { 172.16.0.4; };
};


zone "0.16.172.in-addr.arpa" {
        type master;
        file "db.172.16.0.rev";
        allow-transfer { 172.16.0.4; };

};


pour tester 

dig +dnssec @172.16.0.3 wwww.sodecaf.fr