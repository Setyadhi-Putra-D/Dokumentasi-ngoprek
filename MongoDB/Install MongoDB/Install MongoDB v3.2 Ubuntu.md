# Install MongoDB Community Edition di Ubuntu

MongoDB hanya menyediakan package untuk 64-bit LTS. Misalnya, 12.04 LTS, 14,04 LTS, 16.04 LTS, dan sebagainya.

## Step 1 : Mengimpor public key yang digunakan oleh sistem manajemen package.

> sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927

## Step 2 : Buat file list di source.list.d untuk MongoDB

__Ubuntu 12.04__

> echo "deb http://repo.mongodb.org/apt/ubuntu precise/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

__Ubuntu 14.04__

> echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

__Ubuntu 16.04__

> echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

## Step 3 : Reload local package database dan install MongoDB packages

> sudo apt-get update && sudo apt-get install -y mongodb-org

Jika ingin menginstall MongoDB per package, ada 5 package yang terdapat dalam MongoDB


No | Nama Packages | Deskripsi
--- | --- | ---
1  | mongodb-org | Sebuah metapackage yang secara otomatis akan menginstal empat package komponen yang tercantum di bawah
2  | mongodb-org-server | Berisi daemon mongod dan konfigurasi terkait dan skrip init
3  | mongodb-org-mongos | Berisi daemon mongos
4  | mongodb-org-shell | Berisi shell mongo
5  | mongodb-org-tools | Berisi MongoDB alat berikut: mongoimport bsondump, mongodump, mongoexport, mongofiles, mongooplog, mongoperf, mongorestore, mongostat, dan mongotop

Sebelumnya harus menentukan setiap package komponen individual bersama dengan nomor versi, seperti dalam contoh berikut:

> sudo apt-get install -y mongodb-org-3.2.10 mongodb-org-server-3.2.10 mongodb-org-shell-3.2.10 mongodb-org-mongos-3.2.10 mongodb-org-tools-3.2.10

## Step 4 : Check Packages MongoDB

> sudo dpkg --get-selections | grep mongodb-org

## Step 5 : (Ubuntu 16.04) buat service file

> check apakah sudah terdapat file mongod.service didalam directory /lib/systemd/system/

Jika dalam directory /lib/systemd/system/ belum ada file mongod.service maka buat file mongod.service dan ikuti srcipt di bawah ini :

> [Unit]

> Description=High-performance, schema-free document-oriented database

> After=network.target

> Documentation=https://docs.mongodb.org/manual

> [Service]

> User=mongodb

> Group=mongodb

> ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf

> [Install]

> WantedBy=multi-user.target

## Run MongoDB

### Start MongoDB
> sudo service mongod start

atau

> sudo systemctl start mongod

### Restart MongoDB
> sudo service mongod restart

atau

> sudo systemctl restart mongod

## Stop MongoDB
> sudo service mongod stop

atau

> sudo systemctl stop mongod
