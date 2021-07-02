# ngrok内网穿透

## 安装go环境
中文源地址:https://golang.google.cn/dl/

### linux安装
```
#将go包解压到/usr/local/go中
tar -C /usr/local -xzf go1.4.linux-amd64.tar.gz #注意名字
#加入环境变量
export PATH=$PATH:/usr/local/go/bin

```
### windows安装

```
Windows 下可以使用 .msi 后缀(在下载列表中可以找到该文件，如go1.4.2.windows-amd64.msi)的安装包来安装。

默认情况下 .msi 文件会安装在 c:\Go 目录下。你可以将 c:\Go\bin 目录添加到 Path 环境变量中。添加后你需要重启命令窗口才能生效。
```
---------
- 首先参考怎么安装go环境,上一篇已经讲到了,这里就不提了
- 接着下载git中的那个ngrok.zip并解压到/usr/local/ngrok目录下
### 生成证书
```code
#进入ngrok目录
cd ngrok  
#创建一个cert目录用来保存证书
mkdir cert 
cd cert
#定义我们要映射的域名
export NGROK_DOMAIN="abc.com"
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=$NGROK_DOMAIN" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
```
### 将证书放到/usr/local/ngrok/assets中

```code
cp rootCA.pem ../assets/client/tls/ngrokroot.crt
cp device.crt ../assets/server/tls/snakeoil.crt
cp device.key ../assets/server/tls/snakeoil.key
```
### 生成服务端的文件
在ngrok目录中执行

```
<!--linux服务端/客户端-->
GOOS=linux GOARCH=386 make release-server (32位)
GOOS=linux GOARCH=amd64 make release-server（64位）

GOOS=linux GOARCH=386 make release-client (32位)
GOOS=linux GOARCH=amd64 make release-client（64位）

<!--Mac OS服务端/客户端-->
GOOS=darwin GOARCH=386 make release-server
GOOS=darwin GOARCH=amd64 make release-server

GOOS=darwin GOARCH=386 make release-client
GOOS=darwin GOARCH=amd64 make release-client


<!--windows服务端/客户端-->
GOOS=windows GOARCH=386 make release-server
GOOS=windows GOARCH=amd64 make release-server

GOOS=windows GOARCH=386 make release-client
GOOS=windows GOARCH=amd64 make release-client

```

所有程序都将生成在bin目录中，不同平台将建立不同的子目录
```
（当我生成linux 64位程序时，会直接保存在bin目录下无子目录。所以我个人推测，如果生成是当前系统的程序，则无子目录，直接存放于bin目录下。各位若有条件可验证下）

对应的目录名字如下:

linux
bin/linux_386
bin/linux_amd64

mac os 
bin/darwin_386 
bin/darwin_amd64 

windows
bin/windows_386
bin/windows_amd64
```
目录中，ngrok是客户端，ngrokd是服务端





### 服务端启动
 /usr/local/ngrok/bin/linux_386/ngrokd -domain="$NGROK_DOMAIN" -httpAddr=":1992" -httpsAddr=":1993"
 
 ```
 
 其它配置：

-httpAddr=":80" http服务的访问端口 默认80

-httpsAddr=":443" https服务的访问端口 默认443

-tunnelAddr=":4443" 客户端连接服务端的端口 默认4443

以上端口，如若与系统其他服务有冲突，开启服务时请自行配置其他端口，同时记得配置防火墙
```

### 客户端配置和连接

```
通过sz或者ftp等方式将ngrok下载到你需要使用客户端的电脑中

新建配置文件ngrok.cfg

<!--配置服务端连接地址，也就是基础域名。端口则与服务端-tunnelAddr配置相同-->
server_addr: "abc.com:4443"  
trust_host_root_certs: false
//上面不考虑1992和1993的端口,只用写ngrok的4443端口就行
//也不用考虑nginx的80配置域名啥的,会自动配置
运行客户端

# 客户端一般都是windows所以要下exe也就是ngrok.exe
ngrok -config=ngrok.cfg -subdomain ngrok 80
-subdomain用来指定域名的前缀（也就是映射域名的前缀），如上设置ngrok，当访问http://ngrok.abc.com时，ngrok服务端接收到请求后，便会将客户端http相应返回给访问端。80用来指定本地http服务的端口

此时，ngrok服务便搭建完成
```
 