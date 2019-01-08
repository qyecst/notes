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

<https://docs.openstack.org/install-guide/openstack-services.html>

两台服务器，node1为控制节点，node2为计算节点。控制节点主要用于操控计算节点，主要配置服务包括mysql/rabbitmq/apache/horizon/keystone/glance/nova(api,cert,schedule,consoleauth,conductor,novncproxy)/neutron(server/linuxbridge-agent)/cinder(api,schedule,volume)/可选GFS分布式存储等；计算节点为创建虚拟机的资源池，主要配置服务包括nova(nova-compute,libvirt,kvm)/neutron(linuxbridge-agent)等

配置dns/hosts节点互通，防火墙，时间同步

### 服务安装

<https://docs.openstack.org/install-guide/index.html> # 文档

```bash
## node1 控制节点

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
keystone-manage bootstrap --bootstrap-password <ADMIN_PASS> --bootstrap-admin-url http://controller:5000/v3/ --bootstrap-internal-url http://controller:5000/v3/ --bootstrap-public-url http://controller:5000/v3/ --bootstrap-region-id RegionOne
# vim /etc/httpd/conf/httpd.conf
ServerName controller
# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
# 启动memcache和httpd，修改memcache监听0.0.0.0
# 修改wsgi-keystone.conf
Listen 5000
Listen 35357

# 创建keystone认证用户
# 临时设置admin_token用户环境变量，用于创建用户
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
# 创建admin项目
openstack project create --domain default --description "admin project" admin
# 创建admin用户pwd: admin
openstack user create --domain default --password-prompt admin
# 创建admin角色
openstack role create admin
# 将admin用户加入admin项目，赋予admin角色
openstack role add --project admin --user admin admin
# 创建demo项目，demo用户，demo角色，将demo用户加入demo项目赋予demo角色
openstack project create --domain default --description "demo project" demo
openstack user create --domain default --password=demo demo
openstack role create user
openstack role add --project demo --user demo demo
# 创建service项目，用于管理其他服务
openstack project create --domain default --description "service project" service
# 检查openstack用户/项目
openstack user list
openstack project list

# 注册keystone服务，注册为公共的/内部的/管理的，创建endpoint服务访问入口，每个openstack服务都有指定的端口和专属url称为访问入口
openstack service create --name keystone --description "openstack identity" identity
openstack endpoint create --region RegionOne identity public http://1.1.1.1:5000/v2.0
openstack endpoint create --region RegionOne identity internal http://1.1.1.1:5000/v2.0
openstack endpoint create --region RegionOne identity admin http://1.1.1.1:35357/v2.0
# 查看访问入口
openstack endpoint list
# 验证默认项目/认证用户/获取token，获取到项目id/用户id等表示成功
unset OS_TOKEN
unset OS_URL
openstack --os-auth-url http://1.1.1.1:35357/v3 --os-project-domain-id default --os-user-domain-id default --os-project-name admin --os-username admin --os-auth-type password token issue

# 引用shell脚本，验证默认项目/认证用户，获取token
> export OS_USERNAME=admin
> export OS_PASSWORD=ADMIN_PASS
> export OS_PROJECT_NAME=admin
> export OS_USER_DOMAIN_NAME=Default
> export OS_PROJECT_DOMAIN_NAME=Default
> export OS_AUTH_URL=http://controller:5000/v3
> export OS_IDENTITY_API_VERSION=3
source admin_env.sh
openstack token issue

# 配置glance
vim /etc/glance/glance-api.conf
[default]
notification_driver=noop # glance不需要mq
[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
[glance_store]
default_store=file
filesystem_store_datadir=/var/lib/glance/images/ # 镜像存储位置
[keystone_authtoken]
www_authenticate_uri  = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = GLANCE_PASS
[paste_deploy]
flavor = keystone

vim /etc/glance/glance-registry.conf
[database]
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = GLANCE_PASS
[paste_deploy]
flavor = keystone
# 填充glance数据库
su -s /bin/sh -c "glance-manage db_sync" glance
# 创建glance的keystone用户/注册用户信息
source admin_env.sh
openstack user create --domain default --password=glance glance
openstack role add --project service --user glance admin
# 启动
systemctl start openstack-glance-api|openstack-glance-registry

# 注册keystone服务，注册为公共的/内部的/管理的，创建endpoint服务访问入口
source admin_env.sh
openstack service create --name glance --description "image service" image
openstack endpoint create --region RegionOne image public http://1.1.1.1:9292
openstack endpoint create --region RegionOne image internal http://1.1.1.1:9292
openstack endpoint create --region RegionOne image admin http://1.1.1.1:9292

# 脚本追加
> OS_IMAGE_API_VERSION=2
source admin_env.sh
glance image-list # 查看镜像列表

# 上传镜像到glance，可见性public
wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
openstack image create "cirros" --file cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --public
openstack image list
# glance image-create --name "cirros" --file cirros.xxx.img --disk-format qcow2 --container-format bare --visibility public --progress
# glance image-list

# 配置nova
vim /etc/nova/nova.conf
[DEFAULT]
my_ip = 1.1.1.1 # use the management interface IP address of the controller node
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:RABBIT_PASS@controller
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api_database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
[database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
[placement_database]
connection = mysql+pymysql://placement:PLACEMENT_DBPASS@controller/placement
[api]
auth_strategy = keystone
[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = NOVA_PASS
[vnc]
enabled = true
# novncproxy_base_url=http://1.1.1.1:6080/vnc_auto.html
server_listen = $my_ip
server_proxyclient_address = $my_ip
[glance]
api_servers = http://controller:9292
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = PLACEMENT_PASS

# vim /etc/httpd/conf.d/00-nova-placement-api.conf
<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>

# 同步nova数据库
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
# 创建用户/服务
openstack user create --domain default --password=nova nova
openstack role add --project service --user nova admin
# 启动服务
systemctl start openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
# 注册服务
source admin_env.sh
openstack service create --name nova --description "compute service" compute
openstack endpoint create --region RegionOne compute public http://1.1.1.1:8774/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute internal http://1.1.1.1:8774/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute admin http://1.1.1.1:8774/v2/%\(tenant_id\)s
# 查看
openstack host list

# neutron配置
# vim /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = controller
metadata_proxy_shared_secret = METADATA_SECRET
# vim /etc/nova/nova.conf
[neutron]
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS
service_metadata_proxy = true
metadata_proxy_shared_secret = METADATA_SECRET
# vim /etc/neutron/plugins/ml2/ml2_conf.ini|linuxbridge_agent.ini|dhcp_agent.ini|metadata_agent.ini
# 创建连接/用户/更新数据库/注册服务
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
openstack user create --domain default --password=neutron neutron
openstack role add --project service --user neutron admin
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
source admin_env.sh
openstack service create --name neutron --description "networking" network
openstack endpoint create --region RegionOne network public http://1.1.1.1:9696
openstack endpoint create --region RegionOne network internal http://1.1.1.1:9696
openstack endpoint create --region RegionOne network admin http://1.1.1.1:9696
# 重启openstack-nova-api
systemctl restart openstack-nova-api
# 启动neutron
systemctl start neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
systemctl start neutron-l3-agent.service
# 查看
openstack endpoint list
neutron agent-list
# 控制节点neutron创建网桥
source admin_env.sh
neutron net-create flat --shared --provider:physical_network physnet1 --provider:network_type flat
neutron subnet-create flat 192.168.1.0/24 --name flat-subnet --allocation-pool start=192.168.1.140,end=192.168.1.200 --dns-nameserver 192.168.1.1 --gateway 192.168.1.1
neutron net-list
neutron subnet-list
# 创建虚拟机，控制节点免密登录
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
nova keypair-add --pub-key /root/.ssh/id_rsa.pub mykey
nova keypair-list
# 安全组
nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
nova secgroup-add-rule default tcp 22 22 0.0.0.0/0
nova secgroup-add-rule default tcp 80 80 0.0.0.0/0
# 查看
nova flavor-list
nova image-list
neutron net-list
# 创建虚拟机
nova boot --flavor m1.tiny --image cirros --nic net-id=6277d20f-d033******* --security-group default --key-name mykey hello-myinstance

# 配置dashboard/horizon
yum install openstack-dashboard
vim /etc/openstack-dashboard/local_settings
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['one.example.com', 'two.example.com']
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
OPENSTACK_NEUTRON_NETWORK = {
    ...
    'enable_router': False,
    'enable_quotas': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': False,
    'enable_firewall': False,
    'enable_vpn': False,
    'enable_fip_topology_check': False,
}
TIME_ZONE = "TIME_ZONE"
# vim /etc/httpd/conf.d/openstack-dashboard.conf
WSGIApplicationGroup %{GLOBAL}
# systemctl restart httpd memcached
# GUI配置
http://1.1.1.1/dashboard/

## node2计算节点

yum install epel-release # epel源
yum install centos-release-openstack-rocky # rocky版的openstack源
# yum install https://rdoproject.org/repos/rdo-release.rpm #  RDO源

yum install python-openstackclient # OpenStack client
yum install openstack-selinux # 自动管理openstack服务的selinux

# 安装软件包
yum install openstack-nova-compute sysfsutils
yum install openstack-neutron openstack-neutron-liunxbridge ebtables ipset
yum install openstack-cinder python-cinderclient targetcli python-oslo-policy
yum install
qemu-kvm qemu-img

# 配置 nova
vim /etc/nova/nova.conf
[default]
my_ip=2.2.2.2
# ...
[libvirt]
virt_type=qemu
#virt_type=kvm
# ...

# 启动
systemctl restart libvirtd openstack-nova-compute
# 查看
openstack host list
glance image-list
nova image-list

# 配置 neutron
vim /etc/neutron/neutron.conf
[DEFAULT]
transport_url = rabbit://openstack:RABBIT_PASS@controller
auth_strategy = keystone
[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = NEUTRON_PASS
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp

# vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini|ml2_conf.ini

# vim /etc/nova/nova.conf
[neutron]
url = http://controller:9696
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = NEUTRON_PASS

ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
systemctl restart openstack-nova-compute
systemctl neutron-linuxbridge-agent
```
