# Centos 下安装 Zookeeper



由于 Zookeeper 依赖 Java 环境，所以必须提前安装好 JDK ，具体步骤请移步另外一篇文章：[CentOS7安装JDK(一)](https://www.hyblogs.com/archives/centos7-jdk-01)

## 安装ZooKeeper单机版

### 一、下载安装包

本文所采用的下载地址：https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/
也可以访问官方网站进行下载：https://zookeeper.apache.org/
默认下载最新的版本：
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1585832795549.png)

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1585832834190.png)

### 二、开始安装

首先，本文以 /data/ 为基目录进行安装配置。

1、解压缩 Zookeeper 压缩包，命令行：

```
[root@hecs-x-large-2-linux-xx ~]# tar -zxvf /data/apache-zookeeper-3.6.0-bin.tar.gz -C /data/zookeeper/
```

2、进入 zookeeper/apache-zookeeper-3.6.0-bin/conf/ 目录，将配置文件 zoo_sample.cfg 更名为 zoo.cfg ，命令行：

```
[root@hecs-x-large-2-linux-xx ~]# cd /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/

[root@hecs-x-large-2-linux-xx conf]# cp zoo_sample.cfg zoo.cfg
```

3、创建 Zookeeper 数据文件目录 datadir ，命令行：

```
[root@hecs-x-large-2-linux-xx ~]# mkdir /data/zookeeper/datadir
```

4、修改Zookeeper配置文件： zoo.cfg ，修改后保存并退出，修改项如下：

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/datadir
clientPort=2181
```

5、进入 Zookeeper 的 bin 目录启动 Zookeeper 服务，命令行：

```
[root@hecs-x-large-2-linux-xx ~]# cd /data/zookeeper/apache-zookeeper-3.6.0-bin/bin/

[root@hecs-x-large-2-linux-xx bin]# sh zkServer.sh start
```

6、查看 Zookeeper 服务状态，命令行：

```
[root@hecs-x-large-2-linux-xx bin]# sh zkServer.sh status /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo.cfg
```

如果查看服务状态出现了异常信息，如下面的异常信息：

```
Error contacting service. It is probably not running.
```

产生此异常信息的原因是由于 Zookeeper 服务的每个实例都拥有全局配置信息，他们在启动的时候会随时随地的进行Leader选举操作。此时，第一个启动的Zookeeper 需要和另外两个 Zookeeper 实例进行通信。但是，另外两个Zookeeper 实例还没有启动起来，因此就产生了这的异样信息。

7、启动 Zookeeper 客户端，命令行：

```
[root@hecs-x-large-2-linux-xx bin]# sh zkCli.sh
```

## 安装Zookeeper集群版

<p style="color:red;">
声明：由于 Zookeeper 集群最低要求三个节点，但是由于成本原因未购买那么多服务器，因此本次记录将在一台服务器上面模拟实现三个节点。需要注意的是在集群为分布式模式下我们使用的每个配置文档模拟一台机器，也就是在单台机器上运行多个 Zookeeper 实例。但是，必须保证每个配置文档的各个端口号不能冲突，除了 clientPort 不同之外，dataDir 也不同。另外，还要在 dataDir 所对应的目录中创建 myid 文件来指定对应的 Zookeeper 服务器实例。
</p>

首先，下载Zookeeper，同上面单机版一样，下载对应的版本即可。

### 一、创建必要目录与相关文件

Zookeeper集群中，每一个节点都需创建对应的 data 目录、dataLog 目录以及 myid 文件。

#### 1、先创建三个节点文件目录

对应的命令行如下：

```
[root@hecs-x-large-2-linux-xx zookeeper]# mkdir server0

[root@hecs-x-large-2-linux-xx zookeeper]# mkdir server1

[root@hecs-x-large-2-linux-xx zookeeper]# mkdir server2
```

#### 2、创建每个节点所必须的 data 目录、dataLog 目录以及 myid 文件

对应的命令行如下：

```
[root@hecs-x-large-2-linux-xx zookeeper]# cd server0

