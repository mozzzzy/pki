OPENSSL=openssl
CONFIG_ROOT_CA_REQ=config/root-ca-req.conf
CONFIG_ROOT_CA=config/ca.conf
CONFIG_ROOT_OCSP_REQ=config/root-ocsp-req.conf
ROOT_CA_KEY=private/root-ca.key
ROOT_CA_CSR=root-ca.csr
ROOT_CA_CRT=root-ca.crt
ROOT_CA_CRL=root-ca.crl
ROOT_OCSP_KEY=private/root-ocsp.key
ROOT_OCSP_CSR=root-ocsp.csr
ROOT_OCSP_CRT=root-ocsp.crt
INT_CA_CSR=../intCa/int-ca.csr
INT_CA_CRT=../intCa/int-ca.crt

.PHONY: help
help:
	@echo "Usage:"
	@echo "  Initialize directories                 : make init"
	@echo "  Remove all directories and files       : make clean"
	@echo "  -------- For building Root CA --------"
	@echo "  Create private key for root ca         : make $(ROOT_CA_KEY)"
	@echo "  Create csr for root ca                 : make $(ROOT_CA_CSR)"
	@echo "  Check csr for root ca                  : make check_root_ca_csr"
	@echo "  Create certificate for root ca         : make $(ROOT_CA_CRT)"
	@echo "  Check certificate for root ca          : make check_root_ca_crt"
	@echo "  Create crl for root ca                 : make $(ROOT_CA_CRL)"
	@echo "  Check crl for root ca                  : make check_root_ca_crl"
	@echo "  ----- For buiding OCSP Responder -----"
	@echo "  Create private key for ocsp responder  : make $(ROOT_OCSP_KEY)"
	@echo "  Create csr for ocsp responder          : make $(ROOT_OCSP_CSR)"
	@echo "  Check csr for ocsp responder           : make check_root_ocsp_csr"
	@echo "  Create certificate for ocsp responder  : make $(ROOT_OCSP_CRT)"
	@echo "  Check certificate for ocsp responder   : make check_root_ocsp_crt"
	@echo "  ------------ For CA Tasks ------------"
	@echo "  Create certificate for intermediate ca : make $(INT_CA_CRT)"

#
# Following rules are for building root ca.
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
$(ROOT_CA_KEY):
	$(OPENSSL) genrsa \
		-out $@ \
		-aes256 \
		4096

# Create CSR (Certificate Signing Request) for ca certificate.
$(ROOT_CA_CSR): $(ROOT_CA_KEY) $(CONFIG_ROOT_CA_REQ)
	$(OPENSSL) req \
		-new \
		-config $(CONFIG_ROOT_CA_REQ) \
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
$(ROOT_CA_CRT): $(ROOT_CA_CSR) $(ROOT_CA_KEY) $(CONFIG_ROOT_CA)
	$(OPENSSL) ca \
		-config $(CONFIG_ROOT_CA) \
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

# Create CRL (Certificate Revocation List)
$(ROOT_CA_CRL):
	$(OPENSSL) ca -gencrl \
		-config $(CONFIG_ROOT_CA) \
		-out $@

# Check CRL
.PHONY: check_root_ca_crl
check_root_ca_crl:
	$(OPENSSL) crl \
		-inform pem \
		-in $(ROOT_CA_CRL) \
		-text

#
# Following rules are for building ocsp responder.
#

# Create a rsa private key for ocsp certificate.
# NOTE: I want to encrypt the private key using -aes256,
#       but there is no way to input passphrase to "openssl ocsp" command
#       when I run ocsp responder.
$(ROOT_OCSP_KEY):
	$(OPENSSL) genrsa \
		-out $@ \
		2048

# Create CSR for ocsp certificate.
$(ROOT_OCSP_CSR): $(ROOT_OCSP_KEY) $(CONFIG_ROOT_OCSP_REQ)
	$(OPENSSL) req \
		-new \
		-config $(CONFIG_ROOT_OCSP_REQ) \
		-key $(ROOT_OCSP_KEY) \
		-out $@

# Check CSR for ocsp certificate.
.PHONY: check_root_ocsp_csr
check_root_ocsp_csr:
	$(OPENSSL) req \
		-in $(ROOT_OCSP_CSR) \
		-noout \
		-text

# Create ocsp certificate.
$(ROOT_OCSP_CRT): $(ROOT_OCSP_KEY) $(ROOT_OCSP_CSR) $(CONFIG_ROOT_CA)
	$(OPENSSL) ca \
		-in $(ROOT_OCSP_CSR) \
		-out $@ \
		-config $(CONFIG_ROOT_CA) \
		-notext \
		-extensions ocsp_ext \
		-days 365

# Check ocsp certificate
.PHONY: check_root_ocsp_crt
check_root_ocsp_crt:
	$(OPENSSL) x509 \
		-in $(ROOT_OCSP_CRT) \
		-noout \
		-text

#
# Following rules are for CA tasks.
#

# Create server certificate
$(INT_CA_CRT): $(INT_CA_CSR) $(ROOT_CA_KEY) $(CONFIG_ROOT_CA)
	$(OPENSSL) ca \
		-in $(INT_CA_CSR) \
		-out $@ \
		-config $(CONFIG_ROOT_CA) \
		-notext \
		-extensions int_ca_ext \
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
	@echo "    -config $(CONFIG_ROOT_CA) \\"
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
		$(ROOT_CA_KEY) \
		$(ROOT_CA_CSR) \
		$(ROOT_CA_CRT) \
		$(ROOT_CA_CRL) \
		$(ROOT_OCSP_KEY) \
		$(ROOT_OCSP_CSR) \
		$(ROOT_OCSP_CRT) \
		*.old \
		*.attr
