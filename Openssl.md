# [Openssl] Generating Self Signed Certificate

## Root CA Generation

##### Create the configuration file [./config/root.config]

```
[req]
x509_extensions = v3_ca
distinguished_name = dn
prompt = no

[dn]
C = <Country Name (2 letter code)>
ST = <State or Province Name (eg, Some-State)>
L = <Locality Name (eg, city)>
O = <Organization Name (eg, company)>
OU = <Organizational Unit Name (eg, section)>
CN = <CA Name>

[v3_ca]
basicConstraints = critical, CA:TRUE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always, issuer:always
keyUsage = critical, cRLSign, digitalSignature, keyCertSign
```

##### Generate RootCA key [4096]:

``openssl genrsa -out rootCA.key 4096``

##### Generate RootCA Certificate [SHA384 | 10 years]:

``openssl req -new -x509 -sha384 -days 3650 -nodes -key rootCA.key -config ./config/root.config -out rootCA.pem``

> Verify the certificate

``openssl x509 -in rootCA.pem -text``

> Extract the certificate

``openssl x509 -outform der -in rootCA.pem -out root.crt``

---

## Server Certificate Generation

##### Create the configuration file [./config/serverCertReq.config]

```
[req]
req_extensions = v3_req
distinguished_name = dn
prompt = no

[dn]
C = <Country Name (2 letter code)>
ST = <State or Province Name (eg, Some-State)>
L = <Locality Name (eg, city)>
O = <Organization Name (eg, company)>
OU = <Organizational Unit Name (eg, section)>
CN = <FQDN>

[v3_req]
subjectAltName = DNS:<FQDN>,IP:<IP>
```

##### Create the configuration file [./config/serverCert.ext]

```
subjectAltName = DNS:<FQDN>,IP:<IP>
keyUsage = critical,digitalSignature,keyEncipherment
extendedKeyUsage = serverAuth,clientAuth
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always
```

##### Generate Server Certificate key [4096]:

``openssl genrsa -out serverCert.key 4096``

##### Generate Server Certificate Request [SHA384]:

``openssl req -new -sha384 -key serverCert.key -config ./config/serverCertReq.config -out serverCertReq.csr``

##### Generate Server Certificate [SHA384 | 1 year]:

``openssl x509 -sha384 -days 365 -req -in serverCertReq.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -extfile ./config/serverCert.ext -out serverCert.pem``

> Verify the certificate

``openssl x509 -in serverCert.pem -text``

> Extract the certificate

``openssl x509 -outform der -in serverCert.pem -out serverCert.crt``

> Convert to PFX/P12 format

``openssl pkcs12 -export -inkey serverCert.key -in serverCert.pem -certfile rootCA.pem -out serverCert.pfx``

---

## Server Certificate Renewal
> Creating new certificate key and certificate request is optional. Use same root to generate the new certificate.
