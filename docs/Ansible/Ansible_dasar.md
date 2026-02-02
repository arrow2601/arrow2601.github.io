# Materi Pembelajaran Ansible Dasar

## Tujuan Pembelajaran

Setelah menyelesaikan seluruh rangkaian praktikum, siswa mampu:

* Memahami konsep dasar Ansible
* Mengelola beberapa server menggunakan inventory
* Menjalankan playbook Ansible
* Melakukan deployment service dan aplikasi secara otomatis

---

## Topologi Praktikum

* **Control Node** : Server tempat Ansible dijalankan
* **Managed Node** : Server yang dikonfigurasi oleh Ansible

Daftar managed node:

| IP Address    | Fungsi                |
| ------------- | --------------------- |
| 192.168.169.5 | Nginx Server          |
| 192.168.169.6 | Apache & MySQL Server |

---

## Persiapan Awal (Wajib)

Tahapan ini dilakukan **sebelum praktikum dimulai**.

!!! Warning
    Buat terlebih dahulu 2 container/VM baru dengan OS Debian dan IP Address sesuai tabel managed node.

---

### 1. Konfigurasi Root Login SSH (Managed Node)

Pada praktikum ini, managed node menggunakan **user root**.

Edit konfigurasi SSH:

```bash
nano /etc/ssh/sshd_config
```

Pastikan konfigurasi berikut:

```text
PermitRootLogin yes
```

Restart SSH:

```bash
systemctl restart ssh
```

---

### 2. Konfigurasi SSH Tanpa Password

#### Generate SSH Key (Control Node)

```bash
ssh-keygen
```

Tekan Enter hingga proses selesai.

#### Copy SSH Key ke Managed Node

```bash
ssh-copy-id root@192.168.169.5
ssh-copy-id root@192.168.169.6
```

#### Uji Koneksi

```bash
ssh root@192.168.169.5
ssh root@192.168.169.6
```

---

## Struktur Direktori Ansible

```bash
mkdir ansible-lab
cd ansible-lab
```

```
ansible-lab/
├── inventory
├── nginx.yml
├── apache.yml
├── mysql.yml
├── deploy-web.yml
```

---

## Inventory Multi Host

```
nano inventory
```  

```ini
[nginx]
192.168.169.5

[apache]
192.168.169.6

[database]
192.168.169.6

[all:vars]
ansible_user=root
```

---

# PRAKTIKUM 1 – Deploy Nginx

## Tujuan

Menginstall dan menjalankan Nginx menggunakan Ansible.

### Langkah 1 – Membuat Playbook

```bash
nano nginx.yml
```

Isi file berikut:

```yaml
- name: Install dan Jalankan Nginx
  hosts: nginx
  become: true

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Pastikan nginx berjalan
      service:
        name: nginx
        state: started
        enabled: yes
```

### Langkah 2 – Menjalankan Playbook

```bash
ansible-playbook -i inventory nginx.yml
```

### Langkah 3 – Verifikasi

Akses melalui browser:

```
http://192.168.169.5
```

---

# PRAKTIKUM 2 – Deploy Apache

## Tujuan

Menginstall dan menjalankan Apache Web Server menggunakan Ansible.

### Langkah 1 – Membuat Playbook

```bash
nano apache.yml
```

Isi file berikut:

```yaml
- name: Install dan Jalankan Apache
  hosts: apache
  become: true

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install apache2
      apt:
        name: apache2
        state: present

    - name: Pastikan apache berjalan
      service:
        name: apache2
        state: started
        enabled: yes
```

### Langkah 2 – Menjalankan Playbook

```bash
ansible-playbook -i inventory apache.yml
```

### Langkah 3 – Verifikasi

```
http://192.168.169.6
```

---

# PRAKTIKUM 3 – Deploy MySQL

## Tujuan

Menginstall dan menjalankan MySQL Server menggunakan Ansible.

### Langkah 1 – Membuat Playbook

```bash
nano mysql.yml
```

Isi file berikut:

```yaml
- name: Install dan Jalankan MySQL
  hosts: database
  become: true

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install mysql-server
      apt:
        name: mariadb-server
        state: present

    - name: Pastikan mysql berjalan
      service:
        name: mysql
        state: started
        enabled: yes
```

### Langkah 2 – Menjalankan Playbook

```bash
ansible-playbook -i inventory mysql.yml
```

### Langkah 3 – Verifikasi

```bash
systemctl status mysql
```

---

# PRAKTIKUM 4 – Deploy Website dari GitHub (Virtual Host dengan Ansible)

Pada praktikum ini, deployment website dilakukan menggunakan **dua playbook terpisah** agar alur pembelajaran lebih sederhana dan mudah dipahami.

* `deploy-apache-web.yml` → untuk Apache
* `deploy-nginx-web.yml` → untuk Nginx

---

## PRAKTIKUM 4A – Deploy Website di Apache (Virtual Host)

### Tujuan

Melakukan deployment website statis dari GitHub ke Apache menggunakan Virtual Host.

---

### Langkah 1 – Membuat Playbook Apache

Buat file `deploy-apache-web.yml`:

```bash
nano deploy-apache-web.yml
```

---

### Langkah 2 – Isi Playbook Apache

