<!--
{
    "title": "nginx相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-03 11:25:37",
    "tag": [
        "nginx"
    ],
    "info": []
}
-->

## nginx安装

### 编译安装

```bash
# 依赖
yum install zlib-devel pcre-devel openssl-devel

# 添加用户
useradd -M -s /sbin/nologin nginx

# 配置
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module

#./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf \
#   --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx/nginx.pid \
#   --lock-path=/var/lock/nginx.lock --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module \
#   --with-http_gzip_static_module --http-client-body-temp-path=/var/tmp/nginx/client/ --http-proxy-temp-path=/var/tmp/nginx/proxy/ \
#   --http-fastcgi-temp-path=/var/tmp/nginx/fcgi/ --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi --http-scgi-temp-path=/var/tmp/nginx/scgi --with-pcre

# 编译&安装
make clean
make -j [num_of_processes]
make install

# 开放防火墙端口
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -p tcp --dport 443 -j ACCEPT

# 立即生效
firewall-cmd --add-port=80/tcp --add-port=443/tcp
# firewall-cmd --reload 后生效/重启生效/长期生效
firewall-cmd --add-port=80/tcp --add-port=443/tcp --permanent
```

### 软件源安装

```bash
yum install nginx
apt install nginx
```

## 常用操作

```bash
nginx -t # 检测配置文件语法
nginx -s reload # 重新加载配置
# nginx -s [stop|quit|reopen|reload]
```

## 日志分析

```bash
# 统计独立ip数
awk '{print $1}' access.log | sort -r | uniq -c | wc -l

# 统计pv
awk '{print $1}' access.log | wc -l

# 统计uv
awk '{print $1}' access.log | sort -r | uniq -c | wc -l

# 统计访问前20ip
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -20

# 统计9点到12点访问量
?????

# 统计状态码，打印大于20此的IP
awk '{if($9~/502|499|500|503|404/)print $1,$9}' access.log | sort | uniq -c | sort -nr | awk '{if($1>20)print $2}'

# 分析请求时间大于5s的url，打印url 时间 ip # NR当前行数 NF字段总数 $NF最后一个Field RT指定的那个分隔符 RS记录分隔符 ORS记录输出分隔符 FS分隔符 OFS列输出分隔符
# log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for" "$request_time"';
awk '{if($NF>5)print $NF,$7,$1}' access.log | sort -nr
```

## 日志切分

```bash
cp log log.bak.<date> && echo -n > log
# kill -USR1 <nginx.pid>
# bash脚本定时执行
```

## nginx配置

