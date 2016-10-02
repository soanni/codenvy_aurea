# Copyright (c) 2012-2016 Codenvy, S.A.
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Eclipse Public License v1.0
# which accompanies this distribution, and is available at
# http://www.eclipse.org/legal/epl-v10.html
# Contributors:
# Codenvy, S.A. - initial API and implementation

FROM centos
# 8080 - jboss
# 8787 - debug jboss
EXPOSE 4403 8080 22 8787

#EXPOSE 4403 8080 8000 22 
# debug JBoss
#EXPOSE 8787

RUN yum update -y && \
    yum -y install sudo openssh-server procps wget unzip mc git curl subversion nmap && \
    mkdir /var/run/sshd && \
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && \
    echo "%wheel ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
    useradd -u 1000 -G users,wheel -d /home/user --shell /bin/bash -m user && \
    echo -e "codenvy2016\ncodenvy2016" | passwd user && \
    sed -i 's/requiretty/!requiretty/g' /etc/sudoers

USER user

#LABEL che:server:8080:ref=tomcat8 che:server:8080:protocol=http che:server:8000:ref=tomcat8-debug che:server:8000:protocol=http
LABEL che:server:8080:ref=jboss \
      che:server:8080:protocol=http \
      che:server:8087:ref=jboss-debug \
      che:server:8087:protocol=http

#JAVA_VERSION=6u91 
#JAVA_VERSION_PREFIX=1.6.0_91 
#JAVA_HOME=/opt/jre1.6.0_91/bin
#TOMCAT_HOME=/home/user/tomcat8

ENV MAVEN_VERSION=3.3.9 \
    JAVA_VERSION=8u45 \
    JAVA_VERSION_PREFIX=1.8.0_45
ENV JAVA_HOME=/opt/jdk$JAVA_VERSION_PREFIX \
    M2_HOME=/home/user/apache-maven-$MAVEN_VERSION

RUN mkdir /home/user/apache-maven-$MAVEN_VERSION && \
    wget -qO- "http://apache.ip-connect.vn.ua/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz" | tar -zx --strip-components=1 -C /home/user/apache-maven-$MAVEN_VERSION/

RUN wget \
  --no-cookies \
  --no-check-certificate \
  --header "Cookie: oraclelicense=accept-securebackup-cookie" \
  -qO- \
  "http://download.oracle.com/otn-pub/java/jdk/$JAVA_VERSION-b14/jdk-$JAVA_VERSION-linux-x64.tar.gz" | sudo tar -zx -C /opt/
ENV TERM xterm

USER root
RUN echo -e "JAVA_HOME=/opt/jdk$JAVA_VERSION_PREFIX\nM2_HOME=/home/user/apache-maven-$MAVEN_VERSION\nTOMCAT_HOME=/home/user/tomcat8\nPATH=$JAVA_HOME/bin:$M2_HOME/bin:$PATH" >> /etc/environment
USER user
ENV LANG C.UTF-8

############# download ant
RUN mkdir /home/user/apache-ant-1.9.7 && wget -qO- "http://apache-mirror.rbc.ru/pub/apache//ant/binaries/apache-ant-1.9.7-bin.tar.gz" | tar -zx --strip-components=1 -C /home/user/apache-ant-1.9.7
RUN sudo ln -s /home/user/apache-ant-1.9.7/bin/ant /usr/bin/ant
############# JBoss && JDK 6u45
RUN sudo mkdir /data && sudo mkdir /data/app && \
    cd /data/app && \
    sudo svn checkout --no-auth-cache --username asolodov --password jd6xugLMrr https://subversion.devfactory.com/repos/AureaSpiral_JtoB/docker/codenvy/jboss/ && \
    cd /opt && \
    sudo svn checkout --no-auth-cache --username asolodov --password jd6xugLMrr https://subversion.devfactory.com/repos/AureaSpiral_JtoB/docker/codenvy/install/ && \
    sudo rpm -i /opt/install/jdk-6u45-linux-amd64.rpm

#RabbitMQ
# install EPEL sofware repository
RUN sudo yum -y -q install epel-release --nogpgcheck

# install RabbitMQ server
RUN sudo yum -y -q install rabbitmq-server --nogpgcheck 

# enable RabbitMQ server plugins
RUN sudo rabbitmq-plugins enable amqp_client mochiweb rabbitmq_management rabbitmq_management_visualiser rabbitmq_stomp webmachine rabbitmq_shovel rabbitmq_shovel_management

# user:user owner
RUN sudo chown -R user:user /data

WORKDIR /projects

CMD sudo /usr/bin/ssh-keygen -A && \
    sudo /usr/sbin/sshd -D && \
    tail -f /dev/null


