sudo openssl req -x509 -nodes -newkey rsa:2048 -days 60 \
  -keyout portal.key -out portal.crt \
  -subj "/CN=portal.office.local" \
  -addext "subjectAltName=DNS:portal.office.local,DNS:portal" \
  -addext "basicConstraints=CA:FALSE" \
  -addext "keyUsage=digitalSignature,keyEncipherment" \
  -addext "extendedKeyUsage=serverAuth"
