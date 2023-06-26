
# 特殊操作

```
#docker exec后使用ss -K没有权限，宿主机root用户使用这个命令进入docker，就有权限了kill 连接了
 nsenter -t  $(docker inspect -f '{{.State.Pid}}' xcc-uc) -n /bin/sh
 
 ```