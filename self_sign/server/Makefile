CONFIG=config/server.conf
OPENSSL=openssl
SERVER_KEY=private/server.key
SERVER_CRT=server.crt

.PHONY: help
help:
	@echo "Usage:"
	@echo "  Initialize directories                    : make init"
	@echo "  Create private key for server certificate : make $(SERVER_KEY)"
	@echo "  Create private key for server certificate : make $(SERVER_CRT)"
	@echo "  Check server certificate                  : make check_server_crt"
	@echo "  Remove all files                          : make clean"

.PHONY: init
init:
	mkdir private
	chmod 700 private

# Create a rsa private key for server certificate.
# -aes256 : encrypt the private key with aes256.
$(SERVER_KEY):
	$(OPENSSL) genrsa \
		-out $@ \
		-aes256 \
		2048

$(SERVER_CRT): $(SERVER_KEY)
	$(OPENSSL) req \
		-x509 \
		-new \
		-config $(CONFIG) \
		-key $(SERVER_KEY) \
		-out $@

.PHONY: check_server_crt
check_server_crt:
	$(OPENSSL) x509 \
		-in $(SERVER_CRT) \
		-noout \
		-text

.PHONY: clean
clean:
	rm -rf \
		private \
		$(SERVER_KEY) \
		$(SERVER_CRT)
