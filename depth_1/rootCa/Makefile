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
		-extensions v3_ca \
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

INTERMEDIATE_CA_CSR:=../intermediateCa/IntermediateCaCsr.pem
INTERMEDIATE_CA_CRT:=../intermediateCa/IntermediateCaCrt.pem
$(INTERMEDIATE_CA_CRT): $(INTERMEDIATE_CA_CSR) $(ROOT_PRIVATE_KEY) $(ROOT_CRT) $(CONFIG_FOR_SIGN)
	openssl ca \
		-config $(CONFIG_FOR_SIGN) \
		-batch \
		-in $(INTERMEDIATE_CA_CSR) \
		-out $@ \
		-cert $(ROOT_CRT) \
		-extensions v3_ca \
		-keyfile $(ROOT_PRIVATE_KEY)
	openssl x509 -in $@ -out $@
