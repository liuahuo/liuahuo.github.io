---
layout: post
title: docker的离线安装
category: docker
---
网络上有很多docker安装的教程，大多都是在线安装、一键搞定的。因为网络环境问题，需要离线安装docker，成功后分享下过程，方便学习。  
#### 1.准备工作 
 
- 操作系统  
docker需要64位的内核版本3.10以上的操作系统支持。  
          查看操作系统位数：getconf LONG_BIT  
          查看内核版本：uname -r

- 依赖组件  
docker容器资源管理使用cgroups(control groups)，所以需要在安装docker前下载安装cgroups组件
          推荐一个下载rpm的网站：http://pkgs.org/（下面的所有rpm包都可以在这个网站下载）  
          需要下载以下4个rpm：  
![cgroup]({{site.baseurl}}/images/docker-install/docker-depentence.png)
          
- docker引擎rpm

另外还需要下载：

- febootstrap工具

 febootstrap是制作镜像的工具。需要下载以下7个rpm:
 ![febootstrap1]({{site.baseurl}}/images/docker-install/febootstrap1.png)  
 ![febootstrap2]({{site.baseurl}}/images/docker-install/febootstrap2.png)  
 ![febootstrap1=3]({{site.baseurl}}/images/docker-install/febootstrap3.png)  
 ![febootstrap4]({{site.baseurl}}/images/docker-install/febootstrap4.png)  
#### 2.安装
 将依赖组件、docker引擎、febootstrap的rpm包依次安装。 
安装完成后，输入docker命令，即可查看docker所有的命令（截图有限，没有截到全部的命令）  

 ![docker command]({{site.baseurl}}/images/docker-install/docker-command.png) 