[root@hecs-x-large-2-linux-xx server0]# mkdir data

[root@hecs-x-large-2-linux-xx server0]# mkdir dataLog

[root@hecs-x-large-2-linux-xx server0]# cd data

[root@hecs-x-large-2-linux-xx data]# echo 0 > myid
```

上面只示例了 server0 节点下需要创建的 data 目录、dataLog 目录以及 myid 文件，剩下的 server1和server2可以依葫芦画瓢按照 server0 一样的方式进行创建即可。但是需要注意myid文件中的值不能相同，server1对应的myid文件中的值为1即可，server2对应的myid文件中的值为2即可。myid的值是后续 Zookeeper 配置文件 zoo.cfg 中配置的第几号服务器。

#### 3、修改 Zookeeper 配置文件：zoo.cfg

需修改项如下：

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/server0/data
dataLogDir=/data/zookeeper/server0/dataLog
clientPort=2181
server.0=127.0.0.1:2888:3888
server.1=127.0.0.1:2889:3889
server.2=127.0.0.1:2890:3890
```

#### 4、创建Server1和Server2对应的 zoo1.cfg 和 zoo2.cfg 配置文件

对应的执行命令行如下：

```
[root@hecs-x-large-2-linux-xx conf]# cp zoo.cfg zoo1.cfg
[root@hecs-x-large-2-linux-xx conf]# cp zoo.cfg zoo2.cfg
```

#### 5、分别修改 zoo1.cfg 和 zoo2.cfg 配置文件

需要修改配置文件中的端口号和对应的数据目录等信息，对应的修改项如下：

```
# server1-zoo1.cfg:
clientPort=2182
dataDir=/data/zookeeper/server1/data
dataLogDir=/data/zookeeper/server1/dataLog

# server2-zoo2.cfg:
clientPort=2183
dataDir=/data/zookeeper/server2/data
dataLogDir=/data/zookeeper/server2/dataLog
```

#### 6、启动服务

分别启动三个节点服务，对应的命令行如下：

```
[root@hecs-x-large-2-linux-xx bin]# sh zkServer.sh start /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo.cfg
Starting zookeeper ... STARTED

[root@hecs-x-large-2-linux-xx bin]# sh zkServer.sh start /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo1.cfg
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo1.cfg
Starting zookeeper ... STARTED

[root@hecs-x-large-2-linux-xx bin]# sh zkServer.sh start /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo2.cfg
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo2.cfg
Starting zookeeper ... STARTED
```

#### 7、查看服务状态：

分别查看刚才启动的三个服务状态，对应的命令行如下：

```
[root@hecs-x-large-2-linux-xx bin]# sh zkServer.sh status /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower

[root@hecs-x-large-2-linux-xx bin]# sh zkServer.sh status /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo1.cfg
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo1.cfg
Client port found: 2182. Client address: localhost.
Mode: leader

[root@hecs-x-large-2-linux-xx bin]# sh zkServer.sh status /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo2.cfg
ZooKeeper JMX enabled by default
Using config: /data/zookeeper/apache-zookeeper-3.6.0-bin/conf/zoo2.cfg
Client port found: 2183. Client address: localhost.
Mode: follower
```

由此可见 1 号节点是 Leader 节点！

#### 8、启动客户端连接到 Zookeeper

对应的命令行为：

