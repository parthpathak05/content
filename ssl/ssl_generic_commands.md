#### 1. To get certificate using the openssl tool:
---------------------------------------------

```
#openssl s_client -connect HOST:PORT -showcerts <<<'' | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p
```

Ex:
```
#openssl s_client -connect <AD_HOST>:636 -showcerts <<<'' | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p
```

#### 2. To import SSL certificate from a file:
-----------------------------------------

```
#keytool -import -file <cert file> -alias -keystore <destination keystore> -storepass "changeit"
```

Ex:
```
#keytool -import -file ranger-admin-trust.cer -alias rangeradmintrust -keystore ranger-plugin-truststore.jks -storepass changeit
```

#### 3. To list the entries in the keystore:
---------------------------------------

```
#keytool -list -v -keystore path_to_keystore_file
```

Ex.:

```
#keytool -list -v -keystore gateway.jks
```

#### 4. Generate the self-signed certificate and place it in the KeyStore:
---

- The below command generates a KeyPair of PrivateKeyEntry and a associated Public certificate and places it in the keystore file of JKS format.
- The Key creation algorithm is RSA and the size of the key is 2048 bits. This is valid for 360 days.

```
#keytool -genkey -keyalg RSA -alias 'alias_name' -keystore keystore.jks -storepass Password -validity 360 -keysize 2048
```

Ex.
```
#keytool -genkey -keyalg RSA -alias rangerHdfsAgent -keystore ranger-plugin-keystore.jks -storepass myKeyFilePassword -validity 360 -keysize 2048
```

#### 5. Export the certificate from the JKS keystore/truststore to a certificate file:
---

- The below command will export a public certificate of a given alias in the command, in a DER encoded format.
 
```
#keytool -export -alias alias_name -keystore path_to_keystore_file -file path_to_certificate_file
```

Ex:
```
#keytool -export -keystore keystore.jks -alias alias1 -file public_cert.cer
```

Where:

- `alias_name`: Specifies the same alias that was used to generate the certificate.
- `path_to_keystore_file`: Specifies the same KeyStore path that was used to generate the certificate.
- `path_to_certificate_file`: Specifies the exported certificate file, often given an extension of .cert.

#### 6. To change the password of the private key entry in the JKS keystore: 
----------------------------------------------------------------------------

- Use the `-keypasswd` command to change the password (under which private/secret keys identified by -alias are protected) from `-keypass old_keypass` to `-new new_keypass`. The password value must contain at least six characters.
- If the `-keypass` option isn't provided at the command line and the `-keypass` password is different from the keystore password `(-storepass arg)`, then the user is prompted for it.
- If the `-new` option isn't provided at the command line, then the user is prompted for it.

```
#keytool -keypasswd -keystore keystore.jks

OR

#keytool -keypasswd -keystore keystore.jks -keypass Old_keyspassword -new New_keypassword
```

- The following are the available options for the `-keypasswd` command:

- `{-alias alias}`: Alias name of the entry to process.
- `[-keypass old_keypass]`: Key password.
- `[-new new_keypass]`: New password.
- `{-keystore keystore}`: Keystore name.
- `{-storepass arg}`: Keystore password.
- `{-storetype type}`: Keystore type.
- `{-v}`: Verbose output.

#### 7. To change the JKS keystore file password:
---------------------------------------------

- Use the `-storepasswd` command to change the password used to protect the integrity of the keystore contents. The new password is set by `-new` arg and must contain at least six characters.

Ex:

``` 
#keytool -storepasswd -keystore keystore.jks
```

-`[-new arg]`: New password
-`{-keystore keystore}`: Keystore name
-`[-storepass arg]`: Keystore password
-`{-storetype type}`: Keystore type
-`{-providername name}`: Provider name
-`{-keystore keystore}`: Keystore name
- `{-v}`: Verbose output


#### 8. Extracting the Private key and the certs from the PFX files: (Usually required when SSL certificate renewal cases come in!)
-----------------------------------------------------------------------------------------------------------------------------------

- For extracting the Private Key from the PCKS12 file:

```
# openssl pkcs12 -in [yourfile.pfx] -nocerts -out private_key.key
```

For extracting the public certificate:
```
#openssl pkcs12 -in [yourfile.pfx] -clcerts -nokeys -out public_cert.crt
```

