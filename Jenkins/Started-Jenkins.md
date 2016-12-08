###Jenkins Continuous Integration dan Delivery server

###Install
####Docker
[Klik Disini](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/Jenkins/Docker/Usage.md)

[DockerFile](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/Jenkins/Docker/Usage.md)

####Ubuntu / Debian
Pada distribusi berbasis Debian, seperti Ubuntu, dapat menginstal Jenkins melalui `apt`

```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

sudo apt-get update && sudo apt-get -y install jenkins
```

Paket Instalasi :
* Pengaturan Jenkins sebagai daemon dijalankan pada startup, lihat `/etc/init.d/jenkins` untuk lebih jelasnya.
* Buat user `Jenkins` untuk menjalankan service.
* Log file `/var/log/jenkins/jenkins.log`. Periksa file ini jika terjadi masalah pada Jenkins.
* Kongifure `/etc/default/jenkins`, misal konfigurasi tempat penyimpanan `JENKINS_HOME`
* Set Jenkins pada port 8080. Akses port ini dengan browser untuk memulai konfigurasi lanjutan.
