# 使用DevStack安装OpenStack

注意：

- 0.【可选】尽量不要使用带有桌面的操作系统，NetworkManager可能会在配置网络时有影响

- 1.【可选】不要使用校园网有线网（宿舍），可以尝试使用手机连mobile，再开热点给电脑用

- 2.切换到stack用户后，设置proxy

## 一、单机安装

### 1.设置代理
```shell
curl ip.gs
export all_proxy=http://x.x.x.x:xxxx
export http_proxy=http://x.x.x.x:xxxx
export https_proxy=http://x.x.x.x:xxxx
# 检查proxy设置是否正确
curl ip.gs
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

### 3.配置目录权限

> 如果出现目录没有权限再执行如下操作

```shell
# 权限
sudo chown -R stack:stack ~/.cache
sudo chown -R stack:stack ~/data
```

## 二、集群安装
