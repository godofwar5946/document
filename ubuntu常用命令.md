# ubuntu常用命令





```bash
apt update      				#查询软件包更新（不执行更新操作）
apt upgrade     				#执行软件包更新
passwd    						#重置当前用户密码（可以跟用户名，重置指定用户密码）
ufw	status						#查看防火墙状态  inacvice（不活动） acvice（活动）
ufw allow 22    				#防火墙放行指定端口
ufw delete allow 22 			#防火墙关闭指定端口
ufw enable						#开启防火墙
ufw disable						#关闭防火墙
ufw reload						#重启防火墙（修改防火墙配置后需重启生效）
apt install net-tools           #安装网络工具包
netstat -antup | grep 80        #查看端口占用情况 -a显示所有选项；-t显示tcp；-u显示udp；-n不显示别名；-l列在listen的服务；-p显示程序名；-e显示扩展信息
kill -9 12345					#根据pid强制关闭程序
```



# 开启root用户SSH连接

1.修改root用户密码：`sudo passwd root`

2.修改/etc/ssh/sshd_config文件，找到PermitRootLogin这一行，并将其修改为`PermitRootLogin yes`，表示允许root用户通过SSH登录

3.重启ssh服务：`sudo systemctl restart sshd`

# 免密SSH登录

1. 修改要登录的主机 /etc/ssh/sshd_config文件，找到AuthorizedKeysFile这一行，空格后写公钥内容文件位置，如 ~/.ssh/authorized_keys

2. 在本机上生成ssh证书
```bash
ssh-keygen -t rsa -b 1024 -C "comment here"
# -t 选择密钥类型，可选项有 dsa | ecdsa | ecdsa-sk | ed25519 | ed25519-sk | rsa
# -b 指定长度 
# -C 指定注释，会加到生成的.pub文件最后
```
3. 将生成的文件中，.pub文件的内容，复制到远程主机的 authorized_keys 文件中，每行一个

4. 重启远程主机的ssh
```bash
sudo systemctl restart ssh
```

# 配置代理

### 1.配置wget crul代理

1.修改/etc/profile文件，最下行添加 `export http_proxy=http://127.0.0.1:10809/`

2.使配置生效 `source /etc/profile`

### 2.配置非root用户的apt代理

修改/etc/apt/apt.conf，添加 `Acquire::http::Proxy "http://192.168.1.1:8080/";`

### 3.配置docker代理

修改/etc/docker/daemon.json

```json
{
  "proxies": {
    "http-proxy": "http://127.0.0.1:7890",
    "https-proxy": "http://127.0.0.1:7890"
  },
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://ueo0uggy.mirror.aliyuncs.com",
    "https://docker.m.daocloud.io",
    "https://cf-workers-docker-io-apl.pages.dev",
    "http://95.169.25.181"
  ]
}

```

### 4.配置git代理

命令：`git config --global http.proxy http://192.168.0.164:10809/`

删除：`git config --global --unset http.proxy`

# 将可执行文件添加为命令

给文件添加可执行权限：`chmod +x file.sh`

放至`/usr/local/bin`目录

# 配置路由转发

修改配置文件`/etc/sysctl.conf`，添加`net.ipv4.ip_forward = 1`

使配置生效`sudo sysctl -p /etc/sysctl.conf`

```bash
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```

```bash
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```

- **`sudo`**: 以管理员权限执行命令。
- **`iptables`**: Linux 的防火墙工具。
- **`-A FORWARD`**: 将规则追加到 `FORWARD` 链（负责处理转发的数据包）。
- **`-i eth0`**: 匹配输入接口为 `eth0`（通常指内网或 LAN）。
- **`-o eth1`**: 匹配输出接口为 `eth1`（通常指外网或 WAN，如互联网）。
- **`-j ACCEPT`**: 允许匹配的数据包通过。

**作用**：允许从内网（`eth0`）到外网（`eth1`）的所有流量转发。

```bash
sudo iptables -A FORWARD -i eth1 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

- **`-i eth1 -o eth0`**: 匹配从外网（`eth1`）返回内网（`eth0`）的流量。
- **`-m state`**: 使用 `state` 模块检查连接状态。
- **`--state ESTABLISHED,RELATED`**: 仅允许已建立的连接（如对请求的响应）或相关连接（如 FTP 辅助连接）。
- **`-j ACCEPT`**: 允许匹配的数据包通过。

**作用**：允许外网返回的响应流量（确保内网主机能收到请求的回复）。

```bash
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```

- **`t nat`**: 操作 `nat` 表（用于网络地址转换）。
- **`-A POSTROUTING`**: 将规则追加到 `POSTROUTING` 链（数据包发出前的最后处理阶段）。
- **`-o eth1`**: 匹配从 `eth1` 接口发出的数据包。
- **`-j MASQUERADE`**: 将内网设备的私有 IP 伪装为 `eth1` 的公有 IP（动态 NAT）。

**作用**：使内网设备通过 `eth1` 的公网 IP 访问外网，实现共享上网。

# 开启远程桌面

```bash
sudo apt update
sudo apt install xrdp -y
sudo usermod -a-G ssl-cert xrdp
sudo systemctl restart xrdp
```

**特殊情况**：如果需要在windows上使用远程桌面连接到ubuntu上，可能需要切换到X11
使用以下命令查看当前的协议
```bash
echo $XDG_SESSION_TYPE
```
如果输出不为X11，则可能需要更改配置文件
```bash

sudo vim /etc/gdm3/custom.conf # 编辑 gdm3 配置文件
#找到这一行，取消注释，并设置为false
WaylandEnable=false
sudo systemctl restart gdm3 # 重启 gdm3 服务使配置生效

echo $XDG_SESSION_TYPE #验证是否切换成功
```
# 离线安装mysql

1.创建用户组

```bash
groupadd mysql     #创建用户组
# -r 参数表示 mysql 用户是系统用户，不可用于登录系统，创建用户 mysql 并将其添加到用户组 mysql 中
useradd -r -g mysql mysql
```

2.分配用户组

```bash
chown -R mysql /usr/local/mysql/ # 将文件的所有属性改为 mysql 用户
chgrp -R mysql /usr/local/mysql/ # 将组属性改为 mysql 组
```

3.创建数据目录并授权

```bash
mkdir -p /data/mysql #数据目录
chown mysql:mysql -R /data/mysql
```

4.新建my.cnf文件

```bash
[mysqld]
bind-address=0.0.0.0
port=3307
user=mysql
basedir=/usr/local/mysql-ircs //可执行文件位置
datadir=/data/mysql-ircs		//数据文件存储位置
socket=/tmp/mysql-ircs.sock
log-error=/data/mysql-ircs/mysql-ircs.err  //日志文件位置
pid-file= /data/mysql-ircs/mysql-ircs.pid	//进程位置
#character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
server-id=10  //多实例时需要区分
```

5.初始化数据库

使用命令初始化，需要修改对应参数  `./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize`

6.查看初始密码

在log-error目录中，有对应的初始密码

7.设置开机启动

```bash
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
service mysql start
```

8.登录mysql

```bash
mysql -u root -h 127.0.0.1 -P 22 -p
```

9.修改密码

```bash
ALTER USER 'root'@'localhost' IDENTIFIED BY '1234';
FLUSH PRIVILEGES; 
```