```
[root@hecs-x-large-2-linux-xx bin]# sh zkCli.sh
Connecting to localhost:2181
2020-04-02 23:23:24,335 [myid:] - INFO  [main:Environment@98] - Client environment:zookeeper.version=3.6.0--b4c89dc7f6083829e18fae6e446907ae0b1f22d7, built on 02/25/2020 14:38 GMT
2020-04-02 23:23:24,337 [myid:] - INFO  [main:Environment@98] - Client environment:host.name=localhost
2020-04-02 23:23:24,337 [myid:] - INFO  [main:Environment@98] - Client environment:java.version=1.8.0_241
2020-04-02 23:23:24,338 [myid:] - INFO  [main:Environment@98] - Client environment:java.vendor=Oracle Corporation
2020-04-02 23:23:24,338 [myid:] - INFO  [main:Environment@98] - Client environment:java.home=/usr/local/java/jre
2020-04-02 23:23:24,338 [myid:] - INFO  [main:Environment@98] - Client environment:java.class.path=/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../zookeeper-server/target/classes:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../build/classes:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../zookeeper-server/target/lib/*.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../build/lib/*.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/zookeeper-prometheus-metrics-3.6.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/zookeeper-jute-3.6.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/zookeeper-3.6.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/snappy-java-1.1.7.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/slf4j-log4j12-1.7.25.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/slf4j-api-1.7.25.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/simpleclient_servlet-0.6.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/simpleclient_hotspot-0.6.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/simpleclient_common-0.6.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/simpleclient-0.6.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/netty-transport-native-unix-common-4.1.45.Final.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/netty-transport-native-epoll-4.1.45.Final.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/netty-transport-4.1.45.Final.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/netty-resolver-4.1.45.Final.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/netty-handler-4.1.45.Final.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/netty-common-4.1.45.Final.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/netty-codec-4.1.45.Final.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/netty-buffer-4.1.45.Final.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/metrics-core-3.2.5.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/log4j-1.2.17.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/json-simple-1.1.1.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jline-2.11.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jetty-util-9.4.24.v20191120.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jetty-servlet-9.4.24.v20191120.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jetty-server-9.4.24.v20191120.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jetty-security-9.4.24.v20191120.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jetty-io-9.4.24.v20191120.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jetty-http-9.4.24.v20191120.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/javax.servlet-api-3.1.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jackson-databind-2.9.10.3.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jackson-core-2.9.10.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/jackson-annotations-2.9.10.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/commons-lang-2.6.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/commons-cli-1.2.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../lib/audience-annotations-0.5.0.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../zookeeper-*.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../zookeeper-server/src/main/resources/lib/*.jar:/data/zookeeper/apache-zookeeper-3.6.0-bin/bin/../conf:.:/usr/local/java/jre/lib/rt.jar:/usr/local/java/lib/dt.jar:/usr/local/java/lib/tools.jar
2020-04-02 23:23:24,338 [myid:] - INFO  [main:Environment@98] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:java.io.tmpdir=/tmp
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:java.compiler=<NA>
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:os.name=Linux
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:os.arch=amd64
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:os.version=3.10.0-1062.1.1.el7.x86_64
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:user.name=root
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:user.home=/root
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:user.dir=/data/zookeeper/apache-zookeeper-3.6.0-bin/bin
2020-04-02 23:23:24,339 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.free=51MB
2020-04-02 23:23:24,340 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.max=228MB
2020-04-02 23:23:24,340 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.total=57MB
2020-04-02 23:23:24,343 [myid:] - INFO  [main:ZooKeeper@1005] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@182decdb
2020-04-02 23:23:24,346 [myid:] - INFO  [main:X509Util@77] - Setting -D jdk.tls.rejectClientInitiatedRenegotiation=true to disable client-initiated TLS renegotiation
2020-04-02 23:23:24,350 [myid:] - INFO  [main:ClientCnxnSocket@239] - jute.maxbuffer value is 1048575 Bytes
2020-04-02 23:23:24,357 [myid:] - INFO  [main:ClientCnxn@1703] - zookeeper.request.timeout value is 0. feature enabled=false
2020-04-02 23:23:24,361 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1154] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181.
2020-04-02 23:23:24,361 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1156] - SASL config status: Will not attempt to authenticate using SASL (unknown error)
Welcome to ZooKeeper!
JLine support is enabled
2020-04-02 23:23:24,422 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@986] - Socket connection established, initiating session, client: /0:0:0:0:0:0:0:1:51934, server: localhost/0:0:0:0:0:0:0:1:2181
2020-04-02 23:23:24,445 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1420] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, session id = 0xbd54a670000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
```
