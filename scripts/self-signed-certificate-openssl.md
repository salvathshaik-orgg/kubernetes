# Self-Signed Certificate with CA for xyzlfgtop001.example.com

## Overview
This guide explains how to generate a self-signed Certificate Authority (CA) and use it to issue a server certificate for `xyzlfgtop001.example.com`. This setup helps create a secure HTTPS environment for internal use.

## Prerequisites
- OpenSSL installed on your system (default on most Linux distributions).
- Access to a terminal or command line.

## Step 1: Create a Certificate Authority (CA)
### 1.1 Generate the CA Private Key
```sh
openssl genpkey -algorithm RSA -out CA.key
```

### 1.2 Generate the CA Certificate
```sh
openssl req -x509 -new -nodes -key CA.key -sha256 -days 3650 -out CA.crt
```
- Use a unique **Common Name (CN)** like `My Custom CA`.
- The `CA.crt` file will be used to sign server certificates.

## Step 2: Generate the Server Certificate
### 2.1 Generate a Private Key for the Server
```sh
openssl genpkey -algorithm RSA -out server.key
```

### 2.2 Create a Certificate Signing Request (CSR)
```sh
openssl req -new -key server.key -out server.csr
```
- Set `xyzlfgtop001.example.com` as the **Common Name (CN)**.

### 2.3 Sign the Server Certificate Using the CA
```sh
openssl x509 -req -in server.csr -CA CA.crt -CAkey CA.key -CAcreateserial -out server.crt -days 365 -sha256
```
- The `server.crt` file is now signed by `CA.crt`.

## Step 3: Verify the Certificates
Check the CA certificate:
```sh
openssl x509 -in CA.crt -text -noout
```
Check the server certificate:
```sh
openssl x509 -in server.crt -text -noout
```

## Step 4: Configure Your Web Server
### Nginx Configuration
```nginx
server {
    listen 443 ssl;
    server_name xyzlfgtop001.example.com;

    ssl_certificate /path/to/server.crt;
    ssl_certificate_key /path/to/server.key;
    ssl_trusted_certificate /path/to/CA.crt;

    location / {
        root /var/www/html;
    }
}
```
Restart Nginx:
```sh
sudo systemctl restart nginx
```

### Apache Configuration
```apache
<VirtualHost *:443>
    ServerName xyzlfgtop001.example.com

    SSLEngine on
    SSLCertificateFile /path/to/server.crt
    SSLCertificateKeyFile /path/to/server.key
    SSLCACertificateFile /path/to/CA.crt

    DocumentRoot /var/www/html
</VirtualHost>
```
Restart Apache:
```sh
sudo systemctl restart httpd
```

## Step 5: Distribute the CA Certificate
- Clients must import `CA.crt` into their trusted certificate store to avoid browser warnings.
- In Linux, you can copy the CA certificate to `/etc/pki/ca-trust/source/anchors/` and update trust:
  ```sh
  sudo cp CA.crt /etc/pki/ca-trust/source/anchors/
  sudo update-ca-trust
  ```
 
## 2nd menthod below
## Step 6: Generate a Self-Signed Certificate with SANs
Modern TLS implementations require **Subject Alternative Names (SANs)** for hostname validation.

### 6.1 Create an OpenSSL Config File
Create a file named `openssl.cnf` with the following content:
```ini
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
x509_extensions    = v3_ca
prompt             = no

[ req_distinguished_name ]
C  = US
ST = California
L  = San Jose
O  = MyOrganization
CN = xyzlfgtop001.example.com

[ req_ext ]
subjectAltName = @alt_names

[ v3_ca ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = xyzlfgtop001.example.com
DNS.2 = *.example.com
IP.1  = 192.168.1.100  # Replace with actual IP if needed
```

### 6.2 Generate the Certificate with SAN
#### Generate a Private Key
```sh
openssl genpkey -algorithm RSA -out server.key
```
#### Generate the CSR
```sh
openssl req -new -key server.key -out server.csr -config openssl.cnf
```
#### Sign the Certificate Using the CA
```sh
openssl x509 -req -in server.csr -CA CA.crt -CAkey CA.key -CAcreateserial -out server.crt -days 365 -sha256 -extfile openssl.cnf -extensions req_ext
```

### 6.3 Verify the SANs in the Certificate
```sh
openssl x509 -in server.crt -text -noout | grep -A1 "Subject Alternative Name"
```
You should see:
```
X509v3 Subject Alternative Name:
    DNS:xyzlfgtop001.example.com, DNS:*.example.com, IP Address:192.168.1.100
```

## Conclusion
You have successfully created a self-signed certificate with **Subject Alternative Names (SANs)** and configured it for `xyzlfgtop001.example.com`. This setup ensures encrypted communication for internal usage while adhering to modern TLS requirements.

