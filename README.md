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


