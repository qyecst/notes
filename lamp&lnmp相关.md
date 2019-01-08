<!--
{
    "title": "lamp&lnmp相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "liunx",
        "nginx",
        "apache",
        "mysql",
        "php",
        "lamp",
        "lnmp"
    ],
    "info": []
}
-->

## LAMP&LNMP配置

### LAMP

apache：`./configure --prefix=path/to/apache --enable-so --enable-rewrite --enable-charset-lite --enable-cgi [.etc]`

php：

```bash
./configure --prefix=path/to/php --with-zlib --with-apxs2=path/to/apache/bin/apxs --with-mysql=path/to/mysql --with-config-file-path=/usr/local/php --enable-mbstring --enable-fpm [.etc]

## lamp分开 lap+mysql
./configure --prefix=path/to/php --with-zlib --with-apxs2=path/to/apache/bin/apxs \
--with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd \
--with-config-file-path=/usr/local/php --enable-mbstring --enable-fpm [.etc]
```

mysql：`/usr/local/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data` & `cp support-files/my-medium.cnf /etc/my.cnf`

`vim /usr/local/httpd/conf/httpd.conf`

```apache
LoadModule php5_module modules/libphp5.so

# :53 line
AddType application/x-httpd-php .pho

# :169 line
DirectoryIndex index.html index.php
```

### LNMP

mysql：`cmake -DCMAKE_INSTALL_PREFIX=path/to/mysql -DSYSCONFDIR=/etc -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all`

php：`cp /usr/src/php.../php.ini-production /usr/local/php/php.ini` & `cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf` & `/usr/local/php/sbin/php-fpm`

nginx：

```nginx
location ~ \.(php|php5)?$ {
    root <path>;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME /usr/local/nginx/html/$fastcgi_script_name;
    include fastcgi_params;
}
```

### 添加Redis缓存

```bash
# php添加redis驱动 php-redis
wget http://url/to/phpreids.tgz
./configure --with-php-config=/usr/local/php<ver>/bin/php-config --enable-redis
make && make install

# vim /usr/local/php<ver>/lib/php.ini
extension_dir = "/usr/local/php/lib/php/extensions/no-debug-zts-20090626"
extension = redis.so
```
