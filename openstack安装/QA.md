### 1.kuryr-libnetwork安装报错

- 系统：CentOS 
- 分支：stable/train

#### 尝试解决

- 1.切换到master分支，重新stack.sh

#### 建议

local.conf中的plugin可以不指定分支，直接使用master分支。因为之前没有指定分支时，是没有这个报错的。
