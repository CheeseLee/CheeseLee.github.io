# DevStack搭建环境

------
## 安装前准备
1. 操作系统检查
> * CentOS、RHEL、Ubuntu；
> * 需要连接Pip源，下载python依赖库;

2. 检查是否能通互联网
> * 需要与GitHub（或内部GitLab）连通，下载openstack模块源代码，如Nova，KeyStone等；
> * 需要连接pip源，下载python依赖库;

3. 检查Python、Python的pip工具、git是否已经安装
> python --version
> pip --version
> git --version

4. 设置宿主机网卡混杂模式（devstack安装在用vmwar虚拟化出来的虚拟机上才需要设置）
> ifconfig eth0 promisc

5. 用root用户创建stack用户（devstack安装不能用root用户）：
> adduser -m stack
> passwd stack

## 安装
1. 从github上克隆devstack的源码,获取L版分支
> su - stack
> git clone  https://github.com/openstack-dev/devstack.git
> git checkout  stable/liberty

2. 给stack用户赋予权限
> su - root
> bash /home/stack/devstack/tools/create-stack-user.sh

3. 设置pip源
> 客户端配置pip源，新建文件 ~/.pip/pip.conf 内容如下：
    ```conf
    [global]
	index-url = http://pypi.douban.com/simple/
	trusted-host = pypi.douban.com
	disable-pip-version-check = true
    ```	
> 如果pip源中缺少devstack需要安装的符合版本要求的库文件，需要先补充pip源
> 大部分库文件都在操作系统安装时，自带的python库中安装好，有些特殊的库文件及版本要求，参考/home/stack/requirements/upper-constraints.txt文件

4. 配置安装文件
> 在/home/stack/devstack目录下，创建local.conf文件，增加以下配置。
  该配置为模板，环境迁移需要修改，具体修改哪些，请参照
  http://docs.openstack.org/developer/devstack/configuration.html#local-conf
    ```shell
	[[local|localrc]]
	DEST=/home/stack
	HOST_IP=172.17.140.140
	SERVICE_HOST=172.17.140.141
	MYSQL_HOST=172.17.140.141
	RABBIT_HOST=172.17.140.141
	GLANCE_HOSTPORT=172.17.140.141:9292
	GIT_BASE=https://github.com
	
	# Logging
	LOGDAYS=1
	LOG_COLOR=False
	LOGDIR=$DEST/logs
	LOGFILE=$DEST/logs/stack.sh.log
	VERBOSE=True
	SCREEN_LOGDIR=$DEST/logs/screen
	
	# Credentials
	ADMIN_PASSWORD=devstack
	DATABASE_PASSWORD=$ADMIN_PASSWORD
	RABBIT_PASSWORD=$ADMIN_PASSWORD
	SERVICE_PASSWORD=$ADMIN_PASSWORD
	SERVICE_TOKEN=$ADMIN_PASSWORD
		
	#Disable and enable service
	DISABLED_SERVICES=n-net
	   
	# Enable Neutron - Service
	ENABLED_SERVICES+=,neutron,q-svc,q-agt,q-dhcp,q-l3,q-meta,q-metering,q-vpn,q-lbaas
	# Neutron options
	Q_USE_SECGROUP=True
	FLOATING_RANGE="172.17.140.0/24"
	FIXED_RANGE="10.0.0.0/24"
	Q_FLOATING_ALLOCATION_POOL=start=172.17.140.198,end=172.17.140.202
	PUBLIC_NETWORK_GATEWAY="172.17.140.1"
	Q_L3_ENABLED=True
	PUBLIC_INTERFACE=eth0
	# Open vSwitch provider networking configuration
	Q_USE_PROVIDERNET_FOR_PUBLIC=True
	OVS_PHYSICAL_BRIDGE=br-ex
	PUBLIC_BRIDGE=br-ex
	OVS_BRIDGE_MAPPINGS=public:br-ex
	# Enable Heat Service
	ENABLED_SERVICES+=,heat,h-api,h-api-cfn,h-api-cw,h-eng
	# Enable Swift services
	ENABLED_SERVICES+=,swift,s-proxy,s-object,s-container,s-account
	SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5
	SWIFT_REPLICAS=1
	# Enable Ceilometer Service (metering + alarming)
	ENABLED_SERVICES+=,ceilometer-acompute,ceilometer-acentral,ceilometer-collector,ceilometer-api
	ENABLED_SERVICES+=,ceilometer-alarm-notify,ceilometer-alarm-eval
    
    ```
	
5. 运行安装脚本(用stack用户）
> bash /home/stack/devstack/stack.sh
> 出现以下提示，表示安装成功
>>This is your host IP address: 172.17.140.140
This is your host IPv6 address: ::1
Horizon is now available at http://172.17.140.140/dashboard
Keystone is serving at http://172.17.140.140:5000/
The default users are: admin and demo
The password: devstack
