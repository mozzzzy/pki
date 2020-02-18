# Verify Operation

## Run httpd that use cert file.
```sh
$ docker-compose up -d
```

## Create TLS Connection with httpd using Root CA's cert file.
```sh
$ openssl s_client -connect localhost:443 -CAfile ./rootCa/RootCaCrt.pem
```
