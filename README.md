# Keycloak TLS

TLS-configured Keycloak instance with Postgres and Docker

## Generate certificate

First, copy and edit the `.env` file :

```bash
cp .env.example .env # Edit KEYCLOAK_PASSWORD
```

Then, we can generate our certificates :

```bash
mkdir -p certs/{ca,keycloak}

# Choose an appropriate DN
KEYCLOAK_CERTS_DN="/C=FR/ST=IDF/L=PARIS/O=EXAMPLE"

# Generate root CA (you can ignore if you have already one)
openssl genrsa -out certs/ca/ca.key 2048
openssl req -new -x509 -sha256 -days 1095 -subj "$KEYCLOAK_CERTS_DN/CN=CA" -key certs/ca/ca.key -out certs/ca/ca.pem

# Generate Keycloak certificate, signed by your CA
openssl genrsa -out certs/keycloak/keycloak-temp.key 2048
openssl pkcs8 -inform PEM -outform PEM -in certs/keycloak/keycloak-temp.key -topk8 -nocrypt -v1 PBE-SHA1-3DES -out certs/keycloak/keycloak.key
openssl req -new -subj "$KEYCLOAK_CERTS_DN/CN=keycloak" -key certs/keycloak/keycloak.key -out certs/keycloak/keycloak.csr
openssl x509 -req -in certs/keycloak/keycloak.csr -CA certs/ca/ca.pem -CAkey certs/ca/ca.key -CAcreateserial -sha256 -out certs/keycloak/keycloak.pem
rm certs/keycloak/keycloak-temp.key certs/keycloak/keycloak.csr

# Configuring filenames and rights for Keycloak container
cp certs/keycloak/keycloak.key certs/keycloak/tls.key
cp certs/keycloak/keycloak.pem certs/keycloak/tls.crt
chmod 655 certs/keycloak/tls.crt certs/keycloak/tls.key
```

## Run Keycloak

```bash
docker-compose up -d
```

Enjoy your Keycloak instance : `https://localhost:8443`
