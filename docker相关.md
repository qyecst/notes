<!--
{
    "title": "docker相关",
    "create": "2018-12-03 09:59:37",
    "modify": "2018-12-03 15:13:05",
    "tag": [
        "docker",
        "dockerfile",
        "docker-compose",
        "docker-compose.yaml"
    ],
    "info": []
}
-->

## 常用配置

```conf
# 默认配置文件 /etc/docker/daemon.json

{
  "registry-mirrors" : [ # 仓库源
    "http://some_mirror_url"
  ],
  "insecure-registries" : [ # 允许不安全/http的仓库源
    "registry_host:port"
  ],
  "userns-remap": "some_username" # 用户资源隔离 若值为 default 会自动创建并映射为dockremap用户和用户组
}

## userns-remap: 修改 /etc/subuid 和 /etc/subgid 添加下列两行，建立宿主机用户/用户组到容器用户的映射
# some_username:1000:1
# some_username:100000:65536
## 1000为宿主机的uid，表示容器root映射为宿主机uid:1000的用户，其他用户映射为uid:100000 ~ 100000+65536的用户，也受uid:1000的管理
```

## Dockerfile操作

### Dockerfile文件格式：

```dockerfile
# 指定构建基于的镜像
FROM ubuntu:18.04
# 镜像信息
MAINTAINER qyecst qyecst@qyecst.cn
# 暴露端口
EXPOSE 8080
# 执行命令，尽量同一层执行并最后清除临时文件 以减少镜像尺寸
RUN set -ex && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse" > /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse" >> /etc/apt/sources.list && \
    apt update -y && \
    apt upgrade -y && \
    apt install -y --no-install-recommends \
                                          vim \
                                          ssh \
                                          git \
                                          python3 \
                                          software-properties-common \
                                          language-pack-zh-hans && \
    add-apt-repository -y ppa:webupd8team/java && \
    apt update -y && \
    apt upgrade -y && \
    echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections && \
    apt install -y --no-install-recommends \
                                          oracle-java8-installer && \
    apt clean all && \
    rm -rf /var/cache/*
# 指定环境变量
ENV SHELL /bin/bash
# 添加文件，COPY为复制，单纯复制
COPY ./bashrc /root/.bashrc
# 添加文件，ADD会自动解压文件，xxx.tar会被解压
ADD ./ide_files/ide-backend.jar /root
# 执行命令
RUN update-java-alternatives -s java-8-oracle && \
    update-locale LANG=zh_CN.UTF-8 && \
    git config --global core.quotepath false && \
    git config --global core.editor vim && \
    git config --global commit.template /root/gitcommittemplate
# 工作目录
WORKDIR /root
# CMD 命令设置容器启动后默认执行的命令及其参数，但能被`docker run`命令后面的命令行参数替换
# ENTRYPOINT 命令设置容器启动时的执行命令，一定会被执行，即使`docker run`时指定了其他命令
# CMD echo "Hello world" 或 ENTRYPOINT ["/bin/echo", "Hello"] 
CMD ["java", "-jar", "ide-backend.jar", "--PTY_LIB_FOLDER=/root/lib"]
```

### CMD/ENTRYPOINT

```dockerfile
# ENTRYPOINT 中的参数始终会被使用，CMD 的额外参数可以在容器启动时动态替换掉

CMD echo "Hello World"
# docker run -it [image]  输出  `Hello World`
# docker run -it [image] /bin/bash  则CMD命令被忽略，进入bash交互  `root@[container_id]:/#`

ENTRYPOINT ["/bin/echo", "Hello World "]
# docker run -it [image]  输出  `Hello World`
# docker run -it [image] some_text  因ENTRYPOINT的命令不会被覆盖，输出  `Hello World some_text`

ENTRYPOINT ["/bin/echo", "Hello"]
CMD ["World"]
# docker run -it [image]  输出  `Hello World`
# docker run -it [image] some_text  因CMD被覆盖，输出  `Hello some_text`
```

## docker常用操作

```bash
# 显示容器列表
docker ps [-a|--all]
# 启动容器，-d 后台运行  --rm 容器停止后删除  -i 交互式  -t 虚拟终端
# docker run [options] image_name[:tag] [command]
docker run --rm -it centos:latest /bin/bash
```

## docker-compose常用操作

### 安装

```bash
# https://github.com/docker/compose/releases

curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### 配置文件格式

```dockercompose
# 默认寻找目录下的 docker-compose.[yaml|yml] 文件
# /path/to/conf/file/docker-compose.yaml
# /path/to/conf/file/docker-compose.yml

# 版本
version: "3.5"

# 服务列表
services:
    # 服务名
    nginx:
        # 构建方式，dockerfile目录，image为构建后镜像名
        #build: ./website/
        #image: my_nginx
        # 构建方式，指定详细参数
        #build:
        #    context: ./dir
        #    dockerfile: Dockerfile-somefile
        #    args:
        #        buildno: 1
        # 镜像方式
        image: nginx:latest

        # 重启方式
        restart: always
        # 容器名
        container_name: nginx
        # 暴露端口/非映射/提供container之间的端口访问
        expose:
            - "80"
        # 映射端口
        ports:
            - "80:80"
            - "443:443"
        # 挂载卷
        volumes:
            - ./nginx.conf:/etc/nginx/nginx.conf:ro
            - ./data/:/usr/share/nginx/html/:rw
        # 所属网络
        networks:
            - nginx
        # 主机名
        hostname: nginx
        # 指定dns
        dns:
            - 223.5.5.5
            - 1.1.1.1

        # 环境变量，字典或数组格式
        environment:
            DB_HOST: db_host:3306
            DB_PWD: db_passwd
        #environment:
        #    - DB_HOST=db_host:3306
        #    - DB_PWD=db_passwd

        # 依赖，会后于依赖启动
        depends_on:
            - some_depends_service1
            - some_s2

        # 覆盖缺省命令
        command: bundle exec thin -p 3000
        #command: [bundle, exec, thin, -p, 3000]

        # 连接到其他容器，可以使用别名，默认别名为服务名
        links:
            - db:my_alias_name
            - redis
        # 添加主机hosts映射
        extra_hosts:
            - "somehost:192.168.1.1"
            - "otherhost:192.168.1.2"
# 网络列表
networks:
    # 网络
    nginx:
        # 名称
        name: nginx
```
