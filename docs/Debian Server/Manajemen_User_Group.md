# Manajemen User dan Group 

Sistem operasi linux merupakan sistem operasi multiuser, artinya bisa menangani lebih dari satu user dalam waktu yang bersamaan. Dengan kata lain, satu komputer dengan sistem operasi linux bisa digunakan dua orang atau lebih dengan user yang berbeda-beda dalam waktu yang sama.

Secara garis besar, user dibedakan menjadi dua, yaitu user root dan user biasa. Dimana user root mempunyai simbol (#) dan user biasa mempunyai simbol ($) pada bash. Perbedaan antara keduanya adalah dalam hal hak akses. User root memiliki hak akses maksimal didalam sistem operasi, artinya user root bisa melakukan apa saja dalam sistem operasi. Sedangkan user biasa normalnya hanya bisa melakukan modifikasi file atau direktori yang berada didalam direktori home miliknya saja (/home/nama_user).

Berikut beberapa perintah dasar yang bisa digunakan

## Adduser

root@debian:~# adduser student

``` py 
root@debian:~# adduser student
Adding user `student' ...
Adding new group `student' (1005) ...
Adding new user `student' (1003) with group `student' ...
Creating home directory `/home/student' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for student
Enter the new value, or press ENTER for the default
        Full Name []: Raihan Kamal Ibrahim
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y 
```

Perintah ini digunakan untuk menambahkan User.

## su (Switch User)

### Switch ke User Biasa
``` py
root@debian:~# su - student
student@debian:~$
```
Dengan perintah diatas kita berpindah dari user root menjadi user student.

### Switch ke user Root

``` py student@debian:~$ su -
Password:
root@debian:~#
```

Dengan perintah diatas kita telah berpindah dari user student menjadi user root.






