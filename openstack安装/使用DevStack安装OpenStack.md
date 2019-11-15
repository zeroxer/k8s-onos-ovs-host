# 使用DevStack安装OpenStack

注意：

- 0.【可选】尽量不要使用带有桌面的操作系统，NetworkManager可能会在配置网络时有影响

- 1.【可选】不要使用校园网有线网（宿舍），可以尝试使用手机连mobile，再开热点给电脑用

- 2.切换到stack用户后，设置proxy

## 一、单机安装

### 0.下载并配置Devstack

#### 0.1.添加用户
```shell
sudo useradd -s /bin/bash -d /opt/stack -m stack
sudo su -
echo "stack ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
sudo su stack
cd ~
```

#### 0.2.下载Devstack
```shell
sudo apt-get install git -y
git clone https://opendev.org/openstack/devstack
cd devstack
git checkout stable/train
```

#### 0.3.配置Devstack

##### 0.3.1.[TODO]更换Devstack源

1.把`devstack/stackrc`文件中的`GIT_BASE=${GIT_BASE:-https://opendev.org}`替换成`GIT_BASE=${GIT_BASE:-https://gitee.com}`

- 注意：如果使用的这一步，则之后就不要再设置`GIT_BASE`

2.把`devstack/stackrc`文件中的`${GIT_BASE}/openstack`替换成`${GIT_BASE}/zerocjx`
    
- 注意：此时应该保证`GIT_BASE`为`https://gitee.com`

```shell
cp samples/local.conf .
vim local.conf
# 添加/修改如下内容
[[local|localrc]]
HOST_IP=x.x.x.x
FIXED_RANGE=10.11.12.0/24

Q_USE_SECGROUP=True
FLOATING_RANGE=10.108.144.0/22
PUBLIC_NETWORK_GATEWAY=10.108.144.1
PUBLIC_INTERFACE=ens160

E_PROVIDERNET_FOR_PUBLIC=True
OVS_PHYSICAL_BRIDGE=br-ex
PUBLIC_BRIDGE=br-ex
OVS_BRIDGE_MAPPINGS=public:br-ex

ADMIN_PASSWORD=pass
DATABASE_PASSWORD=pass
RABBIT_PASSWORD=pass
SERVICE_PASSWORD=pass


# GIT_BASE=https://github.com
# GIT_BASE=https://giee.com
```

- 3.不要修改`stackrc`文件中的github.com

```shell
# 【错误】
# sed -i 's/https:\/\/github.com/http:\/\/gitea.xmagicer.com/g' stackrc
```

