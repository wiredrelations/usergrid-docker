# Copyright 2014-2015 Jahn Bertsch
# Copyright 2015 TOMORROW FOCUS News+ GmbH
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
# http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# build usergrid from source

FROM bobbywired/usergrid-java

WORKDIR /root
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64
ENV PATH $PATH:$JAVA_HOME/bin

COPY ./tomcat9/etc/init.d/tomcat  /root/tomcat

# build usergrid
# this is done in one run step so less files are included in the layers of the
# docker image, making it smaller.
RUN \
  echo "+++ install tomcat and packages required for compilation" && \
  apt-get install -y software-properties-common && \
  rm -vrf /var/lib/apt/lists/* && \
  apt-get update && \
  apt-get install -y maven curl git-core && \
  \
#  echo "+++ fix tomcat7 init script: add missing java8 location" && \
#  sed -i "s#/usr/lib/jvm/java-7-oracle#/usr/lib/jvm/java-7-oracle /usr/lib/jvm/java-8-oracle#g" /etc/init.d/tomcat7 && \
#  \
  echo "+++ Setup Tomcat9" && \
  useradd -m -U -d /opt/tomcat -s /bin/false tomcat && \
  VERSION=9.0.48 && \
  wget https://www-eu.apache.org/dist/tomcat/tomcat-9/v${VERSION}/bin/apache-tomcat-${VERSION}.tar.gz -P /tmp && \
  tar -xf /tmp/apache-tomcat-${VERSION}.tar.gz -C /opt/tomcat/ && \
  ln -s /opt/tomcat/apache-tomcat-${VERSION} /opt/tomcat/latest && \
  sh -c 'chmod +x /opt/tomcat/latest/bin/*.sh' && \
  sh -c 'cp /root/tomcat /etc/init.d/tomcat' && \
  sh -c 'chmod 775 /etc/init.d/tomcat' && \
  sh -c 'update-rc.d tomcat defaults' && \
  #printf '[Unit]\nDescription=Tomcat 9 servlet container\nAfter=network.target\n\n[Service]\nType=forking\nUser=tomcat\nGroup=tomcat\nEnvironment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"\nEnvironment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom -Djava.awt.headless=true"\nEnvironment="CATALINA_BASE=/opt/tomcat/latest"\nEnvironment="CATALINA_HOME=/opt/tomcat/latest"\nEnvironment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"\nEnvironment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"\nExecStart=/opt/tomcat/latest/bin/startup.sh\nExecStop=/opt/tomcat/latest/bin/shutdown.sh\n\n[Install]\nWantedBy=multi-user.target' /etc/systemd/system/tomcat.service && \
  #sh -c 'systemctl daemon-reload' && \
  #sh -c 'systemctl enable --now tomcat' && \
  \
  echo "+++ get usergrid-java source" && \
  git clone --single-branch --branch master --depth 50 https://github.com/apache/usergrid-java.git usergrid-java && \
  cd usergrid-java && \
  \
  echo "+++ build usergrid-java" && \
  cd /root/usergrid-java && \
  mvn --quiet clean install -DskipTests -DskipIntegrationTests && \
  \
  echo "+++ get usergrid source, set logging level" && \
  cd /root && \
  git clone --single-branch --branch master --depth 50 https://github.com/apache/usergrid.git usergrid && \
  cd usergrid && \
  git checkout 1bf227b1b6b3913abaac6c0648d0714fd2cacc93 && \
  grep -rl log4j.rootLogger=INFO stack | xargs sed -i 's#log4j.rootLogger=INFO#log4j.rootLogger=WARN#g' && \
  \
  echo "+++ build usergrid" && \
  cd /root/usergrid/stack && \
  mvn --quiet clean install -DskipTests -DskipIntegrationTests && \
  \
  echo "+++ cleanup" && \
#  rm -rf /var/lib/tomcat9/webapps/ROOT && \
#  mv /root/usergrid/stack/rest/target/ROOT.war /var/lib/tomcat9/webapps && \
#  mv /root/usergrid/stack/config/src/main/resources/usergrid-default.properties /usr/share/tomcat9/lib/usergrid-deployment.properties && \
  rm -rf /opt/tomcat/latest/webapps/ROOT && \
  mv /root/usergrid/stack/rest/target/ROOT.war /opt/tomcat/latest/webapps && \
  mv /root/usergrid/stack/config/src/main/resources/usergrid-default.properties /opt/tomcat/latest/lib/usergrid-deployment.properties && \
  apt-get purge --auto-remove -y maven git-core ant && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /root/usergrid-java && \
  rm -rf /root/usergrid

# default command when starting container with "docker run"
CMD /root/run.sh

# exposed ports:
#  8080 usergrid http interface
#  8443 usergrid https interface
EXPOSE 8080 8443

# runtime configuration script: since this is updated frequently during development, add it last
COPY run.sh /root/run.sh
