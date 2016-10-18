# Setup a Elasticsearch Cluster

Lakukan hal berikut ini sebelum mulai mengkonfigurasi server untuk elasticsearch
- Buat 3 Node Server/VM's dengan menggunakan OS(Operating System) Ubuntu Server dengan masing-masing RAM 1 GB.
- Update semua server yang sudah di install Ubuntu Server.
- Ubah hostname server ke __elastic-client01, elastic-master01, dan elastic-data01__.
- Edit file __:~$ sudo vim /etc/hosts__ tambahkan IP Node server juga Hostnamenya. Terapkan ke semua server, konfigurasi di atas sangat penting karena akan menggunakan nama host untuk komunikasi host satu sama lain.
  - 172.16.198.10 elastic-client01
  - 172.16.198.20 elastic-master01
  - 172.16.198.21 elastic-data01

## Install Java 8 di semua Node Server
Elasticsearch membutuhkan Java Runtime. Langkah selanjutnya menginstal versi java terbaru ke semua server dengan menjalankan perintah berikut:
  - Tambahkan repositori oracle Java

    > :~$ sudo add-apt-repository ppa:webupd8team/java
  - Update package list

    > :~$ sudo apt-get update
  - Install Java setelah update selesai

    > :~$ sudo apt-get install oracle-java8-installer -y
  - Setting Java environment variables

    > :~$ sudo apt-get install oracle-java8-set-default -y
  - Check apakah Java sudah terinstall dan Versi Java

    > :~$ java -version

    > :~$ javac -version

## Install Elasticsearch di semua Node Server
- Download elasticsearch

  > :~$ wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.3.5/elasticsearch-2.3.5.deb
- Install download package

  > :~$ sudo dpkg -i elasticsearch-2.3.5.deb
- Complete installation

  > :~$ sudo apt-get -f install -y
- Start elasticsearch service

  > :~$ sudo service elasticsearch start __atau__ :~$ sudo systemctl start elasticsearch
- Restart elasticsearch service

  > :~$ sudo service elasticsearch restart __atau__ :~$ sudo systemctl restart elasticsearch
- Stop elasticsearch service

  > :~$ sudo service elasticsearch stop __atau__ :~$ sudo systemctl stop elasticsearch
- Check apakah elasticsearch sudah terinstall dan jalan servicenya

  > :~$ curl 'http://localhost:9200'

## Configure Elasticsearch

### Client Node (elastic-client01)
- Edit file __:~$ sudo vim /etc/elasticsearch/elasticsearch.yml__ ikuti langkah dibawah ini
  - Uncomment __cluster.name:__ dan tambahkan nama clusternya __devops-production__
  - tambahkan __node.name: elastic-client01__
  - tambahkan __node.client: true__
  - tambahkan __node.data: false__
  - Uncomment __network.host:__ dan tambahkan IP dari node server __172.16.198.10__
  - tambahkan __discovery.zen.ping.multicast.enabled: false__
  - Uncomment __discovery.zen.ping.unicast.hosts:__ dan tambahkan node server yang akan di cluster __["elastic-client01", "elastic-master01",  "elastic-data01"]__
- Save file dan restart elasticsearch service
  - sudo service elasticsearch restart

  atau
  - sudo systemctl restart elasticsearch

### Master Node (elastic-master01)
- Edit file __:~$ sudo vim /etc/elasticsearch/elasticsearch.yml__ ikuti langkah dibawah ini
  - Uncomment __cluster.name:__ dan tambahkan nama clusternya __devops-production__
  - tambahkan __node.name: elastic-master01__
  - tambahkan __node.master: true__
  - tambahkan __node.data: false__
  - Uncomment __network.host:__ dan tambahkan IP dari node server __172.16.198.20__
  - tambahkan __discovery.zen.ping.multicast.enabled: false__
  - Uncomment __discovery.zen.ping.unicast.hosts:__ dan tambahkan node server yang akan di cluster __["elastic-client01", "elastic-master01",  "elastic-data01"]__
- Save file dan restart elasticsearch service
  - sudo service elasticsearch restart

  atau
  - sudo systemctl restart elasticsearch

### Data Node (elastic-data01)
- Edit file __:~$ sudo vim /etc/elasticsearch/elasticsearch.yml__ ikuti langkah dibawah ini
  - Uncomment __cluster.name:__ dan tambahkan nama clusternya __devops-production__
  - tambahkan __node.name: elastic-data01__
  - tambahkan __node.client: false__
  - tambahkan __node.data: true__
  - Uncomment __network.host:__ dan tambahkan IP dari node server __172.16.198.21__
  - tambahkan __discovery.zen.ping.multicast.enabled: false__
  - Uncomment __discovery.zen.ping.unicast.hosts:__ dan tambahkan node server yang akan di cluster __["elastic-client01", "elastic-master01",  "elastic-data01"]__
- Save file dan restart elasticsearch service
  - sudo service elasticsearch restart

  atau
  - sudo systemctl restart elasticsearch

__Elasticsearch boot up__
> sudo update-rc.d elasticsearch defaults 95 10

## Installing elasticsearch GUI plugin
Setelah membuat cluster elasticsearch, status cluster dapat dilihat pada __Client Node(elastic-client01)__ dengan perintah berikut
> curl http://(ip_elastic-client01]:9200/_cluster/stats

atau
> curl -XGET 'http://(ip_elastic-client01]:9200/_cluster/state?pretty'

Menginstal plugin pada __Client Node(elastic-client01)__. Untuk menginstal plugin, masuk ke directory __"/usr/share/elasticsearch/bin"__ dan jalankan perintah berikut
> __:~$ sudo ./plugin install mobz/elasticsearch-head__

Restart elasticsearch service hingga plugin berjalan normal
> __:~$ sudo service elasticsearch restart__

atau
> __:~$ sudo systemctl restart elasticsearch__

Setelah itu untuk dapat mengaksesnya lewat browser (http://(ip_elastic-client01]:9200/_plugin/head/]

## Memory Locking di Elasticsearch

Elastis menyarankan untuk menghindari swapping Elasticsearch di semua proses, karena efek negatif pada kinerja dan stabilitas. Salah satu cara menghindari swapping berlebihan adalah mengkonfigurasi Elasticsearch untuk mengunci memori yang dibutuhkan.

Setelah proses installasi elasticsearch di semua server selesai

Edit Elasticsearch configuration
> __:~$ sudo vim /etc/elasticsearch/elasticsearch.yml__

Cari baris yang berisi __bootstrap.mlockall__ dan Uncomment
> __bootstrap.mlockall: true__

Save dan exit file __elasticsearch.yml__

Selanjutnya, buka file __/etc/default/elasticsearch__ dan edit
> __:~$ sudo vim /etc/default/elasticsearch__

Cari baris yang berisi __ES_HEAP_SIZE__, Uncomment, dan setting di angka 50% dari sisa memory yang dipakai system. misal RAM total 8GB sistem hanya menggunakan 2GB, sisa memory 6GB maka __ES_HEAP_SIZE__ bisa di set 3GB
> __ES_HEAP_SIZE=2g__

Selanjutnya, Cari dan Uncomment baris yang berisi __MAX_LOCKED_MEMORY=unlimited__
> __MAX_LOCKED_MEMORY=unlimited__

Save , exit dan restart Elasticsearch service
> __:~$ sudo service elasticsearch restart__

atau
> __:~$ sudo systemctl restart elasticsearch__

Check apakah memory locking di Elasticsearch sudah jalan
> __:~$ curl http://(ip_elastic-node]:9200/_nodes/process?pretty__
