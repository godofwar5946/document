# openVPN使用指南

### 1. 安装Easy-RSA并生成密钥

下载Easy-RSA，地址：`https://github.com/OpenVPN/easy-rsa/releases`

解压后复制`vars.example` 为`vars`，修改vars配置文件

```bash
if [ -z "$EASYRSA_CALLER" ]; then
	echo "You appear to be sourcing an Easy-RSA *vars* file. This is" >&2
	echo "no longer necessary and is disallowed. See the section called" >&2
	echo "*How to use this file* near the top comments for more details." >&2
	return 1
fi


set_var EASYRSA_DN	"cn_only"


set_var EASYRSA_REQ_COUNTRY	"CN"
set_var EASYRSA_REQ_PROVINCE	"ShanDong"
set_var EASYRSA_REQ_CITY	"YanTai"
set_var EASYRSA_REQ_ORG	"godofwar5946"
set_var EASYRSA_REQ_EMAIL	"godofwar5946@gmail.com"

set_var EASYRSA_KEY_SIZE	2048

```

生成密钥信息

```bash
1. [root@Web01 easy-rsa]# ./easyrsa init-pki    #1、初始化，在当前目录创建PKI目录，用于存储整数
2. [root@Web01 easy-rsa]# ./easyrsa build-ca    #2、创建根证书，会提示设置密码，用于ca对之后生成的server和client证书签名时使用，其他提示内容直接回车即可
3. Enter New CA Key Passphrase:         #注意密码不能太短，我这边设置的是123456
4. Re-Enter New CA Key Passphrase: 
5. [root@Web01 easy-rsa]# ./easyrsa gen-req server nopass    #3、创建server端证书和私钥文件，nopass表示不加密私钥文件，提示内容直接回车即可
6. [root@Web01 easy-rsa]# ./easyrsa sign server server    #4、给server端证书签名，提示内容需要输入yes和创建ca根证书时候的密码
7. [root@Web01 easy-rsa]# ./easyrsa gen-dh    #5、创建Diffie-Hellman文件，密钥交换时的Diffie-Hellman算法
8. [root@Web01 easy-rsa]# ./easyrsa gen-req client nopass    #6、创建client端的证书和私钥文件，nopass表示不加密私钥文件，提示内容直接回车即可
9. [root@Web01 easy-rsa]# ./easyrsa sign client client    #7、给client端证书前面，提示内容输入yes和创建ca根证书时候的密码
```



### 2. 服务端安装openvpn

安装openvpn `apt install openvpn`

修改配置文件`/etc/openvpn/server.conf`

```bash
port 1194													#端口
proto tcp													#协议
dev tun														#采用路由隧道方式（tun/tap）
ca /opt/easy-rsa/EasyRSA-3.2.2/pki/ca.crt					#ca证书位置
cert /opt/easy-rsa/EasyRSA-3.2.2/pki/issued/server.crt		#服务端公钥位置
key /opt/easy-rsa/EasyRSA-3.2.2/pki/private/server.key		#服务端私钥位置
dh /opt/easy-rsa/EasyRSA-3.2.2/pki/dh.pem					#证书校验算法
server 10.224.0.0 255.255.0.0								#给客户端分配的地址池
push "route 192.168.0.0 255.255.255.0"						#推送给客户端的路由信息
ifconfig-pool-persist /var/log/openvpn/ipp.txt				#地址池记录文件位置，未来让openvpn客户端固定ip地址使用的
max-clients 300												#最多允许300个客户端连接
log /var/log/openvpn.log									#openvpn日志记录位置
client-to-client											#允许客户端与客户端之间通信
persist-key													
persist-tun													
verb 3														#设置日志记录冗长级别
duplicate-cn												#openvpn一个证书在同一时刻是否允许多个客户端接入
comp-lzo													#启用允许数据压缩，客户端配置文件也需要有这项。
client-config-dir /etc/openvpn/ccd							#客户端配置文件夹，用于设置路由到客户端转发（可不配置）
route 192.168.0.0 255.255.255.0								#配合客户端配置iroute使用，用于路由到客户端转发（可不配置）
```

