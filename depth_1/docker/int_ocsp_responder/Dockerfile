# base image
FROM centos:7

# install openssl
RUN yum install -y openssl

CMD ["openssl","ocsp","-port","9081","-index","/root/intCa/db/index","-rsigner","/root/intCa/int-ocsp.crt","-rkey","/root/intCa/private/int-ocsp.key","-CA","/root/intCa/int-ca.crt","-text"]
