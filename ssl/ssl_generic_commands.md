#### 1. To get certificate using the openssl tool:
---------------------------------------------

```
# openssl s_client -connect HOST:PORT -showcerts <<<'' | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p
```
```
Ex: openssl s_client -connect <AD_HOST>:636 -showcerts <<<'' | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p
```

#### 2. To import SSL certificate from a file:
-----------------------------------------

```
# keytool -import -file <cert file> -alias -keystore <destination keystore> -storepass "changeit"
```
```
Ex: keytool -import -file ranger-admin-trust.cer -alias rangeradmintrust -keystore ranger-plugin-truststore.jks -storepass changeit
```

#### 3. To list the entries in the keystore:
---------------------------------------

```
# keytool -list -v -keystore path_to_keystore_file
```

#### 4. Generate the self-signed certificate and place it in the KeyStore:
---------------------------------------------------------------------

```
# keytool -genkeypair -alias alias_name -keyalg RSA -validity 999 -keysize 2048 -keystore keystore_name.jks
```

```
Ex. keytool -genkey -keyalg RSA -alias rangerHdfsAgent -keystore ranger-plugin-keystore.jks -storepass myKeyFilePassword -validity 360 -keysize 2048
```

#### 5. Export the certificate to a certificate file:
------------------------------------------------------

```
# keytool -export -alias alias_name -keystore path_to_keystore_file -file path_to_certificate_file
```
```
Ex: keytool -export -keystore /etc/ranger/admin/conf/ranger-admin-keystore.jks -alias rangeradmin -file ranger-admin-trust.cer
```

Where:

`alias_name`: Specifies the same alias that was used to generate the certificate.

`path_to_keystore_file`: Specifies the same KeyStore path that was used to generate the certificate.

`path_to_certificate_file`: Specifies the exported certificate file, often given an extension of .cert.


o Good links to be referred:
-----------------------------

https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html

https://www.ssl.com/article/ssl-tls-handshake-overview/

https://blog.vrypan.net/2013/08/28/public-key-cryptography-for-non-geeks/

https://www.acunetix.com/blog/articles/tls-security-what-is-tls-ssl-part-1/
