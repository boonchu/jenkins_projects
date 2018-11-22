#### Installing Tomcat on Ubuntu 16.04

- Clone VM disk
```
VBoxManage clonehd /Volumes/WDElement2T/Template-Images/Ubuntu\ JVM\ \ 16.04.5\ \(64bit\).vdi /Volumes/WDElement2T/VM\ Images/Tomcat\ Stage\ 1.vdi
```

- Add user 'tomcat'
```
groupadd tomcat
useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
```

- Install tomcat software
```
cd /tmp && curl -O curl -O http://apache.mirrors.ionfish.org/tomcat/tomcat-8/v8.5.35/bin/apache-tomcat-8.5.35.tar.gz
tar xzvf apache-tomcat-8.5.35.tar.gz -C /opt/tomcat --strip-components=1
```

- Change permission location /opt/tomcat
```
chgrp -R tomcat /opt/tomcat
cd /opt/tomcat
chmod -R g+r conf
chmod g+x conf
chown -R tomcat webapps/ work/ temp/ logs/
```

- Append systemd tomcat service /etc/systemd/system/tomcat.service
```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
```
```
systemctl daemon-reload && systemctl start tomcat && systemctl status tomcat
```

- Access web page [IP]:8080

- Change tocmat role /opt/tomcat/conf/tomcat-users.xml
```
  <role rolename="manager-script"/>
  <role rolename="admin-gui"/>
  <user username="tomcat" password="tomcat" roles="manager-script,admin-gui"/>
```