重启服务 `systemctl restart openvpn@server`

如果要配置路由到某个客户端，用来代理网络请求，需要配置以下信息（可选）

在`/etc/openvpn/ccd`目录下新建文件，文件名和客户端证书名相同（比如客户端证书名称是client001.key，则新建的文件名为client001）

写入以下内容

```bash
iroute 192.168.0.0 255.255.255.0
```

重启服务

`note`：需要配合服务端配置文件中的 `client-config-dir /etc/openvpn/ccd` 和`route 192.168.0.0 255.255.255.0`使用

用来路由的客户端证书只能单个设备使用，并且和中转机使用的证书对应



### 3. 客户端安装vpn

安装openVPN客户端

运行openVPN GUI

托盘栏右击openVPN ，点击`选项`，查看`高级`中`配置文件`的`文件夹`

打开该文件夹

将`ca.crt`、`client.crt`、`client.key`、`client.ovpn`放到该文件夹中

右击托盘栏的openVPN，点击连接



如果必要，修改client.ovpn配置文件

```bash
client											#当前为客户端
dev tun											#采用路由隧道方式（tun/tap）
proto tcp										#协议
remote 110.42.57.121 1194						#远程服务器地址，如果有负载均衡，可以出现多个remote
resolv-retry infinite							#始终重新解析Server的IP地址（如果remote后面跟的是域名）
nobind											#定义在本机不邦定任何端口监听incoming数据。
persist-key				
persist-tun
verb 3
ca ca.crt										#ca证书路径
cert client-sanjin.crt							#客户端证书路径
key client-sanjin.key							#客户端密钥路径
comp-lzo										#启用允许数据压缩，这个地方需要严格和Server端保持一致
route 10.224.0.0 255.255.0.0					#添加到服务器的路由（可选）
```



# 参考

## OpenVPN服务端搭建部署

### 一、安装配置证书软件

```bash
1. [root@Web01 ~]# yum -y install easy-rsa
2. [root@Web01 ~]# mkdir /opt/easy-rsa
3. [root@Web01 easy-rsa]# rpm -ql easy-rsa    #查看已安装的RPM包中名为 easy-rsa 的文件列表
4. [root@Web01 easy-rsa]# cp -a /usr/share/easy-rsa/3.0.8/* .
5. [root@Web01 easy-rsa]# cp -a /usr/share/doc/easy-rsa-3.0.8/vars.example ./vars
6. [root@Web01 easy-rsa]# > vars 
7. [root@Web01 easy-rsa]# cat vars
8. if [ -z "$EASYRSA_CALLER" ]; then
9.         echo "You appear to be sourcing an Easy-RSA 
10. 'vars' file." >&2
11.         echo "This is no longer necessary and is 
12. disallowed. See the section called" >&2
13.         echo "'How to use this file' near the top 
14. comments for more details." >&2
15.       return 1
16. fi
17. set_var EASYRSA_DN "cn_only"
18. set_var EASYRSA_REQ_COUNTRY "CN"
19. set_var EASYRSA_REQ_PROVINCE "Beijing"
20. set_var EASYRSA_REQ_CITY "Shanghai"
21. set_var EASYRSA_REQ_ORG "koten"
22. set_var EASYRSA_REQ_EMAIL "888888@qq.comm"
23. set_var EASYRSA_NS_SUPPORT "yes"


```

### 二、创建证书

```bash
1. [root@Web01 easy-rsa]# ./easyrsa init-pki    #1、初始化，在当前目录创建PKI目录，用于存储整数
2. [root@Web01 easy-rsa]# ./easyrsa build-ca    #2、创建根证书，会提示设置密码，用于ca对之后生成的server和client证书签名时使用，其他提示内容直接回车即可
3. Enter New CA Key Passphrase:         #注意密码不能太短，我这边设置的是123456
4. Re-Enter New CA Key Passphrase: 
5. [root@Web01 easy-rsa]# ./easyrsa gen-req server nopass    #3、创建server端证书和私钥文件，nopass表示不加密私钥文件，提示内容直接回车即可
6. [root@Web01 easy-rsa]# ./easyrsa sign server server    #4、给server端证书签名，提示内容需要输入yes和创建ca根证书时候的密码
7. [root@Web01 easy-rsa]# ./easyrsa gen-dh    #5、创建Diffie-Hellman文件，密钥交换时的Diffie-Hellman算法
8. [root@Web01 easy-rsa]# ./easyrsa gen-req client nopass    #6、创建client端的证书和私钥文件，nopass表示不加密私钥文件，提示内容直接回车即可
9. [root@Web01 easy-rsa]# ./easyrsa sign client client    #7、给client端证书前面，提示内容输入yes和创建ca根证书时候的密码
```

