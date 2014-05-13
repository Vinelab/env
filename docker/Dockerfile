##
#
# setup:
#   - generate an ssh key to be used as an authentication key with `ssh-keygen` and call it `docker_ssh_rsa`
#   - make sure your current ssh agent has the identity file added
#     - $ eval `ssh-agent`
#     - $ ssh-add docker_ssh_rsa
#
# build: $ sudo docker build -t <you>/spvssh .
#
# run: $ sudo docker run -d -p 8022:22 <you>/spvssh
#
# ssh access: $ ssh root@localhost -p 8022
#
##

FROM centos:latest

MAINTAINER Abed Halawi <halawi.abed@gmail.com>

# install openssh server
RUN yum -y install openssh-server

# install openssh clients
RUN yum -y install openssh-clients

# make ssh directories
RUN mkdir /root/.ssh
RUN mkdir /var/run/sshd

# create host keys
RUN ssh-keygen -b 1024 -t rsa -f /etc/ssh/ssh_host_key
RUN ssh-keygen -b 1024 -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN ssh-keygen -b 1024 -t dsa -f /etc/ssh/ssh_host_dsa_key

# move public key to enable ssh keys login
ADD docker_ssh.pub /root/.ssh/authorized_keys
RUN chmod 400 /root/.ssh/authorized_keys
RUN chown root:root /root/.ssh/authorized_keys
# tell ssh to not use ugly PAM
RUN sed -i 's/UsePAM\syes/UsePAM no/' /etc/ssh/sshd_config

# make the terminal prettier
RUN echo 'export PS1="[\u@docker] \W # "' >> /root/.bash_profile

# enable networking
RUN echo 'NETWORKING=yes' >> /etc/sysconfig/network

# install supervisord
RUN /bin/rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
RUN yum -y install python-setuptools
RUN easy_install supervisor
RUN /usr/bin/echo_supervisord_conf > /etc/supervisord.conf
RUN mkdir -p /var/log/supervisord

# make supervisor run in foreground
RUN sed -i -e "s/^nodaemon=false/nodaemon=true/" /etc/supervisord.conf

# tell supervisor to include relative .ini files
RUN mkdir /etc/supervisord.d
RUN echo [include] >> /etc/supervisord.conf
RUN echo 'files = /etc/supervisord.d/*.ini' >> /etc/supervisord.conf

# add sshd program to supervisord config
RUN echo [program:sshd] >> /etc/supervisord.d/ssh.ini
RUN echo 'command=/usr/sbin/sshd -D' >> /etc/supervisord.d/ssh.ini
RUN echo  >> /etc/supervisord.d/ssh.ini

EXPOSE 22

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
