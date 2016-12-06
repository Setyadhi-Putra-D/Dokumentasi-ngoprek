----
#Setup JAMES (Java Apache Mail Enterprise Server) di CentOS 7

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
```bash
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
```bash
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
```bash
~]# ./james-cli.sh -h localhost -p 9999 adddomain james.tealinuxos.org && ./james-cli.sh -h localhost -p 9999 adduser username@james.tealinuxos.org password
```

##Step Testing
* Test dengan menggunakan Thunderbird
  * E-mail Address : `user@james.tealinuxos.org`
  * Password E-mail : `password`
  * Incoming > Protocol : `IMAP` > Port : `10143` > Hostname : `james.tealinuxos.org`
  * Outgoing > Protocol : `SMTP` > Port : `10025` > Hostname : `james.tealinuxos.org`
* Coba kirim e-mail dari JAMES ke Gmail

![alt tag](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/JAMES/asset/2016-11-24-173239_1920x1080_scrot.png)

##Konfigurasi Tambahan

####Menambahkan pengait yang membatalkan eksekusi shutdown untuk Derby
Secara default, file `db.lck` pada directory `/opt/james-server/var/store/derby/` akan menghilang terhapus setelah JAMES shutdown. Berarti JAMES tidak mengeksekusi dengan benar prosedur shutdown dari Derby. Hal itu dapat menyebabkan kerusakan data dari database Derby. Menambahkan pengait sehingga Derby dapat dengan baik mengeksekusi shutdown pada saat JAMES shutdown.
* Clone, build jar, dan copy ke directory `/opt/james-server/conf/lib/`

  ```bash
  ~]# git clone https://github.com/lbtc-xxx/derby-shutdown-bean
  ~]# cd derby-shutdown-bean
  ~]# mvn clean package
  ~]# cp target/derby-shutdown-bean.jar /opt/james-server/conf/lib/
  ```
* Buat folder di directory `/opt/james-server/conf/META-INF/org/apache/james/` dan copy kan `spring-server.xml`

  ```bash
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

* Download lib distribution dari Apache Derby [Download Page](http://archive.apache.org/dist/db/derby/db-derby-10.11.1.1/db-derby-10.11.1.1-lib.tar.gz)
* Extract hasil dari download `~]# tar -zxvf db-derby-10.11.1.1-lib.tar.gz`
* File yang digunakan `derby.jar` dan `derbynet.jar` untuk menerima koneksi jaringan dan copy 2 file jar di directory `/opt/james-server/lib/`

  ```bash
  ~]# pwd
  /opt
  ~]# cp db-derby-10.11.1.1-lib/lib/derby.jar james-server/lib
  ~]# cp db-derby-10.11.1.1-lib/lib/derbynet.jar james-server/lib
  ```
* Definisikan file `derby.jar` dan `derbynet.jar` ke file konfigurasi `wrapper.conf`

  ```java
  wrapper.java.classpath.94=%REPO_DIR%/derby.jar
  wrapper.java.classpath.131=%REPO_DIR%/derbynet.jar
  ```
* Konfigurasi `derby.log` dan beberapa tambahan untuk derby

  ```java
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

* Download package Apache James DKIM [Download Page](http://mirror.wanxp.id/apache//james/jdkim/apache-jdkim-0.2-bin.tar.gz)
* Extract hasil dari download `~]# tar -zxvf apache-jdkim-0.2-bin.tar.gz`
* Copy beberapa file lib `.jar` dan paste file `.jar` pada directory `/opt/james-server/lib/`

  ```bash
  ~]# pwd
  /opt/apache-jdkim-0.2
  ~]# cp lib/apache-jdkim-library-0.2.jar /opt/james-server/conf/lib
  ~]# cp lib/apache-jdkim-mailets-0.2.jar /opt/james-server/conf/lib
  ~]# cp lib/not-yet-commons-ssl-0.3.11.jar /opt/james-server/conf/lib
  ```
* Membuat key pair ada dua `dkim-public.pem` dan `dkim-private.pem`, public untuk DNS Record dan Private untuk server james

  ```bash
  ~]# pwd
  /opt/apache-jdkim-0.2
  ~]# openssl genrsa -out dkim-private.pem 1024 -outform PEM
  ~]# openssl rsa -in dkim-private.pem -out dkim-public.pem -pubout -outform PEM
  ```
* Menambahkan / register public key ke DNS Record

  ```
  Domain = 1480566545.tealinuxos._domainkey.tealinuxos.org
  TTL = 14400
  Class = IN
  Type = TXT
  Destination = v=DKIM1\; k=rsa\; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCuAaQ1O5HoBraRJt5BmBsJ+eA5pO/7QO4UX4WQHpNPB2oBRytFqbXHw5X8WBlT2GsfQtJChcMudS56k7veHlK3zgK2+uAQ2RfA25nqVYaAlAPs7tSXmwxEwLc5vkgg/oxerulUx9BZZ8Lw6huHRinIVTYhxkzcC1SEL02Vuxov/QIDAQAB
  ```

![alt tag](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/JAMES/asset/2016-12-01-121837_1920x1080_scrot.png)
* Testing DKIM, Publickey, dan selectornya dengan command

  ```bash
  ~]$ host -t txt 1480566545.tealinuxos._domainkey.tealinuxos.org
  1480566545.tealinuxos._domainkey.tealinuxos.org descriptive text "v=DKIM1\; k=rsa\; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCuAaQ1O5HoBraRJt5BmBsJ+eA5pO/7QO4UX4WQHpNPB2oBRytFqbXHw5X8WBlT2GsfQtJChcMudS56k7veHlK3zgK2+uAQ2RfA25nqVYaAlAPs7tSXmwxEwLc5vkgg/oxerulUx9BZZ8Lw6huHRinIVTYhxkzcC1SEL02Vuxov/QIDAQAB"
  ```

