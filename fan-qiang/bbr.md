# BBR

因为我使用的是Centos 8，不需要升级内核直接改配置就可以了

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