```yaml
- name: Deploy Website Apache dengan Virtual Host
  hosts: apache
  become: false

  tasks:
    - name: Install git
      apt:
        name: git
        state: present
        update_cache: yes

    - name: Buat direktori website
      file:
        path: /var/www/lab-apache
        state: directory

    - name: Clone repository website
      git:
        repo: https://github.com/mdn/beginner-html-site.git
        dest: /tmp/web-src
        force: yes

    - name: Copy website ke document root
      copy:
        src: /tmp/web-src/
        dest: /var/www/lab-apache/
        remote_src: yes

    - name: Konfigurasi Virtual Host Apache
      copy:
        dest: /etc/apache2/sites-available/000-default.conf
        content: |
          <VirtualHost *:80>
              DocumentRoot /var/www/lab-apache
              <Directory /var/www/lab-apache>
                  AllowOverride All
                  Require all granted
              </Directory>
          </VirtualHost>

    - name: Reload Apache
      service:
        name: apache2
        state: reloaded
```

---

### Langkah 3 – Menjalankan Playbook Apache

```bash
ansible-playbook -i inventory deploy-apache-web.yml
```

---

### Verifikasi Apache

Akses melalui browser:

```
http://192.168.169.6
```

Jika website tampil, praktikum Apache dinyatakan berhasil.

---

## PRAKTIKUM 4B – Deploy Website di Nginx (Virtual Host)

### Tujuan

Melakukan deployment website statis dari GitHub ke Nginx menggunakan Virtual Host.

---

### Langkah 1 – Membuat Playbook Nginx

Buat file `deploy-nginx-web.yml`:

```bash
nano deploy-nginx-web.yml
```

---

### Langkah 2 – Isi Playbook Nginx

```yaml
- name: Deploy Website Nginx dengan Virtual Host
  hosts: nginx
  become: false

  tasks:
    - name: Install git
      apt:
        name: git
        state: present
        update_cache: yes

    - name: Buat direktori website
      file:
        path: /var/www/lab-nginx
        state: directory

    - name: Clone repository website
      git:
        repo: https://github.com/mdn/beginner-html-site.git
        dest: /tmp/web-src
        force: yes

    - name: Copy website ke document root
      copy:
        src: /tmp/web-src/
        dest: /var/www/lab-nginx/
        remote_src: yes

    - name: Konfigurasi Virtual Host Nginx
      copy:
        dest: /etc/nginx/sites-available/default
        content: |
          server {
              listen 80 default_server;
              root /var/www/lab-nginx;
              index index.html index.htm;

              location / {
                  try_files $uri $uri/ =404;
              }
          }

    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```

---

### Langkah 3 – Menjalankan Playbook Nginx

```bash
ansible-playbook -i inventory deploy-nginx-web.yml
```

---

### Verifikasi Nginx

Akses melalui browser:

```
http://192.168.169.5
```

Jika website tampil, praktikum Nginx dinyatakan berhasil.

---

## Konsep Penting

Pada praktikum ini:

* Tidak menggunakan DNS server
* Tidak menggunakan file `/etc/hosts`
* Virtual host dikonfigurasi sebagai **default server**

Dengan konfigurasi ini, akses ke IP server akan langsung menampilkan website hasil deployment.

---

## Langkah 1 – Menyiapkan Direktori Website

Website akan dideploy ke direktori berikut:

* Apache  : `/var/www/lab-apache`
* Nginx   : `/var/www/lab-nginx`

---

## Langkah 2 – Konfigurasi Virtual Host Apache

Virtual host Apache dibuat tanpa `ServerName`.

Isi file konfigurasi:

```apache
<VirtualHost *:80>
    DocumentRoot /var/www/lab-apache

    <Directory /var/www/lab-apache>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Konfigurasi ini memastikan Apache menampilkan website ketika diakses melalui IP Address.

---

## Langkah 3 – Konfigurasi Virtual Host Nginx

Server block Nginx dijadikan sebagai **default_server**.

Isi konfigurasi:

```nginx
server {
    listen 80 default_server;

    root /var/www/lab-nginx;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## Langkah 4 – Deploy Website dari GitHub

Repository yang digunakan:

```
https://github.com/mdn/beginner-html-site.git
```

Playbook `deploy-web.yml` digunakan untuk:

* Install Git
* Clone repository
* Copy file ke direktori virtual host
* Reload service web

Jalankan playbook:

```bash
ansible-playbook -i inventory deploy-web.yml
```

---

## Langkah 5 – Verifikasi

Akses website menggunakan browser:

**Apache**

```
http://192.168.169.6
```

**Nginx**

```
http://192.168.169.5
```

Jika halaman website tampil, maka praktikum dinyatakan berhasil.

---

# CHALLENGE AKHIR – UJI KOMPETENSI ANSIBLE

## Deskripsi Challenge

Pada challenge ini, siswa diminta melakukan **full stack deployment** pada satu node baru menggunakan Ansible.

---

## Skenario Challenge

Tambahkan 1 managed node baru:

| IP Address    | Fungsi            |
| ------------- | ----------------- |
| 192.168.169.7 | Web + PHP + MySQL |

---

## Repository Aplikasi

Gunakan repository berikut:

```
https://github.com/dhiraj-01/Todo-PHP
```

---

## Tugas Challenge

1. Tambahkan node ke inventory
2. Install Nginx, PHP, dan MySQL
3. Deploy aplikasi PHP
4. Konfigurasi database lokal
5. Pastikan aplikasi dapat diakses melalui browser

---

## Verifikasi

```
http://192.168.169.7
```

Jika aplikasi dapat diakses dengan baik, maka challenge dinyatakan **berhasil**.

---

**Akhir Materi**
