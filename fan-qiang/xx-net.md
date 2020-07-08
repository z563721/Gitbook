# XX-net



XX-net是一款基于Goagent和GoGoTest的翻墙软件。

github下载最新版本XX-net[https://github.com/XX-net/XX-Net](https://github.com/XX-net/XX-Net)

当然，已经安装git的童鞋也可以用 git clone[  ](https://github.com/XX-net/XX-Net.git) [https://github.com/XX-net/XX-Net.git](https://github.com/XX-net/XX-Net.git)

  
2、运行

* Windows系统

          运行start.bat

* Mac系统

          运行start.py

* PS：应用启动后需要一段时间搜索🔍可用的Google IP地址才可以快速访问，地址可以通过外部导入[http://127.0.0.1:8085/?module=gae\_proxy&menu=advanced](http://127.0.0.1:8085/?module=gae_proxy&menu=advanced)

       3、配置网络设置

* Windows系统

          打开控制面板 -&gt; Internet选项 -&gt; “连接”选项卡 -&gt; 局域网设置-&gt;属性          要使用自动代理控制在pac地址输入 “http://127.0.0.1:8086/proxy.pac”          要使用全局代理在代理地址输入 “http://127.0.0.1:8087”  


* Mac系统

          编好设置 -&gt; 网络 -&gt; 高级... -&gt; 代理          要使用自动代理控制请勾选自动代理配置并输入URL“http://127.0.0.1:8086/proxy.pac”          要使用全局代理请勾选HTTP代理和HTTPS代理并输入URL “http://127.0.0.1:8087”          

* Chrome

          使用Chrome自动代理设置插件[https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=zh-CN](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=zh-CN)  
4、注册

* 注册Google账号 [https://accounts.google.com/SignUp](https://accounts.google.com/SignUp)
* 绑定手机号码 [https://security.google.com/settings/phone?pli=1](https://security.google.com/settings/phone?pli=1)
* 如果你没有启用2步登录验证，请启用弱安全应用 https://www.google.com/settings/security/lesssecureapps

           如果你已经启用2步登录验证，请设置应用专用密码： [https://security.google.com/settings/security/apppasswords](https://security.google.com/settings/security/apppasswords)           设备选"其他"，随便起个名词，比如GoAgent，点"生成"后，会出来一串密码。 以后就拿这个密码来上传appid部署服务端

* 注册Google应用空间[https://appengine.google.com/](https://appengine.google.com/)记录应用专案ID

5、部署服务器

* 打开[http://127.0.0.1:8085/?module=gae\_proxy&menu=deploy](http://127.0.0.1:8085/?module=gae_proxy&menu=deploy)
* * GAE APPLE ID 填你的应用专案名字，用’\|’分割多个专案
  * Google Account填你的Google账号
  * Password填你的Google账号密码，如开启了二步认证请输入获取的二步认证密码
  * 点击开始部署，显示complete update of app即为部署成功，如不成功请检查网络配置和[http://127.0.0.1:8085/?module=gae\_proxy&menu=status](http://127.0.0.1:8085/?module=gae_proxy&menu=status) 的连接池（Collection Number）
  * 打开[http://127.0.0.1:8085/?module=gae\_proxy&menu=config](http://127.0.0.1:8085/?module=gae_proxy&menu=config) 输入你刚刚配置的 GAE APPLE ID

    6、分享设置

* 打开data/gae\_proxy/目录，把下面的配置文件放进去，同目录取得CA签名证书并发送到设备并添加到信任文件
* 打开设备的网络设置，在代理设置里面输入电脑的IP地址，全局代理使用8087端口，pac自动代理使用8086端口，并在后面加上资源路径“/proxy.pac”
* PS：若两个设备处于同一内网，可直接食用内网IP；若使用外网代理，需请求运营商给予路由器真实外网IP，并服用花生壳DDNS服务。

