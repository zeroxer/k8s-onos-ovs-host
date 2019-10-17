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

注意：这一步是`apply`
```shell
K8S_MASTER_IP=10.108.145.16; CONTRAIL_REPO="docker.io\/opencontrailnightly"; CONTRAIL_RELEASE="latest"; sudo mkdir -pm 777 /var/lib/contrail/kafka-logs; cat contrail-single-step-cni-install-ubuntu.yaml | sed "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g; s/{{ CONTRAIL_REPO }}/$CONTRAIL_REPO/g; s/{{ CONTRAIL_RELEASE }}/$CONTRAIL_RELEASE/g" | sudo kubectl apply -f -
```

## 等待初始化完成

### 查看初始化进度

```shell
sudo kubectl get pods --all-namespaces
```
可能会看到如下输出
```shell
╭─fnl@k8smaster ~/k8s-onos-ovs-host/config_files ‹master› 
╰─$ sudo kubectl get pods --all-namespaces
NAMESPACE     NAME                                     READY   STATUS              RESTARTS   AGE
kube-system   config-zookeeper-pwstm                   0/1     ContainerCreating   0          17s
kube-system   contrail-agent-7bdm9                     0/2     Init:0/3            0          14s
kube-system   contrail-analytics-6clf9                 0/3     Init:0/1            0          17s
kube-system   contrail-analytics-alarm-n427w           0/3     Init:0/1            0          17s
kube-system   contrail-analytics-snmp-fcr95            0/3     Init:0/1            0          17s
kube-system   contrail-analyticsdb-967cd               0/3     Init:0/1            0          17s
kube-system   contrail-config-database-nodemgr-2mx89   0/1     Init:0/1            0          17s
kube-system   contrail-configdb-8g9nk                  0/1     ContainerCreating   0          17s
kube-system   contrail-controller-config-7psw4         0/5     Init:0/1            0          16s
kube-system   contrail-controller-control-84rl7        0/4     Init:0/1            0          17s
kube-system   contrail-controller-webui-2f5xd          0/2     Init:0/1            0          16s
kube-system   contrail-kube-manager-9q7bk              0/1     Init:0/1            0          14s
kube-system   coredns-5644d7b6d9-4666l                 0/1     ContainerCreating   0          23h
kube-system   coredns-5644d7b6d9-8mffv                 0/1     ContainerCreating   0          23h
kube-system   etcd-k8smaster                           1/1     Running             0          23h
kube-system   kube-apiserver-k8smaster                 1/1     Running             0          23h
kube-system   kube-controller-manager-k8smaster        1/1     Running             0          23h
kube-system   kube-proxy-qx4ms                         1/1     Running             0          23h
kube-system   kube-scheduler-k8smaster                 1/1     Running             0          23h
kube-system   rabbitmq-pwlfp                           0/1     ContainerCreating   0          15s
kube-system   redis-mqmh6                              0/1     ContainerCreating   0          15s
```
等待初始化完成，如下
```shell
╭─fnl@k8smaster ~/k8s-onos-ovs-host/config_files ‹master› 
╰─$ sudo kubectl get pods --all-namespaces
NAMESPACE     NAME                                     READY   STATUS              RESTARTS   AGE
kube-system   config-zookeeper-pwstm                   1/1     Running             0          3m49s
kube-system   contrail-agent-7bdm9                     2/2     Running             0          3m46s
kube-system   contrail-analytics-6clf9                 3/3     Running             1          3m49s
kube-system   contrail-analytics-alarm-n427w           3/3     Running             2          3m49s
kube-system   contrail-analytics-snmp-fcr95            3/3     Running             2          3m49s
kube-system   contrail-analyticsdb-967cd               3/3     Running             2          3m49s
kube-system   contrail-config-database-nodemgr-2mx89   1/1     Running             2          3m49s
kube-system   contrail-configdb-8g9nk                  1/1     Running             0          3m49s
kube-system   contrail-controller-config-7psw4         5/5     Running             1          3m48s
kube-system   contrail-controller-control-84rl7        4/4     Running             1          3m49s
kube-system   contrail-controller-webui-2f5xd          2/2     Running             0          3m48s
kube-system   contrail-kube-manager-9q7bk              1/1     Running             0          3m46s
kube-system   coredns-5644d7b6d9-4666l                 0/1     ContainerCreating   0          23h
kube-system   coredns-5644d7b6d9-8mffv                 0/1     ContainerCreating   0          23h
kube-system   etcd-k8smaster                           1/1     Running             0          23h
kube-system   kube-apiserver-k8smaster                 1/1     Running             0          23h
kube-system   kube-controller-manager-k8smaster        1/1     Running             0          23h
kube-system   kube-proxy-qx4ms                         1/1     Running             0          23h
kube-system   kube-scheduler-k8smaster                 1/1     Running             0          23h
kube-system   rabbitmq-pwlfp                           0/1     CrashLoopBackOff    5          3m47s
kube-system   redis-mqmh6                              1/1     Running             0          3m47s
```

## 打开TF的web UI

> 关闭我们之前可能设置的Ubuntu的手动代理，不然这里无法访问

网址为：https://10.108.145.16:8143

打开网址，忽略警告，继续访问

```shell
用户名：admin
密码：contrail123
```
