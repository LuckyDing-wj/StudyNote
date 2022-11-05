[TOC]

# centOS7安装docker

```shell
yun install -y docker

# docker --help
```

# 镜像

## 1. 远程仓库

 https://hub.docker.com

## 2. 搜索镜像，例如Redis

## 3. 拉取镜像

```shell
docker pull redis:3.0
```

## 4. 镜像加速

## 5. 注册阿里云，并登录到控制台

## 6. 产品与服务，找到容器服务

## 7. 选择镜像，查看各种镜像

## 8. 页面右上角镜像仓库控制台，选择镜像加速器

​	阿里云镜像搜索地址：https://dev.aliyun.com/search.html

## 9. 操作本地镜像

### 9.1 查询本地镜像

```shell
docker images
```

### 9.2 删除镜像

删除镜像之前，需要删除所有用到该镜像的容器

```shell
docker rmi 'image id'
# 强制删除，但是如果有容器正在使用该镜像，则不会真正的删除
docker rmi -f 'iamge id'
```

### 9.3 搜索镜像

```shell
docker search 'image'
```

## 10. 容器

### 10.1 查询

```shelle
# 查询正在运行的容器列表
docker ps 

# 查询所有容器
docker ps -a
```

### 10.2 创建

```shell
docker create
# 创建 redis 容器 demo
docker create -p 16379:6379 --name redis redis:3.0

# 查看容器，获取id，启动容器
docker start '容器id'
```

### 10.3 启动redis容器

```shell
# 创建容器之后，启动容器，docker start并不常用，常用docker run, 创建并启动容易
docker run --help
# 启动redis deomo，这样启动在 ctrl+c 会停止这个容器
docker run -p 16380:6379 --name redis2 redis:3.0
# 让容器在后台运行，添加 -d 参数
docker run -p 16380:6379 -d --name redis2 redis:3.0
```

### 10.4 停止容器

- docker stop '容器名或id'
- docker kill '容器名或id'

### 10.5 删除容器

+ docker rm --help

### 10.6 进入容器

* docker exec --help
* 进入redis容器： docker exec -it redis /bin/bash
* ctrl + d 推出容器

###   10.7 查看容器日志

+ docker logs -f '容器名或id'

# 仓库

集中存放镜像的地方，可以上传自己的镜像。

## 1. 创建阿里云镜像仓库

个人实例、仓库管理、镜像仓库、创建镜像仓库

## 2. 推送redis镜像到阿里云仓库

+ 在docker中登录

  ```shelle
  docker login --username=aliyunemail registry.cn-hangzhou.aliyuncs.com
  ```

+ 给镜像打tag，需要用到第一步中创建的镜像地址

  ```shell
  docker tag redis:3.0 registry.cn-hangzhou.aliyuncs.com/wangjie/redis:3.0
  ```

+ 推动镜像

  ```shell
  docker push registry.cn-hangzhou.aliyuncs.com/wangjie/redis:3.0
  ```

+ 可在阿里云镜像仓库中查看

  ```shell
  # 删除本地仓库镜像
  docker rmi registry.cn-hangzhou.aliyuncs.com/wangjie/redis:3.0
  # 拉去阿里云镜像
  docker pull registry.cn-hangzhou.aliyuncs.com/wangjie/redis:3.0
  ```

# 数据管理

容器在运行项目时会产生数据，比如运行的mysql容器，那么一定会有数据的产生，那么问题来了，数据是保存在容器内部还是保存在外部？

如果将数据保存在内部，那么也就意味着我们改变了原有镜像，这种做法是不可取的，因为在后期的镜像升级将变得不可能了。*也就是说，运行的镜像，最好不要改变，如果必须改变的（比如说，修改配置文件等），在改变后记得commit提交打成一个新的镜像*。

显然，数据是应该保存在容器的外部，也就是说保存在主机上。那么问题又来了，数据保存在主机上，那么容器该如何读取主机中的数据呢？

## 数据卷

在create或者run容器时，可以通过-v参数指定主机的目录，挂载到容器中的某一个目录上，这样，容器就在这个目录读写数据了。从而实现了容器和数据的分离。

### Demo 

运行mysql（percona）容器，将mysql的数据放到主机的/data/mysql-data中

```shell
# 下载mysql镜像
docker pull percona::5.6

# 创建容器
dcoker create --name percona -v /data/mysql-data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root percona:5.6
# --name percona 指定是容器的名称
# -v /data/mysql-data:/var/lib/mysql 将主机目录/data/mysql-data挂载到容器的/var/lib/mysql上
# -p 33306:3306 设置端口映射，主机端口是33306，容器内部端口3306
# -e MYSQL_ROOT_PASSWORD=root  设置容器参数，设置root用户的密码为root
# percona:5.6 镜像名:版本

# 启动容器
docker start percona
# 链接测试
```

## 构建镜像

前面我们的学习都是直接从仓库中拉取镜像，然后创建容器，最后启动容器来使用的。

在实际开发过程中，仓库中的容器可能不能完全满足我们的需求，比如说，我们项目的部署到docker容器，就不能从仓库中直接拉取镜像，就需要自己构建镜像了。

构建镜像通过编写Dockerfile配置文件完成。

### DockerFile 文件

DockerFile是一个文本文件，在文件中编写多条命令，描述一个镜像构建的细节。

文件分为四部分：基础镜像信、维护者信息、镜像操作指令和容器启动时执行的指令。

例如：

```shell
#第一行必须指令基于的基础镜像
FROM ubutu

#维护者信息
MAINTAINER docker_user  wangjie@163.com

#镜像的操作指令
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y ngnix 
RUN echo "\ndaemon off;">>/etc/ngnix/nignix.conf

#容器启动时执行指令
CMD /usr/sbin/ngnix
```