```nginx
## 定义Nginx运行的用户和用户组
user  nginx nginx;

## nginx进程数，建议设置为等于CPU总核心数 [auto]
worker_processes  8;
## 为进程分配cpu，每个进程1个cpu
worker_cpu_affinity 00000001 00000010 00000100 ... 10000000;

## 进程pid文件
pid  /var/run/nginx.pid;

## 一个nginx进程打开的最多文件描述符数目，理论值应是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不均匀，建议与ulimit -n的值一致
#worker_rlimit_nofile  65535;

## 工作模式与连接数上限
events {
    ## 参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ];
    ## epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型
    use  epoll;

    ## 单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections  1024;

    ## 收到一个新连接通知后接受尽可能多的连接
    multi_accept  on;

    ## 优化同一时刻只有一个请求而避免多个睡眠进程被唤醒的设置，on为防止被同时唤醒
    accept_mutex  on;
}

## 设定http服务器
http {
    ## 关闭版本信息
    server_tokens  off;

    ## 开启目录列表访问
    # autoindex on;

    ## 默认编码
    charset  utf-8;

    ## 文件扩展名与文件类型映射表
    include  mime.types;

    ## 默认文件类型
    default_type  application/octet-stream;

    ## 关闭 等待数据0.2s，将这段时间内的数据打成一个大的包发送
    tcp_nodelay  on;

    ## 当包累计到一定大小后发送
    tcp_nopush  on;

    ## 开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为on
    ## 如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载
    ## 如果图片显示不正常把这个改成off
    sendfile  on;

    ## 工作进程每次调用sendfile()传输的数据最大不能超出这个值，默认值为0表示无限制
    #sendfile_max_chunk  512k;

    ## 日志格式设定
    log_format  main    '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';

    ## 定义访问日志 [off]
    access_log  /var/log/nginx/access.log main;

    ## 错误日志定义类型，[ debug | info | notice | warn | error | crit ]
    error_log  /var/log/nginx/error.log warn;

    ## 连接超时时间，单位是秒 keepalive_timeout timeout [header_timeout];
    ## 要是上传文件比较大，在规定时间内没有上传完成，就会自动断开连接
    keepalive_timeout  120s 120s;

    ## 设置一个keep-alive连接上可以服务的请求的最大数量，当最大请求数量达到时，连接被关闭
    keepalive_requests  300;

    ## 反向代理时，为了支持长连接，注意配置：
        ##http {
        ##    upstream  BACKEND {
        ##        server  192.168.0.1：8080 weight=1 max_fails=2 fail_timeout=30s;
        ##        server  192.168.0.2：8080 weight=1 max_fails=2 fail_timeout=30s;
        ##        keepalive  300; ## 这个很重要，为了保持后端长连接，设置到upstream服务器的空闲keepalive连接的最大数量
        ##    }
        ##    server {
        ##        listen  8080;
        ##        server_name  "";
        ##        location  / {
        ##            proxy_pass  http://BACKEND;
        ##            proxy_set_header  Host $Host;
        ##            proxy_set_header  x-forwarded-for $remote_addr;
        ##            proxy_set_header  X-Real-IP $remote_addr;
        ##            proxy_http_version  1.1; ## 这个最好也设置，HTTP协议中对长连接的支持是从1.1版本之后才有的
        ##            proxy_set_header  Connection ""; ## 这个最好也设置，清理来自client请求中的"Connection" header
        ##        }
        ##    }
        ##}

    ## 请求头和请求体(各自)的超时时间
    client_body_timeout  5s;
    client_header_timeout  5s;

    ## 限制了上传文件的大小
    #client_max_body_size  8m;

    ## 指定客户端的响应超时时间，在这段时间内，客户端没有读取任何数据，nginx就会关闭连接
    #send_timeout  30;

    ## 关闭不响应的客户端连接，这将会释放那个客户端所占有的内存空间
    #reset_timedout_connection  on;

    ## 开启目录列表访问，合适下载服务器，默认关闭
    #autoindex  on;

    ## 开启gzip压缩输出
    gzip  on;

    ## 最小压缩文件大小，0为全压缩
    gzip_min_length  0k;

    ## 压缩缓冲区，以文件大小为准，以16K为单位，以16K的4倍申请内存
    gzip_buffers  4 16k;

    ## 压缩版本（默认1.1，前端如果是squid2.5请使用1.0），不支持压缩的客户端会乱码
    gzip_http_version  1.1;

    ## 压缩等级，1最低9最高，1占用cpu低9高
    gzip_comp_level  6;

    ## 压缩类型，默认包含text/html，写上去也不会有问题，但是会有一个warn
    #gzip_types  text/plain text/css text/javascript application/json application/javascript application/x-javascript;
    gzip_types  *;

    ## vary头，代理时用，有的浏览器支持压缩，有的不支持，避免不支持的也压缩，根据客户端的HTTP头来判断是否压缩
    #gzip_vary  on;

    ## gzip_proxied [off|expired|no-cache|no-store|private|no_last_modified|no_etag|auth|any]
    ## Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头
    ## off - 关闭所有的代理结果数据的压缩
    ## expired - 启用压缩，如果header头中包含 "Expires" 头信息
    ## no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
    ## no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
    ## private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
    ## no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
    ## no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
    ## auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
    ## any - 无条件启用压缩
    #gzip_proxied  off;

    ## 导入外部配置文件
    #include  /etc/nginx/conf.d/*.conf;

    ## FastCGI相关参数是为了改善网站的性能，减少资源占用，提高访问速度
    #fastcgi_connect_timeout  300;
    #fastcgi_send_timeout  300;
    #fastcgi_read_timeout  300;
    #fastcgi_buffer_size  64k;
    #fastcgi_buffers  4 64k;
    #fastcgi_busy_buffers_size  128k;
    #fastcgi_temp_file_write_size  128k;

    ## 限制
    #limit_req_zone  $binary_remote_addr zone=zonenameone:10m rate=100r/s;
    #limit_conn_zone  $binary_remote_addr zone=zonenametwo:10m;
    #limit_conn_zone  $server_name zone=zonenamethree:10m;
        #location  / {
        #    ## 每秒100个请求，允许突发100个
        #    #limit_req  zone=zonenameone burst=100 nodelay;

        #    ## 最大并发连接数
        #    #limit_conn  zonenametwo 30;

        #    ## 该服务提供的总连接数不得超过1000，超过请求的会被拒绝
        #    #limit_conn  zonenamethree 1000;
        #}

    server {
        ## 监听端口
        listen  80;

        ## 服务器名
        server_name  qyecst.cn www.qyecst.cn;

        ## 301跳转
        return  301 https://www.qyecst.cn$request_uri;
    }

    server {
        ## 监听端口
        listen  443;

        ## 服务器名
        server_name  qyecst.cn;

        ## 301跳转
        return  301 https://www.qyecst.cn$request_uri;
    }

    server {
        ## 监听ssl，启用http2
        #listen  443 ssl http2;
        listen  443 ssl http2 default_server;

        ## 服务器名
        server_name  www.qyecst.cn;

        ## 添加头信息--SSL安全part https://cipherli.st/
        ## 开启HSTS，并设置有效期为“6307200秒”（6个月），包括子域名(根据情况可删掉)，预加载到浏览器缓存(根据情况可删掉)
        add_header  Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";

        ## 禁止被嵌入框架
        add_header  X-Frame-Options DENY;

        ## 防止在IE9、Chrome和Safari中的MIME类型混淆攻击
        add_header  X-Content-Type-Options nosniff;

        ## XSS保护
        add_header  X-XSS-Protection "1; mode=block";

        ## SSL证书密匙文件--SSL安全part https://cipherli.st/
        ssl_certificate  /etc/ssl/cert.pem;
        ssl_certificate_key  /etc/ssl/key.pem;

        ## OCSP Stapling的证书位置
        ssl_trusted_certificate  /etc/ssl/cert.pem;

        ## DH-Key交换密钥文件位置
        ssl_dhparam  /etc/ssl/dhparam.pem;

        ## SSL安全设置--SSL安全part https://cipherli.st/
        ## SSL session过期时间
        ssl_session_timeout  10m;

        ## 只允许TLS协议
        ssl_protocols  TLSv1.3; # nginx版本大等于1.13.0 否则使用 TLSv1.2

        ## 加密套件
        ssl_ciphers  ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS;

        ## secp384r1 是一个曲线名，openssl ecparam -list_curves 可以看所有的曲线名
        ssl_ecdh_curve  secp384r1;

        ## 由服务器协商最佳的加密算法
        ssl_prefer_server_ciphers  on;

        ## Session Cache
        ssl_session_cache  shared:SSL:10m;

        ##浏览器的Session Ticket缓存
        ssl_session_tickets  off;

        ## OCSP Stapling开启,OCSP是用于在线查询证书吊销情况的服务，使用OCSP Stapling能将证书有效状态的信息缓存到服务器，提高TLS握手速度
        ssl_stapling  on;

        ## OCSP Stapling验证开启
        ssl_stapling_verify  on;

        ## 用于查询OCSP服务器的DNS
        resolver  1.1.1.1 223.5.5.5 valid=300s;

        ## 查询域名超时时间
        resolver_timeout  1s;

        ## 访问日志
        access_log  /var/log/nginx/cn.qyecst.access.log;

        ## 错误日志
        error_log  /var/log/nginx/cn.qyecst.error.log;

        location / {
            ## 加载uwsgi
            include uwsgi_params;

            ## 使用socket文件通信
            uwsgi_pass unix:/tmp/uwsgi.sock;

            # root /usr/share/nginx/html;
            # index index.html index.htm;
        }
        #location / {
        #    proxy_pass http://127.0.0.1:8080/;
        #    proxy_http_version 1.1;
        #    proxy_set_header Upgrade $http_upgrade;
        #    proxy_set_header Connection $connection_upgrade;
        #    proxy_set_header X-Forwarded-For $remote_addr;
        #    proxy_set_header X-Real-IP $remote_addr;
        #}
    }
}
```

