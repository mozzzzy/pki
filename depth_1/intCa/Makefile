OPENSSL=openssl
CONFIG_INT_CA_REQ=config/int-ca-req.conf
CONFIG_INT_CA=config/ca.conf
CONFIG_INT_OCSP_REQ=config/int-ocsp-req.conf
INT_CA_KEY=private/int-ca.key
INT_CA_CSR=int-ca.csr
INT_CA_CRT=int-ca.crt
INT_CA_CRL=int-ca.crl
INT_OCSP_KEY=private/int-ocsp.key
INT_OCSP_CSR=int-ocsp.csr
INT_OCSP_CRT=int-ocsp.crt
SERVER_CSR=../server/server.csr
SERVER_CRT=../server/server.crt

.PHONY: help
help:
	@echo "Usage:"
	@echo "  Initialize directories                 : make init"
	@echo "  Remove all directories and files       : make clean"
	@echo "  -------- For building Intermediate CA --------"
	@echo "  Create private key for intermediate ca : make $(INT_CA_KEY)"
	@echo "  Create csr for intermediate ca         : make $(INT_CA_CSR)"
	@echo "  Check csr for intermediate ca          : make check_int_ca_csr"
	@echo "  Check certificate for intermediate ca  : make check_int_ca_crt"
	@echo "  Create crl for intermediate ca         : make $(INT_CA_CRL)"
	@echo "  Check crl for intermediate ca          : make check_int_ca_crl"
	@echo "  --------- For buiding OCSP Responder ---------"
	@echo "  Create private key for ocsp responder  : make $(INT_OCSP_KEY)"
	@echo "  Create csr for ocsp responder          : make $(INT_OCSP_CSR)"
	@echo "  Check csr for ocsp responder           : make check_int_ocsp_csr"
	@echo "  Create certificate for ocsp responder  : make $(INT_OCSP_CRT)"
	@echo "  Check certificate for ocsp responder   : make check_int_ocsp_crt"
	@echo "  ---------------- For CA Tasks ----------------"
	@echo "  Create certificate for server : make $(SERVER_CRT)"

#
# Following rules are for building intermediate ca.
#

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
$(INT_CA_KEY):
	$(OPENSSL) genrsa \
		-out $@ \
		-aes256 \
		4096

# Create CSR (Certificate Signing Request) for ca certificate.
$(INT_CA_CSR): $(INT_CA_KEY) $(CONFIG_INT_CA_REQ)
	$(OPENSSL) req \
		-new \
		-config $(CONFIG_INT_CA_REQ) \
		-key $(INT_CA_KEY) \
		-out $@

# Check CSR for ca certificate.
.PHONY: check_int_ca_csr
check_int_ca_csr:
	$(OPENSSL) req \
		-in $(INT_CA_CSR) \
		-noout \
		-text

# Check ca certificate.
.PHONY: check_int_ca_crt
check_int_ca_crt:
	$(OPENSSL) x509 \
		-in $(INT_CA_CRT) \
		-noout \
		-text

# Create CRL (Certificate Revocation List)
$(INT_CA_CRL):
	$(OPENSSL) ca -gencrl \
		-config $(CONFIG_INT_CA) \
		-out $@

# Check CRL
.PHONY: check_int_ca_crl
check_int_ca_crl:
	$(OPENSSL) crl \
		-inform pem \
		-in $(INT_CA_CRL) \
		-text

#
# Following rules are for building ocsp responder.
#

# Create a rsa private key for ocsp certificate.
# NOTE: I want to encrypt the private key using -aes256,
#       but there is no way to input passphrase to "openssl ocsp" command automatically
#       when I run ocsp responder.
$(INT_OCSP_KEY):
	$(OPENSSL) genrsa \
		-out $@ \
		2048

# Create CSR for ocsp certificate.
$(INT_OCSP_CSR): $(INT_OCSP_KEY) $(CONFIG_INT_OCSP_REQ)
	$(OPENSSL) req \
		-new \
		-config $(CONFIG_INT_OCSP_REQ) \
		-key $(INT_OCSP_KEY) \
		-out $@

# Check CSR for ocsp certificate.
.PHONY: check_int_ocsp_csr
check_int_ocsp_csr:
	$(OPENSSL) req \
		-in $(INT_OCSP_CSR) \
		-noout \
		-text

# Create ocsp certificate.
$(INT_OCSP_CRT): $(INT_OCSP_KEY) $(INT_OCSP_CSR) $(CONFIG_INT_CA)
	$(OPENSSL) ca \
		-in $(INT_OCSP_CSR) \
		-out $@ \
		-config $(CONFIG_INT_CA) \
		-notext \
		-extensions ocsp_ext

# Check ocsp certificate
.PHONY: check_int_ocsp_crt
check_int_ocsp_crt:
	$(OPENSSL) x509 \
		-in $(INT_OCSP_CRT) \
		-noout \
		-text

#
# Following rules are for CA tasks.
#

# Create server certificate
$(SERVER_CRT): $(SERVER_CSR) $(INT_CA_KEY) $(CONFIG_INT_CA)
	$(OPENSSL) ca \
		-in $(SERVER_CSR) \
		-out $@ \
		-config $(CONFIG_INT_CA) \
		-notext \
		-extensions server_ext \
		-days 365

# Revoke certificate
# -crl_reason : The reason why the certificate was revoked. Following values can be used.
#                 unspecified
#                 keyCompromise
#                 CACompromise
#                 affiliationChanged
#                 superseded
#                 cessationOfOperation
#                 certificateHold
#                 removeFromCRL
.PHONY: revoke
revoke:
	@echo "Please execute following command."
	@echo ""
	@echo "  $(OPENSSL) ca \\"
	@echo "    -config $(CONFIG_INT_CA) \\"
	@echo "    -revoke certs/<CRT_FILE_NAME> \\"
	@echo "    -crl_reason <REASON>"
	@echo ""
	@echo "You can choose one of the followings as <REASON>"
	@echo "  * unspecified"
	@echo "  * keyCompromise"
	@echo "  * CACompromise"
	@echo "  * affiliationChanged"
	@echo "  * superseded"
	@echo "  * cessationOfOperation"
	@echo "  * certificateHold"
	@echo "  * removeFromCRL"
	@echo ""

.PHONY: clean
clean:
	rm -rf \
		certs \
		db \
		private \
		$(INT_CA_KEY) \
		$(INT_CA_CSR) \
		$(INT_CA_CRT) \
		$(INT_CA_CRL) \
		$(INT_OCSP_KEY) \
		$(INT_OCSP_CSR) \
		$(INT_OCSP_CRT) \
		*.old \
		*.attr
