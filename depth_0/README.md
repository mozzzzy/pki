# pki sandbox environment (depth 0)

## What
This is sandbox environment of PKI.
We can create root CA and tls server.

## Create root CA certificate
```bash
$ cd rootCa
$ make clean init root-ca.crt
```

## Create server certificate
```bash
$ cd server
$ make init server.csr
$ cd ../rootCa
$ make ../server.crt
```

## Run httpd using server certificate
Set server certificate passphrase into file/pass_input.sh.
This script is executed by httpd as SSLPassPhraseDialog script.
```bash
$ vim files/pass_input.sh
#!/bin/sh
echo -n "<PASSPHRASE>"
```

Now we can run httpd on docker container.
```bash
$ docker-compose up -d
```

## Send https request to httpd
```
$ curl https://origin.dev.local/ -v --cacert rootCa/root-ca.crt --resolv origin.dev.local:443:127.0.0.1
```
