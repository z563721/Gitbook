# shadowsocks-libev部署文档 \(Snap\)

## 安装和启用EPEL Repository

```text
dnf install epel-release -y
```

## Snap

### 安装

```text
yum install snapd
```

### 添加snap启动通信 socket

```text
systemctl enable --now snapd.socket
```

### 创建软链接

```text
ln -s /var/lib/snapd/snap /snap
```

## Shadowsocks-libev

### 安装

```text
snap install shadowsocks-libev
```

### 创建配置文件

跟直接make install源码不同，snap安装的配置文件需要放置在snap目录下

```text
vim /snap/bin/config.json
```

这里注意最好使用以下几种AHEAD加密：

| Name | Alias | Key Size | Salt Size | Nonce Size | Tag Size |
| :--- | :--- | :--- | :--- | :--- | :--- |
| AEAD\_CHACHA20\_POLY1305 | chacha20-ietf-poly1305 | 32 | 32 | 12 | 16 |
| AEAD\_AES\_256\_GCM | aes-256-gcm | 32 | 32 | 12 | 16 |
| AEAD\_AES\_192\_GCM | aes-192-gcm | 24 | 24 | 12 | 16 |
| AEAD\_AES\_128\_GCM | aes-128-gcm | 16 | 16 | 12 | 16 |

输入以下内容，密码可以自己改，端口和下面的防火墙端口相对应

```text
{
"server": "0.0.0.0",
"nameserver": "8.8.8.8",
"server_port": 8888,
"password": "mypassword",
"method": "aes-256-gcm",
"timeout": 600,
"no_delay": true,
"mode": "tcp_and_udp",
"plugin": "",
"plugin_opts": ""
}
```



### Service单元配置

新建Service配置文件

```text
vim /lib/systemd/system/ss.service
```

插入内容

```text
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStart=/usr/bin/ss-server -c /etc/shadowsocks-libev/config.json -u
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

### 关闭SELinux

由于snap本身的原因，会被SELinux拒绝执行，打开`/etc/selinux/config`文件

修改`SELINUX=disabled`

### 开机启动

```text
systemctl daemon-reload
systemctl start ss.service
systemctl enable ss.service
```

## 防火墙

防火墙开放对应的shadowsocks服务端口和http/https服务端口：

```text
firewall-cmd --permanent --add-port=8888/tcp
firewall-cmd --permanent --add-port=8888/udp
sudo firewall-cmd --zone=public --permanent --add-service=https
sudo firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --reload
```







