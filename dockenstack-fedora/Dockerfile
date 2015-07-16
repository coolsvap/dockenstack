# Swapnil Kulkarni 2015
# Forked from code by original author: Eric Windisch

FROM fedora
MAINTAINER Swapnil Kulkarni <me@coolsvap.net>

EXPOSE 80 5000 8773 8774 8776 9292

# Install Docker
RUN yum -y install docker

# Install utilities
RUN yum -y install git socat curl sudo vim wget

# Extra requirements for pip-requirements
RUN yum -y install libffi-devel

# Configure and install MySQL
RUN yum -y install mysql-server; \
    mysql_install_db --datadir=/var/lib/mysql/; \
    chown -R mysql.mysql /var/lib/mysql;

# Install RabbitMQ
RUN yum -y install rabbitmq-server

RUN yum -y install sqlite-devel
RUN yum -y install openldap-devel
RUN yum -y install gcc-c++

# Copy in docker images
RUN mkdir -m 755 -p /opt/dockenstack/images
WORKDIR /opt/dockenstack/images
RUN umask 022; \
    wget --progress=dot:mega \
         http://35d32e50e9d5a9dddd5f-7400ae4d3b198289837a7cdf652ffd5a.r94.cf1.rackcdn.com/cirros.img
RUN umask 022; \
    wget --progress=dot:mega \
    http://35d32e50e9d5a9dddd5f-7400ae4d3b198289837a7cdf652ffd5a.r94.cf1.rackcdn.com/registry.img

# Setup devstack user
RUN mkdir -p /opt; \
    useradd -m -s /bin/bash -d /opt/stack devstack && \
    usermod -a -G docker devstack
ADD devstack.sudo /etc/sudoers.d/devstack
RUN chown root:root /etc/sudoers.d/devstack

# Local scripts
ADD scripts /opt/dockenstack/bin
RUN chmod 755 /opt/dockenstack/bin/*

# Install devstack scripts
RUN git clone https://github.com/openstack-dev/devstack /devstack

# Install prereq packages.
RUN /devstack/tools/install_prereqs.sh || :

# Pre-download all "NOPRIME" packages
RUN /bin/bash /opt/dockenstack/bin/rpm-cache-devstack

RUN yum -y install python-pip

# Install all pip requirements
RUN pip install -r https://raw.github.com/openstack/requirements/master/global-requirements.txt
RUN pip install -r https://raw2.github.com/openstack/tempest/master/requirements.txt

# Pre-checkout git repos
RUN su devstack -c '/bin/bash /opt/dockenstack/bin/openstack-git-checkout'

RUN git clone https://github.com/stackforge/nova-docker /opt/stack/nova-docker
WORKDIR /opt/stack/nova-docker
RUN cp contrib/devstack/extras.d/70-docker.sh /devstack/extras.d/; \
    cp contrib/devstack/lib/nova_plugins/hypervisor-docker /devstack/lib/nova_plugins/;

WORKDIR /devstack
ADD localrc /devstack/localrc
ADD localrc.d /devstack/localrc.d

WORKDIR /
# Fix ownership of all files
RUN chown -R devstack /devstack

CMD ["/opt/dockenstack/bin/start"]
