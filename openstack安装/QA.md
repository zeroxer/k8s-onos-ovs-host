### 1.kuryr-libnetwork安装报错

```shell
jsonschme版本不符合要求，系统的pip安装的是3.0.2，不符合<3.0.0的要求
```

- 系统：CentOS 
- 分支：stable/train

#### 尝试解决

- 1.~~切换到master分支，重新stack.sh~~
- 命令：
```shell
sudo pip uninstall jsonschem..
./stack.sh
```

#### 建议

~~local.conf中的plugin可以不指定分支，直接使用master分支。因为之前没有指定分支时，是没有这个报错的。~~

### 2.openstack实例启动后，无法进入系统Booting from Hard Disk...GRUB

参考：https://www.yunforum.net/group-topic-id-1741.html

```shell
sudo vim /etc/nova/nova.conf 
# 虽然在虚拟机中开了硬件虚拟加速，但是可能BIOS中没有打开，所以这里修改virt_type为qemu
virt_type = qemu
```
