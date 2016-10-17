# Setup a Elasticsearch Cluster

Lakukan hal berikut ini sebelum mulai mengkonfigurasi server untuk elasticsearch
- Buat 3 Node Server/VM's dengan menggunakan OS(Operating System) Ubuntu Server dengan masing-masing RAM 1 GB.
- Update semua server yang sudah di install Ubuntu Server.
- Ubah hostname server ke elastic-client01, elastic-master01, dan elastic-data01.
- Edit file __:~$ sudo vim /etc/hosts__ tambahkan IP Node server juga Hostnamenya. Terapkan ke semua server, konfigurasi di atas sangat penting karena akan menggunakan nama host untuk komunikasi host satu sama lain.
  - 172.16.198.10 elastic-client01
  - 172.16.198.20 elastic-master01
  - 172.16.198.21 elastic-data01

## Install Java 8 
Elasticsearch membutuhkan Java Runtime. Langkah selanjutnya menginstal versi java terbaru dengan menjalankan perintah berikut:
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

## Install Elasticsearch
- Download elasticsearch

  > :~$ wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.3.5/elasticsearch-2.3.5.deb
- Install download package

  > :~$ sudo dpkg -i elasticsearch-2.3.5.deb
- Complete installation

  > :~$ sudo apt-get -f install -y
- Start elasticsearch service

  > :~$ sudo service elasticsearch start atau :~$ sudo systemctl start elasticsearch
- Restart elasticsearch service

  > :~$ sudo service elasticsearch restart atau :~$ sudo systemctl restart elasticsearch
- Stop elasticsearch service

  > :~$ sudo service elasticsearch stop atau :~$ sudo systemctl stop elasticsearch
- Check apakah elasticsearch sudah terinstall dan jalan servicenya

  > :~$ curl 'http://localhost:9200'

## Configure Elasticsearch

