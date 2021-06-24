# SSL

## Creating SSL certificate and it's CA

1. Create CA's private key and self-signed certificate
   ```bash
   rm *.pem

    # 1. Generate CA's private key 	and self-signed certificate
    openssl req -x509 -newkey rsa:4096 -days 365 -nodes -keyout ca-key.pem -out ca-cert.pem -subj "/C=FR/ST=Occitanie/L=Toulouse/O=Tech School/OU=Education/CN=*.techschool.guru/emailAddress=techschool.guru@gmail.com"

    echo "CA's self-signed certificate"
    openssl x509 -in ca-cert.pem -noout -text

    # 2. Generate web server's private key and certificate signing request (CSR)
    openssl req -newkey rsa:4096 -nodes -keyout server-key.pem -out server-req.pem -subj "/C=FR/ST=Ile de Franch/L=Paris/O=PC Book/OU=Computer/CN=*.pcbook.com/emailAddress=pcbook@gmail.com"

    # 3. Use CA's private key to sign web server's CSR and get back the signed certificate
    openssl x509 -req -in server-req.pem -days 60 -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile server-ext.cnf

    echo "Server's signed certificate"
    openssl x509 -in server-cert.pem -noout -text

   ```
## Step By Step Create CA, Intermediate CA and Signed Certificate
1. Generate Root CA Private Key
   ```bash
   openssl genrsa -out MyRootCA.key 4096
   ```
2. Generate Root CA Certificate
   ```bash
   openssl req -new -x509 -days 356 -key MyRootCA.key -out MyRootCA.crt
   ```
3. Generate Intermediate CA private key
   ```bash
   openssl genrsa -out MyIntermediateCA.key 4096
   ```
4. Generate Intermediate CA Signing Request (CSR)
   ```bash
   openssl req -new -key MyIntermediateCA.key -out MyIntermediateCA.csr
   ```
5. Generate Intermediate Certificate Signed by Root CA
   ```bash
   openssl x509 -req -days 365 -in MyIntermediateCA.csr -CA MyRootCA.crt -CAkey MyRootCA.key -CAcreateserial -out MyIntermediateCA.crt
   ```   
6. Generate RSA Server Key
   ```bash
   openssl genrsa -out server.key 2048
   ```  
7. Create server certificate signing request, to be signed by Intermediate CA
   ```bash
   openssl req -new -key server.key -out server.csr
   ```     
8. Sign CSR by intermediate CA
   ```bash
   openssl x509 -req -days 365 -in server.csr -CA MyIntermediateCA.crt -CAkey MyIntermediateCA.key -CAcreateserial -out server.crt -sha1
   ```    
9. Bundle Root CA and Intermediate CA
   ```bash
   cat MyIntermediateCA.crt MyRootCA.crt > ca-bundle
   ```    
10. Create P12 from server certificate and also fill the certificate hirarchy
    ```bash
    openssl pkcs12 -export -in server.crt -inkey server.key -out server.p12 -chain -CAfile ca-bundle.crt
    ```
11. Create Java Keystore `(JKS)` for two way SSL
    ```bash
    keytool -importkeystore -srckeystore server.p12 -destkeystore server.jks -deststoretype JKS
    keytool -import -trustcacerts -keystore server.jks -storepass secret -alias rootca -file MyRootCA.crt
    ```


   