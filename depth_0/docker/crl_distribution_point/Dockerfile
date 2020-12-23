# base image
FROM httpd:2.4

# copy root-ca.crt and root-ca.crl
COPY rootCa/root-ca.crt /usr/local/apache2/htdocs/
COPY rootCa/root-ca.crl /usr/local/apache2/htdocs/
