# pki sandbox environment (self sign)

## What
This is sandbox environment of PKI.
We can create following environments.
* https server (httpd)

## Create server certificate
```bash
$ cd server
$ make clean init server.key server.crt
```

### Run each docker container
Now we can run https server on docker container.
```bash
$ docker-compose up -d --build
```

### Send https request to https server
```bash
$ curl https://origin.dev.local/ -k -v --resolv origin.dev.local:443:127.0.0.1
```
