FROM fedora:latest

MAINTAINER Avesh Agarwal <avagarwa@redhat.com>

RUN \
    yum -y update && \
    yum -y \
#       --disablerepo=epel \
        install etcd && \
# Keep layers small by removing useless cache
    yum clean all 

ADD root /

EXPOSE 4001 7001 2379 2380

ENTRYPOINT ["etcd-env.sh", "/usr/bin/etcd"]
