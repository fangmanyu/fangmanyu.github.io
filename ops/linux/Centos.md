## 安装Centos7

[centos7安装教程](https://linux.cn/article-8048-1.html)

## 使用ifconfig

如果提示没有找到命令，可以安装`net-tools`包使用`ifconfig`命令

```shell
yum install net-tools	
```

## 连接网络

修改网卡配置文件，打开网络连接

```shell
cd /etc/sysconfig/network-scripts
vi ifcfg-enoXXXX # 这里XXX代表随机数

# 设置ONBOOT
ONBOOT=true
```

## 命令提示符显示完整路径

- `vi /etc/profile`
- 修改环境变量PS1：`export PS1='[\u@\h $PWD]\$'`
- 刷新profile:`source /etc/profile`

## 配置ssh远程登录

### 安装和启动ssh

- 查找系统是否安装SSH服务端软件：`rpm -qa|grep openssh `，如果出现 `openssh-server-XXX`，则说明已经安装。

- 启动sshd服务：`service sshd start`，查看服务是否正常： `netstat -a | grep ssh`,`如果出现`

```shell
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
```

​	则说明已经启动服务

### 修改配置信息

- 打开ssh配置：`vi /etc/ssh/sshd_config`
- 禁止root账户远程登录：`PermitRootLogin no`
- 使用密码方式登录：`PasswordAuthentication yes`
- 重启ssh服务：`service sshd restart`

### 添加远程连接用户

创建普通账号ssh：

- 设置用户：`useradd ssh`
- 设置密码：`passwd ssh`

使用普通用户进行ssh登录，之后使用 `su root`切换root账号

## 环境配置

### java环境

修改全局配置

```shell
vi /etc/profile
```

配置jdk环境

```shell
export JAVA_HOME=/usr/local/jdk #jdk所在目录
export PATH=$JAVA_HOME/bin:$PATH
```

刷新配置

```shell
source /etc/profile
```

## 防火墙设置

查看防火墙状态:`running`防火墙开启；`dead`防火墙未开启

```shell
systemctl status firewalld
```

查看端口是否开启

```shell
firewall-cmd --query-port=666/tcp
```

添加永久开放端口

```shell
firewall-cmd --add-port=2888/tcp --permanent
```

重新载入配置（刷新配置）

```shell
firewall-cmd --reload
```

移除端口

```shell
firewall-cmd --permanent --remove-port=666/tcp
```

