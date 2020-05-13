# Setting-up-LoRa-Server-on-Google-Cloud-Platform-Import
This is in Setting up LoRa Server on Google Cloud Platform

First

Look at : https://cloud.google.com/community/tutorials/setting-up-loraserver
or https://www.chirpstack.io/


#### 我的安装步骤

##### 第一部分网关网桥的安装

1. 创建 谷歌 vm 这个看文档

2. 生成密钥 
```
   openssl genpkey -algorithm RSA -out rsa_private.pem -pkeyopt rsa_keygen_bits:2048

   openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem
```

3. 上传密钥文件 其中把 rsa_private.pem 文件上传到 谷歌 vm中 并查询一下文件所在的路径 find / -name rsa_private.pem 我的是这个路径 /home/radiancegooplay/rsa_private.pem

4. 安装MQTT sudo apt-get install mosquitto

5. 安装网关网桥  sudo apt-get install chirpstack-gateway-bridge

6. 进入配置文件 sudo vim /etc/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml

7. 修改配置文件如下
   
   ```
   [integration.mqtt.auth]
   type="gcp_cloud_iot_core"

   [integration.mqtt.auth.gcp_cloud_iot_core]
   server="ssl://mqtt.googleapis.com:8883"
   device_id="gw-0102030405060708"
   project_id="tmw023-ril023"
   cloud_region="us-central1"
   registry_id="eu868-gateways"
   jwt_key_file="/home/radiancegooplay/rsa_private.pem"
   ```
  
8. 去GCP中点击 Iot Core中创建注册表 id为 eu868-gateways 区域选择 us-central1 可以按照文档中去做 还可以创建主题这些步骤是必须进行的这个只要细心基本不会出现错误

9. 然后创建设备 设备id命名为 gw-0102030405060708 然后关键的地方在于 设备的 身份验证 把刚创建的 密钥 rsa_public.pem添加设备中去，有两种方式你可以选择其中一种添加 如果不添加将会报通讯失败错误

10. 完成以上步骤后我们首先来测试下 运行 sudo chirpstack-gateway-bridge

如果出现以下代码我们可以看到应该是成功了

```
radiancegooplay@tmw023-vm:~$ sudo chirpstack-gateway-bridge
INFO[0000] starting ChirpStack Gateway Bridge            docs="https://www.chirpstack.io/gateway-bridge/" version=3.8.0
INFO[0000] backend/semtechudp: starting gateway udp listener  addr="0.0.0.0:1700"
INFO[0000] integration/mqtt: connected to mqtt broker   

```
然后按 Ctrl+C 退出 再正式的启动 sudo systemctl [start|stop|restart|status] chirpstack-gateway-bridge
运行 sudo systemctl start chirpstack-gateway-bridge
再检查是否成功启动  sudo systemctl status chirpstack-gateway-bridge

没有问题的话 应该是以下的显示

```
radiancegooplay@tmw023-vm:~$ sudo systemctl status chirpstack-gateway-bridge
● chirpstack-gateway-bridge.service - ChirpStack Gateway Bridge
   Loaded: loaded (/lib/systemd/system/chirpstack-gateway-bridge.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-05-13 02:44:23 UTC; 41s ago
     Docs: https://www.chirpstack.io/
 Main PID: 2196 (chirpstack-gate)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/chirpstack-gateway-bridge.service
           └─2196 /usr/bin/chirpstack-gateway-bridge

May 13 02:44:23 tmw023-vm systemd[1]: Started ChirpStack Gateway Bridge.
May 13 02:44:23 tmw023-vm chirpstack-gateway-bridge[2196]: time="2020-05-13T02:44:23Z" level=info msg="starting Chi
rpStack Gateway Bridge" docs="https://www.chirpstack.io/gateway-bridge/" version=3.8.0
May 13 02:44:23 tmw023-vm chirpstack-gateway-bridge[2196]: time="2020-05-13T02:44:23Z" level=info msg="backend/semt
echudp: starting gateway udp listener" addr="0.0.0.0:1700"
May 13 02:44:24 tmw023-vm chirpstack-gateway-bridge[2196]: time="2020-05-13T02:44:24Z" level=info msg="integration/
mqtt: connected to mqtt broker"
```

如果出现了问题 请查看日志根据日志来解决发生的错误 journalctl -u chirpstack-gateway-bridge -f -n 50

##### 第二部分网关网桥的安装

1. 前面一部分基本看文档去做 文档讲解非常好 关键是 创建 functions的时候 一定要执行的函数修改成 Send

2. 创建数据库 按照文档的说明去创建就行

