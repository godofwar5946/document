# jitsi使用指南

### 1. 下载jitsi-meet-docker

1.下载最新版本 `wget $(curl -s https://api.github.com/repos/jitsi/docker-jitsi-meet/releases/latest | grep 'zip' | cut -d\" -f4)`

2.解压压缩包 `unzip <filename>`

3.从`env.example`创建`.env`文件 `cp env.example .env`

4.生成安全密钥 `./gen-passwords.sh`

5.创建配置文件目录（可以根据需求修改目录信息） `mkdir -p ~/.jitsi-meet-cfg/{web,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}`

### 2. 配置jitsi-meet-docker

1.修改.env文件

```bash
CONFIG=/opt/jitsi-meet-cfg		//声明配置文件映射地址
HTTP_PORT=18000					//http端口
HTTPS_PORT=18443				//https端口
TZ=Asia/Shanghai				//时区
PUBLIC_URL=https://xx.xxx.xx	//最终使用的域名地址
ETHERPAD_URL_BASE=http://etherpad.meet.jitsi:9001	//如果需要etherpad则放开
ETHERPAD_PUBLIC_URL=https://xx.xxx.xx/p/			//使用域名+/P/
ENABLE_AUTH=1					//开启权限校验
ENABLE_GUESTS=1					//允许访客模式
AUTH_TYPE=internal				//设置权限校验方式

```



2.修改docker-compose.yml文件

```yml
services:
	web:
		volumes:
			- ${CONFIG}/web/ssl:/config/keys	//配置ssl证书的路径
			

```



### 3. 运行服务，修改运行后配置

启动服务 `docker-compose -f docker-compose.yml -f etherpad.yml up -d`

（其中 `-f etherpad.yml`为可选项，具体参考官方文档 `[Self-Hosting Guide - Docker | Jitsi Meet](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker)`）

修改`${CONFIG}/web`目录下的配置文件，`cp config.js  custom-config.js` 和 `cp interface_config.js custom-interface_config.js`  （可选）

修改 `custom-config.js`  （可选）

```bash
config.requireDisplayName = true;		//必须输入名称入会
```

修改 `custom-interface_config.js`

```bash
SHOW_JITSI_WATERMARK: false  //去除会议中jitsi水印
```



### 4. 添加用户

```bash
docker compose exec prosody /bin/bash  #在容器路径下执行此命令
prosodyctl --config /config/prosody.cfg.lua register 用户名 meet.jitsi 密码  #配置用户名和密码
find /config/data/meet%2ejitsi/accounts -type f -exec basename {} .dat \;  #验证是否配置成功   
prosodyctl --config /config/prosody.cfg.lua unregister 用户名 meet.jitsi  #用此命令可以删除用户
```



### 5. 参考

[搭建〖jitsi meet〗视频会议服务 - scarsong](https://scarsong.com/2024/07/01/搭建〖jitsi-meet〗视频会议服务/)