参考配置文件
cat << EOF > ~/devstack/local.conf 
```conf
# Sample ``local.conf`` for user-configurable variables in ``stack.sh``

# NOTE: Copy this file to the root DevStack directory for it to work properly.

# ``local.conf`` is a user-maintained settings file that is sourced from ``stackrc``.
# This gives it the ability to override any variables set in ``stackrc``.
# Also, most of the settings in ``stack.sh`` are written to only be set if no
# value has already been set; this lets ``local.conf`` effectively override the
# default values.

# This is a collection of some of the settings we have found to be useful
# in our DevStack development environments. Additional settings are described
# in https://docs.openstack.org/devstack/latest/configuration.html#local-conf
# These should be considered as samples and are unsupported DevStack code.

# The ``localrc`` section replaces the old ``localrc`` configuration file.
# Note that if ``localrc`` is present it will be used in favor of this section.
[[local|localrc]]
HOST_IP=10.108.147.182

Q_USE_SECGROUP=True
FLOATING_RANGE=10.108.144.0/22
PUBLIC_NETWORK_GATEWAY=10.108.144.1
PUBLIC_INTERFACE=ens160

E_PROVIDERNET_FOR_PUBLIC=True
OVS_PHYSICAL_BRIDGE=br-ex
PUBLIC_BRIDGE=br-ex
OVS_BRIDGE_MAPPINGS=public:br-ex

GIT_BASE=http://10.108.145.62

TARGET_BRANCH=stable/train

enable_plugin zun http://10.108.145.62/openstack/zun $TARGET_BRANCH
enable_plugin zun-tempest-plugin http://10.108.145.62/openstack/zun-tempest-plugin
enable_plugin devstack-plugin-container http://10.108.145.62/openstack/devstack-plugin-container $TARGET_BRANCH

KURYR_CAPABILITY_SCOPE=global
KURYR_PROCESS_EXTERNAL_CONNECTIVITY=False
enable_plugin kuryr-libnetwork http://10.108.145.62/openstack/kuryr-libnetwork $TARGET_BRANCH

LIBS_FROM_GIT="python-zunclient"

enable_plugin zun-ui http://10.108.145.62/openstack/zun-ui $TARGET_BRANCH
enable_plugin heat http://10.108.145.62/openstack/heat $TARGET_BRANCH


# Minimal Contents
# ----------------

# While ``stack.sh`` is happy to run without ``localrc``, devlife is better when
# there are a few minimal variables set:

# If the ``*_PASSWORD`` variables are not set here you will be prompted to enter
# values for them by ``stack.sh``and they will be added to ``local.conf``.
ADMIN_PASSWORD=pass
DATABASE_PASSWORD=pass
RABBIT_PASSWORD=pass
SERVICE_PASSWORD=pass

# ``HOST_IP`` and ``HOST_IPV6`` should be set manually for best results if
# the NIC configuration of the host is unusual, i.e. ``eth1`` has the default
# route but ``eth0`` is the public interface.  They are auto-detected in
# ``stack.sh`` but often is indeterminate on later runs due to the IP moving
# from an Ethernet interface to a bridge on the host. Setting it here also
# makes it available for ``openrc`` to include when setting ``OS_AUTH_URL``.
# Neither is set by default.
#HOST_IP=w.x.y.z
#HOST_IPV6=2001:db8::7


# Logging
# -------

# By default ``stack.sh`` output only goes to the terminal where it runs.  It can
# be configured to additionally log to a file by setting ``LOGFILE`` to the full
# path of the destination log file.  A timestamp will be appended to the given name.
LOGFILE=$DEST/logs/stack.sh.log

# Old log files are automatically removed after 7 days to keep things neat.  Change
# the number of days by setting ``LOGDAYS``.
LOGDAYS=2

# Nova logs will be colorized if ``SYSLOG`` is not set; turn this off by setting
# ``LOG_COLOR`` false.
#LOG_COLOR=False


# Using milestone-proposed branches
# ---------------------------------

# Uncomment these to grab the milestone-proposed branches from the
# repos:
#CINDER_BRANCH=milestone-proposed
#GLANCE_BRANCH=milestone-proposed
#HORIZON_BRANCH=milestone-proposed
#KEYSTONE_BRANCH=milestone-proposed
#KEYSTONECLIENT_BRANCH=milestone-proposed
#NOVA_BRANCH=milestone-proposed
#NOVACLIENT_BRANCH=milestone-proposed
#NEUTRON_BRANCH=milestone-proposed
#SWIFT_BRANCH=milestone-proposed

# Using git versions of clients
# -----------------------------
# By default clients are installed from pip.  See LIBS_FROM_GIT in
# stackrc for details on getting clients from specific branches or
# revisions.  e.g.
# LIBS_FROM_GIT="python-ironicclient"
# IRONICCLIENT_BRANCH=refs/changes/44/2.../1

# Swift
# -----

# Swift is now used as the back-end for the S3-like object store. Setting the
# hash value is required and you will be prompted for it if Swift is enabled
# so just set it to something already:
SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5

# For development purposes the default of 3 replicas is usually not required.
# Set this to 1 to save some resources:
SWIFT_REPLICAS=1

# The data for Swift is stored by default in (``$DEST/data/swift``),
# or (``$DATA_DIR/swift``) if ``DATA_DIR`` has been set, and can be
# moved by setting ``SWIFT_DATA_DIR``. The directory will be created
# if it does not exist.
SWIFT_DATA_DIR=$DEST/data

```



### 1.设置代理
```shell
curl ip.gs
# export all_proxy=http://x.x.x.x:xxxx # 不推荐使用
export http_proxy=http://x.x.x.x:xxxx
export https_proxy=http://x.x.x.x:xxxx
# 添加HOST_IP到no_proxy
export no_proxy='mirrors.ustc.edu.cn,mirrors.tuna.tsinghua.edu.cn,gitee.com,127.0.0.1,10.108.145.62,x.x.x.x'
# 检查proxy设置是否正确
curl ip.gs
```

