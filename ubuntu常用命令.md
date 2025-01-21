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









