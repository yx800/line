# iptables拒绝国外IP地址

确保安装了`apt install ipset -y`

0、获取网段源

`wget --no-check-certificate -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/china.txt`

1、设置好rc.local开机执行(步骤再其他文章中)

rc.local的内容

```
#!/bin/sh -e

ipset create china hash:net maxelem 65536
ipset flush china
while read ip; do
    /sbin/ipset add china $ip
done < /etc/china.txt
ipset add china 192.168.0.0/16
ipset add china 172.16.0.0/12
ipset add china 10.0.0.0/8

iptables -A INPUT -p tcp -m multiport --dport 39001,39003,39004  -m set --match-set china src -j ACCEPT
iptables -A INPUT -p udp -m multiport --dport 39001,39003,39004  -m set --match-set china src -j ACCEPT
iptables -A INPUT -p udp -m multiport --dport 39001,39003:39004 -j DROP
iptables -A INPUT -p tcp -m multiport --dport 39001,39003:39004 -j DROP

exit 0
```

