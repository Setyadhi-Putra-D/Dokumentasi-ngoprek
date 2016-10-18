# Step by step MongoDB sharded cluster deployment

Tutorial ini berisi urutan-urutan proses membuat mongodb cluster dengan cara sharding. Mongodb yang digunakan adalah versi 3.2, dalam tutorial ini kita menggunakan 2 os ubuntu 14.04 dan 16.04

__Referensi:__
- Dokumentasi resmi dari MongoDB tentang MongoDB sharded cluster  
[Install MongoDB](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/tree/master/MongoDB/Install%20MongoDB)

__Spesifikasi Minimal:__
- 3 node Desktop/Server (2 node Desktop/Server 2core and 4GB RAM | 1 node Desktop/Server 4core and 8GB RAM)
- OS(Operating System) 14.04 LTS Trusty and 16.04 LTS Xenial

__Topology Sistem:__

![alt tag](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/MongoDB/Cluster MongoDB/Asset/sharded-cluster-production-architecture.png)


Type server | Component install | Deskripsi | Ip address | Hostname
--- | --- | --- | --- | ---
App Server / Router / Mongos | MongoDB, Apps , mongos service | Server ini akan berperan ganda server aplikasi serta server Mongos | 172.16.198.30 | mongos01
Server Config | Server MongoDB Config | Digunakan sebagai mongodb config server | 172.16.198.20 | mongoconfig01
Shard sever (primary) | MongoDB | DB server | 172.16.198.10 | mongod01


## Step 1: Configure hostname disemua server
Setiap node server harus disetting hostname sesuai dengan spesifikasi / requirement yang sudah dibuat

__:~$ sudo hostname (nama_hostname)__

__:~$ sudo hostnamectl --set-hostname (nama_hostname)__

Edit juga file __:~$ sudo vim /etc/hosts__ dan tambah __hostname__ lalu restart node servernya

## Step 2: Install mongo DB di semua servers
Dokumentasi resmi dari MongoDB tentang cara menginstall MongoDB pada OS(Operating System)