启用trgm和hstore扩展
在PostgreSQL实例的“ 概述”选项卡中，单击“ 使用Cloud Shell连接”，然后gcloud sql connect ...在控制台中显示命令 时，按Enter。它将提示您输入postgres用户密码（您在创建PostgreSQL实例时配置的密码）。

然后执行以下SQL命令：

```
-- change to the ChirpStack Application Server database
\c chirpstack_as

-- enable the pg_trgm extension
-- (this is needed to facilitate the search feature)
create extension pg_trgm;

-- enable the hstore extension
-- (this is needed for storing additional k/v meta-data)
create extension hstore;

-- exit psql
\q
您可以关闭Cloud Shell。
```


3. 安装ChirpStack网络服务器 sudo apt install chirpstack-network-server

4. 配置chirpstack-network-server.toml文件 sudo vim /etc/chirpstack-network-server/chirpstack-network-server.toml 修改完成后记得:wq保存 

注意： 这个是 ns的千万不能弄错

```
[postgresql]
dsn="postgres://chirpstack_ns:[PASSWORD]@[POSTGRESQL_IP]/chirpstack_ns?sslmode=disable"

[redis]
url="redis://xxxx:6379"

[network_server]
net_id="000000"

  [network_server.band]
  name="EU_863_870"

  [network_server.network_settings]
  rx1_delay=3

  [network_server.gateway.backend]
  type="gcp_pub_sub"

    [network_server.gateway.backend.gcp_pub_sub]
    project_id="tmw023-ril023"
    uplink_topic_name="eu868-gateway-events"
    downlink_topic_name="eu868-gateway-commands"

[metrics]
timezone="Local"
```
或者是 US915带1的

```
[postgresql]
dsn="postgres://chirpstack_ns:[PASSWORD]@[POSTGRESQL_IP]/chirpstack_ns?sslmode=disable"

[redis]
url="redis://[REDIS_IP]:6379"

[network_server]
net_id="000000"

  [network_server.band]
  name="US_902_928"

  [network_server.network_settings]
  rx1_delay=3
  enabled_uplink_channels=[0, 1, 2, 3, 4, 5, 6, 7, 64]

  [network_server.gateway.backend]
  type="gcp_pub_sub"

    [network_server.gateway.backend.gcp_pub_sub]
    project_id="tmw023-ril023"
    uplink_topic_name="us915-gateway-events"
    downlink_topic_name="us915-gateway-commands"

[metrics]
timezone="Local"
```

以上就是配置
下面我们 试运行下 

```
radiancegooplay@tmw023-vm:~$ sudo chirpstack-network-server
INFO[0000] starting ChirpStack Network Server            band=EU_863_870 docs="https://www.chirpstack.io/" net_id=000000 version=3.9.0
INFO[0000] storage: setting up storage module           
INFO[0000] storage: setting up Redis client             
INFO[0000] storage: connecting to PostgreSQL            
INFO[0000] storage: applying PostgreSQL data migrations 
INFO[0000] storage: PostgreSQL data migrations applied   count=0
INFO[0000] gateway/gcp_pub_sub: setting up client       
INFO[0000] gateway/gcp_pub_sub: setup downlink topic     topic=eu868-gateway-commands
INFO[0000] gateway/gcp_pub_sub: setup uplink topic       topic=eu868-gateway-events
INFO[0000] gateway/gcp_pub_sub: check if uplink subscription exists  subscription=eu868-gateway-events-chirpstack
INFO[0000] no geolocation-server configured             
INFO[0000] configuring join-server client                ca_cert= server="http://localhost:8003" tls_cert= tls_key=
INFO[0000] api: starting network-server api server       bind="0.0.0.0:8000" ca-cert= tls-cert= tls-key=
INFO[0000] starting downlink device-queue scheduler     
INFO[0000] starting multicast scheduler                 

```
出现上面这个应该是正确的启动了 接着我们 Ctrl+C退出 正式的去启动

sudo systemctl start chirpstack-network-server

然后检查是否是启动了 sudo systemctl status chirpstack-network-server

```
radiancegooplay@tmw023-vm:~$ sudo systemctl status chirpstack-network-server
● chirpstack-network-server.service - ChirpStack Network Server
   Loaded: loaded (/lib/systemd/system/chirpstack-network-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-05-13 03:02:35 UTC; 41s ago
     Docs: https://www.chirpstack.io/
 Main PID: 2293 (chirpstack-netw)
    Tasks: 8 (limit: 4915)
   CGroup: /system.slice/chirpstack-network-server.service
           └─2293 /usr/bin/chirpstack-network-server

May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="storage: Pos
tgreSQL data migrations applied" count=0
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="gateway/gcp_
pub_sub: setting up client"
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="gateway/gcp_
pub_sub: setup downlink topic" topic=eu868-gateway-commands
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="gateway/gcp_
pub_sub: setup uplink topic" topic=eu868-gateway-events
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="gateway/gcp_
pub_sub: check if uplink subscription exists" subscription=eu868-gateway-events-chirpstack
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="no geolocati
on-server configured"
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="configuring 
join-server client" ca_cert= server="http://localhost:8003" tls_cert= tls_key=
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="api: startin
g network-server api server" bind="0.0.0.0:8000" ca-cert= tls-cert= tls-key=
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="starting dow
nlink device-queue scheduler"
May 13 03:02:35 tmw023-vm chirpstack-network-server[2293]: time="2020-05-13T03:02:35Z" level=info msg="starting mul
ticast scheduler"
```

