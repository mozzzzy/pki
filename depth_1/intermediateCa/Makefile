INTERMEDIATE_PRIVATE_KEY=IntermediateCaKey.pem
INTERMEDIATE_CSR=IntermediateCaCsr.pem
INTERMEDIATE_CRT=IntermediateCaCrt.pem

.PHONY: all
all: clean init $(INTERMEDIATE_PRIVATE_KEY) $(INTERMEDIATE_CSR)

.PHONY: init
init:
	mkdir newcerts
	echo "01" > serial
	echo "00" > crlnumber
	touch index.txt

$(INTERMEDIATE_PRIVATE_KEY):
	openssl genrsa \
		-out $@ \
		2048

$(INTERMEDIATE_CSR): $(INTERMEDIATE_PRIVATE_KEY)
	openssl req \
		-new \
		-subj "/C=JP/ST=Tokyo/O=EXAMPLE Intermediate CA/CN=rootca.dev.local" \
		-key $^ \
		-out $@

.PHONY: clean
clean:
	rm -rf \
		newcerts \
		serial \
		crlnumber \
		index.txt \
		$(INTERMEDIATE_PRIVATE_KEY) \
		$(INTERMEDIATE_CSR) \
		$(INTERMEDIATE_CRT) \
		*.old \
		*.attr

CLIENT_CSR:=../client/ClientCsr.pem
CLIENT_CRT:=../client/ClientCrt.pem
CONFIG_FOR_SIGN=config/openssl_sign.cnf
$(CLIENT_CRT): $(CLIENT_CSR) $(INTERMEDIATE_PRIVATE_KEY) $(INTERMEDIATE_CRT) $(CONFIG_FOR_SIGN)
	openssl ca \
		-config $(CONFIG_FOR_SIGN) \
		-batch \
		-in $(CLIENT_CSR) \
		-out $@ \
		-cert $(INTERMEDIATE_CRT) \
		-keyfile $(INTERMEDIATE_PRIVATE_KEY)
	openssl x509 -in $@ -out $@
