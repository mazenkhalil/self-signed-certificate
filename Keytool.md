# [Java Keytool] Generating Self Signed Certificate

## Root CA Generation

> Commands below are used for generating self signed local CA keystore

##### Generate 2048 root CA store [2048 | 10 years]:

``keytool -genkeypair -v -alias root -keystore root.p12 -storetype pkcs12 -keyalg RSA -keysize 2048 -validity 3650 -ext KeyUsage:critical="keyCertSign,digitalSignature,cRLSign" -ext BasicConstraints:critical="ca:true"``

##### Extract the root certificate:
> This certificate can be installed on client devices inorder to get all CA child certificates accepted

``keytool -export -v -alias root -file root.crt -keystore root.p12 -rfc``

---

## Domain Certificate Generation
> You can add extra SAN's if needed by separating them with ',' comma  (e.g. *san=dns:my.domain.local,dns:\*.domain.local,ip:127.0.0.1*)

##### Generate certificate store [2048 | 1 year]:

``keytool -genkeypair -v -alias FQDN -keystore serverCert.p12 -storetype pkcs12 -keyalg RSA -keysize 2048 -validity 365 -ext san=dns:<FQDN>,IP:<IP>``

##### Create certificate signing request:

``keytool -certreq -v -alias FQDN -keystore serverCert.p12 -file signingRequest.csr -ext san=dns:<FQDN>,IP:<IP>``

##### Sign the certificate request [1 year]:

``keytool -gencert -v -alias root -keystore root.p12 -infile signingRequest.csr -validity 365 -outfile signedCert.crt -rfc -ext KeyUsage:critical="digitalSignature,keyEncipherment" -ext EKU="serverAuth,clientAuth" -ext san=dns:<FQDN>,IP:<IP>``

##### Import the root certificate to the store:

``keytool -import -v -alias root -file root.crt -keystore serverCert.p12 -storetype pkcs12``

##### Import the signed certificate to the store:

``keytool -import -v -alias FQDN -file signedCert.crt -keystore serverCert.p12 -storetype pkcs12``

##### Convert the pkcs12 store to JKS format:

``keytool -importkeystore -srckeystore serverCert.p12 -srcstoretype PKCS12 -destkeystore keystore.jks -deststoretype JKS``
