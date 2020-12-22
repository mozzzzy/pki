config = root-ca.conf
root_ca_crl = root-ca.crl
root_ca_crt = root-ca.crt
root_ca_csr = root-ca.csr
root_ca_key = private/root-ca.key
sub_ca_csr = ../sub-ca/sub-ca.csr
sub_ca_crt = ../sub-ca/sub-ca.crt
sub_ocsp_crt = ../sub-ca/sub-ocsp.crt
sub_ocsp_csr = ../sub-ca/sub-ocsp.csr
root_ocsp_crt = root-ocsp.crt
root_ocsp_csr = root-ocsp.csr
root_ocsp_key = private/root-ocsp.key

# directory と file の初期化
.PHONY: init
init:
	mkdir -p certs db private
	chmod 700 private
	touch db/index
	openssl rand -hex 16 > db/serial
	echo 1001 > db/crlnumber

# private key と csr の生成
${root_ca_csr}: ${config}
	openssl req \
	-new \
	-config ${config} \
	-out ${root_ca_csr} \
	-keyout ${root_ca_key}

# 自己署名証明書の生成
${root_ca_crt}: ${config} ${root_ca_csr} ${root_ca_key}
	openssl ca \
	-config ${config} \
	-selfsign \
	-in ${root_ca_csr} \
	-out ${root_ca_crt} \
	-extensions ca_ext

#
# これより以下は、CAとしての操作
#

# CRL(証明書失効リスト)の生成
# db から Revoke の情報をとってきているぽい
${root_ca_crl}: ${config}
	openssl ca \
	-gencrl \
	-config ${config} \
	-out ${root_ca_crl}

# CRLの内容確認 (bulletproof には書かれていなかったが、確認のため)
.PHONY: check_crl
check_crl:
	openssl crl \
	-inform pem \
	-in ${root_ca_crl} \
	-text

# sub CA 証明書の発行
.PHONY: ${sub_ca_crt}
${sub_ca_crt}:
	openssl ca \
	-config ${config} \
	-in ${sub_ca_csr} \
	-out ${sub_ca_crt} \
	-extensions sub_ca_ext

# sub OCSP 署名のための証明書生成
${sub_ocsp_crt}: ${sub_ocsp_csr}
	openssl ca \
	-config ${config} \
	-in ${sub_ocsp_csr} \
	-out ${sub_ocsp_crt} \
	-extensions ocsp_ext \
	-days 30

# 証明書の失効
# -crl_reason に指定できるのは
#   unspecified          : 指定しない
#   keyCompromise        : 鍵が危殆化した
#   CACompromise         : CA のセキュリティが侵害された
#   affiliationChanged   : 所属が変わった
#   superseded           : 破棄する
#   cessationOfOperation : 運用を停止する
#   certificateHold      : 証明書を保留する
#   removeFromCRL        : CRLから削除
#.PHONY: revoke
#revoke:
#	openssl ca \
#	-config ${config} \
#	-revoke certs/1002.pem \
#	-crl_reason keyCompromise

# OCSP 署名のための証明書のための CSR 生成
${root_ocsp_csr}:
	openssl req -new \
	-newkey rsa:2048 \
	-subj "/C=GB/O=Example/CN=OCSP Root Responder" \
	-keyout ${root_ocsp_key} \
	-out ${root_ocsp_csr}

# OCSP 署名のための証明書生成
${root_ocsp_crt}: ${root_ocsp_csr}
	openssl ca \
	-config ${config} \
	-in ${root_ocsp_csr} \
	-out ${root_ocsp_crt} \
	-extensions ocsp_ext \
	-days 30

# OCSP responder の起動
.PHONY: run_ocsp_responder
run_ocsp_responder:
	openssl ocsp \
	-port 9080 \
	-index db/index \
	-rsigner ${root_ocsp_crt} \
	-rkey ${root_ocsp_key} \
	-CA ${root_ca_crt} \
	-text

# OCSP へリクエスト送信
# -issuer : 証明書発行元の証明書ファイルパス
# -CAfile : 信頼する追加の証明書ファイル
# -cert   : 検査する証明書ファイル
.PHONY: test_ocsp_responder
test_ocsp_responder:
	openssl ocsp \
	-issuer ${root_ca_crt} \
	-CAfile ${root_ca_crt} \
	-cert ${root_ocsp_crt} \
	-url http://127.0.0.1:9080

.PHONY: clean
clean:
	rm -rf \
	certs db private \
	root-ca.crl root-ca.csr root-ca.crt \
	root-ocsp.csr root-ocsp.crt
