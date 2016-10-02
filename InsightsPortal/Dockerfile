# Copyright (c) 2012-2016 Codenvy, S.A.
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Eclipse Public License v1.0
# which accompanies this distribution, and is available at
# http://www.eclipse.org/legal/epl-v10.html
# Contributors:
# Codenvy, S.A. - initial API and implementation

FROM debian:jessie
MAINTAINER Andrey Solodov andrey.solodov@aurea.com
# 8083 frontend
# 3307 MySql
# 8082 API
EXPOSE 4403 8080 8000 22 3307 8082 8083


RUN apt-get update && \
    apt-get -y install curl sudo

# update repos for installing nodejs
RUN curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -

RUN apt-get update && \
    apt-get -y install locales openssh-server sudo procps wget zip unzip mc git curl subversion nmap net-tools libfontconfig nodejs bzip2 python-pip ruby-full && \
    mkdir /var/run/sshd && \
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && \
    echo "%sudo ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
    useradd -u 1000 -G users,sudo -d /home/user --shell /bin/bash -m user && \
    echo "secret\nsecret" | passwd user &&  apt-get -y autoremove \
    && apt-get -y clean \
    && rm -rf /var/lib/apt/lists/*

# supervisord
RUN pip install supervisor
# supervisord config file
RUN echo_supervisord_conf > /etc/supervisord.conf && \
    echo "[program:sshd]" >> /etc/supervisord.conf && \
    echo "command=sudo /usr/sbin/sshd -D" >> /etc/supervisord.conf && \
    echo "numprocs=1" >> /etc/supervisord.conf && \
    echo "autostart=true" >> /etc/supervisord.conf && \
    echo "[program:dockerd]" >> /etc/supervisord.conf && \
    echo "command=sudo /usr/bin/dockerd" >> /etc/supervisord.conf && \
    echo "numprocs=1" >> /etc/supervisord.conf && \
    echo "autostart=true" >> /etc/supervisord.conf

# installing docker
RUN wget -qO- https://get.docker.com/ | sh
#RUN curl -fsSLO https://get.docker.com/builds/Linux/x86_64/docker-1.11.1.tgz && tar --strip-components=1 -xvzf docker-1.11.1.tgz -C /usr/local/bin
# installing docker-compose
RUN curl -L https://github.com/docker/compose/releases/download/1.8.0-rc1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
RUN chmod +x /usr/local/bin/docker-compose

#installing Compass
RUN sudo gem update --system && \
    sudo gem install compass

USER user

LABEL che:server:8080:ref=tomcat8 \
      che:server:8080:protocol=http \
      che:server:8000:ref=tomcat8-debug \
      che:server:8000:protocol=http \
      che:server:8082:ref=InsightsPortal-API \
      che:server:8082:protocol=http \
      che:server:8083:ref=InsightsPortal-FRONTEND \
      che:server:8083:protocol=http \
      che:server:3307:ref=InsightsPortal-DB \
      che:server:3307:protocol=http
    
ENV MAVEN_VERSION=3.3.9 \
    JAVA_VERSION=8u45 \
    JAVA_VERSION_PREFIX=1.8.0_45 \
    TOMCAT_HOME=/home/user/tomcat8 \
    GRADLE_HOME=/opt/gradle-3.1

ENV JAVA_HOME=/opt/jdk$JAVA_VERSION_PREFIX \
M2_HOME=/home/user/apache-maven-$MAVEN_VERSION

ENV PATH=$GRADLE_HOME/bin:$JAVA_HOME/bin:$M2_HOME/bin:$PATH

RUN mkdir /home/user/cbuild /home/user/tomcat8 /home/user/apache-maven-$MAVEN_VERSION && \
  wget \
  --no-cookies \
  --no-check-certificate \
  --header "Cookie: oraclelicense=accept-securebackup-cookie" \
  -qO- \
  "http://download.oracle.com/otn-pub/java/jdk/$JAVA_VERSION-b14/jdk-$JAVA_VERSION-linux-x64.tar.gz" | sudo tar -zx -C /opt/ && \
  wget -qO- "http://apache.ip-connect.vn.ua/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz" | tar -zx --strip-components=1 -C /home/user/apache-maven-$MAVEN_VERSION/

ENV TERM xterm
ENV COMPOSE_HTTP_TIMEOUT 600

RUN wget -qO- "http://archive.apache.org/dist/tomcat/tomcat-8/v8.0.24/bin/apache-tomcat-8.0.24.tar.gz" | tar -zx --strip-components=1 -C /home/user/tomcat8 && \
    rm -rf /home/user/tomcat8/webapps/*


ENV LANG C.UTF-8
RUN echo "export JAVA_HOME=/opt/jdk$JAVA_VERSION_PREFIX\nexport M2_HOME=/home/user/apache-maven-$MAVEN_VERSION\nexport TOMCAT_HOME=/home/user/tomcat8\nexport PATH=$JAVA_HOME/bin:$M2_HOME/bin:$PATH" >> /home/user/.bashrc && \
    sudo localedef -i en_US -f UTF-8 en_US.UTF-8
RUN sudo usermod -aG docker user

# installing PhantomJS
WORKDIR /opt
#RUN sudo wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2 && \
#    sudo tar xjvf phantomjs-2.1.1-linux-x86_64.tar.bz2 && \
#    sudo ln -s /opt/phantomjs-2.1.1-linux-x86_64/bin/phantomjs /usr/bin/phantomjs

# installing Gradle
RUN sudo wget https://services.gradle.org/distributions/gradle-3.1-bin.zip && \
    sudo unzip gradle-3.1-bin.zip && \
    sudo rm gradle-3.1-bin.zip
WORKDIR /projects

CMD supervisord -c /etc/supervisord.conf && tail -f /dev/null