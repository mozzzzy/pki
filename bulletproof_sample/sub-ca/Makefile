config = sub-ca.conf
sub_ca_crl = sub-ca.crl
sub_ca_crt = sub-ca.crt
sub_ca_csr = sub-ca.csr
sub_ca_key = private/sub-ca.key
sub_ocsp_csr = sub-ocsp.csr
sub_ocsp_crt = sub-ocsp.crt
sub_ocsp_key = private/sub-ocsp.key
server_csr = ../server/server.csr
server_crt = ../server/server.crt
client_csr = ../client/client.csr
client_crt = ../client/client.crt

# directory と file の初期化
.PHONY: init
init:
	mkdir -p certs db private
	chmod 700 private
	touch db/index
	openssl rand -hex 16 > db/serial
	echo 10001 > db/crlnumber

# private key と csr の生成
${sub_ca_csr}: ${config}
	openssl req \
	-new \
	-config ${config} \
	-out ${sub_ca_csr} \
	-keyout ${sub_ca_key}

#
# これより以下は、CAとしての操作
#

# CRL（証明書失効リスト）の生成
# db から Revoke の情報をとってきているぽい
${sub_ca_crl}: ${config}
	openssl ca \
	-gencrl \
	-config ${config} \
	-out ${sub_ca_crl}

# CRLの内容確認（bulletproof には書かれていなかったが、確認のため）
.PHONY: check_crl
check_crl:
	openssl crl \
	-inform pem \
	-in ${sub_ca_crl} \
	-text

# サーバ証明書の発行
${server_crt}:
	openssl ca \
	-config ${config} \
	-in ${server_csr} \
	-out ${server_crt} \
	-extensions server_ext

# クライアント証明書の発行
${client_crt}:
	openssl ca \
	-config ${config} \
	-in ${client_csr} \
	-out ${client_crt} \
	-extensions client_ext

# OCSP 署名のための証明書のための CSR 生成
${sub_ocsp_csr}:
	openssl req -new \
	-newkey rsa:2048 \
	-subj "/C=GB/O=Example/CN=OCSP Root Responder" \
	-keyout ${sub_ocsp_key} \
	-out ${sub_ocsp_csr}

# OCSP 署名のための証明書発行
${sub_ocsp_crt}: ${sub_ocsp_csr}
	openssl ca \
	-config ${config} \
	-in ${sub_ocsp_csr} \
	-out ${sub_ocsp_crt} \
	-extensions ocsp_ext \
	-days 30

# OCSP responder の起動
.PHONY: run_ocsp_responder
run_ocsp_responder:
	openssl ocsp \
	-port 9081 \
	-index db/index \
	-rsigner ${sub_ocsp_crt} \
	-rkey ${sub_ocsp_key} \
	-CA ${sub_ca_crt} \
	-text

# OCSP へリクエスト送信
# -issuer : 証明書発行元の証明書ファイルパス
# -CAfile : 信頼する追加の証明書ファイル
# -cert   : 検査する証明書ファイル
.PHONY: test_ocsp_responder
test_ocsp_responder:
	openssl ocsp \
	-issuer ${sub_ca_crt} \
	-CAfile ${sub_ca_crt} \
	-cert ${sub_ocsp_crt} \
	-url http://127.0.0.1:9081

.PHONY: clean
clean:
	rm -rf \
	certs db private \
	sub-ca.crl sub-ca.csr sub-ca.crt \
	sub-ocsp.csr sub-ocsp.crt