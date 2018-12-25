<!--
{
    "title": "openstack相关",
    "create": "2018-12-25 15:53:49",
    "modify": "2018-12-25 15:53:49",
    "tag": [
        "openstack",
        "kvm"
    ],
    "info": []
}
-->

## 信息

### 组件

计算compute， Nova，一套控制器，用于为单个用户/使用群组管理虚拟机实例的整个生命周期，依据用户需求提供虚拟服务。负责虚拟机创建/开关机/挂起暂停/调整/迁移/重启/销毁等，配置cpu/内存信息等

对象存储object storage，Swift，一套用于在大规模可扩展系统中通过内置冗余及高容错机制实现对象存储的系统，允许进行存储或检索文件，可为Glance提供镜像存储，为Cinder提供卷备份服务

镜像服务image service，Glance，一套虚拟机镜像查找及检索系统，支持多种虚拟机镜像格式AKI/AMI/ARI/ISO/QCOW2/Raw/VDI/VHD/VMDK等，有创建上传删除编辑镜像的功能

身份服务identify service，Keystone，为openstack其他服务提供身份验证，服务规则和服务令牌等功能，管理domains/projects/users/groups/roles

网络&地址管理network，Neutron，提供云计算的网络虚拟化技术，为open stack的其他服务提供网络连接服务，为用户提供接口定义network/subnet/router/dns/dhcp/负载均衡/l3服务/vlan等。插件架构支持多种主流网络厂家/技术

块存储block storage，Cinder，为运行实例提供稳定的数据块存储服务，插件驱动架构利于块设v欸的创建/管理，创建删除卷/挂载卸载卷等

ui界面dashboard，Horizon，openstack各服务的web管理门户，用户简化用户对服务的操作，例如启动/分配ip/配置访问控制等

测量metering，Ceilometer，收集openstack内的事件，为计费/监控以及其他服务提供数据支撑

部署编排orchestration，Heat，提供了一种通过模板定义的协同部署方式，实现云基础设施软件运行环境(计算/存储/网络资源等 的自动化部署

数据库服务database service，Trove，为用户在openstack的环境提供可扩展和可靠的关系/非关系数据库引擎服务

## 安装

### 环境

https://docs.openstack.org/install-guide/openstack-services.html

两台服务器，node1为控制节点，node2为计算节点。控制节点主要用于操控计算节点，主要配置服务包括mysql/rabbitmq/apache/horizon/keystone/glance/nova(api,cert,schedule,consoleauth,conductor,novncproxy)/neutron(server/linuxbridge-agent)/cinder(api,schedule,volume)/可选GFS分布式存储等；计算节点为创建虚拟机的资源池，主要配置服务包括nova(nova-compute,libvirt,kvm)/neutron(linuxbridge-agent)等

配置dns/hosts节点互通，防火墙，时间同步

### 服务安装

https://docs.openstack.org/install-guide/index.html

```bash
## node1 控制节点 安装Rocky版

# subscription-manager repos --enable=rhel-7-server-optional-rpms --enable=rhel-7-server-extras-rpms --enable=rhel-7-server-rh-common-rpms # 只有RHEL需要

yum install epel-release # epel源
yum install centos-release-openstack-rocky # rocky版的openstack源
# yum install https://rdoproject.org/repos/rdo-release.rpm #  RDO源

yum install python-openstackclient # OpenStack client
yum install openstack-selinux # 自动管理openstack服务的selinux

yum install mariadb mariadb-server python2-PyMySQL mariadb-devel MySQL-python # 安装数据库和python数据库模块支持

yum install rabbitmq-server etcd memcached python-memcached # 安装mq消息队列服务/etcd/memcache

yum install openstack-keystone httpd httpd-devel mod_wsgi # 安装keystone/http服务

yum install openstack-glance python-glance python-glanceclient # 安装glance服务

yum install openstack-nova-api openstack-nova-cert openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api python-novaclient # 安装nova服务

yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables ipset python-neutronclient # 安装neutron服务

yum install openstack-dashboard openstack-cinder python-cinderclient # 安装dashboard cinder

yum install qemu-kvm qemu-img # 安装qemu服务

# 配置数据库
[mysqld]
max_connections=2000

# 创建数据库/用户
> create database keystone|glance|nova|neutron|cinder;
> grant all privileges on [above_db].* to '[above_user]'@'localhost|%' identified by '[above_pwd]';
> flush privileges;

# 配置mq
rabbitmqctl add_user openstack openstack
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmq-plugins list
rabbitmq-plugins enable rabbitmq_management # 监听15672

# 配置keystone
ID=`openssl rand -hex 10`;echo $ID # 基于openssl生成随机数并设置为admin_token
vim /etc/keystone/keystone.conf 
[default]
admin_token=$ID
[database]
connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
[token]
provider = fernet # fernet|uuid
driver=memcache

# 填充keystone数据库
su -s /bin/sh -c "keystone-manage db_sync" keystone
# Initialize Fernet key repositories
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
# Bootstrap the Identity service
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```
