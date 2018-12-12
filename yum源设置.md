<!--
{
    "title": "yum源设置",
    "create": "2018-12-12 17:06:32",
    "modify": "2018-12-12 17:06:32",
    "tag": [
        "yum",
        "centos"
    ],
    "info": [
        "待测//todo"
    ]
}
-->

## 配置文件

默认路径`/etc/yum.repos.d/`

```bash
mv /etc/yum.repos.d/CentOS-Base.repo CentOS-Base.repo.bak

wget http://mirrors.aliyun.com/repo/Centos-7.repo
mv Centos-7.repo /etc/yum.repos.d/CentOS-Base.repo

wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
mv CentOS7-Base-163.repo /etc/yum.repos.d/CentOS-Base.repo
```

## iso的yum源

```bash
# /etc/yum.repos.d/Media.repo
[yum]
name = CentOS7-Media # 名称
baseurl = file:///mnt # 路径
enable = 1 # 启用该源
gpgcheck = 1 # 是否检查gpgkey
gpgkey = file:///mnt/RPM-GPG-KEY-CentOS-7 # gpgkey文件
```

## http的yum源

```bash
yum install createrepo*
yum install httpd

mkdir /var/www/html/centos/
cp -r /mnt/Packaages/* /var/www/html/centos/
createrepo centos/
#createrepo --update centos/
# 使用httpd发布 systemctl start httpd

# /etc/yum.repos.d/Http.repo
[base]
name = CentOS7-Http # 名称
baseurl = http://192.168.1.1/centos/ # 路径
enable = 1 # 启用该源
gpgcheck = 1 # 是否检查gpgkey
gpgkey = http://192.168.1.1/centos/RPM-GPG-KEY-CentOS-7 # gpgkey文件
[updates]
name = CentOS7-Http # 名称
baseurl = http://192.168.1.1/centos/ # 路径
enable = 1 # 启用该源
gpgcheck = 1 # 是否检查gpgkey
gpgkey = http://192.168.1.1/centos/RPM-GPG-KEY-CentOS-7 # gpgkey文件
```

## 同步外部yum源

```bash
yum install yum-utils createrepo
yum repolist

# reposync同步 -r指定repolist_id 默认同步所有源 -p指定下载路径
reposync -r base -p /var/www/html/centos/
#reposync -p /var/www/html/centos
#createrepo --update centos/
createrepo /var/www/html/centos
```
