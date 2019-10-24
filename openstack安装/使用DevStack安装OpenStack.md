# 使用DevStack安装OpenStack

注意：

- 0.尽量不要使用带有桌面的操作系统，NetworkManager可能会在配置网络时有影响

- 1.不要使用校园网有线网（宿舍），可以尝试使用手机连mobile，再开热点给电脑用

- 2.切换到stack用户后，设置proxy
  ```shell
  export all_proxy=http://x.x.x.x:xxxx
  export http_proxy=http://x.x.x.x:xxxx
  export https_proxy=http://x.x.x.x:xxxx
  # 检查proxy设置是否正确
  curl ip.gs
  # 配置pip国内源
  
  ```

## 一、单机安装



## 二、集群安装
