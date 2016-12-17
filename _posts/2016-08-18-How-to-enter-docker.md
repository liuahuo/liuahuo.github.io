---
layout: post
title: 进入docker容器的方法
category: docker

---

|  登陆容器的方式 | ssh登录|第三方工具(nsenter、nsinit)|docker提供的工具(attach、exec)|
| ------------- |:-------------:| -----:| -----: |
| 优点 | 符合平时登录服务器习惯，不用额外学习| 使用方便快捷 | 使用方便快捷     |
| 缺点           | 1.秘钥管理2.ssh升级3.监控 | 1.需要学习第三方工具使用规则2.docker宿主机root权限|1.使用attach登录容器，exit退出容器2.docker宿主机root权限3.同屏|
| 适用范围       | 1.适用docker宿主机登录至容器内部2.远程登录该容器       |适用docker宿主机登录至容器内部| 适用docker宿主机登录至容器内部|


## SSH

使用方法： ssh 用户@IP地址 -p 端口
一般ssh登录走22端口，但是在docker中bridge网络模式使用NAT做端口映射，端口需要特殊标注

适用范围：docker宿主机内部登录容器，外部终端直接登录容器

优点：符合管理员、开发者登录服务器的习惯，不需要进行额外的学习

缺点：
- 1.秘钥管理
如果将秘钥写到镜像中，当需要更新秘钥时需要重新制作镜像，部署，重启容器，虽然这个步骤繁琐，但是个人认为还是安全的。如果将秘钥写到卷中，首先要保证该容器没有这个卷的写权限，否则存在秘钥被篡改的风险。
- 2.ssh升级或打补丁
ssh有漏洞或者版本升级时，需要对每一个容器进行打补丁或者升级操作

## nsenter工具

nsenter工具的安装：

wget https://www.kernel.org/pub/linux/utils/util-linux/v2.28/util-linux-2.28.tar.gz

tar -xzvf util-linux-2.28.tar.gz

cd util-linux-2.28
./configure --without-ncurses

make nsenter

cp nsenter /usr/local/bin

安装完成后，使用nsenter —help 命令查看，可以看到使用方法及参数则证明安装成功。

![nsenter-option]({{site.baseurl}}/images/enter-docker/nsenter-option.png)

使用nsenter 命令登录docker 容器

![nsenter-option]({{site.baseurl}}/images/enter-docker/nsenter.png)

在使用nsenter登录docker 容器时，一般使用前6个参数(我看别人都这么写的，想了想原因，可能使用前6个参数就可以涵盖网络，磁盘，进程管理操作系统基本的信息，我尝试只使用其中一个或几个参数登录容器，成功了但是操作系统功能使用上受限制，比如无法使用网络配置功能等)。

## docker attach

使用方法：docker attach  [container name]

适用范围：docker宿主机内部登录容器
          
优点：快捷方便
       
缺点：
- 1.exit后直接退出该container
- 2.多屏同步 这相当于同一时间最多只能有一个终端连接容器

![attach]({{site.baseurl}}/images/enter-docker/attach.png)

![attach same screen]({{site.baseurl}}/images/enter-docker/attach-same-screen.png)

## docker exec
使用方法：docker exec -it [container name] [command]

适用范围：docker宿主机内部登录容器

优点：快捷方便

缺点：外部终端无法使用这种方法登录容器

使用参数介绍：

-i, --interactive               Keep STDIN open even if not attached ————交互

-t, --tty                        Allocate a pseudo-TTY————分配伪终端

一般情况会使用-it这个组合命令，如果单用也只能单独使用-i命令

-i 参数不会产生伪终端，但是会有正确的返回

![attach same screen]({{site.baseurl}}/images/enter-docker/exec.png)

使用-it时，则和我们平常操作console界面类似。而且也不会像attach方式因为退出，导致整个容器退出。这种方式可以替代ssh或者nsenter、nsinit方式，在容器内进行操作。