### 命令详解

#### FROM

格式为 FROM <image> 或 FROM <image>:<tag>

第一条指令必须为FROM指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令（每个镜像一次）。

#### MAINTAINER

格式为 MAINTAINER <name>, 指定维护者信息。

#### RUN

格式为 RUN <command> 或 RUN ["executable", "param1", "param2"]

前者将在shell终端中运行命令，即 /bin/sh -c；后者则使用exec执行。指定使用其他终端可以通过第二种方式实现，例如 RUN ["/bin/bash", "-c", "echo hello"]

每条RUN指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令过长时可以使用\来换行。

#### CMD

三种格式：

+ CMD ["executable", "param1", "param2"] 使用exec执行，推荐方式
+ CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用
+ CMD ["param1", "param2"] 提供给ENTRYPOINT 的默认参数

指定启动容器时执行的命令，每个Dockerfile只能由一条CMD命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时指定了运行时的命令，则会覆盖掉CMD指定的命令。

#### EXPOSE

格式为 EXPOSE <port> [<port>...]

```shell
EXPOSE 22 80 8443
```

告知Docker服务器容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -P， Docker主机会自动分配一个端口转发到指定的端口；使用-p，则可以具体指定哪个本地端口映射过来。

#### ENV

格式为 ENV <key> <value>。

指定一个环境变量，会被后续RUN指令使用，并在容器运行时保持。例如：

```shell
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/sr/postgrss && ...
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

#### ADD

格式为： ADD <src> <dest>

该命令将复制指定的<src>到容器中的<dest>。其中<src>可以是Dockerfile所在目录的一个相对路径（文件或目录）；也可以事一个URL； 还可以是一个tar文件（自动解压为目录）。

#### COPY

格式为 COPY <src> <dest>

复制本地主机的<src> (为Dockerfile所在目录的相对路径，文件或目录)为容器中的<dest>。目标路径不存在时，会自动创建。

当使用本地目录为源目录时，推荐使用COPY。

#### ENTRYPOINT （入口）

有两种格式:

+ ENTRYPOINT ["executable", "param1", "param2"]
+ ENTRYPOINT command paraml param2 (shell中执行)。

配置容器启动后执行的命令，并且不可被docker run 提供的参数覆盖。

每个Dockerfile中只能有一个ENTRYPOINT,当指定多个ENTRYPOINT时，只有最后一个生效。

#### VOLUME （挂载）

格式为VOLUME  [" /data"]

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的
数据等。

#### USER

格式为USER daemon

指定运行容器时的用户名或UID,后续的RUN也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如: 

```shell
RUN groupadd -r postgres && useradd -r -g postgres postgres
```

要临时获取管理员权限可以使用gosu,而不推荐sudo。

#### WORKDIR

格式为WORKDIR /path/ to/workdir

为后续的RUN、CMD、 ENTRYPOINT 指令配置工作目录。

可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如：

```shell
WORKDIR /a
WORKDIR b
WORKDIR C
RUN pwd

# 最终路径为 /a/b/c
```

#### ONBUILD

格式为 ONBUILD [INSTRUCTION]

配置当所创建的镜像作为其他新创建镜像的基础镜像时，所执行的操作指令。

例如，Dockerfile使用如下的内容创建了镜像image-A。

```shell
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/1oca1/bin/python-build --dir /app/src
[...]
```

如果基于image-A创建新的镜像时，新的Dockerfile中使用FROM image-A 指定基础镜像时，会自动执行ONBUILD指令内容，等价于在后面添加了两条指令。

```shell
FROM image-A

#Automatically run the following
ADD . /app/src 
RUN /usr/1ocal/bin/python-build --dir /app/src !
```

使用ONBUILD指令的镜像，推荐在标签中注明，例如ruby:1.9-onbuild

## Demo 构建redis镜像

Dockerfile如下：

```shell
# 构建Redis镜像
# itcast

#基于Centos6.6构建
FROM centos:6.6

#安装依赖
RUN yum -y install tar cpp binutils glibc glibc-kernheaders glibc-common glibc-devel gcc make gcc-c++ libstdc++-devel tcl

#创建安装目录
RUN mkdir -p /redis/data &&  cd /redis

#拷贝redis的安装包
COPY ./redis-3.0.2.tar.gz /redis

#解压
RUN cd /redis && tar -xvf redis-3.0.2.tar.gz && rm -rf redis-3.0.2.tar.gz && cd redis-3.0.2

#编译、安装
RUN cd /redis/redis-3.0.2 &&  make && make install

#复制配置文件到/redis中，并且修改redis为后台运行
RUN cp /redis/redis-3.0.2/redis.conf /redis/ && echo "daemonize yes" >> redis.conf

#设置数据挂载目录以及工作目录
VOLUME /redis/data
WORKDIR /redis/data

#容器启动后执行该命令
ENTRYPOINT ["/usr/local/bin/redis-server", "/redis/redis.conf"]

#设置对外的端口号
EXPOSE 6379
```

构建命令：

```shell
docker build -t registry.cn-hangzhou.aliyuncs.com/wangjie/redis:my-3.0 /tmp/build-redis-docker-image/

# 查看是否构建完成
docker images
```

创建容器：

```shell
docker create -t --name myRedis -p 26379:6379 registry.cn-hangzhou.aliyuncs.com/wangjie/redis:my-3.0

# 启动容器
docker start myRedis
```

上传镜像到阿里云

```shell
docker push registry.cn-hangzhou.aliyuncs.com/w/redis:my-3.0
```

