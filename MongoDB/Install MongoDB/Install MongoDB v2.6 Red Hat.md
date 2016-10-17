# Install MongoDB Community Edition di Rad Hat atau CentOS

MongoDB hanya menyediakan package untuk Red Hat Enterprise Linux atau CentOS Linux versi 6 and 7 menggunakan .rpm packages.

## Step 1 : Konfigurasi sistem manajemen package (yum)
- Buatlah file mongodb-org-2.6.repo di directory /etc/yum.repos.d/
- Gunakan file repositori berikut:

>[mongodb-org-2.6]

>name=MongoDB 2.6 Repository

>baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/

>gpgcheck=0

>enabled=1

## Step 2 : Install MongoDB packages dan Tools
Install versi stabil dari MongoDB, ikuti command:
> sudo yum install -y mongodb-org

Jika ingin menginstall MongoDB per package, ada 5 package yang terdapat dalam MongoDB

No | Nama Packages | Deskripsi
--- | --- | ---
1  | mongodb-org | Sebuah metapackage yang secara otomatis akan menginstal empat package komponen yang tercantum di bawah
2  | mongodb-org-server | Berisi daemon mongod dan konfigurasi terkait dan skrip init
3  | mongodb-org-mongos | Berisi daemon mongos
4  | mongodb-org-shell | Berisi shell mongo
5  | mongodb-org-tools | Berisi MongoDB alat berikut: mongoimport bsondump, mongodump, mongoexport, mongofiles, mongooplog, mongoperf, mongorestore, mongostat, dan mongotop

Sebelumnya harus menentukan setiap package komponen individual bersama dengan nomor versi, seperti dalam contoh berikut:
> sudo yum install -y mongodb-org-2.6.10 mongodb-org-server-2.6.10 mongodb-org-shell-2.6.10 mongodb-org-mongos-2.6.10 mongodb-org-tools-2.6.10

## Step 3 : Check Packages MongoDB
> sudo rpm -qa | grep mongodb-org

## Run MongoDB

### Konfigurasi SELinux
Jika menggunakan SELinux, harus mengkonfigurasi SELinux untuk memungkinkan MongoDB memulai pada sistem berbasis Linux Red Hat (Red Hat Enterprise Linux atau CentOS Linux).

__ada tiga opsi untuk konfigurasi SELinux:__

- Menghidupkan akses ke port yang digunakan MongoDB
> semanage port -a -t mongod_port_t -p tcp 27017

- Mematikan SELinux, setting SELinux dengan edit file di /etc/selinux/config
> SELINUX=disabled

- Set SELINUX ke mode __permissive__
> SELINUX=permissive

__Restart system dan lihat efeknya__

### Start MongoDB
> sudo service mongod start

### Restart MongoDB
> sudo service mongod restart

### Stop MongoDB
> sudo service mongod stop