[ https://docs.mongodb.com/manual/administration/install-community/ ]

## Step 3: Config Shard 1 & Shard 2 DLL replica set
Jika ada 1 node server, maka status server tersebut adalah PRIMARY. Jika ada < 1, maka salah satu node server menjadi PRIMARY dan lainnya menjadi Secondary. Kita bisa memberi nama untuk replikasi dengan “__rsn0__”

### Step 3.1: Tambah hostname secara urut untuk yang memiliki Shard Server < 1
Edit file __:~$ sudo vim /etc/hosts__ dan tambahkan IP Node server dan Hostnamenya contoh :  
- 172.16.198.10    __mongod01__
- 172.16.198.11    __mongod02__
- 172.16.198.12    __mongod03__

### Step 3.2: Edit configuration file dari mongodb
Edit file __:~$ sudo vim /etc/mongod.conf__ ikuti langkah dibawah ini
- Ubah __bindIp__ ke __0.0.0.0__
- Uncomment __replication:__ dan tambahkan __replSetName: rsn0__
- Overall configuration file
  - __uncomment storage:__
    - __dbPath: /var/lib/mongodb__
    - __journal: enabled: true__
  - __uncomment systemLog:__
    - __destination: file__
    - __logAppend: true__
    - __path: /var/log/mongodb/mongod.log__
  - __uncomment net:__
    - __port: 27017__
    - __bindIp: 0.0.0.0__
  - __uncomment replication:__
    - __replSetName: rsn0__

![alt tag](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/MongoDB/Cluster MongoDB/Asset/mongod.png)

### Step 3.3: Restart mongo db service
Setelah selesai config lalu restart service mongod nya

__:~$sudo service mongod restart__

__:~$sudo systemctl restart mongod__

### Step 3.4: Configure the replica set
Pada tahap ini, kita akan mengaktifkan replication pada mongo. Perintah replication ini dilakukan pada Primary node ,  dari mulai shard single node / multi node. ikuti langkah-langkah berikut untuk mengkonfigurasi set replika
- Connect ke mongodb > __:~$mongo__
- Config MongoDB rsn0:> __rs.initiate()__
- Config MongoDB rsn0:> __rs.add( "mongod02" )__
- Config MongoDB rsn0:> __rs.add( "mongod03" )__

Check replica set status dengan command rsn0:PRIMARY> __rs.status()__

## Step 4: Configure Mongo config servers replica set
Jika ada 1 node server, maka status server tersebut adalah PRIMARY. Jika ada < 1, maka salah satu node server menjadi PRIMARY dan lainnya menjadi Secondary. Kita bisa memberi nama untuk replikasi dengan “__configReplSet__”

### Step 4.1: Tambah hostname semua node secara urut
Edit file __:~$ sudo vim /etc/hosts__ dan tambahkan IP Node server dan Hostnamenya contoh :  
- 172.16.198.10    __mongod01__
- 172.16.198.11    __mongod02__
- 172.16.198.12    __mongod03__
- 172.16.198.20    __mongoconfig01__
- 172.16.198.21    __mongoconfig02__
- 172.16.198.22    __mongoconfig03__  
- 172.16.198.30    __mongos01__
- 172.16.198.31    __mongos02__

### Step 4.2: Edit configuration file dari mongodb
Edit file __:~$ sudo vim /etc/mongod.conf__ ikuti langkah dibawah ini
- Ubah __bindIp__ ke __0.0.0.0__
- Ubah __port__ ke __27019__
- Uncomment __replication:__ dan tambahkan __replSetName: configReplSet__
- Uncomment __sharding:__ dan tambahkan __clusterRole: "configsvr"__
- Overall configuration file
  - __uncomment storage:__
    - __dbPath: /var/lib/mongodb__
    - __journal: enabled: true__
  - __uncomment systemLog:__
    - __destination: file__
    - __logAppend: true__
    - __path: /var/log/mongodb/mongod.log__
  - __uncomment net:__
    - __port: 27019__
    - __bindIp: 0.0.0.0__
  - __uncomment replication:__
    - __replSetName: configReplSet__
  - __uncomment sharding:__
    - __clusterRole: "configsvr"__

![alt tag](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/MongoDB/Cluster MongoDB/Asset/mongo-config-server.png)

### Step 4.3: Restart mongo db service
Setelah selesai config lalu restart service mongod nya

__:~$sudo service mongod restart__

__:~$sudo systemctl restart mongod__

### Step 4.4: Configure the replica set
Pada tahap ini, kita akan mengaktifkan replication pada mongo. Perintah replication ini dilakukan pada Primary node ,  dari mulai config server single node / multi node. ikuti langkah-langkah berikut untuk mengkonfigurasi set replika
- Connect ke mongodb > __:~$mongo mongoconfig01:27019__
- Config MongoDB configReplSet:> __rs.initiate( { _id: "configReplSet", configsvr: true, members: [ { _id: 0, host: "mongoconfig01:27019" }, { _id: 1, host: "config_server:port" }, { _id: 2, host: "config_server:port" } ] } )__

Check replica set status dengan command configReplSet:PRIMARY> __rs.status()__

## Step 5: Configure Mongos servers
Ikuti beberapa tahap untuk konfigurasi mongos / router, jika ada 2 mongos maka bisa menambah metode load balancing

### Step 5.1: Tambah hostname semua node secara urut
Edit file __:~$ sudo vim /etc/hosts__ dan tambahkan IP Node server dan Hostnamenya contoh :  
- 172.16.198.10    __mongod01__
- 172.16.198.11    __mongod02__
- 172.16.198.12    __mongod03__
- 172.16.198.20    __mongoconfig01__
- 172.16.198.21    __mongoconfig02__
- 172.16.198.22    __mongoconfig03__  
- 172.16.198.30    __mongos01__
- 172.16.198.31    __mongos02__

### Step 5.2: Edit configuration file dari mongodb
Edit file __:~$ sudo vim /etc/mongod.conf__ ikuti langkah dibawah ini
- Ubah __bindIp__ ke __0.0.0.0__
- __Comment__ script configurasi __storage:__
- Uncomment __sharding:__ dan tambahkan __configDB: configReplSet/mongoconfig01:27019, config_server:port__
- Overall configuration file
  - __#storage:__
    - __#dbPath: /var/lib/mongodb__
    - __#journal: enabled: true__
  - __uncomment systemLog:__
    - __destination: file__
    - __logAppend: true__
    - __path: /var/log/mongodb/mongod.log__
  - __uncomment net:__
    - __port: 27017__
    - __bindIp: 0.0.0.0__
  - __uncomment sharding:__
    - __configDB: configReplSet/mongoconfig01:27019, config_server:port__

![alt tag](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/MongoDB/Cluster MongoDB/Asset/mongo-mongos.png)

### Step 5.3: Setup mongos as a service
- __Ubuntu 16.04 (systemd)__
    - duplicate file service mongod ke mongos
      - :~$ sudo cp /lib/systemd/system/__mongod.service__ ke /lib/systemd/system/__mongos.service__
    - edit file mongos.service
      - :~$ sudo vim /lib/systemd/system/__mongos.service__
      - ubah pada bagian __ExecStart=/usr/bin/mongod__ ke __ExecStart=/usr/bin/mongos__
- __Ubuntu 14.04__
    - duplicate file service mongod ke mongos
      - :~$ sudo cp /etc/init/__mongod.conf__ ke /etc/init/__mongos.conf__
    - edit file mongos.conf
      - :~$ sudo vim /etc/init/__mongod.conf__
      - ubah pada bagian __DAEMON=/usr/bin/mongod__ ke __DAEMON=/usr/bin/mongos__
      - ubah pada bagian __if [ -f /etc/default/mongod ]; then . /etc/default/mongod; fi__ ke __if [ -f /etc/default/mongos ]; then . /etc/default/mongos; fi__

![alt tag](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/MongoDB/Cluster MongoDB/Asset/mongo-mongos-service.png)

![alt tag](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/MongoDB/Cluster MongoDB/Asset/mongo-mongos-service2.png)

### Step 5.4: Restart mongodb service
setelah selesai config lalu restart service mongod dan mongos nya
- __:~$sudo service mongod restart__
- __:~$sudo systemctl restart mongod__
- __:~$sudo service mongos restart__
- __:~$sudo systemctl restart mongos__

### Step 5.5: Configure the shards
ikuti langkah-langkah berikut untuk mengkonfigurasi
- Connect ke mongodb > __:~$mongo mongos01:27017__
- Config MongoDB mongos:> __sh.addShard( “rsn0/mongod01:27017” )__
- Config MongoDB mongos:> __sh.addShard( “replication_shard/primary_shard_server:port” )__

Check status dengan command
- mongos:> __sh.status()__

__atau__

Check status dengan command
- mongos:> __use admin__
- mongos:> __db.runCommand("getShardMap")__

### Step 5.6: Cluster Balancer (optional)
Metode ini untuk membuat Load Balance antara mongos01 dan mongos02
- Connect ke mongodb > __:~$mongo mongos01:27017__
- Config MongoDB mongos:> __sh.getBalancerState()__
