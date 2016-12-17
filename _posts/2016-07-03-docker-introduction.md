---
layout: post
title: docker入门
category: docker
---
1.什么是docker  
     Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。提到Docker人们总会将它列为虚拟化产品，将其类比VMware产品，了解了docker运行原理和container的概念后，你会发现docker不仅可以实现虚拟化，还能应用于以下场景：

- 应用的打包和发布
- 部署和调整数据库或其他应用
- 标准化开发、测试环境

2.什么是container  
![container]({{site.baseurl}}/images/docker-introduction/container.jpg)   
VMs将整个操作系统运行在虚拟的硬件平台上，提供完整的运行环境给应用程序。对比VMs来看Containers，Container和普通的虚拟机Image相比, 最大的区别是它并不包含操作系统内核，也就是省掉了GuestOS这一层，直接在Host上加载运行应用程序。  
     为什么Container能够直接在host上直接加载运行程序呢？这个和linux操作系统特性密切相关。
首先先普及几个基础知识：

- AUFS（Another Union File System）AUFS (AnotherUnionFS) 是一种 Union FS, 简单来说就是支持将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)的文件系统。Docker 在 AUFS 上构建的 container image 就使用了这种特性，接下来从启动 container 中的 linux 为例来介绍 docker 对AUFS特性的运用。

典型的linux启动需要2个FS：bootfs+rootfs。  
bootfs (boot file system) 主要包含 bootloader 和 kernel, bootloader主要是引导加载kernel, 当boot成功后 kernel被加载到内存中后 bootfs就被umount了。  
rootfs (root file system) 包含的就是典型 Linux 系统中的 /dev, /proc,/bin, /etc 等标准目录和文件。  
对于不同的linux发行版, bootfs基本是一致的, 但rootfs会有差别, 因此不同的发行版可以公用bootfs。

- container和image
典型的Linux在启动后，首先将 rootfs 设置为 readonly, 进行一系列检查, 然后将其切换为 "readwrite" 供用户使用。  
在Docker中，初始化时也是将 rootfs 以readonly方式加载并检查，然而接下来利用 union mount 的方式将一个readwrite 文件系统挂载在 readonly 的rootfs之上，并且允许再次将下层的 FS(file system) 设定为readonly 并且向上叠加, 这样一组readonly和一个writeable的结构构成一个container的运行时态, 每一个FS被称作一个FS层。如下图:  
![container]({{site.baseurl}}/images/docker-introduction/container.png)  
得益于AUFS的特性, 每一个对readonly层文件/目录的修改都只会存在于上层的writeable层中。这样由于不存在竞争,多个container可以共享readonly的FS层。 所以Docker将readonly的FS层称作 "image" - 对于container而言整个rootfs都是read-write的，但事实上所有的修改都写入最上层的writeable层中, image不保存用户状态，只用于模板、新建和复制使用。  
![container]({{site.baseurl}}/images/docker-introduction/container-image.png)  
上层的image依赖下层的image，因此Docker中把下层的image称作父image，没有父image的image称作base image。因此想要从一个image启动一个container，Docker会先加载这个image和依赖的父images以及base image，用户的进程运行在writeable的layer中。所有parent image中的数据信息以及 ID、网络和lxc管理的资源限制等具container
的配置，构成一个Docker概念上的container。如下图:  
![container]({{site.baseurl}}/images/docker-introduction/container-image2.png)  

3.container资源管理  
Cgroups（control groups） 实现了对资源的配额和度量。cgroups 的使用非常简单，提供类似文件的接口，在 /cgroup目录下新建一个 文件夹即可新建一个group，在此文件夹中新建task文件，并将pid写入该文件，即可实现对该进程的资源控制。  
在安装docker前，已经安装了cgroup，所以进入到/sys/fs/cgroup目录下，可以看到cgroup可以限制的资源目录：  
[root@localhost cgroup]# pwd  
/sys/fs/cgroup  
[root@localhost cgroup]# ls
blkio  cpu  cpuacct  cpu,cpuacct  cpuset  devices  freezer  hugetlb  memory  net_cls  perf_event  systemd  
groups可以限制blkio、cpu、cpuacct、cpuset、devices、freezer、memory、net_cls、ns九大子系统的资源，以下是每个子系统的详细说明：

    - blkio 这个子系统设置限制每个块设备的输入输出控制。例如:磁盘，光盘以及usb等等。
    - cpu 这个子系统使用调度程序为cgroup任务提供cpu的访问。
    - cpuacct 产生cgroup任务的cpu资源报告。
    - cpuset 如果是多核心的cpu，这个子系统会为cgroup任务分配单独的cpu和内存。
    - devices 允许或拒绝cgroup任务对设备的访问。
    - freezer 暂停和恢复cgroup任务。
    - memory 设置每个cgroup的内存限制以及产生内存资源报告。
    - net_cls 标记每个网络包以供cgroup方便使用。
    - ns 名称空间子系统。

以memory(内存)为例：system.slice目录里已经自动生成了docker相关的文件夹：  
docker-c549b04fd139dc128d9e74dd05c6bc05526194ab3e60137a316b36c3fb04ba78.scope——这个是 docker启动着的container的资源管理目录  
docker.service——这个是docker的资源管理目录  
打开tasks文件，可以看到里面有被管理的进程id号  
![container]({{site.baseurl}}/images/docker-introduction/task.png)   
目录中各个文件的作用如下：  
![container]({{site.baseurl}}/images/docker-introduction/folder-function.png) 
  