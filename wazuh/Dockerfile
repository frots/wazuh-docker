FROM milcom/centos7-systemd

RUN rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
ADD config/elastic.repo /etc/yum.repos.d/elastic.repo

RUN yum -y update; yum clean all;
RUN yum -y install epel-release openssl useradd; yum clean all


RUN yum -y install filebeat

COPY config/filebeat.yml /etc/filebeat/filebeat.yml

RUN chkconfig --add filebeat
RUN /usr/share/filebeat/bin/filebeat -c /etc/filebeat/filebeat.yml &

RUN yum -y install wget
RUN yum -y install vim expect gcc make openssl-devel unzip
RUN yum -y install curl && curl -sL https://rpm.nodesource.com/setup_6.x | bash -
RUN yum -y install nodejs

RUN wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.rpm"

RUN yum -y localinstall jdk-8u60-linux-x64.rpm &&\
    rm jdk-8u60-linux-x64.rpm &&\
	export JAVA_HOME=/usr/java/jdk1.8.0_60/jre
	


RUN cd root && mkdir wazuh_tmp && cd wazuh_tmp

RUN wget -O wazuh.zip https://github.com/wazuh/wazuh/archive/master.zip &&\
    unzip wazuh.zip &&\
    mv wazuh-master /root/wazuh_tmp/wazuh
	
COPY config/preloaded-vars.conf /root/wazuh_tmp/wazuh/etc/preloaded-vars.conf

RUN /root/wazuh_tmp/wazuh/install.sh	

RUN wget -O wazuh-api.zip https://github.com/wazuh/wazuh-api/archive/master.zip &&\
    unzip wazuh-api.zip &&\
    mkdir -p /var/ossec/api && cp -r wazuh-api-master/* /var/ossec/api &&\
    cd /var/ossec/api && npm install
	
RUN rm -rf wazuh-api-master && rm -rf wazuh-api.zip && rm -rf wazuh-master && rm -rf wazuh.zip

ADD config/default_agent /var/ossec/default_agent

RUN /var/ossec/bin/ossec-control stop

RUN /var/ossec/bin/manage_agents -f /default_agent &&\
  rm /var/ossec/default_agent &&\
  echo -n "" /var/ossec/logs/ossec.log

RUN /var/ossec/bin/ossec-control start
  
ADD config/data_dirs.env /data_dirs.env
ADD config/init.bash /init.bash
# Sync calls are due to https://github.com/docker/docker/issues/9547
RUN chmod 755 /init.bash &&\
  sync && /init.bash &&\
  sync && rm /init.bash

ADD config/run.sh /tmp/run.sh
RUN chmod 755 /tmp/run.sh  

VOLUME ["/var/ossec/data"]  
  
EXPOSE 55000/tcp 1514/udp 1515/tcp 514/udp  
  
  
ENTRYPOINT ["/tmp/run.sh"]