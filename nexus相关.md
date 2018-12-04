<!--
{
    "title": "nexus相关",
    "create": "2018-12-04 15:14:15",
    "modify": "2018-12-04 15:14:15",
    "tag": [
        "nexus"
    ],
    "info": [
        "创建仓库//todo"
    ]
}
-->

## Sonatype Nexus

Sonatype Nexus: Maven仓库管理软件，常用于搭建私服。其他还有Apache Archiva、Artifactory

### 相关链接

```url
# software
https://www.sonatype.com/nexus-repository-oss

# docker
https://hub.docker.com/r/sonatype/nexus3/

# document
https://help.sonatype.com/repomanager3/download
```

## docker-compose部署

### docker-compose文件

```dockercompose
version: "3.5"
services:
    nexus3:
        image: sonatype/nexus3
        container_name: nexus3
        ports:
            - "8081:8081"
        volumes:
            - ./datadir/:/nexus-data/:rw
        restart: always
```
