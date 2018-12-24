<!--
{
    "title": "docker相关",
    "create": "2018-12-03 09:59:37",
    "modify": "2018-12-25 01:04:16",
    "tag": [
        "docker",
        "dockerfile",
        "docker-compose",
        "docker-compose.yaml"
    ],
    "info": [
        " Flocker：容器的分布式存储平台//todo"
    ]
}
-->

## 信息

### 虚拟化

虚拟化技术KVM，Xen，VMware Esxi，Virtual Box，Docker等。虚拟化物理机器，该物理机器可以直接或间接支持虚拟化，虚拟机管理程序(virtual machine monitor/VMM)，可以看作是平台硬件和操作系统之间的抽象化/中间件。

VMM为每个虚拟机分配一套数据结构来管理其状态，包括虚拟处理器的全套寄存器，物理内存使用情况，网络设备状态，虚拟设备状态等。

VMM可以对底层HostOS硬件资源(物理cpu，内存，磁盘，网卡，显卡等)进行封装/隔离/抽象为另一种形式的逻辑资源，提供给上层GuestOS虚拟机使用，通过虚拟化技术实现的虚拟机称为GuestOS，作为其载体的物理主机称为HostOS。

完全虚拟化技术，通过软件实现对操作系统资源的再分配，KVM，Virtual Box等

半虚拟化技术，通过代码修改已有系统，形成新的可虚拟化系统，调用硬件资源安装多个系统，整体速度相对较高，Xen等

轻量级虚拟化，介于上述之间，如docker

### docker信息

镜像，一个静态模板，与ISO类似，一个样板，不能直接修改，通过封装生成。

容器，基于镜像运行启动的应用/系统，称之为容器/虚拟机。

仓库，用于存放镜像，常见分为公开/私有镜像仓库。