通过检查我们已经成功的启动了

5. 现在我们可以安装ChirpStack应用服务器 

执行 sudo apt install chirpstack-application-server

安装完成后还是老样子 配置 chirpstack-application-server.toml文件

sudo vim /etc/chirpstack-application-server/chirpstack-application-server.toml

注意需要增加一个参数 JWT_SECRET 这个你可以通过自己电脑运行 openssl rand -base64 32 随机生成然后复制进去就可以
记得 :wq保存

 ```
 [postgresql]
dsn="postgres://chirpstack_as:[PASSWORD]@[POSTGRESQL_IP]/chirpstack_as?sslmode=disable"

[redis]
url="redis://[REDIS_IP]:6379"

[application_server]

  [application_server.integration]
  enabled=["gcp_pub_sub"]

  [application_server.integration.gcp_pub_sub]
  project_id="tmw023-ril023"
  topic_name="chirpstack-application-server"

  [application_server.external_api]
  bind="0.0.0.0:8080"
  jwt_secret="[JWT_SECRET]"
 
 ```
现在我们运行测试

sudo chirpstack-application-server

出现以下代码说明运行成功

```
radiancegooplay@tmw023-vm:~$ sudo chirpstack-application-server
INFO[0000] starting ChirpStack Application Server        docs="https://www.chirpstack.io/" version=3.10.0
INFO[0000] storage: setting up storage package          
INFO[0000] storage: setup metrics                       
INFO[0000] storage: setting up Redis client             
INFO[0000] storage: connecting to PostgreSQL database   
INFO[0000] storage: applying PostgreSQL data migrations 
INFO[0000] storage: PostgreSQL data migrations applied   count=0
INFO[0000] integration/gcp_pub_sub: setting up client   
INFO[0000] integration/gcp_pub_sub: setup topic          topic=chirpstack-application-server
INFO[0000] api/as: starting application-server api       bind="0.0.0.0:8001" ca_cert= tls_cert= tls_key=
INFO[0000] api/external: starting api server             bind="0.0.0.0:8080" tls-cert= tls-key=
INFO[0000] api/external: registering rest api handler and documentation endpoint  path=/api
INFO[0000] api/js: starting join-server api              bind="0.0.0.0:8003" ca_cert= tls_cert= tls_key=

```
Ctrl+C退出来
然后启动

sudo systemctl start chirpstack-application-server

检查是否启动成功

sudo systemctl status chirpstack-application-server

当出现以下代码的时候说明启动成功

```
radiancegooplay@tmw023-vm:~$ sudo systemctl status chirpstack-application-server
● chirpstack-application-server.service - ChirpStack Application Server
   Loaded: loaded (/lib/systemd/system/chirpstack-application-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-05-13 03:15:28 UTC; 31s ago
     Docs: https://www.chirpstack.io/
 Main PID: 2431 (chirpstack-appl)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/chirpstack-application-server.service
           └─2431 /usr/bin/chirpstack-application-server

May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="storage:
 setting up Redis client"
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="storage:
 connecting to PostgreSQL database"
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="storage:
 applying PostgreSQL data migrations"
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="storage:
 PostgreSQL data migrations applied" count=0
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="integrat
ion/gcp_pub_sub: setting up client"
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="integrat
ion/gcp_pub_sub: setup topic" topic=chirpstack-application-server
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="api/as: 
starting application-server api" bind="0.0.0.0:8001" ca_cert= tls_cert= tls_key=
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="api/exte
rnal: starting api server" bind="0.0.0.0:8080" tls-cert= tls-key=
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="api/exte
rnal: registering rest api handler and documentation endpoint" path=/api
May 13 03:15:29 tmw023-vm chirpstack-application-server[2431]: time="2020-05-13T03:15:29Z" level=info msg="api/js: 
starting join-server api" bind="0.0.0.0:8003" ca_cert= tls_cert= tls_key=
```

现在你可以打开你的 http://[你的域名]:8080进行测试


#### 错误处理

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


