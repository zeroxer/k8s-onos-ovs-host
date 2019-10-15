# 配置ONOS容器

### 0.修改国内源
```shell
sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
apt update
```

### 1.安装sshpass
```shell
apt install sshpass -y
```

### 2.ssh配置参数
```shell
-q 安静模式
-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```

### 3.使用sshpass连接karaf命令行
```shell
sshpass -p rocks ssh -q -p 8101 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null onos@127.0.0.1 "app activate org.onosproject.fwd"
```
