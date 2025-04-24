## SSL (Secure Sockets Layer)

CA (Certificate Authority) Provider => Verify if certificate is authentic ir not

- crt, key file (main, 2 intermediate)

CSR key

CSR validaye => gives CRT file ====> done by CA

- ca.crt
- own.crt
- own.key

### Generating Key

```bash
mkdir certs
cd certs

# generate ca
openssl genpkey -algorithm RSA -out ca.key
openssl req -key ca.key -new -x509 -out ca.crt -days 3650 -subj "/C=NP/ST=Bagmati/L=Kathmandu/O=BISHAL/OU=IT/CN=caauthority.com"

#generate certificate
openssl genpkey -algorithm RSA -out server.key


permission 600
permission RW to www-data


```

## nginx

```
listen 443 ssl;
ssl_certificate
```

microsoft adcs
freeipa
dogtag pki
centralized certificate
