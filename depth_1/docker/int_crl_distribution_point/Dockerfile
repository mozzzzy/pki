# base image
FROM httpd:2.4

# copy int-ca.crt and int-ca.crl
COPY intCa/int-ca.crt /usr/local/apache2/htdocs/
COPY intCa/int-ca.crl /usr/local/apache2/htdocs/
