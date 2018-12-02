<!--
{
    "title": "php相关",
    "create": "2018-12-02 22:54:55",
    "modify": "2018-12-02 19:54:55",
    "tag": [
        "php",
        "php-fpm"
    ],
    "info": []
}
-->

## 安装

```bash
# http://php.net/downloads.php

# 依赖
yum install libxml2 libxml2-devel openssl openssl-devel curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel pcre pcre-devel libxslt libxslt-devel bzip2 bzip2-devel

# 编译
./configure --prefix=/usr/local/php<version> --with-curl --with-freetype-dir --with-gd --with-gettext --with-iconv-dir --with-kerberos --with-libdir=lib64 --with-libxml-dir --with-mysqli --with-openssl --with-pcre-regex --with-pdo-mysql --with-pdo-sqlite --with-pear --with-png-dir --with-jpeg-dir --with-xmlrpc --with-xsl --with-zlib --with-bz2 --with-mhash --enable-fpm --enable-bcmath --enable-libxml --enable-inline-optimization --enable-gd-native-ttf --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-sysvshm --enable-xml --enable-zip
make -j <num_of_processes>
make install

# 配置/安装目录下
cp php.ini-development php.ini
cp php-fpm.conf.default php-fpm.conf
cp www.conf.default www.conf
```

## php&php-fpm配置

```conf
# /path/php-fpm.d/....
# vim www.conf

# 指定用户/组
user=php-fpm_user
group=php-fpm_user

# 监听方式
listen=127.0.0.1:9000
#listen=/var/run/php-fpm.sock
#listen.mode=0666
#listen.owner=php-fpm_user
#listen.group=php-fpm_user

# 使用方式 dynamic|static
pm=dynamic
# 静态时php-fpm进程数，动态下限定最大进程数
pm.max_children=6
# 动态起始进程数
pm.start_servers=2
# 动态闲时最小进程数
pm.min_spare_servers=1
# 动态闲时最大进程数
pm.max_spare_servers=3
# 多少次请求之后结束进程，重新生成新进程
pm.max_requests = 1000

# 内存限制
php_admin_value[memory_limit] = 128M
```
