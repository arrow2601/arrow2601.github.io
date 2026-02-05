# Ansible dasar Part2 

!!! Note
    Nah sekarang kita akan mengulangi apa yang sudah kita lakukan di `node1` pada `node2` namun semua langkah kita lakukan dari `control node`

## Mengurutkan langkah yang harus di lakukan pada Node2
!!! Note
    Sebelum melakukan konfigurasi dari `control node` terlebih dahulu kita urutkan langkah-langkah yang telah kita lakukan pada `node1` agar nantinya langkah-langkah tersebut kita `convert` kedalam `playbook`
    1. update repository
    2. Install Curl
    3. Download NodeJS Setup
    4. Install NodeJS
    5. Install git
    6. Clone Repo
    7. Install MySql
    8. Import Database
    9. Membuat user mysql Baru
    10. memberikan kewenangan (Grant Privilege)
    11. Mengedit Konfigurasi database di NodeJS app
    12. npm install
    13. npm install -g pm2
    14. Menjalankan app dengan pm2
    15. simpan konfigurasi pm2
    dari sini kita akan membuat 3 playbook :
    1. installasi kebutuhan dan git repo
    2. perdatabase-an
    3. menjalankan App

## Membuat folder baru dan konfigurasi inventory
```
root@pve:~# mkdir nodejs
root@pve:~# cd nodejs
root@pve:~/nodejs# nano inventory
```
```
[nodejs]
node2 ansible_host=192.168.169.8 ansible_user=root
```
!!! Note
    Inventory merupakan file yang berisi detail dari `managed node` disini kita membuat group `[nodejs]` didalam group tersebut ada `node2` berikut dengan ip addressnya. nantinya didalam playbook `managed host` dapat kita panggil bisa dengan `group`nya atau hostnamenya yakni `node2`.

```
root@pve:~/nodejs# ansible -i inventory nodejs -m ping
```
## Membuat Playbook 1 - Installasi+repo
```
root@pve:~/nodejs# nano playbook-01-install.yml
```
```
- name: Install NodeJS, Git, MariaDB, dan clone repo
  hosts: nodejs
  become: true

  tasks:
    - name: Update repository
      apt:
        update_cache: yes

    - name: Install dependencies dasar
      apt:
        name:
          - curl
          - git
        state: present

    - name: Download NodeJS setup script
      shell: curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -
      args:
        creates: /etc/apt/sources.list.d/nodesource.list

    - name: Install NodeJS
      apt:
        name: nodejs
        state: present

    - name: Install MariaDB Server
      apt:
        name: mariadb-server
        state: present

    - name: Pastikan MariaDB running
      service:
        name: mariadb
        state: started
        enabled: true

    - name: Clone repository CRUD NodeJS
      git:
        repo: https://github.com/fazt/crud-nodejs-mysql.git
        dest: /root/crud-nodejs-mysql
        force: yes
```
## Jalankan Playbook1
```
root@pve:~/nodejs# ansible-playbook -i inventory playbook-01-install.yml
```

