# Verify Operation

## Run httpd that use cert file.
```sh
$ docker-compose up -d
```

## Create chained certificate for verify
```sh
$ cat rootCa/RootCaCrt.pem intermediateCa/IntermediateCaCrt.pem > ChainedCrt.pem
```

## Create TLS Connection with httpd using chained cert file.
```sh
$ openssl s_client -connect localhost:443 -CAfile ChainedCrt.pem
```
