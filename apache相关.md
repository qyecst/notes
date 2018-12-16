<!--
{
    "title": "apache相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-16 11:31:07",
    "tag": [
        "apache",
        "httpd"
    ],
    "info": [
        "待测//todo"
    ]
}
-->

## apache安装

### 源码编译安装

```bash
# 安装依赖 apr apr-util pcre
yum install apr apr-devel apr-util apr-util-devel pcre pcre-devel
./configure --prefix=path/to/apache --enable-rewrite --enable-so
make && make install

# 依赖 apr apr-util pcre
./configure --prefix=path/to/apr
make && make install
./configure --prefix=path/to/apr-util --with-apr=path/to/apr
make && make install
./configure --prefix=path/to/pcre
make && make install
# apache # http://httpd.apache.org/download.cgi
./configure --prefix=path/to/apache --sysconfdir=/etc --enable-so --enable-rewrite --with-apr=path/to/par --with-apr-util=path/to/apr-util --with-pcre=path/to/pcre
make && make install
```

### 软件源安装

```bash
yum install httpd
apt install apache2
```

### httpd/apache升级

```bash
# 确认升级兼容性
# 在已安装的构建目录中找到文件 config.nice 该文件包含确切的 configure 命令配置
# 复制到新的构建目录中，编辑所需的更改
./config.nice
make && make install
httpd -k graceful-stop # stop old version
httpd -k start # start new version
#apachectl -k graceful-stop
#apachectl -k start
# Passing arguments to httpd using apachectl is no longer supported

# 可以添加新的参数给 config.nice ，会添加到原有的configure配置选项
./config.nice --prefix=/path/to/apache --with-port=90 # 添加新的参数 --with-port=90
```

## 常用操作/命令

```bash
# 检测配置文件语法
httpd -t
# 启动/重启/优雅重启/停止/优雅停止
httpd -k start|restart|graceful|stop|graceful-stop
```

## apache配置

```httpd
# vim /etc/httpd/conf/httpd.conf

ServerRoot "/etc/httpd"
Listen 80
User apache
Group apache
ServerName www.example.com:80
<Directory />
    AllowOverride none
    Require all denied
</Directory>
DocumentRoot "/var/www/html"
<Directory "/var/www">
    AllowOverride None
    # Allow open access:
    Require all granted # Require [all [grant|denied] | not [ip|env] | host xxx.com | ip x.x.x.x]
</Directory>
# .etc
```

## 屏蔽特定UA请求

```httpd
<Directory xxx/www/yoursite>
    SetEnvIfNoCase User-Agent ".*(FeedDemon|JikeSpider|AskTbFXTV|CrawlDaddy|Feedly|Swiftbot|ZmEu|oBot).*" BADBOT
    SetEnvIfNoCase User-Agent "brandwatch" BADBOT
    SetEnvIfNoCase User-Agent "rogerbot" BADBOT
    <RequireAll>
        Require all granted
        Require not env BADBOT
        Require not ip 192.168.1.1
    </RequireAll>
</Directory>
```

## 虚拟主机/域名配置

```httpd
# vim /etc/httpd/conf.d/httpd-vhosts.conf

# NameVirtualHost *:80 # NameVirtualHost 已弃用
<VirtualHost *:80>
    <Directory "/var/www/html">
        Allow from all
        AllowOverride all
        Options -Indexes -FollowSymLinks
    </Directory>
    ServerAdmin root@localhost
    DocumentRoot "/var/www/html"
    Servername www.test.com
    ErrorLog "logs/www.test.com.err.log"
    CustomLog "logs/www.test.com.cut.log" common
</VirtualHost>

# yum install mod_ssl
# vim conf.modules.d/00-ssl.conf => LoadModule ssl_module modules/mod_ssl.so
# vim conf.d/ssl.conf
<VirtualHost *:443>
    Servername www.test.com
    ErrorLog logs/ssl_error_log
    TransferLog logs/ssl_access_log
    LogLevel warn
    SSLEngine on
    SSLProtocol all -SSLv2 -SSLv3
    SSLCipherSuite HIGH:3DES:!aNULL:!MD5:!SEED:!IDEA
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    <Files ~ "\.(cgi|shtml|phtml|php3?)$">
        SSLOptions +StdEnvVars
    </Files>
    <Directory "/var/www/cgi-bin">
        SSLOptions +StdEnvVars
    </Directory>
    BrowserMatch "MSIE [2-5]" nokeepalive ssl-unclean-shutdown downgrade-1.0 force-response-1.0
    CustomLog logs/ssl_request_log "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
</VirtualHost>
```

## url rewrite规则/url重写规则

```httpd
# vim /etc/httpd/conf/httpd.conf
> RewriteEngine on # 全局/vhosts局部

# conf.modules.d/00-base.conf
> LoadModule rewrite_module modules/mod_rewrite.so

## httpd.conf / vhosts.conf
RewriteEngine on
RewriteCond %{HTTP_HOST} www.test.com [NC,OR] # 访问路径为www.test.com NC不区分大小写 OR'或'规则
RewriteCond %{HTTP_HOST} ^test.com [NC] # 访问路径以test.com开头
RewriteRule ^/(.*)$ http://www.test1.com/$1 [L] # L最后一天规则，停止匹配后续规则 重写规则，将上述访问跳转为www.test1.com访问

RewriteEngine on
RewriteRule ^/$ http://www.test.com/index/ [L,R=301] # 访问跳转为www.test.com/index访问，R使用301永久重定向

# 重写规则，手机访问移动端网页
RewriteEngine on
RewriteCond %{HTTP_USER_AGENT} ^Android [NC,OR]
RewriteCond %{HTTP_USER_AGENT} ^iPhone [NC,OR]
RewriteCond %{HTTP_USER_AGENT} ^WAP [NC]
RewriteRule ^/$ http://m.test.com [L,R=301]
RewriteRule ^/(.*)$ http://m.test.com/$1 [L,R=301]
```
