#### Jenkins CI/CD
- change hostname
```
hostnamectl set-hostname node-1
```

- change /etc/hosts
```
127.0.0.1 node-1
```

- change /etc/network/interfaces to add default gw.
```
up route add -net 0.0.0.0/0 gw 192.168.200.2 dev enp0s
```

- add resolvconf to provison /etc/resolv.conf
```
cd /etc/resolvconf/resolv.conf.d
edit head
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
nameserver 10.0.0.1
nameserver 8.8.8.8

# resolvconf -u
```

- add NTP
```
apt-get update
apt-get install -y ntp ntpdate ntp-doc
ntpdate 0.us.pool.ntp.org
hwclock --systohc
systemctl enable ntp
systemctl start ntp
```

- install MISC tools
apt-get install jq tmux python python-pip parted
echo Console with IP address. Add the script to /etc/network/if-up.d/show-up-address
```
#!/bin/bash

if [ "$METHOD" = loopback ]; then
    exit 0
fi

# Only run from ifup.
if [ "$MODE" != start ]; then
    exit 0
fi

cp /etc/issue-standard /etc/issue
IP_ADDRESS=$(ifconfig enp0s3| awk '/inet / {print $2}' | cut -f2 -d:)
echo $IP_ADDRESS >> /etc/issue
echo "" >> /etc/issue
```

- JDK requirements
```
https://jenkins.io/doc/administration/requirements/java/
OpenJDK JDK / JRE 8 - 64 bits
```
```
apt-get update && apt install openjdk-8-jre-headless && apt install openjdk-8-jdk

root@node-1:~# java -version
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-8u181-b13-1ubuntu0.16.04.1-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```
```
setenv JAVA_HOME jdk-install-dir
setenv PATH $JAVA_HOME/bin:$PATH
export PATH=$JAVA_HOME/bin:$PATH
```

- Instlal Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
echo deb https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
apt-get update
apt-get install jenkins
systemctl start jenkins
```

- Create the first Jenkins job and understand how it works.

#### TASK 1: Create Maven Project

- Add github plugin

- Install Maven
https://www.rosehosting.com/blog/how-to-install-maven-on-ubuntu-16-04/
```
apt-cache show maven | grep Version
Version: 3.3.9-3

apt-get -y install maven

# mvn -v
Apache Maven 3.3.9
Maven home: /usr/share/maven
Java version: 1.8.0_181, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-8-openjdk-amd64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.4.0-131-generic", arch: "amd64", family: "unix"
```

- Configure Global Tool Configuration
```
Configure Java SDK
Configure git {name: git, path to git: git}
Configure maven {name: mvn, MAVEN_HOME: /user/share/maven}
```

- Setup Job
```
Git:
https://github.com/jleetutorial/maven-project

Build:
Invoke top-level Maven targets:
- Maven Version:mvn
- Goals:clean install -U -X
- JVM options:-Djdk.net.URLClassPath.disableClassPathURLCheck=true
```

- Error from the jo, 
```
su - jenkins
cd ~/workspace/maven-project

method: without debugging output.
mvn clean install -U 
method: with debugging output.
mvn clean install -U -X 

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Forking command line: /bin/sh -c cd /var/lib/jenkins/workspace/maven-project/server && /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java -jar /var/lib/jenkins/workspace/maven-project/server/target/surefire/surefirebooter6484219378912071020.jar /var/lib/jenkins/workspace/maven-project/server/target/surefire/surefire523676550179335272tmp /var/lib/jenkins/workspace/maven-project/server/target/surefire/surefire1682245626820082404tmp
Error: Could not find or load main class org.apache.maven.surefire.booter.ForkedBooter
```

- The job failure was resolved by this setup.
```
method: with env variable. *resolved*
_JAVA_OPTIONS=-Djdk.net.URLClassPath.disableClassPathURLCheck=true mvn clean install -U

https://talk.openmrs.org/t/error-could-not-find-or-load-main-class-org-apache-maven-surefire-booter-forkedbooter/20509
https://stackoverflow.com/questions/53010200/maven-surefire-could-not-find-forkedbooter-class
https://stackoverflow.com/questions/50661648/spring-boot-fails-to-run-maven-surefire-plugin-classnotfoundexception-org-apache/50661649#50661649
```
