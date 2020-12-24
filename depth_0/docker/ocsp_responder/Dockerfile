# base image
FROM centos:7

# install openssl
RUN yum install -y openssl

CMD ["openssl","ocsp","-port","9080","-index","/root/rootCa/db/index","-rsigner","/root/rootCa/root-ocsp.crt","-rkey","/root/rootCa/private/root-ocsp.key","-CA","/root/rootCa/root-ca.crt","-text"]
