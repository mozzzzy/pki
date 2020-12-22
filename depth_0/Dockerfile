# base image
FROM httpd:2.4

#
# html files
#

COPY files/htdocs/ /usr/local/apache2/htdocs/

#
# enable tls
#

COPY server/server.crt /usr/local/apache2/conf/server.crt
COPY server/private/server.key /usr/local/apache2/conf/server.key

#
# cert passphrase (This is not secure. but it's ok because this is demo env.)
#
COPY files/pass_input.sh /usr/local/bin/pass_input.sh
RUN chmod 500 /usr/local/bin/pass_input.sh

RUN sed -i \
  -e 's/^#\(Include .*httpd-ssl.conf\)/\1/' \
  -e 's/^#\(LoadModule .*mod_ssl.so\)/\1/' \
  -e 's/^#\(LoadModule .*mod_socache_shmcb.so\)/\1/' \
  /usr/local/apache2/conf/httpd.conf

RUN sed -i 's/<\/IfModule>/SSLPassPhraseDialog "exec:\/usr\/local\/bin\/pass_input.sh"\n<\/IfModule>/g' /usr/local/apache2/conf/httpd.conf
