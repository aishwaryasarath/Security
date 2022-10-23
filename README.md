# Security
Notes on SSL/TLS

Become a CA
1. Generate the private Key `myCA.key`:  `openssl genrsa -des3 -out myCA.key 2048`
2. Generate a Root Certificate `myCA.pem`:  `openssl req -x509 -new -nodes -key myCA.key -sha256 -days 1825 -out myCA.pem`
3. Install the Root certificate on devices

   Add Root certificate on Linux
    If it isn’t already installed, install the ca-certificates package.
    `sudo apt-get install -y ca-certificates`
    Copy the myCA.pem file to the `/usr/local/share/ca-certificates` directory as a myCA.crt file.
    `sudo cp ~/certs/myCA.pem /usr/local/share/ca-certificates/myCA.crt`
    Update the certificate store.
    `sudo update-ca-certificates`
    You can test that the certificate has been installed by running the following command:
    `awk -v cmd='openssl x509 -noout -subject' '/BEGIN/{close(cmd)};{print | cmd}' < /etc/ssl/certs/ca-certificates.crt | grep Hellfish`
    If installed you see this o/p
   ` subject=C = US, ST = IL, L = BG, O = AishCA, OU = 7G, CN = AishCA, emailAddress = abraham@hellfish.media`
    
4. Now we’re a CA on all our devices and we can sign certificates for any new dev sites that need HTTPS. First, we create a private key for the dev site.
    `openssl genrsa -out AishCA.test.key 2048`
5. Then we create a CSR
    `openssl req -new -key AishCA.test.key -out AishCA.test.csr`
Finally, we’ll create an X509 V3 certificate extension config file, which is used to define the Subject Alternative Name (SAN) for the certificate. In our case, we’ll create a configuration file called hellfish.test.ext containing the following text
```authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = AishCA.test
```

`openssl x509 -req -in AishCA.test.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out AishCA.test.crt -days 825 -sha256 -extfile AishCA.test.ext`

We now have three files: `AishCA.test.key` (the private key), `AishCA.test.csr` (the certificate signing request, or csr file), and `AishCA.test.crt` (the signed certificate)

References
1. https://www.golinuxcloud.com/openssl-create-client-server-certificate/
2. https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/
3. https://tomcat.apache.org/tomcat-8.5-doc/ssl-howto.html
