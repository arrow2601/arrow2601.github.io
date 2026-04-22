# High Availability Web Server

## Konfigurasi IP Address & Hostname  
### DNS Server  
#### Set Hostname  
```
hostnamectl set-hostname ns1
```  
#### Konfigurasi Network
```
nano /etc/network/interfaces
```  
```
# Interface Internet
auto enp0s3
iface enp0s3 inet dhcp

# Interface Lokal
auto enp0s8
iface enp0s8 inet static
    address 192.168.10.1
    netmask 255.255.255.0  
```
```  
systemctl restart networking  
```  

### Web-01  
#### Set Hostname  
```
hostnamectl set-hostname web01  
```
#### Konfigurasi Network  
```
nano /etc/network/interfaces  
```  
```  
auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
    address 192.168.10.2
    netmask 255.255.255.0  
```  
```  
systemctl restart networking  
```  
### Web-02  
#### set hostname  
```
hostnamectl set-hostname web02  
```  
#### konfigurasi network  
```  
nano /etc/network/interfaces  
```  
```  
auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
    address 192.168.10.3
    netmask 255.255.255.0  
```  
```  
systemctl restart networking  
```  

## Installasi dan Konfigurasi DNS (ns1)
### Install Bind9  
```
apt update && apt install bind9 -y  
```  
### Membuat Zone  
```  
nano /etc/bind/named.conf.local  
```  
```  
zone "web.lan" {
    type master;
    file "/etc/bind/db.web";
};  
```  
### Konfigurasi File DB  
```  
nano /etc/bind/db.web  
```  
```  
$TTL    604800
@       IN      SOA     ns1.web.lan. root.web.lan. (
                      1         ; Serial
                 604800         ; Refresh
                  86400         ; Update
                2419200         ; Expire
                 604800 )       ; Negative Cache
;
@       IN      NS      ns1.web.lan.
ns1     IN      A       192.168.10.1
www     IN      A       192.168.10.100  
```  
```
systemctl restart bind9  
```  
## Installasi Web Server (Web01 & Web02)  
Jalankan perintah ini di kedua server web:  
```
apt update && apt install nginx -y  
```  
### Membuat halaman web  
pada web 1  
```
echo "<h1>WEB 01 - MASTER</h1>" > /var/www/html/index.html  
```  
pada web2  
```  
echo "<h1>WEB 02 - SLAVE</h1>" > /var/www/html/index.html  
```  

## Konfigurasi KeepAlived( High Availability)  
install di Web01 dan Web02  
```
apt install keepalived -y  
```  
### Konfigurasi KeepAlived Master (web01)  
``` 
nano /etc/keepalived/keepalived.conf  
```  
```  
vrrp_instance VI_1 {
    state MASTER
    interface enp0s8
    virtual_router_id 51
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123
    }
    virtual_ipaddress {
        192.168.10.100
    }
}  
```  
### KOnfigurasi KeepAlived Master (Web02) 
``` 
vrrp_instance VI_1 {
    state BACKUP
    interface enp0s8
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123
    }
    virtual_ipaddress {
        192.168.10.100
    }
}  
```  
``` 
systemctl restart keepalived  
```  
##  Pengujian  
1. Akses www.web.lan dari client (pastikan DNS client diarahkan ke 192.168.10.1).  
2. Harusnya muncul konten dari Web01.  
3. Matikan Web01 (systemctl stop nginx atau matikan VM).  
4. Refresh browser, konten harusnya berubah menjadi Web02 secara otomatis tanpa mengganti IP/Domain.  

