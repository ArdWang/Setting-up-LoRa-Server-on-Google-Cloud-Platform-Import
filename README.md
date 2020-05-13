# Setting-up-LoRa-Server-on-Google-Cloud-Platform-Import
This is in Setting up LoRa Server on Google Cloud Platform

First

Look at : https://cloud.google.com/community/tutorials/setting-up-loraserver
or https://www.chirpstack.io/

配置过程中一定不能把 项目id填错 填错将会出现以下bug

```

xxxxgooplay@xxxx:~$ sudo chirpstack-network-server
INFO[0000] starting ChirpStack Network Server band=EU_863_870 docs="https://www.chirpstack.io/" net_id=0
00000 version=3.9.0
INFO[0000] storage: setting up storage module
INFO[0000] storage: setting up Redis client
INFO[0000] storage: connecting to PostgreSQL
INFO[0000] storage: applying PostgreSQL data migrations
INFO[0000] storage: PostgreSQL data migrations applied count=0
INFO[0000] gateway/gcp_pub_sub: setting up client
INFO[0000] gateway/gcp_pub_sub: setup downlink topic topic=eu868-gateway-commands
FATA[0000] gateway-backend setup failed: gateway/gcp_pub_sub: downlink topic 'eu868-gateway-commands' does not exis
t

```

还有Cloud functions方法中
要执行的函数
Send
一定只能用Send 用其它的就会出现错误
如以下:
```
部署失败：
Build failed: go: finding google.golang.org/genproto v0.0.0-20190605220351-eb0b1bdb6ae6
go: finding cloud.google.com/go v0.39.0
go: finding google.golang.org/genproto v0.0.0-20190508193815-b515fa19cec8
go: finding google.golang.org/api v0.5.0
go: downloading cloud.google.com/go v0.39.0
go: downloading google.golang.org/genproto v0.0.0-20190605220351-eb0b1bdb6ae6
go: downloading google.golang.org/api v0.5.0
go: downloading google.golang.org/grpc v1.19.0
go: downloading github.com/googleapis/gax-go/v2 v2.0.4
go: downloading github.com/golang/protobuf v1.3.1
go: downloading golang.org/x/net v0.0.0-20190311183353-d8887717615a
go: downloading golang.org/x/sys v0.0.0-20190215142949-d0b11bdaac8a
go: downloading golang.org/x/text v0.3.1-0.20180807135948-17ff2d5776d2
go: downloading golang.org/x/oauth2 v0.0.0-20190226205417-e64efc72b421
go: downloading go.opencensus.io v0.21.0
go: downloading github.com/hashicorp/golang-lru v0.5.0
# serverlessapp
./worker.go:48:28: cannot refer to unexported name iotpubsub.eu868
./worker.go:48:28: undefined: iotpubsub.eu868
./worker.go:48:48: undefined: gateway
./worker.go:48:56: undefined: commands
./worker.go:173:29: cannot refer to unexported name iotpubsub.eu868
./worker.go:173:29: undefined: iotpubsub.eu868
./worker.go:173:49: undefined: gateway
./worker.go:173:57: undefined: commands

```

Ubuntu Linux系统使用命令操作的时候,当命令执行的时候,如果我们想退出当前的执行,可以使用快捷键: Ctrl + c 即可退出当前命令行



安装网关桥的时候 一定要提前安装 

ChirpStack Gateway Bridge利用MQTT来发布事件和接收命令。Mosquitto是流行的开源MQTT代理，但是任何实现MQTT 3.1.1的MQTT代理都可以使用。如果您安装Mosquitto，请确保您安装了最新版本

sudo apt-get install mosquitto

然后执行安装

安装需要 sudo apt-get install lora-gateway-bridge  //这个最后会变为 chirpstack-gateway-bridge

配置 lora-gateway-bridge.toml
安装需要 sudo apt-get install lora-gateway-bridge

配置好之后运行 

测试成功的话

sudo chirpstack-gateway-bridge

执行

sudo systemctl [start|stop|restart|status] chirpstack-gateway-bridge