## nginx代理

```nginx
http {
    upstream <group_name> {
        server 192.168.0.10:88;
        server 192.168.0.11;
        # .etc
    }
    server {
        location / {
            root <path>;
            index index.html;
            proxy_pass http://<group_name>;
            # 后端服务器返回504/502/超时等错误，自动将请求转发upstream的另一台服务器/故障转移
            proxy_next_upstream http_502 http_504 error timeout invalid_header;
            # proxy_redirect http://192.168.1.154:8080/ http://www.test.com/;
        }
    }
}
```

## 负载均衡

权重、轮询、iphash .etc

```nginx
http {
    upstream <group_name> { # 轮询
        server 192.168.0.10:88;
        server 192.168.0.11;
        # .etc
    }
    upstream <group_name> { # 权重
        server 192.168.0.10:88 weight=2;
        server 192.168.0.11 weight=4;
        # .etc
    }
    upstream <group_name> {
        server 192.168.0.10:88 weight=2 max_fails=2 fail_timeout=30s;
        server 192.168.0.11 weight=4 max_fails=2 fail_timeout=30s;
        # .etc
    }
    upstream <group_name> { # iphash
        ip_hash;
        server 192.168.0.10:88;
        server 192.168.0.11;
        # .etc
    }
    server {
        location / {
            root <path>;
            index index.html;
            proxy_pass http://<group_name>;
        }
    }
}
```

## fastcgi配置

```conf
location ~ \.php$ {
    root    html;
    #fastcgi_pass   127.0.0.1:9000;
    fastcgi_pass    unix:/var/run/php-fpm.sock;
    fastcgi_index   index.php;
    fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include     fastcgi_params;
}
# php-fpm配置 默认在www.conf中
# listen.owner=nginx
# listen.group=nginx
# listen.mode=0660
# listen=/var/run/php-fpm.sock
```