#### 配置git代理

> 如果配置了上一步，则不要再单独配置git代理

```shell
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 2.配置pip

#### 2.1配置主用户的pip
```shell
# 配置pip国内源
mkdir ~/.pip
vim ~/.pip/pip.conf
# 添加如下内容
[global]
index-url = https://mirrors.aliyun.com/pypi/simple
[install]
trusted-host=mirrors.aliyun.com
```

#### 2.2配置stack的pip

- 方式一、修改配置文件

```shell
# 配置pip国内源
mkdir ~/.pip
vim ~/.pip/pip.conf
# 添加如下内容
[global]
index-url = https://mirrors.aliyun.com/pypi/simple
[install]
trusted-host=mirrors.aliyun.com
```

- 方式二、升级pip，使用命令行配置
```shell
# 升级 pip 到最新的版本 (>=10.0.0) 后进行配置：
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 2.3更新系统的pip版本
```shell
# pip升级可选
sudo pip install --upgrade pip
sudo pip install --upgrade setuptools
```

注意：不升级pip可能出现的问题
```shell
pip无法找到指定版本的包
```

#### 2.4创建python虚拟环境
```shell
cd devstack
virtualenv ../requirements/.venv/
```

### 3.配置目录权限

> 如果出现目录没有权限再执行如下操作

```shell
# 权限
sudo chown -R stack:stack ~/.cache
sudo chown -R stack:stack ~/data
```

### (根据实际情况)4.关闭Ubuntu的NetworkManager，配置DNS
```shell
sudo vim /etc/systemd/resolved.conf
[Resolve]
DNS=8.8.8.8
sudo systemctl restart systemd-resolved.service
```

## 二、集群安装


## Q&A

### 1.pip找不到指定版本的包

- 问题

```shell
openstack no matching distribvution found for wrapt==1.11.2
```
- 分析原因

    - 在ubuntu系统中直接使用`pip install wrapt==1.11.2`发现该包已经安装到python2.7中，说明包是可以找到的
    - 怀疑是网络问题导致的某些步骤不完整。因为在网络流畅的情况下，没有出现该问题。
        - 网络问题：校园网在晚上基本处于网络断开状态，连接github速度为20k，使用代理也是同样的速度。
        - 只能通过白天解决该问题
    - 搜索引擎没有找到对应的问题的解决方式，有一个类似的是通过升级到python3解决的类似问题
    - 经测试不是网速导致的。以上可能是ubuntu系统用户和stack的pip.conf配置不同，虽然ubuntu系统可以安装，但是stack用户的pip配置有问题，所以才无法安装

- 原因：PyPi源设置错误
```shell
# 设置stack用户的pip.conf
vim ~/.pip/pip.conf
# 统一使用aliyun
[global]
index-url = https://mirrors.aliyun.com/pypi/simple
[install]
trusted-host=mirrors.aliyun.com
```
也许可以统一使用`https://pypi.tuna.tsinghua.edu.cn/simple`

- 【完成】终于解决了一次

~~
- 尝试解决方法
重新安装devstack
```shell
./unstack.sh
./clean.sh
sudo rm -rf /opt/stack
sudo reboot
```

- 或者尝试：
```
如果遇到pip源的版本找不到，请自行根据报错提示修改/opt/stack/requirements/upper-constraints.txt的相应内容
```

- CentOS中解决方法

> CentOS使用清华的pypi源，虽然pip.conf中配置的使ustc的:)

1.新开一个终端，切换到stack用户，source tempest/.tox/** 切换到指定的虚拟环境

2.根据报错的信息，使用pip重新执行安装命令`pip install -c .... -r ....`

3.重新执行`stack.sh`

- 找到问题原因

使用手动到指定的位置使用pip安装依赖，例如`tempest`、`glance对应的系统的依赖`


执行`stack.sh`的过程中会修改`/opt/stack/requirements`仓库中的`uper-constraints.txt`。

使用如下命令撤销更改
```shell
cd /opt/stack/requiremts/
git checkout .
```
~~

