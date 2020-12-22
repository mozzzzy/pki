CONFIG=config/root-ca.conf
OPENSSL=openssl
ROOT_CA_KEY=private/root-ca.key
ROOT_CA_CSR=root-ca.csr
ROOT_CA_CRT=root-ca.crt
SERVER_CSR:=../server/server.csr
SERVER_CRT:=../server/server.crt

.PHONY: help
help:
	@echo "Usage:"
	@echo "  Initialize directories           : make init"
	@echo "  Create private key for root ca   : make $(ROOT_CA_KEY)"
	@echo "  Create csr for root ca           : make $(ROOT_CA_CSR)"
	@echo "  Check csr for root ca            : make check_root_ca_csr"
	@echo "  Create certificate for root ca   : make $(ROOT_CA_CRT)"
	@echo "  Check certificate for root ca    : make check_root_ca_crt"
	@echo "  Create certificate for server    : make $(SERVER_CRT)"
	@echo "  Remove all directories and files : make clean"

# Create some directories and files.
.PHONY: init
init:
	mkdir certs db private
	chmod 700 private
	touch db/index
	$(OPENSSL) rand -hex 16 > db/serial
	echo 1001 > db/crlnumber

# Create a rsa private key for ca certificate.
# -aes256 : encrypt the private key with aes256.
$(ROOT_CA_KEY):
	$(OPENSSL) genrsa \
		-out $@ \
		-aes256 \
		4096

# Create CSR (Certificate Signing Request) for ca certificate.
$(ROOT_CA_CSR): $(ROOT_CA_KEY) $(CONFIG)
	$(OPENSSL) req \
		-new \
		-config $(CONFIG) \
		-key $(ROOT_CA_KEY) \
		-out $@

# Check CSR for ca certificate.
.PHONY: check_root_ca_csr
check_root_ca_csr:
	$(OPENSSL) req \
		-in $(ROOT_CA_CSR) \
		-noout \
		-text

# Create ca certificate.
$(ROOT_CA_CRT): $(ROOT_CA_CSR) $(ROOT_CA_KEY) $(CONFIG)
	$(OPENSSL) ca \
		-config $(CONFIG) \
		-in $(ROOT_CA_CSR) \
		-out $@ \
		-selfsign \
		-notext

# Check ca certificate.
.PHONY: check_root_ca_crt
check_root_ca_crt:
	$(OPENSSL) x509 \
		-in $(ROOT_CA_CRT) \
		-noout \
		-text

.PHONY: clean
clean:
	rm -rf \
		certs \
		db \
		private \
		$(ROOT_CA_KEY) \
		$(ROOT_CA_CSR) \
		$(ROOT_CA_CRT) \
		*.old \
		*.attr

#
# Following rules are for CA tasks.
#

$(SERVER_CRT): $(SERVER_CSR) $(ROOT_CA_KEY) $(CONFIG)
	$(OPENSSL) ca \
		-in $(SERVER_CSR) \
		-out $@ \
		-config $(CONFIG) \
		-notext \
		-extensions server_ext

#$(CLIENT_CRT): $(CLIENT_CSR) $(ROOT_CA_KEY) $(ROOT_CA_CRT) $(CONFIG)
#	openssl ca \
#		-config $(CONFIG) \
#		-batch \
#		-in $(CLIENT_CSR) \
#		-out $@ \
#		-cert $(ROOT_CA_CRT) \
#		-keyfile $(ROOT_CA_KEY)
#	openssl x509 -in $@ -out $@