### 三、安装openvpn并写入服务端配置文件

```bash
1. [root@Web01 easy-rsa]# yum -y install openvpn
2. [root@Web01 easy-rsa]# cat /etc/openvpn/server.conf
3. port 1194                                    #端口
4. proto udp                                    #协议
5. dev tun                                      #采用路由隧道模式
6. ca /opt/easy-rsa/pki/ca.crt                  #ca证书的位置
7. cert /opt/easy-rsa/pki/issued/server.crt     #服务端公钥的位置
8. key /opt/easy-rsa/pki/private/server.key     #服务端私钥的位置
9. dh /opt/easy-rsa/pki/dh.pem                  #证书校验算法  
10. server 10.8.0.0 255.255.255.0                #给客户端分配的地址池
11. push "route 172.16.1.0 255.255.255.0"        #允许客户端访问的内网网段
12. ifconfig-pool-persist ipp.txt                #地址池记录文件位置，未来让openvpn客户端固定ip地址使用的
13. keepalive 10 120                             #存活时间，10秒ping一次，120秒如果未收到响应则视为短线
14. max-clients 100                              #最多允许100个客户端连接
15. status openvpn-status.log                    #日志位置，记录openvpn状态
16. log /var/log/openvpn.log                     #openvpn日志记录位置
17. verb 3                                       #openvpn版本
18. client-to-client                             #允许客户端与客户端之间通信
19. persist-key                                  #通过keepalive检测超时后，重新启动VPN，不重新读取
20. persist-tun                                  #检测超时后，重新启动VPN，一直保持tun是linkup的，否则网络会先linkdown然后再linkup
21. duplicate-cn                                 #客户端密钥（证书和私钥）是否可以重复
22. comp-lzo                                     #启动lzo数据压缩格式
```

### 四、启动并检查端口

```bash
1. [root@Web01 easy-rsa]# systemctl start openvpn@server
2. [root@Web01 easy-rsa]# systemctl enable openvpn@server
3. Created symlink from /etc/systemd/system/multi-user.target.wants/openvpn@server.service to /usr/lib/systemd/system/openvpn@.service.
4. [root@Web01 easy-rsa]# ip a s tun0    #查看网段
5. 4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
6.     link/none 
7.     inet 10.8.0.1 peer 10.8.0.2/32 scope global tun0
8. valid_lft forever preferred_lft forever
9.     inet6 fe80::b1:7ea6:1178:8a1a/64 scope link flags 800
10. valid_lft forever preferred_lft forever
11. [root@Web01 easy-rsa]# ss -lntup|grep 1194    #检查端口
12. udp    UNCONN     0      0         *:1194                  *:*                   users:(("openvpn",pid=47104,fd=6))
13. [root@Web01 easy-rsa]# ps -ef|grep openvpn    #检查pid
14. root      47104      1  0 10:59 ?        00:00:00 /usr/sbin/openvpn --cd /etc/openvpn/ --config server.conf
15. root      47202  40565  0 11:01 pts/0    00:00:00 grep --color=auto openvpn

```



## OpenVPN服务端搭建部署

### 一、配置openvpn

