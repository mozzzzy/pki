# pki sandbox environment (depth 0)

## What
This is sandbox environment of PKI.
We can create following environments.
* root CA
* CRL distribution point (httpd)
* OCSP responder (openssl)
* https server (httpd)

## Create root CA certificate, OCSP certificate, CRL
```bash
$ cd rootCa
$ make clean init root-ca.crt root-ocsp.crt root-ca.crl
```

## Create server certificate
```bash
$ cd server
$ make clean init server.csr
$ cd ../rootCa
$ make ../server/server.crt
```

## Run CRL distribution point, OCSP responder, https server
### Configure https server
Set server certificate passphrase into file/pass_input.sh.
This script is executed by httpd as SSLPassPhraseDialog script.
```bash
$ vim docker/server/files/pass_input.sh
#!/bin/sh
echo -n "<PASSPHRASE>"
```

### Run each docker container
Now we can run CRL distribution point, OCSP responder and https server on docker containers.
```bash
$ docker-compose up -d --build
```

### Get root CA certificate and CRL
```bash
$ curl http://localhost/root-ca.crt -v
$ curl http://localhost/root-ca.crl -v
```

### Get OCSP response
```bash
$ openssl ocsp -issuer rootCa/root-ca.crt -CAfile rootCa/root-ca.crt -cert rootCa/root-ocsp.crt -url http://127.0.0.1:9080
```

### Send https request to https server
```bash
$ curl https://origin.dev.local/ -v --cacert rootCa/root-ca.crt --resolv origin.dev.local:443:127.0.0.1
```
