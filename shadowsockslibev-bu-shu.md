# \# shadowsocks-libev部署

System: CentOS 7 1、通过ssh连接上服务器：（PS：ip是VPS提供给你的ip，password同）

* 输入 ssh root@ip
* 输入网站上的password
* 新建目录

  ```text
       cd ~
       mkdir shadowsocks
       cd shadowsocks
  ```

2、安装shadowsocks：

* 安装依赖

  ```text
    yum install -y gcc make libtool build-essential git vim
    yum install -y yum install gettext gcc autoconf libtool automake make asciidoc xmlto c-ares-devel libev-devel
  ```

* 下载

  ```text
       git clone --recursive https://github.com/shadowsocks/shadowsocks-libev.git
  ```

* 安装Libsodium 和MbedTLS

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

* 编译&安装

  ```text
       cd shadowsocks-libev
       ./autogen.sh
       ./configure --prefix=/usr && make
         make install
  ```

* 准备必须文件

  ```text
        mkdir -p /etc/shadowsocks-libev
        cp ./rpm/SOURCES/etc/init.d/shadowsocks-libev /etc/init.d/shadowsocks-libev
        cp ./debian/config.json /etc/shadowsocks-libev/config.json
        chmod +x /etc/init.d/shadowsocks-libev
  ```

3、设置shadowsocks配置文件：

* 新建文件

  ```text
   vim /etc/shadowsocks-libev/config.json
  ```

* 按I进入insert模式，输入:（ PS:服务器IP，服务端口（建议自定义），本地监听IP，本地监听端口，密码（建议自定义），超时时间，[加密算法](https://github.com/shadowsocks/shadowsocks/wiki/Encryption)。以下文本编辑操作同\)

  ```text
    {
     "server":"0.0.0.0",
     "server_port":8888,
     "local_address": "127.0.0.1",
     "local_port":1080,
     "password":"mypassword",
     "timeout":300,
     "method":"aes-256-cfb",
    }
  ```

* 多用户配置可以采用下列配置

  ```text
    {
    "server": "0.0.0.0",
    "local_address": "127.0.0.1",
    "local_port": "1080",
    "port_password": {
        "8881": "8881password",
        "8882": "8882password"
    },
    "method": "aes-256-cfb"
    }
  ```

* 按ESC退出编辑状态
* 输入 :wq 退出并保存

4、新建shadowsocks的service单元配置文件： vim /etc/systemd/system/shadowsocks-server.service 输入:（PS：如果服务端口数值小于1024，把nobody改为root。） \[Unit\] Description=Shadowsocks service After=network.target

```text
[Service]
Type=simple
User=nobody
ExecStart=/usr/local/bin/ss-server -c /etc/shadowsocks/config.json
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
KillMode=process
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

5、运行shadowsocks服务并设置为开机自启：

```text
/etc/init.d/shadowsocks-libev start
systemctl start shadowsocks-libev
systemctl enable shadowsocks-libev
```

6、 防火墙开放对应的shadowsocks服务端口和http/https服务端口：

```text
firewall-cmd --permanent --add-port=8888/tcp
sudo firewall-cmd --zone=public --permanent --add-service=https
sudo firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --reload
```