## uwsgi配置

```conf
location / {
    # 加载uwsgi
    include uwsgi_params;
    # 使用socket文件通信
    uwsgi_pass unix:/tmp/uwsgi.sock;
    #uwsgi_pass some_uwsgi_server:80;
}
# uwsgi配置，默认在uwsgi.ini中
# socket=/tmp/uwsgi.sock
# #socket=127.0.0.1:80
# chmod-socket=660
# #stats=127.0.0.1:9001
```

## nginx升级/降级

```bash
nginx -V # 获取编译参数
./configure --prefix=...etc # 使用所需编译参数
make # 构建
# 备份旧版nginx可执行文件，复制新版
mv /path/to/nginx/binary/nginx /path/to/backup/nginx.bak
cp path/to/new/nginx /path/to/nginx
# 测试新版本是否正常
/path/to/new/nginx -t
# 重启nginx
```

## 设定查看nginx状态

```nginx
location /nginxstatus {
    stub_status on;
}
```

## location匹配规则

```nginx
 = 字面精确匹配
 ^~ 最大前缀匹配
 ~ 区分大小写
 ~* 不区分大小写
 !~ 区分大小写不匹配
 !~* 不区分大小写不匹配
# location = 匹配 > location 完整路径 > location ^~ 路径 > location ~ ~* 正则 > location 部分起始路径 > location / 路径

location = / {
    ...etc;
    # 只会匹配/，比location / 低
}
location = /index.html {
    ...etc;
    # 只会匹配/index.html，最高优先级
}
location = /images/ {
    ...etc;
    # 匹配以/images/开始的请求，并停止其他匹配
}
location ~ .*\.(html|txt|gif|jpg|jpeg|png)$ {
    ...etc;
    # 匹配以上述结尾的url请求，但是/images/的请求会被上述location处理
}
location / {
    ...etc;
    # 匹配任何请求，请求都以/开头
    # 优先级最低
}
```

## nginx rewrite/nginx重写规则

```nginx
 last # 表示完成匹配
 break # 本规则匹配后终止匹配
 redirect # 返回302临时重定向
 permanent # 返回301永久重定向

# nginx.conf | vhosts.conf

# test.com跳转www.test.com
if ($host = 'test.com') {
    rewrite ^/(.*)$ http://www.test.com/$1 permanent;
}

# 访问 www.test.com跳转www.test1.com/index.html
rewrite ^/$ http://www.test1.com/index.html permanent;

# 访问 /test/dir/跳转至/other.html 且浏览器地址不变
rewrite ^/test/dir/$ /other.html last;

# 多域名跳转
if ($host != 'www.test.com') {
    rewrite ^/(.*)$ http://www.test.com/$1 permanent;
}

# 访问不存在的跳转至index.html
if ( ! -e $request_filename) {
    rewrite ^/(.*)$ /index.html last;
}

# 目录替换 /xxx/123 => /xxx?id=123
rewrite ^/(.+)/(\d+) /$1?id=$2 last;

# 判断浏览器ua
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /ie/$1 break;
}

# 禁止访问
location ~ .*\.(sh|flv|mp3)$ {
    return 403;
}

# 移动端跳转
if ($http_user_agent ~* "(Android)|(iPhone)|(Mobile)|(WAP)|(UCWEB)") {
    rewrite ^/(.*)$ http://m.test.com/$1 permanent;
}

# 匹配url跳转
if ($args ~* tid=12) {
    return 404;
}
```

## 防盗链

```nginx
# 如果valid_referers列表中没有Referer头的值 $invalid_referer 将被设置为1
location ~ .*\.(gif|jpg|png|swf|flv) {
    valid_referers none blocked test.com *.test.com;
    # none没有referers直接通过浏览器访问/其他工具访问
    # blocked有，但是被代理服务器或防火墙隐藏
    # 域名 通过指定域名访问的
    if ($invalid_referer) {
        return 403;
        # rewrite ^/ http://www.test.com/403.html;
    }
}

location ~* \.(gif|jpg|png|swf|flv) {
    if ($host != '*.test.com') {
        return 403;
    }
}
```

## nginx的ssl/https

```bash
# 生成ssl证书
openssl genrsa -des3 -out server.key 1024
# 生成csr
openssl req -new -key server.key -out server.csr
# 去除口令，nginx使用
mv server.key server.key.bak
openssl rsa -in server.key.bak -out server.key
# 生成自签证书
openssl x509 -req -days 10240 -in server.csr -signkey server.key -out server.crt

# 配置nginx的ssl
server {
    listen 443 ssl;
    server_name www.test.com;
    ssl_certificate /path/to/server.crt;
    ssl_certificate_key /path/to/server.key;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 5m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    location / {
        root html;
        index index.html;
    }
}
```
