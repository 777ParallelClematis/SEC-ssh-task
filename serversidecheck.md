openssl s_client -connect localhost:443 -servername portal.office.local </dev/null | openssl x509 -noout -subject -issuer -dates
