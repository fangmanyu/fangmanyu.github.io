# Docker使用

## 安装docker

[Centos7安装教程](https://docs.docker.com/install/linux/docker-ce/centos/)

## docker 添加阿里源

[阿里镜像源地址](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

## push到docker Hub

1. 查看镜像ID

```shell
docker image ls
```

2. 登录 Hub

```shell
docker login
```

3. 打标签

```shell
docker tag <imageID> <namespace>/<imageName>:<version>
```

4. 推送到仓库

```shell
docker push <namespace>/<imageName>
```

---

## commit保存容器

```shell
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

Options:
  -a, --author string    Author (e.g., "John Hannibal Smith
                         <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
```

实例

```shell
docker commit -m "添加fonttools依赖" 952789f31805 fangmanyu/novel_spider:v1.0.1
```

## 路径映射

在运行容器是通过 `-v 本地路径:容器内路径`参数设置路径映射

```shell
docker run -it -v $(pwd)/host-dava:/container-data alpine sh
```

## 拷贝容器内文件

将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。

```
docker cp /www/runoob 96f7f14e99ab:/www/
```

将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。

```
docker cp /www/runoob 96f7f14e99ab:/www
```

将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。

```
docker cp  96f7f14e99ab:/www /tmp/
```

##  docker daemon远程连接设置

[参考CSDN文章](https://blog.csdn.net/qq_37467907/article/details/79537801)



# 常用Dockerfile

## ssh

```shell
FROM centos
MAINTAINER fmy

ENV ROOT_PASSWORD 123456

RUN yum install -y openssh-server

RUN echo $ROOT_PASSWORD | passwd --stdin root

RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D
```

构建

```shell
docker build -t ssh:1.0 .
```

