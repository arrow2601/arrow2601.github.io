# LDAP + Mail Server

!!! Note
    Clone 2 buah server , LDAP server dan Mail Server 

## Konfigurasi dasar LDAP Server
### Konfigurasi IP Address dan Hostname
```  
root@belajar:~# hostnamectl set-hostname ldap-server
```
```
root@belajar:~# su -
```
```
root@ldap-server:~# nano /etc/network/interfaces
```
```
# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
address 192.168.10.1/24
```
```
root@ldap-server:~# service networking restart
```

## Konfigurasi dasar Mail Server
### Konfigurasi IP Address dan Hostname

```
root@belajar:~# hostnamectl set-hostname mail-server
root@belajar:~# su -
```
```
root@mail-server:~# nano /etc/network/interfaces
```
```
auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
address 192.168.10.2/24
``` 

## Konfigurasi DNS (LDAP-Server)
### Install Bind9
```
root@ldap-server:~# apt-get update
root@ldap-server:~# apt-get install bind9  
```
### Konfigurasi zone
```
root@ldap-server:~# nano /etc/bind/named.conf.local
```
```
zone "ldap.lan" {
    type master;
    file "/etc/bind/db.ldap";
};
```
### Konfigurasi file db
```
root@ldap-server:~# cp /etc/bind/db.local /etc/bind/db.ldap
root@ldap-server:~# nano /etc/bind/db.ldap
```
```
;
$TTL    604800
@       IN      SOA     localhost. root.localhost. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      int-srv.ldap.lan.
@       IN      MX  10  mail.ldap.lan.
int-srv IN      A       192.168.10.1
mail    IN      A       192.168.10.2
```

```
root@ldap-server:~# service bind9 restart
```

## Konfigurasi LDAP Server
### installasi slapd
```
root@ldap-server:~# apt install slapd ldap-utils -y
```
![alt text](image-43.png)  
Masukkan Password untuk admin LDAP, masukkan saja 1234. masukkan sebanyak 2 kali  

### Konfigurasi OU(Organizational Unit) dan user LDAP

```
root@ldap-server:~# nano struktur.ldif  
```
```
root@ldap-server:~# dpkg-reconfigure slapd
```


```
dn: ou=mail,dc=ldap,dc=lan
objectClass: organizationalUnit
ou: mail

dn: uid=user1,ou=mail,dc=ldap,dc=lan
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: user1
sn: user1
uid: user1
userPassword: 1234
uidNumber: 2000
gidNumber: 2000
homeDirectory: /home/user1
```
```


