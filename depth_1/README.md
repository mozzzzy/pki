# pki sandbox environment (depth 1)

## What
This is sandbox environment of PKI.
We can create following environments.
* root CA
* CRL distribution point for root CA (httpd)
* OCSP responder for root CA (openssl)
* intermediate CA
* CRL distribution point for intermediate CA (httpd)
* OCSP responder for intermediate CA (openssl)
* https server (httpd)

## Create root CA certificate, OCSP certificate, CRL
```bash
$ cd rootCa
$ make clean init root-ca.crt root-ocsp.crt root-ca.crl
```

## Create intermediate CA certificate, OCSP certificate, CRL
First, create intermediate CA CRT.
```bash
$ cd intCa
$ make clean init int-ca.csr
$ cd ../rootCa
$ make ../intCa/int-ca.crt
```
Next, create OCSP certificate and CRL.
```
$ cd intCa
$ make int-ocsp.crt int-ca.crl
```

## Create server certificate
```bash
$ cd server
$ make clean init server.csr
$ cd ../intCa
$ make ../server/server.crt
```

## Run each docker containers
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
$ curl http://localhost:8080/int-ca.crt -v
$ curl http://localhost:8080/int-ca.crl -v
```

### Get root OCSP response
```bash
$ openssl ocsp -issuer rootCa/root-ca.crt -CAfile rootCa/root-ca.crt -cert rootCa/root-ocsp.crt -url http://127.0.0.1:9080
```

### Get intermediate CA certificate and CRL
```bash
$ curl http://localhost/int-ca.crt -v
$ curl http://localhost/int-ca.crl -v
```

### Get intermediate OCSP response
```bash
$ openssl ocsp -issuer intCa/int-ca.crt -CAfile rootCa/root-ca.crt -cert intCa/int-ocsp.crt -url http://127.0.0.1:9081
```

### Send https request to https server
```bash
$ cat rootCa/root-ca.crt intCa/int-ca.crt > chained.crt
$ curl https://origin.dev.local/ -v --cacert chained.crt --resolv origin.dev.local:443:127.0.0.1
```
