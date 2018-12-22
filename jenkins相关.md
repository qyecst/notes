<!--
{
    "title": "jenkins相关",
    "create": "2018-12-22 22:47:50",
    "modify": "2018-12-22 22:47:50",
    "tag": [
        "jenkins",
        "ci",
        "cd"
    ],
    "info": []
}
-->

## 安装

```bash
# https://jenkins.io/download/
# 2.54 (2017-04) and newer: Java 8 / 依赖Java
yum install java

# lts version
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins

# non lts version
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins

## /etc/sysconfig/jenkins /etc/rc.d/init.d/{functions,jenkins}
## /var/log/jenkins, /var/lib/jenkins, and /var/cache/jenkins

# docker
docker pull jenkins/jenkins:lts # latest LTS
docker pull jenkins/jenkins # latest weekly

## store the workspace in /var/jenkins_home including plugins and configuration
docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

## https://github.com/jenkinsci/docker/blob/master/README.md
docker run --name myjenkins -p 8080:8080 -p 50000:50000 --env JAVA_OPTS=-Dhudson.footerURL=http://mycompany.com jenkins/jenkins:lts

## logging
mkdir data
cat > data/log.properties <<EOF
handlers=java.util.logging.ConsoleHandler
jenkins.level=FINEST
java.util.logging.ConsoleHandler.level=FINEST
EOF
docker run --name myjenkins -p 8080:8080 -p 50000:50000 --env JAVA_OPTS="-Djava.util.logging.config.file=/var/jenkins_home/log.properties" -v `pwd`/data:/var/jenkins_home jenkins/jenkins:lts

## change the default slave agent port # dockerfile
FROM jenkins/jenkins:lts
ENV JENKINS_SLAVE_AGENT_PORT 50001
# or
docker run --name myjenkins -p 8080:8080 -p 50001:50001 --env JENKINS_SLAVE_AGENT_PORT=50001 jenkins/jenkins:lts
```
