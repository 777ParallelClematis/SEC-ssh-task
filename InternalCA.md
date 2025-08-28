To be run on ubuntu server (via SSH from windows host)

sudo mkdir -p /root/pki && cd /root/pki

1. working directory and root Cert Auth

openssl req -x509 -newkey rsa:4096 -days 3650 -nodes \
  -subj "/C=NZ/ST=YourRegion/L=YourCity/O=YourOrg/OU=IT/CN=Office Root CA" \
  -keyout ca.key -out ca.crt

2. server key and CSR for the portal

openssl req -new -newkey rsa:2048 -nodes \
  -subj "/C=NZ/ST=YourRegion/L=YourCity/O=YourOrg/OU=IT/CN=portal.office.local" \
  -keyout portal.key -out portal.csr

3. Add SAN/KU/EKU extensions and then sign the server cert with the CA

cat > server-ext.cnf <<'EOF'
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = portal.office.local
DNS.2 = portal
EOF

openssl x509 -req -in portal.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out portal.crt -days 365 -sha256 -extfile server-ext.cnf

4. Install certs & keys to standard paths

sudo install -m 600 portal.key /etc/ssl/private/portal.key
sudo install -m 644 portal.crt  /etc/ssl/local/portal.crt
sudo install -m 644 ca.crt      /etc/ssl/local/office-root-ca.crt

5. Install + configure Apache for HTTPS

sudo apt update && sudo apt install -y apache2
sudo a2enmod ssl headers http2
echo 'OK: portal over HTTPS' | sudo tee /var/www/html/index.html

sudo tee /etc/apache2/sites-available/portal-ssl.conf >/dev/null <<'EOF'
<VirtualHost *:443>
    ServerName portal.office.local
    ServerAlias portal

    SSLEngine on
    SSLCertificateFile      /etc/ssl/local/portal.crt
    SSLCertificateKeyFile   /etc/ssl/private/portal.key
    Protocols h2 http/1.1

    DocumentRoot /var/www/html
    <Directory /var/www/html>
        Require all granted
        Options -Indexes
    </Directory>

    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
</VirtualHost>

<VirtualHost *:80>
    ServerName portal.office.local
    ServerAlias portal
    Redirect permanent / https://portal.office.local/
</VirtualHost>
EOF

sudo a2ensite portal-ssl
sudo systemctl reload apache2

6. Verification on ubuntu

openssl x509 -in /etc/ssl/local/portal.crt -noout -subject -issuer -ext subjectAltName -ext extendedKeyUsage
openssl s_client -connect portal.office.local:443 -servername portal.office.local </dev/null \
 | openssl x509 -noout -subject -issuer -dates

7. On Win 11, trust the root CA so the browser turns green/happy/secure

scp <youruser>@<vm-ip>:/root/pki/ca.crt $HOME\Downloads\office-root-ca.crt

Run mmc.exe, import cert

Restart the browser and visit https://portal.office.local 