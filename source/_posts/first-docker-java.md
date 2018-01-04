---
title: linux下使用docker部署第一个Java应用
date: 2017-12-14 10:50:08
tags:
  - docker
  - linux 
categories: 
  - Linux服务器系列学习
---

## 前言

>Docker是一个划时代的开源项目，它彻底的释放了计算虚拟化的威力，极大的提高了应用的运算效率，降低了云计算资源供应的成本！无论是应用开发者、运维人员、还是其他信息技术从业人员，都有必要认识和掌握 Docker，节约有限的时间。
>来自于：[Docker —— 从入门到实践](https://www.gitbook.com/book/yeasy/docker_practice/details) 

根据自己的学习和记录，我就讲自己使用docker过程当中遇到的问题以及技巧记录下来，以便自己日后学习温故，同时也可以方便大家共同进步。

## 安装docker

### 安装前的检查

1. 系统要求
    * 64位系统
    * 内核版本3.10+
2. 检查内核版本，返回值大于3.10就行，若不是自行升级系统
```java
    [root@iZj1fkye8uu7o0Z ~]# uname -r
    3.10.0-693.2.2.el7.x86_64
```
3. 使用sudo或者root权限的用户登入终端
4. 确保yum是最新的 `yum update`

### 开始安装docker

1.  使用`yum`安装docker,`yum install -y docker` 安装详细过程如下
```java
[root@iZj1fkye8uu7o0Z ~]# yum install -y docker
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package docker.x86_64 2:1.12.6-61.git85d7426.el7.centos will be installed
--> Processing Dependency: docker-common = 2:1.12.6-61.git85d7426.el7.centos for package: 2:docker-1.12.6-61.git85d7426.el7.centos.x86_64
--> Processing Dependency: docker-client = 2:1.12.6-61.git85d7426.el7.centos for package: 2:docker-1.12.6-61.git85d7426.el7.centos.x86_64
--> Running transaction check
---> Package docker-client.x86_64 2:1.12.6-61.git85d7426.el7.centos will be installed
---> Package docker-common.x86_64 2:1.12.6-61.git85d7426.el7.centos will be installed
--> Finished Dependency Resolution
Dependencies Resolved
================================================================================================================================================
Package                         Arch                     Version                                                Repository                Size
================================================================================================================================================
Installing:
docker                          x86_64                   2:1.12.6-61.git85d7426.el7.centos                      extras                    15 M
Installing for dependencies:
docker-client                   x86_64                   2:1.12.6-61.git85d7426.el7.centos                      extras                   3.4 M
docker-common                   x86_64                   2:1.12.6-61.git85d7426.el7.centos                      extras                    80 k
Transaction Summary
================================================================================================================================================
Install  1 Package (+2 Dependent packages)
Total download size: 18 M
Installed size: 63 M
Downloading packages:
(1/3): docker-client-1.12.6-61.git85d7426.el7.centos.x86_64.rpm                                                          | 3.4 MB  00:00:00     
(2/3): docker-common-1.12.6-61.git85d7426.el7.centos.x86_64.rpm                                                          |  80 kB  00:00:00     
(3/3): docker-1.12.6-61.git85d7426.el7.centos.x86_64.rpm                                                                 |  15 MB  00:00:01     
------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                            12 MB/s |  18 MB  00:00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 2:docker-common-1.12.6-61.git85d7426.el7.centos.x86_64                                                                       1/3
  Installing : 2:docker-client-1.12.6-61.git85d7426.el7.centos.x86_64                                                                       2/3
  Installing : 2:docker-1.12.6-61.git85d7426.el7.centos.x86_64                                                                              3/3
  Verifying  : 2:docker-1.12.6-61.git85d7426.el7.centos.x86_64                                                                              1/3
  Verifying  : 2:docker-common-1.12.6-61.git85d7426.el7.centos.x86_64                                                                       2/3
  Verifying  : 2:docker-client-1.12.6-61.git85d7426.el7.centos.x86_64                                                                       3/3
Installed:
  docker.x86_64 2:1.12.6-61.git85d7426.el7.centos                                                                                               
Dependency Installed:
  docker-client.x86_64 2:1.12.6-61.git85d7426.el7.centos                 docker-common.x86_64 2:1.12.6-61.git85d7426.el7.centos                
Complete!

```

2. 查看上面使用`yum`安装docker是否成功并且已经启动了，查看`docker version` 显示`docker`的`client`和`server`的信息如下：

```java
[root@iZj1fkye8uu7o0Z ~]# docker version
Client:
Version:         1.12.6
API version:     1.24
Package version: docker-1.12.6-61.git85d7426.el7.centos.x86_64
Go version:      go1.8.3
Git commit:      85d7426/1.12.6
Built:           Tue Oct 24 15:40:21 2017
OS/Arch:         linux/amd64
Server:
Version:         1.12.6
API version:     1.24
Package version: docker-1.12.6-61.git85d7426.el7.centos.x86_64
Go version:      go1.8.3
Git commit:      85d7426/1.12.6
Built:           Tue Oct 24 15:40:21 2017
OS/Arch:         linux/amd64
```
如果显示以上信息 ，说明已经成功安装`docker`，同时`docker`的`client`和`server`都已经成功运行了。

那么问题来了，成功运行了`docker`，但是里面没有镜像，我们如何才能够在其中运行自己的应用程序呢。下面就来介绍如何使用[网易蜂巢](https://c.163.com/#/m/home/)的镜像中心来安装我们的软件。
> 也可以使用`docker`的[镜像中心](https://hub.docker.com/),只不过在国内使用这个速度非常慢，我是用的是`网易镜像中心`，读者们也可以使用[阿里镜像中心](https://dev.aliyun.com/search.html)或者其他的都行。

## 安装镜像

### 使用网易蜂巢镜像中心安装所需镜像

#### 安装Tomcat

1. 安装`Tomcat`，我们只需要在里面搜索`tomcat`就行了。其实我们平时使用的过程中就知道，如果想让`tomcat`跑起来就必须安装`Java`环境。但是在这里使用`docker`镜像安装`Tomcat`就已经包含`Java`环境了，无需另外安装。

2. 按照上面的网址进入`网易蜂巢镜像中心`后，**需要登录才会进入到入口**，然后搜索`Tomcat`就行了。
![image](http://osal57kgi.bkt.clouddn.com/wangyi_fengchao.png)
在镜像中心上搜索`Tomcat`会发现第一个`tomcat`会有一个`docker`鲸鱼的标志，这说明这个镜像资源和官方[docker镜像](https://hub.docker.com/)保持同步的。那我们点进去，在右边就会有一个**下载地址**的链接。
```java
docker pull hub.c.163.com/library/tomcat:latest
```
这个就是我们需要在终端上面进行`git`拉取的仓库地址。`docker`命令标志，`pull`git命令熟悉的话知道这是下载拉取的意思，`hub.c.163.com/library/`仓库地址，`tomcat`是镜像名称，`:`后面的`latest`为`tag`标签，表示当前拉取的是最新版本，如果需要其他版本的也可以在这个页面的下方寻找`tomcat`镜像的其他版本。

3. 根据上面的地址镜像下载`tomcat进项`,操作命令记录如下：
  - 下载镜像
    ```java
    [root@iZj1fkye8uu7o0Z ~]# docker pull hub.c.163.com/library/tomcat:latest
    Trying to pull repository hub.c.163.com/library/tomcat ...
    latest: Pulling from hub.c.163.com/library/tomcat
    9af7279b9dbd: Pull complete
    31816c948f2f: Pull complete
    c59a1cdf83d3: Pull complete
    232c7a75d568: Pull complete
    de412d312979: Pull complete
    80315ba34693: Pull complete
    5d3f97bd90e8: Pull complete
    dc8dc63f6baa: Pull complete
    f6c6e2d67f03: Pull complete
    9123b340aa92: Pull complete
    76abaea2279d: Pull complete
    4476602e3346: Pull complete
    12e1fda011bd: Pull complete
    Digest: sha256:db1a8ca2fe44449d265e5505f300be6f34fc63211a5506400a0a8c24653af91f
    ```
最后面的这个` sha256:db1a8ca2fe44449d265e5505f300be6f34fc63211a5506400a0a8c24653af91f`表示改镜像在仓库当中的地址编号。

  - 查看当前docker当中的镜像 `docker images`,操作记录如下：
    ```java
    [root@iZj1fkye8uu7o0Z ~]# docker images
    REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
    hub.c.163.com/library/tomcat   latest              72d2be374029        3 months ago        292.4 MB  
    ```

4. 运行下载的镜像

```java
[root@iZj1fkye8uu7o0Z ~]# docker run -d -p 80:8080 hub.c.163.com/library/tomcat
80e53db67f4bfb711ab3491f49dece1f7f9aa782e7c8d95153f36e39b6940e65
[root@iZj1fkye8uu7o0Z ~]# docker ps
CONTAINER ID        IMAGE                          COMMAND             CREATED             STATUS              PORTS                  NAMES
80e53db67f4b        hub.c.163.com/library/tomcat   "catalina.sh run"   16 seconds ago      Up 15 seconds       0.0.0.0:80->8080/tcp  evil_leakey
```

上面这个运行命令 `docker run -d -p 80:8080 hub.c.163.com/library/tomcat` 说明如下
`docker run`表示运行， `-d`表示后台运行*docker 分为前台运行，后台运行，前台运行可以通过Ctrl+C进行停止* `-p`表示制定宿主机和容器之间的端口映射，在这里`-p 80:8080`表示将宿主机`80`端口映射到容器的`8080`端口。
> 这里也可以使用`-P`*大写*来分配端口，表示宿主机将随机分配端口映射到容器端口，容器端口不变。操作如下：

```java
[root@iZj1fkye8uu7o0Z ~]# docker run -d -P hub.c.163.com/library/tomcat
965a99cdb70730716aa466ddefe28c55fb7161e9030f8d42d0055711b26c7b25
[root@iZj1fkye8uu7o0Z ~]# docker ps
CONTAINER ID        IMAGE                          COMMAND             CREATED             STATUS              PORTS                     NAMES
965a99cdb707        hub.c.163.com/library/tomcat   "catalina.sh run"   4 seconds ago       Up 3 seconds        0.0.0.0:32768->8080/tcp   kickass_swirles
80e53db67f4b        hub.c.163.com/library/tomcat   "catalina.sh run"   5 minutes ago       Up 5 minutes        0.0.0.0:80->8080/tcp      evil_leakey
```

上面通过命令`docker ps`查看当前正在运行的容器，可以看出 `Tomcat`正在运行的有两个，而且两个的端口号还不一样，`80`是我们上面`-p`指定的，`32768`是通过`-P`随机分配的。而且`localhost:80`和`localhost:32768`都是可以访问到`tomcat`的首页的。

5. 暂停容器

在上面提到了使用命令`docker ps`可以查看当前正在运行的容器。那么我们可以使用 `docker stop CONTAINER ID`进行停止正在运行的容器，比如上面的`80`端口的容器的`CONTAINER ID `是`80e53db67f4b`,现在想将其停止就可以使用`docker stop`命令了，而且这个`CONTAINER ID`还可以简写，操作记录如下：

```java
[root@iZj1fkye8uu7o0Z ~]# docker stop 80
80
[root@iZj1fkye8uu7o0Z ~]# docker ps
CONTAINER ID        IMAGE                          COMMAND             CREATED             STATUS              PORTS                     NAMES
965a99cdb707        hub.c.163.com/library/tomcat   "catalina.sh run"   6 minutes ago       Up 6 minutes        0.0.0.0:32768->8080/tcp   kickass_swirles
```

通过`docker ps`可以很清楚的看到当前只剩下端口号为`32768`的正在运行。同时地址栏当中也已经无法使用`localhost:80`进行访问`Tomcat`的首页了。

到现在为止，我们就已经在`docker`当中安装好了我们的第一个应用， 并且成功运行了。有了`Tomcat`我们就可以运行自己的Web应用了，那么问题来了，数据库怎么办呢？下面我就来说说怎么安装数据库吧，其实答题的操作都差不多，镜像中心找到`MySQL`的拉取地址，然后终端运行下载镜像，运行，映射端口。

#### 安装MySQL

1. 同样进入网易蜂巢镜像中心，搜索`MySQL`，找到其下载地址。`docker pull hub.c.163.com/library/mysql:latest`

2. 进入`linux`终端，输入上面下载地址的命令，详细过程如下：

    ```java
    [root@iZj1fkye8uu7o0Z ~]# docker pull hub.c.163.com/library/mysql:latest
    Trying to pull repository hub.c.163.com/library/mysql ...
    latest: Pulling from hub.c.163.com/library/mysql
    42cb69312da9: Pull complete
    e2cf5467c4b5: Pull complete
    871ec0232f66: Pull complete
    3c0ae7ec690d: Pull complete
    d39b43089b70: Pull complete
    aa0e7cb4b67c: Pull complete
    738db9902d06: Pull complete
    ae333863ac05: Pull complete
    6d014992204a: Pull complete
    09aeca0c9a82: Pull complete
    0162083b2de0: Pull complete
    Digest: sha256:b2bce1a792237ac5df78877d583f34b09ab023a77130921a6bcce67ce2d24ff0
    ```

3. `docker run`命令运行下载好的`MySQL`镜像。可能运行`MySQL`和`Tomcat`不太一样，在这里我们不止需要制定映射端口，还需要为`MySQL`指定访问的用户名和密码。

```java
[root@iZj1fkye8uu7o0Z ~]# docker run --name root -e MYSQL_ROOT_PASSWORD=123456 -d -p 3306:3306 hub.c.163.com/library/mysql
0e00273b42da6c30ca21798c0fed807cc91d758b08087324668338172b6eb62d
```

如上面命令所示，用户名为`root`，密码为`123456`,我们可以用`navicate`测试是否连接上。
上面命令参数的说明详细见网易蜂巢镜像文档当中。[详细说明](https://c.163.com/hub#/m/repository/?repoId=2955) 关于如何创建用户名，如何跨容器连接MySQL等

```java
How to use this image

Start a mysql server instance

Starting a MySQL instance is simple:

$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
... where some-mysql is the name you want to assign to your container, my-secret-pw is the password to be set for the MySQL root user and tag is the tag specifying the MySQL version you want. See the list above for relevant tags.
```

### 使用镜像

####  进入容器

在刚刚我们创建了`Tomcat`和`MySQL`的容器，那么如何进入到这些正在运行的容器当中呢。可以使用命令`docker ps`查看正在运行的容器。然后使用命令`docker exit -it [NAMES] bash` 指定需要进入的具体容器是哪一个。这个参数`-it`表示使用**交互模式**运行，同时返回一个可以操作的终端，这里的`NAMES`就是`docker ps`查看到的容器运行时候最后一个名字。操作记录如下：

```java
[root@iZj1fkye8uu7o0Z ~]# docker ps
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                    NAMES
5fea4d975b62        hub.c.163.com/library/tomcat   "catalina.sh run"        16 seconds ago      Up 15 seconds       0.0.0.0:8080->8080/tcp   hungry_noether
ed96dabaff1f        hub.c.163.com/library/mysql    "docker-entrypoint.sh"   7 minutes ago       Up 7 minutes        0.0.0.0:3306->3306/tcp   root
8c93f7088b22        my-nginx                       "nginx -g 'daemon off"   46 minutes ago      Up 46 minutes       0.0.0.0:80->80/tcp       adoring_hopper
```

上面很清楚的显示`MySQL`的`NAMES`是`root`，那么我们就可以使用命令`docker exec -it root bash`来进入容器，记录如下

```java
docker exec -it root bash
```

但是此时只是进入了容器，还需要进入`MySQL`怎么操作呢，跟我们平常在`Linux`上一样操作，`mysql -uroot -p`操作记录如下：

```java
[root@iZj1fkye8uu7o0Z ~]# docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS              PORTS                    NAMES
b7ae64383c2c        tomcat:mydocs2                "catalina.sh run"        6 days ago          Up 6 days           0.0.0.0:80->8080/tcp     prickly_borg
87680cea7eb4        hub.c.163.com/library/mysql   "docker-entrypoint.sh"   6 days ago          Up 6 days           0.0.0.0:3306->3306/tcp   root
[root@iZj1fkye8uu7o0Z ~]# docker exec -it root bash
root@87680cea7eb4:/# mysql -uroot -p                                                                                                                                                          
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 108
Server version: 5.7.18 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```

现在就可以愉快的玩耍了。

#### 退出容器

1. 如果使用上面`docker exec -it NAMES bash`方式进入容器，可以直接使用`exit`，退出容器，而不会将其停止。

## 后记

通过学习`linux`上部署`docker`，平时需要独立安装的各种环境变得简单，同时也可以自行通过`Dockerfile`文件来`build`自己的镜像，然后`pash`到仓库，或者厂商的镜像中心当中，以便日后再用，都是非常方便的。同时也可以在一台宿主机当中部署多个`docker`，实现集群服务也是可以的。当然我们也可以使用`Web`端的工具来管理`docker`,相应的工具名称大家可以自行搜索使用。


## 联系

[聪聪](https://ccoder.cc/)的独立博客 ，一个喜欢技术，喜欢钻研的95后。如果你看到这篇文章，千里之外，我在等你联系。

- [Blog@ccoder's blog](https://ccoder.cc/)
- [CSDN@ccoder](http://blog.csdn.net/chencong3139)
- [Github@ccoder](https://github.com/chencong-plan)
- [Email@ccoder](mailto:admin@ccoder.top) *or* [Gmail@ccoder](mailto:chencong3139@gmail.com)
