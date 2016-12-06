#Java KeyStore
* Environment
  * Java JDK
  * OpenSSL
* File Requirement
  * key dan certificate >> `privateKey.key` & `certificate.crt`

    ```bash
    ~]# pwd
    /opt
    ~]# openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out certificate.crt
    ```
* Membuat CAFile

  ```bash
  ~]# pwd
  /opt
  ~]# curl -O https://raw.githubusercontent.com/bagder/ca-bundle/master/ca-bundle.crt
  ~]# cat certificate.crt ca-bundle.crt > allcacerts.crt
  ```
* Membuat PKCS12 key store

  ```bash
  ~]# pwd
  /opt
  ~]# openssl pkcs12 -export -out certificate.pkcs12 -inkey privateKey.key -in certificate.crt -certfile allcacerts.crt -name java
  ```
* Membuat Java KeyStore

  ```bash
  ~]# pwd
  /opt
  ~]# keytool -importkeystore -srckeystore certificate.pkcs12 -srcstoretype pkcs12 -srcalias java -destkeystore mykeystore.jks -deststoretype jks -destalias mykey
  ```
* Menampilkan daftar entri dalam keystore

  ```bash
  ~]# pwd
  /opt
  ~]# keytool -v -list -storetype jks -keystore /opt/mykeystore.jks
  ```

###References
* (https://docs.oracle.com/cd/E19509-01/820-3503/ggfhb/index.html)
* (https://james.apache.org/server/3/config-ssl-tls.html)
* (http://cunning.sharp.fm/2008/06/importing_private_keys_into_a.html)
* (https://www.tbs-certificates.co.uk/FAQ/en/626.html)
* (http://www.fourproc.com/2010/06/23/create-a-ssl-keystore-for-a-tomcat-server-using-openssl-.html)
* (http://stackoverflow.com/questions/906402/importing-an-existing-x509-certificate-and-private-key-in-java-keystore-to-use-i)
* (http://curl.haxx.se/docs/caextract.html)
* (https://www.sslshopper.com/article-most-common-openssl-commands.html)
