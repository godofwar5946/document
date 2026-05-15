## 1.执行命令，安装acme
curl https://get.acme.sh | sh -s email=xx@gmail.com

## 2.进入acme目录
cd ~/.acme.sh/

## 3.修改配置文件，添加clouflare的api密钥和区域ID
vi account.conf
CF_Token="xxx"
CF_Zone_ID="xxx"

## 4.执行命令获取SSL证书
./acme.sh --issue --dns dns_cf -d xxx.com -d '*.xxx.com'

## 5.将证书拷贝至指定位置
acme.sh --install-cert -d xxx.com -d *.xxx.com --key-file   /opt/ssl/key.pem --fullchain-file /opt/ssl/fullchain.pem --reloadcmd "systemctl reload nginx"