####Mendefinisikan script DKIMSign di mailet
Edit file `mailetcontainer.xml` pada directory `/opt/james-server/conf/`. Pastikan nilai `s` (selector) sesuai dengan yang sudah terdaftar di DNS. Dan harus mengganti `d` (domain) dengan domain yang terpasang server. Tambahkan beberapa script sebelum `<mailet match="All" class="RemoteDelivery">`.

  ```xml
  <mailet match="All" class="org.apache.james.jdkim.mailets.ConvertTo7Bit"/>

  <mailet match="All" class="org.apache.james.jdkim.mailets.DKIMSign">
    <signatureTemplate>v=1; s=1480566545.tealinuxos; d=tealinuxos.org; c=relaxed/relaxed; h=Message-ID:Date:Subject:From:To:MIME-Version:Content-Type; a=rsa-sha256; bh=; b=;</signatureTemplate>
    <privateKey>
    -----BEGIN RSA PRIVATE KEY-----
    isi dari PRIVATE KEY
    -----END RSA PRIVATE KEY-----
    </privateKey>
  </mailet>

  <!-- Attempt remote delivery using the specified repository for the spool, -->
  <!-- using delay time to retry delivery and the maximum number of retries -->
  <mailet match="All" class="RemoteDelivery">
    <outgoingQueue>outgoing</outgoingQueue>
  ```

####Restart JAMES

###Referensi
* (http://james.apache.org/jdkim/mailets/)
* (http://mail-archives.apache.org/mod_mbox/james-server-user/201410.mbox/%3C544FD474.2040906%40malcolms.com%3E)
* (http://www.gettingemaildelivered.com/dkim-explained-how-to-set-up-and-use-domainkeys-identified-mail-effectively)
* (http://www.dataenter.com/doc/general_domainkeys.htm)
* (https://scottlinux.com/2012/10/27/how-to-fetch-dkim-records-from-dns/)
* (http://dkimcore.org/)

----
#Konfigurasi SSL JAMES

##Requirement
* Port IMAPS > `993`
* Port SMTPS > `465` (untuk mail client)
* Port SMTP > `25` (untuk menerima koneksi dari server SMTP lain. STARTTLS diaktifkan)
* Mengekspos port dengan forwarding pada `iptables`
* [Java KeyStore](https://github.com/Setyadhi-Putra-D/Dokumentasi-ngoprek/blob/master/SSL/JKS.md)
  copy file `mykeystore.jks` ke directory `/opt/james-server/conf`

### Konfigurasi IMAPS
* Masuk ke file konfigurasi `imapserver.xml` dan ganti `port` + `TLS`

  ```xml
  <bind>0.0.0.0:10993</bind>

  <tls socketTLS="true" startTLS="false">
   <keystore>file://conf/mykeystore.jks</keystore>
   <secret>PASSPHRASE</secret>
   <provider>org.bouncycastle.jce.provider.BouncyCastleProvider</provider>
  </tls>
  ```

### Konfigurasi SMTPS
* Membuat seluruh salinan elemen `smtpserver` di `smtpserver.xml`
* Mengubah `jmxName` pada elemen kedua `smtpserver`, `port`, dan `TLS`

  ```xml
  <jmxName>smtpsserver</jmxName>

  <bind>0.0.0.0:10465</bind>

  <tls socketTLS="true" startTLS="false">
   <keystore>file://conf/mykeystore.jks</keystore>
   <secret>PASSPHRASE</secret>
   <provider>org.bouncycastle.jce.provider.BouncyCastleProvider</provider>
   <algorithm>SunX509</algorithm>
  </tls>

  <authRequired>announce</authRequired>
  ```

### Konfigurasi SMTP
* Edit `TLS` pada elemen pertama `smtpserver`

  ```xml
  <tls socketTLS="false" startTLS="true">
   <keystore>file://conf/mykeystore.jks</keystore>
   <secret>PASSPHRASE</secret>
   <provider>org.bouncycastle.jce.provider.BouncyCastleProvider</provider>
   <algorithm>SunX509</algorithm>
  </tls>
  ```

###Menghapus `mailet` dari file konfigurasi `mailetcontainer.xml`

```xml
<mailet match="RemoteAddrNotInNetwork=127.0.0.1" class="ToProcessor">
  <processor>relay-denied</processor>
  <notice>550 - Requested action not taken: relaying denied</notice>
</mailet>
```

###Konfigurasi iptables
* Edit file `iptables` pada directory `/etc/sysconfig/`

  ```
  *nat
  :PREROUTING ACCEPT [0:0]
  :OUTPUT ACCEPT [0:0]
  :POSTROUTING ACCEPT [0:0]
  -A PREROUTING -i eth0 -p tcp --dport 25 -j DNAT --to-destination :10025
  -A PREROUTING -i eth0 -p tcp --dport 465 -j DNAT --to-destination :10465
  -A PREROUTING -i eth0 -p tcp --dport 993 -j DNAT --to-destination :10993
  COMMIT
  *filter
  :INPUT DROP [0:0]
  :FORWARD DROP [0:0]
  :OUTPUT ACCEPT [0:0]
  -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
  -A INPUT -i lo -j ACCEPT
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 10025 -j ACCEPT
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 10465 -j ACCEPT
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 10993 -j ACCEPT
  COMMIT
  ```
* `service iptables restart`
