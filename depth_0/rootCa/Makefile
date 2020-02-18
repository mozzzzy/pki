ROOT_PRIVATE_KEY=RootCaKey.pem
ROOT_CSR=RootCaCsr.pem
ROOT_CRT=RootCaCrt.pem
CONFIG_FOR_SIGN=config/openssl_sign.cnf

.PHONY: all
all: clean init $(ROOT_PRIVATE_KEY) $(ROOT_CSR) $(ROOT_CRT)

.PHONY: init
init:
	mkdir newcerts
	echo "01" > serial
	echo "00" > crlnumber
	touch index.txt

$(ROOT_PRIVATE_KEY):
	openssl genrsa \
		-out $@ \
		2048

$(ROOT_CSR): $(ROOT_PRIVATE_KEY)
	openssl req \
		-new \
		-subj "/C=JP/ST=Tokyo/O=EXAMPLE Root CA/CN=rootca.dev.local" \
		-key $^ \
		-out $@

$(ROOT_CRT): $(ROOT_CSR) $(ROOT_PRIVATE_KEY) $(CONFIG_FOR_SIGN)
	openssl ca \
		-config $(CONFIG_FOR_SIGN) \
		-batch \
		-in $(ROOT_CSR) \
		-out $@ \
		-selfsign \
		-keyfile $(ROOT_PRIVATE_KEY)
	openssl x509 -in $@ -out $@

.PHONY: clean
clean:
	rm -rf \
		newcerts \
		serial \
		crlnumber \
		index.txt \
		$(ROOT_PRIVATE_KEY) \
		$(ROOT_CSR) \
		$(ROOT_CRT) \
		*.old \
		*.attr

CLIENT_CSR:=../client/ClientCsr.pem
CLIENT_CRT:=../client/ClientCrt.pem
$(CLIENT_CRT): $(CLIENT_CSR) $(ROOT_PRIVATE_KEY) $(ROOT_CRT) $(CONFIG_FOR_SIGN)
	openssl ca \
		-config $(CONFIG_FOR_SIGN) \
		-batch \
		-in $(CLIENT_CSR) \
		-out $@ \
		-cert $(ROOT_CRT) \
		-keyfile $(ROOT_PRIVATE_KEY)
	openssl x509 -in $@ -out $@
