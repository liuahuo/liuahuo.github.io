---
layout: post
title: 制作docker image
categories: docker
comment: true

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

![bulid image]({{sit.baseurl}}/images/make-docker-image/build-image.png)

在centos7-image目录下，执行build命令，centos7-ssh 是生成的image名字，后面跟Dockfile所在目录。
此时，我们可以看到image多了一个centos7-ssh的镜像。

![docker image]({{sit.baseurl}}/images/make-docker-image/docker-image.png)

- 验证ssh登录 

 1.docker run -d centos-ssh:latest /bin/bash  
   运行centos-ssh镜像生成一个容器，-d表示在后台运行。  
 2.此时，使用docker ps -a 命令可以看到启动的容器。  
 ![docker ps]({{sit.baseurl}}/images/make-docker-image/docker-ps.png)  
 3.使用docker inspect CONTAINER ID查看容器的信息
 ![docker inspect]({{sit.baseurl}}/images/make-docker-image/docker-inspect.png)  
 ![docker inspect ip]({{sit.baseurl}}/images/make-docker-image/docker-inspect-ip.png)    
 找到IP地址  
4.使用ssh root@172.17.0.2登录容器，密码是dockfile中修改的redhat  
具有其他功能的image也是同样的方法制作，验证。 
 
### 3.用容器快速制作镜像
为了更好的理解和使用docker镜像、容器，决定使用base image 安装oracle后，打包容器制作为oracle镜像。使用容器制作镜像的好处有:  
     第一就是可以步步为营。当你制作一个比较复杂点的镜像时，不可能一步就能做成功，所以当你觉得下面一步可能要出错时（不可逆的错误），可以先把该镜像打个包，如果接下来失误了，删掉这个容器，再用刚才打包好的镜像做个容器，继续前面的步骤。  
        第二就是不管你在容器中怎么折腾都没关系，大不了删除掉这个容器，如果你在物理服务器上就要相当注意了，要不然duang的一下，整个服务器就瘫痪了。  
        第三好处和第一个差不多，因为有些数据库不能测试的，你一测试就会产生很大的数据（日志，还有些默认的数据）（我做mongodb镜像时，测试了下打包后的镜像竟然达到4个多G，而没测试的就几百MB，相差太大了），因为docker镜像不能太大，否则不好上传。所以一般是做好了，先打个包成镜像，然后接着测试下，如果成功。那就可以了。  
     理想很丰满，显示很骨干，在安装oracle过程中，报错啦。
     
  ![oracle error]({{sit.baseurl}}/images/make-docker-image/oracle-error.png) 
 
 这个报错是因为，oracle安装对hostname有要求，要求hostname中不能包含数字，但是docker在运行镜像为容器的过程中会自动为容器分配一个id，这个id是由数字和字母组成的。无法使用一般修改hostname的办法修改容器的hostname，看到网上有使用docker-compose工具修改容器hostname的，我没有尝试(恩，是因为懒，如果尝试了再来更新)。ps：docker run —add-host这个并不是修改hostname啊~  
     虽然没有成功使用容器制作oracle镜像，但是理论上使用容器制作镜像的语句为：  
     docker commit  xxx(容器的id)   xxxx（要制作成的镜像名）  
     最后使用docker search oracle查找了下oracle镜像，  
      ![docker search]({{sit.baseurl}}/images/make-docker-image/docker-search.png)  
然后 docker pull docker.io/wnameless/oracle-xe-11g，下载完成后可以看到镜像列表中添加了一个oracle-xe-11g的镜像。  
![docker images]({{sit.baseurl}}/images/make-docker-image/docker-images.png)   
运行该镜像为容器，进入容器后，su - oracle 切换到oracle用户，可以看到oracle的安装目录，可以运行sqlplus / as sysdba命令进入到数据库中。

     