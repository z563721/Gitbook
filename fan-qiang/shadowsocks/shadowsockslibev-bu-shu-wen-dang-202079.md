# shadowsocks-libev部署文档 2020-7-9

System: CentOS 8

Shadowsocks-libev version: 3.3.4

由于系统升级与软件版本的变动，新版本的配置流程跟之前略有差异

## 安装shadowsocks

### 安装依赖

```text
yum install epel-release -y
yum install gcc gettext autoconf libtool automake make pcre-devel asciidoc xmlto c-ares-devel libev-devel libsodium-devel mbedtls-devel -y
yum install git vim -y
```

### 新建目录**shadowsocks**

```text
cd ~
mkdir shadowsocks
cd shadowsocks
```

### 下载源码

```text
git clone --recursive https://github.com/shadowsocks/shadowsocks-libev.git
```

### 安装Libsodium 和MbedTLS

```text
cd ~
mkdir downloads
cd downloads

# Installation of Libsodium
export LIBSODIUM_VER=1.0.13
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
sudo make install
popd
sudo ldconfig

# Installation of MbedTLS
export MBEDTLS_VER=2.6.0
wget https://tls.mbed.org/download/mbedtls-$MBEDTLS_VER-gpl.tgz
tar xvf mbedtls-$MBEDTLS_VER-gpl.tgz
pushd mbedtls-$MBEDTLS_VER
make SHARED=1 CFLAGS=-fPIC
sudo make DESTDIR=/usr install
popd
sudo ldconfig

cd ../shadowsocks
```

### 编译&安装

```text
cd shadowsocks-libev
./autogen.sh
./configure --prefix=/usr && make
make install
```

### 准备必须文件

```text
mkdir -p /etc/shadowsocks-libev
cp ./rpm/SOURCES/etc/init.d/shadowsocks-libev /etc/init.d/shadowsocks-libev
cp ./debian/config.json /etc/shadowsocks-libev/config.json
chmod +x /etc/init.d/shadowsocks-libev
```

### 

## 设置

### 设置服务参数

* 新建文件

  ```text
   vim /etc/shadowsocks-libev/config.json
  ```

* 输入服务器，密码，加密方式，传输模式

  ```text
  {
  "server": "0.0.0.0",
  "nameserver": "8.8.8.8",
  "server_port": 8888,
  "password": “cptbtptpbcptdtptp”,
  "method": "aes-256-gcm",
  "timeout": 600,
  "no_delay": true,
  "mode": "tcp_and_udp",
  "plugin": "",
  "plugin_opts": ""
  }
  ```



### 设置启动服务

新建shadowsocks的service单元配置文件： 

```text
vim /lib/systemd/system/shadowsocks-libev.service
```

 输入:（PS：如果服务端口数值小于1024，把nobody改为root。） \[Unit\] Description=Shadowsocks service After=network.target

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

## 开机启动

运行shadowsocks服务并设置为开机自启：

```text
systemctl start ss-server.service
systemctl enable ss-server.service
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

