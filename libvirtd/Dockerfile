FROM debian:stretch
ENV LC_ALL C
ENV DEBIAN_FRONTEND noninteractive
ENV container docker
ENV init /lib/systemd/systemd

MAINTAINER Aleksej Gavrilov <le9i0nx@gmail.com>


RUN DEBIAN_FRONTEND=noninteractive \
    apt-get update  && \
    apt-get dist-upgrade -y && \
    apt-get install -y \
        locales ca-certificates systemd \
        libvirt-daemon-system libvirt-clients bridge-utils \
        ebtables iproute2 qemu-kvm kmod ovmf sgabios libguestfs-tools && \
    locale-gen --purge en_US.UTF-8 && \
    echo -e 'LANG="en_US.UTF-8"\nLANGUAGE="en_US:en"\n' > /etc/default/locale && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*  /tmp/* /var/cache/apt/*

RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
systemctl enable libvirtd; \
systemctl enable virtlockd 

RUN systemctl mask \
    systemd-remount-fs.service \
    dev-hugepages.mount \
    sys-fs-fuse-connections.mount \
    systemd-logind.service \
    getty.target \
    console-getty.service \
    dev-mqueue.mount \
    sys-kernel-config.mount \
    sys-kernel-debug.mount \
    display-manager.service \
    graphical.target \
    exim4.service \
    systemd-tmpfiles-clean.timer

EXPOSE 16509

RUN echo "listen_tls = 0" >> /etc/libvirt/libvirtd.conf; \
echo 'listen_tcp = 1' >> /etc/libvirt/libvirtd.conf; \
echo 'tls_port = "16514"' >> /etc/libvirt/libvirtd.conf; \
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf; \
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf

#RUN echo 'LIBVIRTD_ARGS="--listen"' >> /etc/sysconfig/libvirtd
RUN mkdir -p /var/lib/libvirt/images/

# Edit the service file which includes ExecStartPost to chmod /dev/kvm

RUN sed -i "/Service/a ExecStartPost=\/bin\/chmod 666 /dev/kvm" /usr/lib/systemd/system/libvirtd.service

VOLUME ["/sys/fs/cgroup"]
ENTRYPOINT ["/lib/systemd/systemd"]
# docker run --name t0 --rm -e 'container=docker' --privileged --net=host -ti -v /sys/fs/cgroup:/sys/fs/cgroup:ro le9i0nx/test:libvirtd
# docker exec -ti t0 /bin/bash

