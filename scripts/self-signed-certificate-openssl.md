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

## Conclusion
You have successfully created a self-signed certificate and configured it for `xyzlfgtop001.example.com`. This setup ensures encrypted communication for internal usage.

