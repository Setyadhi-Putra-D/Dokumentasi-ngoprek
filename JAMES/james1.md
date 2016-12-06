----
#Setup JAMES (Java Apache Mail Enterprise Server) di CentOS 7
----
##Step 0 : Requirements

####Install JAVA (JDK & JRE) di CentOS 7
* Download JAVA di [Download Page](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* Install Package JAVA `~]# rpm -ivh jdk-8uxxx-linux-x64.rpm`
* Check JAVA Version `​~]# java -version && javac -version`
* Setting setup global environment variables pakai bash script
  * Buat file untuk setting environment variables java `~]# vim /etc/profile.d/java.sh`
  * Tambahkan bash script dibawah ini
  ```bash
  #!/bin/bash
  JAVA_HOME=/usr/java/jdk1.8.0_xxx
  PATH=$JAVA_HOME/bin:$PATH
  export PATH JAVA_HOME
  export CLASSPATH=.
  ```
  * Setting permision file `java.sh` dengan command `~]# chmod +x /etc/profile.d/java.sh`
  * Setting environment variables agar permanen saat startup `~]# source /etc/profile.d/java.sh`

####Install Dependency
* GIT
  * `~]# yum -y update && yum -y install git`
* libc6
  * `~]# yum -y update && yum -y install glibc-devel glibc-devel.i686`

####System Resources
`~]# export MAVEN_OPTS="-Xmx2048m -XX:MaxPermSize=1024m"`

##Step 1 : Download dan Build JAMES
* Masuk ke directory opt `~]# cd /opt/`
* Clone source code dari repository JAMES di Github official `~]# git clone https://github.com/apache/james-project.git`
* Masuk ke directory hasil clone `~]# cd /opt/james-project/`
* Compile dan Build JAMES `~]# mvn package -DskipTests=true -Pwith-assembly`
* Hasil compile dan build di directory `/opt/james-project/server/app/target/`

##Step 2 : Configure
Copy file ber-extensi `.tar.gz / .zip` hasil compile dan build JAMES dari directory `/opt/james-project/server/app/target/` ke directory `/opt` lalu extract file tadi.
```
~]# cp /opt/james-project/server/app/target/james-server-app-3.0.0-betax-SNAPSHOT-app.tar.gz /opt/
~]# tar -zxvf james-server-app-3.0.0-betax-SNAPSHOT-app.tar.gz
```
Masuk ke directory `~]# cd /opt/james-server/conf/` dalam directory conf semua file konfigurasi memiliki tambahan nama `....-template.xml`, lalu rename file konfigurasi untuk menghilangkan `template` ada beberapa file utama yang digunakan agar server JAMES jalan.
```bash
~]# cp events-template.xml events.xml
~]# cp imapserver-template.xml imapserver.xml
~]# cp log4j-template.properties log4j.properties
~]# cp mailetcontainer-template.xml mailetcontainer.xml
~]# cp managesieveserver-template.xml managesieveserver.xml
~]# cp pop3server-template.xml pop3server.xml
~]# cp quota-template.xml quota.xml
~]# cp smtpserver-template.xml smtpserver.xml
~]# cp sqlResources-template.xml sqlResources.xml
```
Konfigurasi pertama, akan merubah nilai port dari settingan defaultnya. Port smtpserver, port pop3server, dan port imapserver
* Pertama ubah port pada file `smtpserver.xml`
  ```xml
  <bind>0.0.0.0:10025</bind>
  ```
* Ubah port pada file `pop3server.xml`
  ```xml
  <bind>0.0.0.0:10110</bind>
  ```
* Ubah port pada file `imapserver.xml`
  ```xml
  <bind>0.0.0.0:10143</bind>
  ```

Setelah selesai mengedit port pindah ke directory `~]# cd /opt/james-server/bin/` dan jalankan service JAMES `~]# ./james start`, ada 6 fungsi yang bisa dijalankan pada JAMES `Usage: ./james { console | start | stop | restart | status | dump }`, Jika JAMES sudah berjalan check portnya `~]# netstat -tupln`. Setelah itu jika sudah up dan running test JAMES dengan menggunakan command `telnet localhost 10025`, jika ada error / tidak dapat connect ke JAMES coba check `wrapper.log` pada directory `~]# vim /opt/james-server/log/wrapper.log`.
```
[root@james ~]# telnet localhost 10025
Trying ::1...
Connected to localhost.
Escape character is '^]'.
220 james james SMTP Server Server (james SMTP Server ) ready
QUIT
221 2.0.0 james Service closing transmission channel
Connection closed by foreign host.
```

##Step 3 : Buat Account User and Domain
Domain yang dipakai adalah subdomain `james.tealinuxos.org` dan menggunakan user untuk testing JAMES. Masuk ke directory `~]# cd /opt/james-server/bin/` dan jalankan command untuk menambah user dan domain
```
~]# ./james-cli.sh -h localhost -p 9999 adddomain james.tealinuxos.org && ./james-cli.sh -h localhost -p 9999 adduser username@james.tealinuxos.org password
```

##Step Testing
* Test dengan menggunakan Thunderbird
  * E-mail Address : `user@james.tealinuxos.org`
  * Password E-mail : `password`
  * Incoming > Protocol : `IMAP` > Port : `10143` > Hostname : `james.tealinuxos.org`
  * Outgoing > Protocol : `SMTP` > Port : `10025` > Hostname : `james.tealinuxos.org`
* Coba kirim e-mail dari JAMES ke Gmail

##Konfigurasi Tambahan

####Menambahkan pengait yang membatalkan eksekusi shutdown untuk Derby
Secara default, file `db.lck` pada directory `/opt/james-server/var/store/derby/` akan menghilang terhapus setelah JAMES shutdown. Berarti JAMES tidak mengeksekusi dengan benar prosedur shutdown dari Derby. Hal itu dapat menyebabkan kerusakan data dari database Derby. Menambahkan pengait sehingga Derby dapat dengan baik mengeksekusi shutdown pada saat JAMES shutdown.
* Clone, build jar, dan copy ke directory `/opt/james-server/conf/lib/`
  ```
  ~]# git clone https://github.com/lbtc-xxx/derby-shutdown-bean
  ~]# cd derby-shutdown-bean
  ~]# mvn clean package
  ~]# cp target/derby-shutdown-bean.jar /opt/james-server/conf/lib/
  ```
* Buat folder di directory `/opt/james-server/conf/META-INF/org/apache/james/` dan copy kan `spring-server.xml`
  ```
  ~]# pwd
  /opt/james-server/conf/META-INF
  ~]# mkdir -p org/apache/james
  ~]# cp /opt/james-project/server/container/spring/src/main/resources/META-INF/org/apache/james/spring-server.xml /opt/james-server/conf/META-INF/org/apache/james
  ```
* Configure file `~]# vim /opt/james-server/conf/META-INF/org/apache/james/spring-server.xml`
  ```xml
  <bean id="derbyShutdownSpringBean" class="org.nailedtothex.derby.DerbyShutdownSpringBean"/>
  ```
* Configure file `~]# vim /opt/james-server/conf/log4j.properties`
  ```
  log4j.logger.org.nailedtothex.derby=INFO, CONS, FILE
  ```

####Menyesuaikan timeout Java Service Wrapper
Edit file `wrapper.conf` pada directory `/opt/james-server/conf/`
```
wrapper.jvm_exit.timeout=120
wrapper.startup.timeout=120
wrapper.shutdown.timeout=120
wrapper.ping.timeout=300
```
####Restart JAMES

###Referensi
* (http://wrapper.tanukisoftware.com/doc/english/qna-freeze.html)
* (http://wrapper.tanukisoftware.com/doc/english/prop-jvm-exit-timeout.html)
* (http://wrapper.tanukisoftware.com/doc/english/prop-shutdown-timeout.html)
* (http://wrapper.tanukisoftware.com/doc/english/prop-ping-timeout.html)
* (http://james.apache.org/server/3/quick-start.html)

----
#Updating dan Configuring Derby
----
* Download lib distribution dari Apache Derby [Download Page](http://archive.apache.org/dist/db/derby/db-derby-10.11.1.1/db-derby-10.11.1.1-lib.tar.gz)
* Extract hasil dari download `~]# tar -zxvf db-derby-10.11.1.1-lib.tar.gz`
* File yang digunakan `derby.jar` dan `derbynet.jar` untuk menerima koneksi jaringan dan copy 2 file jar di directory `/opt/james-server/lib/`
  ```
  ~]# pwd
  /opt
  ~]# cp db-derby-10.11.1.1-lib/lib/derby.jar james-server/lib
  ~]# cp db-derby-10.11.1.1-lib/lib/derbynet.jar james-server/lib
  ```
* Definisikan file `derby.jar` dan `derbynet.jar` ke file konfigurasi `wrapper.conf`
  ```
  wrapper.java.classpath.94=%REPO_DIR%/derby.jar
  wrapper.java.classpath.131=%REPO_DIR%/derbynet.jar
  ```
* Konfigurasi `derby.log` dan beberapa tambahan untuk derby
  ```
  # Java Additional Parameters
  # wrapper.java.additional.1=
  wrapper.java.additional.18=-Dderby.infolog.append=true
  wrapper.java.additional.17=-Dderby.stream.error.file=../log/derby.log
  wrapper.java.additional.16=-Dderby.drda.portNumber=11527
  wrapper.java.additional.15=-Dderby.drda.startNetworkServer=true
  ```
* Restart James server, untuk log lokasinya ada di directory `/opt/james-server/log`

----
#DKIM JAMES
----
* Download package Apache James DKIM [Download Page](http://mirror.wanxp.id/apache//james/jdkim/apache-jdkim-0.2-bin.tar.gz)
* Extract hasil dari download `~]# tar -zxvf apache-jdkim-0.2-bin.tar.gz`
* Copy beberapa file lib `.jar` dan paste file `.jar` pada directory `/opt/james-server/lib/`
  ```
  ~]# pwd
  /opt/apache-jdkim-0.2
  ~]# cp lib/apache-jdkim-library-0.2.jar /opt/james-server/conf/lib
  ~]# cp lib/apache-jdkim-mailets-0.2.jar /opt/james-server/conf/lib
  ~]# cp lib/not-yet-commons-ssl-0.3.11.jar /opt/james-server/conf/lib
  ```
* Membuat key pair ada dua `dkim-public.pem` dan `dkim-private.pem`, public untuk DNS Record dan Private untuk server james
  ```
  ~]# pwd
  /opt/apache-jdkim-0.2
  ~]# openssl genrsa -out dkim-private.pem 1024 -outform PEM
  ~]# openssl rsa -in dkim-private.pem -out dkim-public.pem -pubout -outform PEM
  ```
* Menambahkan / register public key ke DNS Record
