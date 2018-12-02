<!--
{
    "title": "apache相关",
    "create": "2018-05-16 15:02:26",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "apache"
    ],
    "info": [
        "待测//todo"
    ]
}
-->

## apache安装

源码编译：

- 依赖：`apr apr-util pcre`
- 安装：
    - apr：`./configure --prefix=path/to/apr`
    - apr-util：`./configure --prefix=path/to/apr-util --with-apr=path/to/apr`
    - pcre：`./configure --prefix=path/to/pcre`
- 安装：`./configure --prefix=path/to/apache --sysconfdir=/etc --enable-so --enable-rewrite --with-apr=path/to/par --with-apr-util=path/to/apr-util --with-pcre=path/to/pcre`

## apache配置

`/etc/httpd.conf`