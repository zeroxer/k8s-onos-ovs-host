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

### 1.设置代理
```shell
curl ip.gs
export all_proxy=http://x.x.x.x:xxxx
export http_proxy=http://x.x.x.x:xxxx
export https_proxy=http://x.x.x.x:xxxx
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
```shell
# 配置pip国内源
mkdir ~/.pip
vim ~/.pip/pip.conf
# 添加如下内容
[global]
index-url = https://mirrors.ustc.edu.cn/pypi/web/simple
format = columns
```

#### 2.3更新系统的pip版本
```shell
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

## 二、集群安装