docker虚拟化，早期为LXC+联合文件系统(another union file system/AUFS组合，后引入libcontainer，视作LXC替代。LXC负责资源管理，AUFS负责镜像管理，LXC包括Cgroup(control groups，Namespace，Chroot等组件，通过Cgroups资源管理。Cgroup在底层资源管理，LXC在Cgroups封装一层，docker在LXC封装一层。

Cgroups为linux内核提供可限制/记录/隔离进程组所使用的物理资源(cpu，内存，磁盘io，网卡流量等。

docker容器可看作一个或多个运行进程，占有相应的内存/cpu计算资源/虚拟网络设备/文件系统资源等，其占用的文件系统资源通过镜像的镜像层文件提供。基于每个镜像层的json文件，docker可解析镜像的json文件，获知镜像之上运行的进程，如何配置环境变量，docker守护进程实现了静态向动态的转变。

镜像分层，每个镜像由一个或多个镜像层组成，可以在某个镜像上加一定的镜像层获得新的镜像(通过dockerfile实现，每个镜像层由唯一id，镜像在存储和使用时共享相同的镜像层(id，镜像层是只读的，启动容器时，在其上添加一层空的可读写的镜像层，容器进程只写读写层。

### docker网络

基于docker run创建容器，可以--net指定容器网络模式

host模式 --net=host  基于host模式，容器不会获得独立的network namespace，与宿主机公用一个network namespace，容器不会虚拟自己的网卡，而是使用宿主机的ip和端口

container模式 --net=container  基于此模式，指定新创建的容器和已存在的容器共享一个network namespace，而不是和宿主机共享。容器不会虚拟自己的网卡，而是和指定的容器共享ip/端口等，两容器除了网络外，其他文件系统/进程列表等还是隔离的

none模式 --net=none  基于此模式，容器有自己的network namespace，但是不为容器进行网络配置，需要手动为容器添加网卡，配置ip等。pipework工具等

bridge模式 --net=bridge 默认模式，基于此模式，为容器分配network namespace，设置ip/路由等，默认将容器连接到虚拟网桥交换机docker0上

## dokcer安装

```bash
# centos7.x ubuntu/debian
yum install docker
apt install docker.io

## centos 6.x
# yum install lxc libcgroup device-mapper-event-libs
# yum install docker-io device-mapper
# /etc/init.d/docker restart
```

## docker常用操作

```bash
## 搜索
docker search <keyword>

## 拉取/推送镜像
docker pull <image>:<tag>
docker push <image>:<tag>

## 显示容器列表
docker ps [-a|--all]

## 启动容器，-d 后台运行  --rm 容器停止后删除  -i 交互式  -t 虚拟终端
# docker run [options] image_name[:tag] [command]
docker run --rm -it centos:latest /bin/bash

## 导入镜像/导出镜像
cat image.tar | docker import - centos # 导入centos镜像
docker export <container_id> > centos.tar # 导出镜像

## 提交修改的容器/从修改容器创建镜像
docker commit <container_id> <image_name>:<tag>

## 其他操作
docker [stop|start|rm] <container_id>
docker rmi <image_id>
docker -p 80:80 # docker -P
docker exec -it <container_id> /bin/bash
docker exec <container_id> df -h

## 容器卷
docker volume create --name vol0 # 创建vol0卷
docker volume ls # 查看
docker volume inspect vol0 # 查看vol0详细信息，驱动/存储点等
docker run -d -P --name test -v vol0:/path <image> [command] # 使用vol0卷
docker run -d -P --name test -v /data <image> [command] # 会默认创建一个卷，路径在/var/lib/docker/... 使用inspect查看
docker rm -v # 删除容器时删除volume卷
docker volume rm $(docker volume ls -qf dangling=true) # 删除未使用的volume卷
```

## 常用配置

### 环境配置

```conf
# 需要开启ip_forward以支持docker内网络转发
# /etc/sysctl.conf `sysctl -p`

net.ipv4.ip_forward=1
```

### 配置文件

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
```

### user-remap/用户映射

```bash
# /etc/docker/daemon.json

{
    "userns-remap": "some_username" # 用户资源隔离 若值为 default 会自动创建并映射为dockremap用户和用户组
}
## userns-remap: 修改 /etc/subuid 和 /etc/subgid 添加下列两行，建立宿主机用户/用户组到容器用户的映射
some_username:1000:1
some_username:100000:65536
## 1000为宿主机的uid，表示容器root映射为宿主机uid:1000的用户，其他用户映射为uid:100000 ~ 100000+65536的用户，也受uid:1000的管理
```

### 桥接配置

自定义docker桥接网卡，设置docker容器ip与宿主机同网段，无需NAT即可访问容器，也可以基于pipework为容器指定静态ip

#### 网卡配置文件

```bash
## vim /etc/sysconfig/network-scripts/ifcfg-ens33
DEVICE=ens33
BOOTPROTO=none
NM_CONTROLLED=no
ONBOOT=yes
TYPE=Ethernet
BRIDGE=br0
IPADDR=1.1.1.1
NETMASK=255.255.255.0
GATEWAY=1.1.1.254
USERCTL=no # 是否允许非root用户控制该设备

## vim /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE=br0
BOOTPROTP=none
IPV6INIT=no
NM_CONTROLLED=no
ONBOOT=yes
TYPE=Bridge
IPADDR=1.1.1.1
NETMASK=255.255.255.0
GATEWAY=1.1.1.254
USERCTL=no

## 重启网络服务
systemctl restart network

## 配置docker
# centos 7.x
vim /etc/sysconfig/docker-network
> DOCKER_NETWORK_OPTIONS="-b=br0"
# centos 6.x
vim /etc/sysconfig/docker
> other_args="-b=br0"
systemctl restart docker
```

#### 命令操作

```bash
## 设置网桥
yum install bridge-utils # 安装相关库
ifconfig docker0 down
# ip link set dev docker0 down
brctl delbr docker0
brctl addbr br0
ip link set dev br0 up # 开启br0
ip addr add 1.1.1.1/24 dev br0 # 设置br0的ip
ip addr del 1.1.1.1/24 dev ens33 # 删除宿主机网卡ip
brctl addif br0 ens33 # 将宿主机网卡挂到br0
ip route del default
ip route add default via 1.1.1.1 dev br0 # 设置路由

## 设置docker
# centos 7.x
vim /etc/sysconfig/docker-network
> DOCKER_NETWORK_OPTIONS="-b=br0"
# centos 6.x
vim /etc/sysconfig/docker
> other_args="-b=br0"
systemctl restart docker
```

#### pipework指定静态ip

```bash
# https://github.com/jpetazzo/pipework
# 启动容器，网络none，名称test
docker run -itd --net=none --name=test centos7 /bin/bash
# pipework设置ip 1.1.1.3 网关 1.1.1.1 掩码24
pipework br0 test 1.1.1.3/24@1.1.1.1
```

### docker磁盘扩容

存储格式为devicemapper

```bash
vim /etc/sysconfig/docker-storage
> DOCKER_STORAGE_OPTIONS="--storage-driver [overlay2|devicemapper|aufs|btrfs] "

# --storage-driver devicemapper

# vim /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

使用Device Mapper的存储插件。device mapper是linux2.6后的从逻辑设备到物理设备映射框架机制，device mapper driver默认创建100GB存储文件，用于存储镜像和容器，每个容器被限制于10G之内，也可以基于lookback自动创建稀疏文件，具体为用/var/lib/docker/devicemapper/devicemapper下的data和metadata实现动态磁盘扩容

利用docker volume绕过存储驱动，提升io效率

```bash
## 配置文件设置
vim /etc/sysconfig/docker-storage
> DOCKER_STORAGE_OPTIONS="--storage-opt dm.loopdatasize=2000G --storage-opt dm.loopmetadatasize=10G --storage-opt dm.fs=ext4"
systemctl restart docker

## 启动配置/命令操作
docker -d --storage-opt dm.foo=bar
dm.basesize # 默认10G，限制容器/镜像大小
dm.loopdatasize # 存储池大小，默认100G
dm.datadev # 存储池设备 /var/lib/docker/devicemapper/devicemapper/data
dm.loopmetadatadev # 元数据设备 /var/lib/docker/devicemapper/devicemapper/metadata
dm.fs # 文件系统，默认ext4
dm.blocksize <size> # 默认64K
dm.blkdiscard # 默认true
# 将默认池100G=>2T 元数据2G=>10G
rm -rf /var/lib/docker/devicemapper/devicemapper
mkdir -p /var/lib/docker/devicemapper/devicemapper
dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/data bs=1G count=0 seek=2000 # seek从输出文件开头跳过blocks个块后再开始复制
dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/metadata bs=1G count=0 seek=10

## 在线扩容方式
df -h # 查看device mapper名称
ll /dev/mapper/docker-*
dmsetup table docker-*
> 0 20971520 thin 253:0 30 # 20971520 表示多少个512B扇区
echo $((15*1024*1024*1024/512))
> 31457280
# 修改大小
echo 0 31457280 thin 253:0 30 | dmsetup load docker-*
dmsetup resume docker-*
dmsetup table docker-*
resize2fs /dev/mapper/docker-*
# 验证大小
df -h
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
# ENTRYPOINT ["/bin/echo", "Hello World"]    exec执行格式，推荐
# ENTRYPOINT /bin/echo Hello World    shell执行格式
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
