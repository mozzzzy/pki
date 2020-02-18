CLIENT_PRIVATE_KEY=ClientKey.pem
CLIENT_CSR=ClientCsr.pem
CLIENT_CRT=ClientCrt.pem

.PHONY: all
all: clean $(CLIENT_PRIVATE_KEY) $(CLIENT_CSR)

$(CLIENT_PRIVATE_KEY):
	openssl genrsa \
		-out $@ \
		2048

$(CLIENT_CSR): $(CLIENT_PRIVATE_KEY)
	openssl req \
		-new \
		-subj "/C=JP/ST=Tokyo/O=EXAMPLE SERVER/CN=origin.dev.local" \
		-key $^ \
		-out $@

.PHONY: clean
clean:
	rm -rf \
		$(CLIENT_PRIVATE_KEY) \
		$(CLIENT_CSR) \
		$(CLIENT_CRT)