#### 9. Change the PFX type to JKS type (Usually required when SSL certificate renewal cases come in!):
---

- PFX to JKS:

1. Here source keystore is of type PKCS12 & destination keystore will be of JKS type.
2. You need to know the PrivateKey passwords and the old password for `pkcs12` certifiacte file.

```
# keytool -importkeystore  -srckeystore PFK/p12_file_name -srcstoretype pkcs12 -destkeystore JKS_file_name -deststoretype jks -srcstorepass <Password> -deststorepass <Password> -srcalias 'alias_name' -destalias 'alias_name' -destkeypass <Password>
```

Ex:

```
# keytool -importkeystore  -srckeystore host1.pfx -srcstoretype pkcs12 -destkeystore host.jks -deststoretype jks -srcstorepass <Password> -deststorepass <Password> -srcalias 'alias1' -destalias 'alias1' -destkeypass <Password>
```

#### 10. To list the content of a key/certificate in human readable format:
---

- For reading the key created using RSA algorithm:

```
#openssl rsa -in <name>.key -text
```

- For reading the certificate created using x509 standard:

```
#openssl x509 -in <name>.crt -text
```

- As a note please refer the man page for the above commands: 


#### 11. Deleting the alias in the JKS keystore:
---

- This command will delete the mentioned alias name from the JKS file.
- You need to know the password for the keystore and if trying to remove the `PrivateKeyEntry` the key password.
 
```
#keytool -delete -alias <alias_name> -keystore <keystorename>
```

Ex:

```
#keytool -delete -alias gateway-identity -keystore gateway.jks
```

#### 12. Changing the alias in the SSL keystores JKS:
---

- This command will help you rename the alias in the JKS files.
- It can also be executed without the `-storepass` argument. In which case you will hae to provide the password on the prompt when asked.

```
#keytool -changealias -alias "alias_name" -destalias "new_alias"  -keystore /path/to/keystore -storepass storepass
```

Ex:

```
#keytool -changealias -alias "mykey" -destalias "gateway-identity" -keystore /etc/ambari-server/conf/ambari-server-truststore.jks
```

- Optionally you can also use the above example to change the alias of a `PrivateKeyEntry` using argument `-keypass key-password`.


#### 13. Inspect the truststore again to verify complete cert chain is imported:
---

- This command will tell you the Owners and the Issuers of the SSL certificates in a JKS keystore.
- Can be used to check if the certificate chain is complete in a give file.

```
#echo "" | $JAVA_HOME/bin/keytool -list -keystore test.jks -v | egrep "Owner:|Issuer:"
```

#### 14. To print the contents of the certificate:
---

- We can use the `keytool` command to read the certificate details from a PEM/DER encoded files.

```
#keytool -printcert -file <file.crt>
```

#### 15. To check if the `.crt` file corresponds to the `.key` file use below:
---

```
#openssl x509 –noout –modulus –in <file>.crt | openssl md5
```

```
#openssl rsa –noout –modulus –in <file>.key | openssl md5
```

- The MD5 should be same for both the key and the CRT file which verifies the keypair. 


#### 16. Converting the DER encoded certs to PEM encoded certs:
---

```
#openssl x509 -inform der -in certificate.cer -out certificate.pem
```



---
#### *** Good links that can be referred for better understanding of SSL: ***
---

- [Detail series of 5 parts on SSL/TLS](https://www.acunetix.com/blog/articles/tls-security-what-is-tls-ssl-part-1/)

- [The SSL/TLS Handshake: an Overview](https://www.ssl.com/article/ssl-tls-handshake-overview/)

- [What is Public/PrivateKeyCryptogrphy? And use of it in Digital Signatures](https://blog.vrypan.net/2013/08/28/public-key-cryptography-for-non-geeks/)

- [Export Certificates and Private Key from a PKCS#12 File with OpenSSL](https://www.ssl.com/how-to/export-certificates-private-key-from-pkcs12-file-with-openssl/)

- ["keytool" command manual Page](https://docs.oracle.com/en/java/javase/13/docs/specs/man/keytool.html)

- ["openssl" manual Page](https://www.openssl.org/docs/manmaster/man1/openssl-pkcs12.html)