如果没有导入就报以下的错误
```
xx@xxx:~$ sudo chirpstack-gateway-bridge
INFO[0000] starting ChirpStack Gateway Bridge            docs="https://www.chirpstack.io/gateway-bridge/" version=3.8.0
INFO[0000] backend/semtechudp: starting gateway udp listener  addr="0.0.0.0:1700"
FATA[0000] setup integration error: setup mqtt integration error: integration/mqtt: new GCP Cloud IoT Core authentication error: parse jwt key-file error: asn1: structure error: length too large 
radiancegooplay@tmw023:~$ sudo vim private-key.pem
radiancegooplay@tmw023:~$ sudo apt-get install mosquitto

```

查看系统的日志

journalctl -u chirpstack-gateway-bridge -f -n 50

我又碰到了以下的错误

```
xxxxgooplay@xxx:~$ sudo systemctl status chirpstack-gateway-bridge
● chirpstack-gateway-bridge.service - ChirpStack Gateway Bridge
   Loaded: loaded (/lib/systemd/system/chirpstack-gateway-bridge.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since Tue 2020-05-12 06:26:28 UTC; 8s ago
     Docs: https://www.chirpstack.io/
  Process: 1934 ExecStart=/usr/bin/chirpstack-gateway-bridge (code=exited, status=1/FAILURE)
 Main PID: 1934 (code=exited, status=1/FAILURE)
May 12 06:26:27 tmw023 systemd[1]: chirpstack-gateway-bridge.service: Main process exited, code=exited, status=1/FAILURE
May 12 06:26:27 tmw023 systemd[1]: chirpstack-gateway-bridge.service: Unit entered failed state.
May 12 06:26:27 tmw023 systemd[1]: chirpstack-gateway-bridge.service: Failed with result 'exit-code'.
May 12 06:26:28 tmw023 systemd[1]: chirpstack-gateway-bridge.service: Service hold-off time over, scheduling restart.
May 12 06:26:28 tmw023 systemd[1]: Stopped ChirpStack Gateway Bridge.
May 12 06:26:28 tmw023 systemd[1]: chirpstack-gateway-bridge.service: Start request repeated too quickly.
May 12 06:26:28 tmw023 systemd[1]: Failed to start ChirpStack Gateway Bridge.
May 12 06:26:28 tmw023 systemd[1]: chirpstack-gateway-bridge.service: Unit entered failed state.
May 12 06:26:28 tmw023 systemd[1]: chirpstack-gateway-bridge.service: Failed with result 'exit-code'.

```
或者
```
FATA[0000] setup integration error: setup mqtt integration error: integration/mqtt: new GCP Cloud IoT Core authentication error: parse jwt key-file error: asn1: structure error: tags don’t match (16 vs {class:0 tag:21 length:5 isCompound:false}) {optional:false explicit:false application:false private:false defaultValue: tag: stringType:0 timeType:0 set:false omitEmpty:false} pkcs8 @2

```

#### 关键的处理办法

当出现上面这些错误的时候 第一个想到的是4096的太长了 需要更换成谷歌官方的2048 我还在 

有关创建密钥对 请看谷歌文档的介绍

https://cloud.google.com/iot/docs/how-tos/credentials/keys


文档中看到这段话

从社区提交的Google Cloud社区教程不代表Google Cloud产品的官方文档。

所以我更换了 这种形式的

You can generate a 2048-bit RSA key pair with the following commands:

生成私钥

openssl genpkey -algorithm RSA -out rsa_private.pem -pkeyopt rsa_keygen_bits:2048 

生成公钥

openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem

果然困惑我二天的问题解决了

期间我咨询过
https://forum.chirpstack.io/top/monthly 开发人员 他们也没有给我找到合适的解决办法 不过我敢肯定这种问题没有引起他们的注意

#### Tips

试运行

sudo chirpstack-gateway-bridge

查找文件所在的路径

find / -name "filename"

修改文件权限

chmod a+rw private-key.pem


#### 我的安装步骤

1. 创建 谷歌 vm 这个看文档

2. 生成密钥 

openssl genpkey -algorithm RSA -out rsa_private.pem -pkeyopt rsa_keygen_bits:2048

openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem

3. 上传密钥文件 其中把 rsa_private.pem 文件上传到 谷歌 vm中






