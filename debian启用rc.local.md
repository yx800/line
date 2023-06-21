# debian10< 启用rc.local

## 对systemd注入rc-local服务

```
cat>>/etc/systemd/system/rc-local.service<<EOF  
[Unit]  
Description=/etc/rc.local  
ConditionPathExists=/etc/rc.local

[Service]  
Type=forking  
ExecStart=/etc/rc.local start  
TimeoutSec=0  
StandardOutput=tty  
RemainAfterExit=yes  
SysVStartPriority=99

[Install]  
WantedBy=multi-user.target  
EOF
```

## 执行daemon reload和启动服务开机启动

```
systemctl daemon-reload && systemctl enable rc-local
```

## 创建一个空白的rc.local文件，并写入头尾

···
touch /etc/rc.local

cat /etc/rc.local

#!/bin/sh -e

<your command>

exit 0
···

## 对rc.local赋予权限以执行

`chmod a+x /etc/rc.local`


到这里，就全部完成了，在下一次重启中，该文件将会被自动调用并执行。
