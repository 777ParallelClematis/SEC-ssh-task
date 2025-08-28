sudo tee /etc/apache2/sites-available/portal-ssl.conf >/dev/null <<'EOF'
<VirtualHost *:443>
    ServerName portal.office.local
    ServerAlias portal

    SSLEngine on
    SSLCertificateFile      /etc/ssl/local/portal.crt
    SSLCertificateKeyFile   /etc/ssl/private/portal.key
    Protocols h2 http/1.1

    DocumentRoot /var/www
    <Directory /var/www>
        Require all granted
        Options -Indexes
    </Directory>

    # Basic hardening
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
</VirtualHost>

<VirtualHost *:80>
    ServerName portal.office.local
    ServerAlias portal
    Redirect permanent / https://portal.office.local/
</VirtualHost>
EOF
