# Tungstem Fabric + k8s(Ubuntu18.04) 安装

> CNI: 容器网络插件

## 安装kubeadm工具

### 启动k8s

#### 1.安装k8s Master
```shell
sudo kubeadm init
# 稍等一会儿，例如18s
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 2.查看安装结果
```shell
sudo kubectl get nodes
# 此时发现master的状态是NotReady
```

## 一条命令安装TF

> 参考：https://tungstenfabric.github.io/website/Tungsten-Fabric-Ubuntu-one-line-install-on-k8s.html

### 【失败】尝试直接使用脚本
```shell
K8S_MASTER_IP=10.108.145.16; CONTRAIL_REPO="docker.io\/opencontrailnightly"; CONTRAIL_RELEASE="latest"; sudo mkdir -pm 777 /var/lib/contrail/kafka-logs; curl https://raw.githubusercontent.com/Juniper/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-ubuntu.yaml | sed "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" | sudo kubectl apply -f -
```


#### 原因

因为k8s版本升级导致api变动-v1.16.0的变动,导致DaemonSet从extensions/v1beta1移动到apps/v1中。

```shell
daemonsets, deployments, replicasets resources under extensions/v1beta1 - use apps/v1 instead
```


### 【尝试解决】下载contrail-single-step-cni-install-ubuntu.yaml文件，手动替换apiVersion

#### 首先：删除刚才失败的遗留内容
> 把刚才的`applf`改成`delete`

```shell
K8S_MASTER_IP=10.108.145.16; CONTRAIL_REPO="docker.io\/opencontrailnightly"; CONTRAIL_RELEASE="latest"; sudo mkdir -pm 777 /var/lib/contrail/kafka-logs; curl https://raw.githubusercontent.com/Juniper/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-ubuntu.yaml | sed "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" | sudo kubectl delete -f -
```

#### 然后，下载yaml文件，手动修改
```shell
wget https://raw.githubusercontent.com/Juniper/contrail-kubernetes-docs/master/install/kubernetes/templates/contrail-single-step-cni-install-ubuntu.yaml
```

##### 1.替换api版本信息 
把原来的
```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
```
替换成
```yaml
apiVersion: apps/v1
kind: DaemonSet
```

##### 2.添加selector选项
同时，在所有DaemonSet对应位置添加一个配置项`spec.selector.matchLabels:`,内容为对应DaemonSet的labels下的内容，例如如下配置中为`app: config-zookeeper`
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: config-zookeeper
  namespace: kube-system
  labels:
    app: config-zookeeper
spec:
  selector:
    matchLabels:
      app: config-zookeeper
  template:
    metadata:
      labels:
        app: config-zookeeper
```

## 手动重新安装

```shell
K8S_MASTER_IP=10.108.145.16; CONTRAIL_REPO="docker.io\/opencontrailnightly"; CONTRAIL_RELEASE="latest"; sudo mkdir -pm 777 /var/lib/contrail/kafka-logs; cat contrail-single-step-cni-install-ubuntu.yaml | sed "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" | sudo kubectl apply -f -
```

## 耐心等待-依赖于网络环境

> 请保证网络的带宽、以及从dockerhub中下载镜像的速度

后台会自动下载各种docker镜像，并自动配置相关内容

`重要`：等待下载完成所有contrail需要的镜像

## 使用kubectl删除刚才的任务

注意：这一步是`delete`
```shell
K8S_MASTER_IP=10.108.145.16; CONTRAIL_REPO="docker.io\/opencontrailnightly"; CONTRAIL_RELEASE="latest"; sudo mkdir -pm 777 /var/lib/contrail/kafka-logs; cat contrail-single-step-cni-install-ubuntu.yaml | sed "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" | sudo kubectl delete -f -
```

上一步中充当`pull images`命令，把我们需要的docker镜像下载下来。因为是第一次下载，所有有很多初始化内容可能会因为等待时间过长导致超时报错。

所以，删除我们在k8s创建的内容，再重新执行一次上一步手动安装使用的命令。

注意：这一步是`create`
```shell
K8S_MASTER_IP=10.108.145.16; CONTRAIL_REPO="docker.io\/opencontrailnightly"; CONTRAIL_RELEASE="latest"; sudo mkdir -pm 777 /var/lib/contrail/kafka-logs; cat contrail-single-step-cni-install-ubuntu.yaml | sed "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" | sudo kubectl create -f -
```

## 等待初始化完成

### 查看初始化进度

```shell
sudo kubectl get pods --all-namespaces
```
可能会看到如下输出
```shell

```
等待初始化完成，如下
```shell
```

## 打开TF的web UI

> 关闭我们之前可能设置的Ubuntu的手动代理，不然这里无法访问

网址为：https://10.108.145.16:8143

打开网址，忽略警告，继续访问

```shell
用户名：admin
密码：contrail123
```
