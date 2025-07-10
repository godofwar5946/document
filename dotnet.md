#

## 1. 安装sdk
``` bash
sudo apt install dotnet-sdk-x.0 # x为想要安装的版本号

#使用dotnet 命令查看是否安装成功
dotnet
#使用以下命令查看已经安装的sdk版本
dotnet --list-sdks
#使用以下命令查看可以创建的项目类型
dotnet new list
```
## 2. 创建项目
以控制台项目为例
```bash
dotnet new console
```
## 3. 添加包
以Newtonsoft.json为例
```bash
dotnet add package Newtonsoft.json
```
## 4. 打包
```bash
dotnet publish --sc true -c Release -r linux-x64 -p:PublishSingleFile=true

# 参数说明：
# --sc true                # 启用 ReadyToRun 预编译（可选，提高启动速度）
# -c Release               # 使用 Release 模式编译 或 Debug
# -r linux-x64             # 指定目标运行平台（如 linux-x64、win-x64、osx-x64 等）
# -p:PublishSingleFile=true # 打包为单一可执行文件
# -o ./publish             # 指定输出目录（可选）


```