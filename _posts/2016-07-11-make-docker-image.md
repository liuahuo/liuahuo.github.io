---
layout: post
title: 制作docker image
category: test
---
### 1.使用febootstrap制作base image

- 制作镜像目录

在Docker入门（http://blog.csdn.net/woshiluahuo/article/details/51838451）中，我们讲到base image，现在要使用febootstrap工具制作base image：

febootstrap -i bash -i wget -i yum -i iputils -i iproute -i man -i vim -i openssh-server -i openssh-clients -i tar -i
      gzip centos7 centos7-image http://mirrors.aliyun.com/centos/7/os/x86_64/

- 1.-i 表示需要安装的package，例如上面的语句：想要base image中安装：bash、wget、yum、iputils、iproute 、man、vim 、openssh-server、openssh-clients、tar 、gzip

这个可以根据自己的需要进行安装，其中openssh-server、openssh-clients这2个包是为了后面ssh登录准备的。

- 2.centos7 是操作系统版本

- 3.centos7-image 是目录（base image想放在服务器的位置）

- 4.源地址，我选择的是阿里云的，也可以使用其他源地址

执行完上面的语句会生成centos7-image文件目录，包含以下文件：

![directory]({{sit.baseurl}}/images/make-docker-image/directory.png)

root目录下是空的，如果这时制作出来的镜像使用ssh登录，会直接进入根目录下，而一般镜像都是进入root目录下的，          
所以执行：cd centos7-image && cp etc/skel/.bash* root/语句，将.bash_logout  .bash_profile  .bashrc这三个文件拷贝至root文件夹下。

- 生成base image

执行：cd centos7-image && tar -c .|docker import - centos7-base 语句，将centos7-image目录下内容打包并将打包内容创建为一个docker镜像。
查看镜像，发现镜像中有一个名为：centos7-base的镜像

![base image]({{sit.baseurl}}/images/make-docker-image/base-image.png)

此时，我们已经可以将centos7-base镜像运行在docker的container中了。

![run base image]({{sit.baseurl}}/images/make-docker-image/run-base-image.png)

### 2.使用Dockfile制作可以使用ssh登录的image

- dockfile

以我们刚刚制作的centos7-base作为base image，使用dockfile制作可以使用ssh登录的image。创建名为：Dockfile的文件，内容如下：  
 #Dockerfile   
FROM centos7-base  #将centos7-base作为base image，类似继承的概念，制作出的新的image具有base image的功能  
MAINTAINER ahuo  
RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key  
RUN ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsh_key  
RUN sed -ri 's/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g' /etc/pam.d/sshd  
RUN mkdir -p /root/.ssh && chown root.root /root && chmod 700 /root/.ssh  
 #上面几行都是配置ssh登录目录和登录验证的，而ssh的安装时在base image centos7-base中完成的(-i openssh-server -i openssh-client)  
EXPOSE 22 #开端口  
RUN echo 'root:redhat' | chpasswd #重置root密码为redhat  
RUN yum install tar gzip gcc vim wget -y  
ENV LANG en_US.UTF-8  
ENV LC_ALL en_US.UTF-8  
CMD /usr/sbin/sshd -D  
 #End