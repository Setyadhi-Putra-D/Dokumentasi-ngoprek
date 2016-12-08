##Build Jenkins di Docker

```bash
docker run --name devjenkins -d -p 18080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -t jenkins
```
* `50000` port mapping
* `18080` port ke public, agar developer, DevOps, PM, dkk bisa akses jenkins diluar.
* `8080` Jenkins port
* `-v jenkins_home:/var/jenkins_home` penyimpan (volume), semua data Jenkins ada disana termasuk plugin dan konfigurasi

##Akses Jenkins
```
http://IP_PUBLIC:18080
```

##Back up data
Jika saat deploy me-mount volume, dapat melakukan backup pada direktori `jenkins_home` setiap saat. Jika volume di dalam container, dapat menggunakan perintah `docker cp $ ID:/var/jenkins_home` untuk mengambil data, atau pilihan lain untuk menemukan di mana data volume.

##Accses Container
Apabila ada dependency yang belum terinstall / kurang, dapat masuk kedalam Container
* `docker exec -it [CONTAINER ID] /bin/sh`
* `docker exec -it [CONTAINER ID] /bin/bash`
* `docker exec -it -u root [CONTAINER ID] /bin/bash`
