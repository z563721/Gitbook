# BBR

System: Centos 8

在Centos 8中，内核本身已经支持BBR加速，只需要去设置开启即可

```text
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

设置完后检查配置和进程

```text
sysctl -n net.ipv4.tcp_congestion_control
lsmod | grep bbr
```

