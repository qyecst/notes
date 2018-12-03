<!--
{
    "title": "docker相关",
    "create": "2018-12-03 09:59:37",
    "modify": "2018-12-03 09:59:37",
    "tag": [
        "docker",
        "docker-compose"
    ],
    "info": []
}
-->

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
