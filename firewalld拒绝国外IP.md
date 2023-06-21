### firewalld 只允许国内ip访问

`firewall-cmd --zone=public --list-all` 查看public区域大致情况

```
[root@desh install]# firewall-cmd --zone=public --list-all
public (active)
  target: default #firewalld 默认动作，接受 icmp 包并拒绝其它的一切
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: ssh dhcpv6-client
  ports: 80/tcp 12345/tcp 40002-40010/udp
  protocols: 
  masquerade: yes
  forward-ports: port=12345:proto=tcp:toport=80:toaddr=192.168.207.72
  source-ports: 
  icmp-blocks: 
  rich rules: 
        rule family="ipv4" source ipset="china" port port="40001" protocol="tcp" accept
        rule family="ipv4" source ipset="china" port port="40001" protocol="udp" accept
```





#### sip端口拒绝国外IP访问

1、获取china ip源

`wget --no-check-certificate -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/china.txt`



2、firewalld 加载 ipset

```
firewall-cmd --permanent --new-ipset=china --type=hash:net
firewall-cmd --permanent --ipset=china --add-entries-from-file=/etc/china.txt
firewall-cmd --permanent --ipset=china --add-entry='192.168.0.0/16'
firewall-cmd --permanent --ipset=china --add-entry='172.16.0.0/12'
firewall-cmd --permanent --ipset=china --add-entry='10.0.0.0/8'
firewall-cmd --permanent --ipset=china --add-entry='127.0.0.0/24'
```


3、放行udp和tcp的sip端口（40001）

```
firewall-cmd --permanent --zone=public --add-rich-rule 'rule family="ipv4" source ipset="china"  port port=40001 protocol=tcp accept'

firewall-cmd --permanent --zone=public --add-rich-rule 'rule family="ipv4" source ipset="china"  port port=40001 protocol=udp accept'
```

 4、放行rtp范围udp端口（目前暂时定在40002-40010）

 `firewall-cmd --zone=public --add-port=40002-40010/udp --permanent`

5、设置生效

`firewall-cmd --reload`

