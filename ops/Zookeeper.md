# Zookeeper使用

## 主要目录结构

- bin:主要一些运行命令
- conf:配置文件
- contrib:附加功能
- dist-maven:mvn编译后的目录
- docs:帮助文档
- lib:需要依赖jar包
- recipes:案例demo代码
- src:源码

## zookeeper配置

在conf目录下有一个 `zoo_sample.cfg`文件，这是官方提供文件，建议备份后修改。

```shell
cp zoo_sample.cfg zoo.cfg
```

主要配置项：

- tickTime:时间基本单元
- initLimit:用于集群， 允许从节点连接并同步到master节点的初始化连接时间，时间为tickTime倍数
- syncLimit:用于集群，master结点与从节点之间发送消息，请求和应答时间长度(心跳机制)
- dataDir: **必须配置**，zk数据存放目录
- dataLogDir:日志目录，默认和dataDir一致
- clientPort:连接服务器的端口，默认2181

## ACL命令

### getAcl/setAcl

- `getAcl`查看节点权限
- `setAcl`设置节点权限

### 权限

- `world:anyone:cdrwa`
- `auth:user:pwd:cdrwa`
- `digest:user:base64(sha1(pwd)):crawd`
- `ip:192.168.1.4:crawd`
- usper

#### 1. world:anyone

`world:anyone`表示开放权限，任何人都可以进行操作

`cdrwa`表示五种权限：c-create; d-delete; r-read; w-write; a-admin(设置权限)

#### 2. auth:user:pwd

`auth:user:pwd`表示用户权限，需要用户进行登录才可以授权。登录使用明文方式，因此不推荐使用。

使用`addauth digest user:pwd`命令进行用户登录

#### 3. digest:user:base64(sha1(pwd))

`digest:user:base64(sha1(pwd))`表示用户权限，登录使用加密密文。登录和`auth:user:pwd`相似

#### 4. ip:192.168.1.4

`ip:192.168.1.4`表示IP权限，一般用于客户端授权

#### 5. super

超级管理员权限，可对所有节点进行修改

```shell
nohup "$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" "-Dzookeeper.DigestAuthenticationProvider.superDigest=fmy:5mb0scnhEdOM6ScPpUTiXUaRJ+E=" \
```

在`zkServer.sh`中添加 `"-Dzookeeper.DigestAuthenticationProvider.
superDigest=<user>:<base64(sha1(password))>"`增加管理员权限

## 集群

### 搭建集群环境

- 同一服务器下的伪集群
- 不同服务器下的集群

#### 同一服务器下的伪集群

##### 1. 修改每个zk配置文件

```shell
server.1=10.43.2.237:2888:3888
server.2=10.43.2.237:2889:3889
server.3=10.43.2.237:2890:3890
```

- `server.1`表示第一台服务器
- `10.43.2.237`表示本机IP
- `2888`表示集群成员信息交换，是服务器与集群中leader服务器交换信息端口
- `3888`表示leader挂掉后进行选举leader端口

##### 2. 添加myid文件

在dataDir目录下添加myid文件，myid内容为数字，用于标识每一个服务器，不能重复。

#### 不同服务器下的集群

搭建和伪集群相似，不同点在于配置中服务器的IP需要换成每台服务器的IP。



# Zookeeper java API

## 使用Apache Curator 操作Zookeeper

> Versions
> The are currently two released versions of Curator, 2.x.x and 3.x.x:
> Curator 2.x.x - compatible with both ZooKeeper 3.4.x and ZooKeeper 3.5.x
> Curator 3.x.x - compatible only with ZooKeeper 3.5.x and includes support for new features such as dynamic reconfiguration, etc.​	

**不同版本的Curator对于zookeeper版本的支持不同，选择Curator版本时一定要注意zookeeper的版本！ **