```bash

1. [root@Web02 ~]# yum -y install openvpn
2. [root@Web02 ~]# cat /etc/openvpn/client.conf
3. client
4. dev tun
5. proto udp
6. remote 10.0.0.7 1194
7. resolv-retry infinite
8. nobind
9. ca ca.crt
10. cert client.crt
11. key client.key
12. verb 3
13. persist-key
14. comp-lzo
15. 
16. [root@Web01 pki]# scp private/client.key 172.16.1.8:/etc/openvpn/
17. [root@Web01 pki]# scp issued/client.crt  172.16.1.8:/etc/openvpn/
18. [root@Web01 pki]# scp ca.crt 172.16.1.8:/etc/openvpn/
19. 
20. [root@Web02 ~]# systemctl start openvpn@client
21. [root@Web02 ~]# systemctl enable openvpn@client
22. Created symlink from /etc/systemd/system/multi-user.target.wants/openvpn@client.service to /usr/lib/systemd/system/openvpn@.service.

```

### 二、测试连接

```bash
1. [root@Web02 ~]# ifdown eth1    #关闭内网IP
2. [root@Web02 ~]# ip a
3. 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
4. link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
5.     inet 127.0.0.1/8 scope host lo
6.        valid_lft forever preferred_lft forever
7.     inet6 ::1/128 scope host 
8.        valid_lft forever preferred_lft forever
9. 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
10. link/ether 00:0c:29:36:19:29 brd ff:ff:ff:ff:ff:ff
11.     inet 10.0.0.8/24 brd 10.0.0.255 scope global eth0
12.        valid_lft forever preferred_lft forever
13.     inet6 fe80::20c:29ff:fe36:1929/64 scope link
14.        valid_lft forever preferred_lft forever
15. 3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN group default qlen 1000
16. link/ether 00:0c:29:36:19:33 brd ff:ff:ff:ff:ff:ff
17. 4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
18. link/none 
19.     inet 10.8.0.6 peer 10.8.0.5/32 scope global tun0
20.        valid_lft forever preferred_lft forever
21.     inet6 fe80::b198:27bb:5967:356b/64 scope link flags 800 
22.        valid_lft forever preferred_lft forever
23. [root@Web02 ~]# ping 172.16.1.7    #成功通过openvpn ping通
24. PING 172.16.1.7 (172.16.1.7) 56(84) bytes of data.
25. 64 bytes from 172.16.1.7: icmp_seq=1 ttl=64 time=0.686 ms
26. 64 bytes from 172.16.1.7: icmp_seq=2 ttl=64 time=1.13 ms
27. ^C
28. --- 172.16.1.7 ping statistics ---
29. 2 packets transmitted, 2 received, 0% packet loss, time 1001ms
30. rtt min/avg/max/mdev = 0.686/0.908/1.130/0.222 ms
```





### ==================服务器端配置文件==================

#### ;local a.b.c.d

定义openvpn监听的IP地址，如果是服务器单网卡的也可以不注明，但是服务器是多网卡的建议注明。

#### port 1194

定义openvpn监听的的端口，默认为1194端口。

#### proto tcp

;proto udp

定义openvpn使用的协议，默认使用UDP。如果是生产环境的话，建议使用TCP协议。

#### dev tun

;dev tap

定义openvpn运行时使用哪一种模式，openvpn有两种运行模式一种是tap模式，一种是tun模式。

tap模式也就是桥接模式，通过软件在系统中模拟出一个tap设备，该设备是一个二层设备，同时支持链路层协议。

tun模式也就是路由模式，通过软件在系统中模拟出一个tun路由，tun是ip层的点对点协议。

具体使用哪一种模式，需要根据自己的业务进行定义。

#### ca ca.crt

定义openvpn使用的CA证书文件，该文件通过build-ca命令生成，CA证书主要用于验证客户证书的合法性。

#### cert vpnilanni.crt

定义openvpn服务器端使用的证书文件。

#### key vpnilanni.key

定义openvpn服务器端使用的秘钥文件，该文件必须严格控制其安全性。

#### dh dh2048.pem

定义Diffie hellman文件。

#### server 10.8.0.0 255.255.255.0

定义openvpn在使用tun路由模式时，分配给client端分配的IP地址段。

#### ifconfig-pool-persist ipp.txt

定义客户端和虚拟ip地址之间的关系。特别是在openvpn重启时,再次连接的客户端将依然被分配和断开之前的IP地址。

#### ;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100

定义openvpn在使用tap桥接模式时，分配给客户端的IP地址段。

