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
netstat -antup | grep 80        #查看端口占用情况 -a显示所有选项；-t显示tcp；-u显示udp；-n不显示别名；-l列在listen的服务；-p显示程序名；-e显示扩									展信息
kill -9 12345					#根据pid强制关闭程序


```



# 开启root用户SSH连接

1.修改root用户密码：`sudo passwd root`

2.修改/etc/ssh/sshd_config文件，找到PermitRootLogin这一行，并将其修改为`PermitRootLogin yes`，表示允许root用户通过SSH登录

3.重启ssh服务：`sudo systemctl restart sshd`

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





