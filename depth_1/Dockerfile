# base image
FROM httpd:2.4

#
# html files
#

COPY files/htdocs/ /usr/local/apache2/htdocs/

#
# enable tls
#

COPY client/ClientCrt.pem /usr/local/apache2/conf/server.crt
COPY client/ClientKey.pem /usr/local/apache2/conf/server.key

RUN sed -i \
  -e 's/^#\(Include .*httpd-ssl.conf\)/\1/' \
  -e 's/^#\(LoadModule .*mod_ssl.so\)/\1/' \
  -e 's/^#\(LoadModule .*mod_socache_shmcb.so\)/\1/' \
  /usr/local/apache2/conf/httpd.conf