#### ;push “route 192.168.10.0 255.255.255.0”

向客户端推送的路由信息，假如客户端的IP地址为10.8.0.2，要访问192.168.10.0网段的话，使用这条命令就可以了。

#### ;client-config-dir ccd

这条命令可以指定客户端IP地址。

使用方法是在/etc/openvpn/创建ccd目录，然后创建在ccd目录下创建以客户端命名的文件。比如要设置客户端 ilanni为10.8.0.100这个IP地址，只要在 /etc/openvpn/ccd/ilanni文件中包含如下行即可:

#### ifconfig-push 10.8.0.200 255.255.255.0

push “redirect-gateway def1 bypass-dhcp”

这条命令可以重定向客户端的网关，在进行翻墙时会使用到。

#### ;push “dhcp-option DNS 208.67.222.222”

向客户端推送的DNS信息。

假如客户端的IP地址为10.8.0.2，要访问192.168.10.0网段的话，使用这条命令就可以了。如果有网段的话，可以多次出现push route关键字。同时还要配合iptables一起使用。

#### client-to-client

这条命令可以使客户端之间能相互访问，默认设置下客户端间是不能相互访问的。

#### duplicate-cn

定义openvpn一个证书在同一时刻是否允许多个客户端接入，默认没有启用。

#### keepalive 10 120

定义活动连接保时期限

#### comp-lzo

启用允许数据压缩，客户端配置文件也需要有这项。

#### ;max-clients 100

定义最大客户端并发连接数量

#### ;user nobody

;group nogroup

定义openvpn运行时使用的用户及用户组。

#### persist-key

通过keepalive检测超时后，重新启动VPN，不重新读取keys，保留第一次使用的keys。

#### persist-tun

通过keepalive检测超时后，重新启动VPN，一直保持tun或者tap设备是linkup的。否则网络连接，会先linkdown然后再linkup。

#### status openvpn-status.log

把openvpn的一些状态信息写到文件中，比如客户端获得的IP地址。

#### log openvpn.log

记录日志，每次重新启动openvpn后删除原有的log信息。也可以自定义log的位置。默认是在/etc/openvpn/目录下。

#### ;log-append openvpn.log

记录日志，每次重新启动openvpn后追加原有的log信息。

#### verb 3

设置日志记录冗长级别。

#### ;mute 20

重复日志记录限额

以上就是openvpn服务器端server.conf配置文件的内容。

### ==================客户端配置文件==================

#### client

定义这是一个client，配置从server端pull拉取过来，如IP地址，路由信息之类，Server使用push指令推送过来。

#### dev tun

定义openvpn运行的模式，这个地方需要严格和Server端保持一致。

#### proto tcp

定义openvpn使用的协议，这个地方需要严格和Server端保持一致。

#### remote 192.168.1.8 1194

设置Server的IP地址和端口，这个地方需要严格和Server端保持一致。

如果有多台机器做负载均衡，可以多次出现remote关键字。

#### ;remote-random

随机选择一个Server连接，否则按照顺序从上到下依次连接。该选项默认不启用。

#### resolv-retry infinite

始终重新解析Server的IP地址（如果remote后面跟的是域名），保证Server IP地址是动态的使用DDNS动态更新DNS后，Client在自动重新连接时重新解析Server的IP地址。这样无需人为重新启动，即可重新接入VPN。

#### nobind

定义在本机不邦定任何端口监听incoming数据。

persist-key

persist-tun

#### ca ca.crt

定义CA证书的文件名，用于验证Server CA证书合法性，该文件一定要与服务器端ca.crt是同一个文件。

#### cert laptop.crt

定义客户端的证书文件。

#### key laptop.key

定义客户端的密钥文件。

#### ns-cert-type server

Server使用build-key-server脚本生成的，在x509 v3扩展中加入了ns-cert-type选项。防止client使用他们的keys ＋ DNS hack欺骗vpn client连接他们假冒的VPN Server，因为他们的CA里没有这个扩展。

#### comp-lzo

启用允许数据压缩，这个地方需要严格和Server端保持一致。

#### verb 3

设置日志记录冗长级别。
