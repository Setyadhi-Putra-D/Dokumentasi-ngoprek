# Install MongoDB Community Edition di Debian

MongoDB hanya menyediakan package untuk 64-bit Debian 7 “Wheezy”. Packages ini dapat bekerja di versi lain Debian, tapi ini bukan konfigurasi yang di support.

## Step 1 : Mengimpor public key yang digunakan oleh sistem manajemen package.

> sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927

## Step 2 : Buat file list di source.list.d untuk MongoDB

__Debian 7 “Wheezy”__

> echo "deb http://repo.mongodb.org/apt/debian wheezy/mongodb-org/3.2 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

Saat ini packages hanya tersedia untuk Debian 7 "Wheezy".

## Step 3 : Reload local package database dan install MongoDB packages

> sudo apt-get update && sudo apt-get install -y mongodb-org

Jika ingin menginstall MongoDB per package, ada 5 package yang terdapat dalam MongoDB

No | Nama Packages | Deskripsi
--- | --- | ---
1 | mongodb-org | Sebuah metapackage yang secara otomatis akan menginstal empat package komponen yang tercantum di bawah.
2 | mongodb-org-server | Berisi daemon mongod dan konfigurasi terkait dan skrip init.
3 | mongodb-org-mongos | Berisi daemon mongos
4 | mongodb-org-shell | Berisi shell mongo
5 | mongodb-org-tools | Berisi MongoDB alat berikut: mongoimport bsondump, mongodump, mongoexport, mongofiles, mongooplog, mongoperf, mongorestore, mongostat, dan mongotop.

Sebelumnya harus menentukan setiap package komponen individual bersama dengan nomor versi, seperti dalam contoh berikut:

> sudo apt-get install -y mongodb-org-3.2.10 mongodb-org-server-3.2.10 mongodb-org-shell-3.2.10 mongodb-org-mongos-3.2.10 mongodb-org-tools-3.2.10

## Step 4 : Check Packages MongoDB

> sudo dpkg --get-selections | grep mongodb-org

## Run MongoDB

### Start MongoDB
> sudo service mongod start

### Restart MongoDB
> sudo service mongod restart

## Stop MongoDB
> sudo service mongod stop
