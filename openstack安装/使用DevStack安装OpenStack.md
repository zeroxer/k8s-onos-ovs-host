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
FLOATING_RANGE=x.x.x.x/27
FIXED_RANGE=10.11.12.0/24

ADMIN_PASSWORD=pass
DATABASE_PASSWORD=pass
RABBIT_PASSWORD=pass
SERVICE_PASSWORD=pass

HOST_IP=x.x.x.x
# GIT_BASE=https://github.com
# GIT_BASE=https://giee.com
```

参考配置文件
cat << EOF > ~/devstack/local.conf 
```conf
[[local|localrc]] 
HOST_IP=192.168.145.141
SERVICE_IP_VERSION=4
DATABASE_PASSWORD=password
RABBIT_PASSWORD=password
SERVICE_TOKEN=password
SERVICE_PASSWORD=password
ADMIN_PASSWORD=password
WSGI_MODE=mod_wsgi
NOVA_USE_MOD_WSGI=False
CINDER_USE_MOD_WSGI=False
TARGET_BRANCH=stable/rocky
DOWNLOAD_DEFAULT_IMAGES=False
NEUTRON_CREATE_INITIAL_NETWORKS=False
disable_service tempest
GIT_BASE=http://git.trystack.cn
NOVNC_REPO=http://git.trystack.cn/kanaka/noVNC.git
enable_plugin zun http://git.trystack.cn/openstack/zun stable/rocky
enable_plugin zun-tempest-plugin http://git.trystack.cn/openstack/zun-tempest-plugin
#This below plugin enables installation of container engine on Devstack. 
#The default container engine is Docker 
enable_plugin devstack-plugin-container http://git.trystack.cn/openstack/devstack-plugin-container stable/rocky
# In Kuryr, KURYR_CAPABILITY_SCOPE is ‘local’ by default, 
# but we must change it to ‘global’ in the multinode scenario. 
KURYR_CAPABILITY_SCOPE=local
KURYR_ETCD_PORT=2379
enable_plugin kuryr-libnetwork http://git.trystack.cn/openstack/kuryr-libnetwork stable/rocky
# install python-zunclient from git 
#LIBS_FROM_GIT="python-zunclient" 
# Optional: uncomment to enable the Zun UI plugin in Horizon 
enable_plugin zun-ui http://git.trystack.cn/openstack/zun-ui stable/rocky
```

### 1.设置代理
```shell
curl ip.gs
# export all_proxy=http://x.x.x.x:xxxx # 不推荐使用
export http_proxy=http://x.x.x.x:xxxx
export https_proxy=http://x.x.x.x:xxxx
# 添加HOST_IP到no_proxy
export no_proxy='mirrors.ustc.edu.cn,mirrors.tuna.tsinghua.edu.cn,127.0.0.1,x.x.x.x'
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
index-url = https://mirrors.ustc.edu.cn/pypi/web/simple
format = columns
trusted-host = mirrors.ustc.edu.cn
```

#### 2.2配置stack的pip

- 方式一、修改配置文件

```shell
# 配置pip国内源
mkdir ~/.pip
vim ~/.pip/pip.conf
# 添加如下内容
[global]
index-url = https://mirrors.ustc.edu.cn/pypi/web/simple
format = columns
trusted-host = mirrors.ustc.edu.cn
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

- 尝试解决方法
重新安装devstack
```shell
./unstack.sh
./clean.sh
sudo rm -rf /opt/stack
sudo reboot
```

- 找到问题原因

执行`stack.sh`的过程中会修改`/opt/stack/requirements`仓库中的`uper-constraints.txt`。

使用如下命令撤销更改
```shell
cd /opt/stack/requiremts/
git checkout .
```