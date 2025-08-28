sudo mkdir -p /etc/ssl/{private,local}

sudo openssl req -x509 -nodes -newkey rsa:2048 -days 60 \
  -keyout /etc/ssl/private/portal.key \
  -out /etc/ssl/local/portal.crt \
  -subj "/CN=portal.office.local" \
  -addext "subjectAltName=DNS:portal.office.local,DNS:portal" \
  -addext "basicConstraints=CA:FALSE" \
  -addext "keyUsage=digitalSignature,keyEncipherment" \
  -addext "extendedKeyUsage=serverAuth"

sudo chmod 600 /etc/ssl/private/portal.key
sudo chmod 644 /etc/ssl/local/portal.crt