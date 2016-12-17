---
layout: post
title: Linux单用户模式及grub使用
category: linux

---
### 对于遗忘root用户密码的解决方法就是，启动时进入单用户模式，重新修改root用户口令。

#### CentOS6.x

在打完oepnssh补丁升级完openssl后，没有验证ssh功能是否恢复正常就将telnet服务关闭了，导致该服务器无法远程。直连服务器发现图形化界面无法正常进入，此时只能使用单用户模式进入服务器进行操作。
     
单用户模式，就是你现在站在这台机器面前能干的活，再通俗点就是你能够接触到这个物理设备。一般干这个活的话，基本上是系统出现严重故障或者root密码忘记等等。
     
使用单用户登录时，我们会使用到一个叫grub的工具。GRUB是Linux系统的自举程序，也就是用来引导系统启动的程序，其主要作用就是指定Linux系统内核的位置，然后指定根目录所在的分区。当Linux系统得到可以运行的内核和根文件系统后，才可以运行，所以启动程序是非常重要的。
   
##### grub工具的使用类似vim编辑器， 使用grub工具进行单用户登录的步骤如下：

- 在开机启动的时候能看到引导目录，用上下方向键选择你忘记密码的那个系统，然后按“e”表示进入编辑模式。

![选择启动系统内核]({{site.baseurl}}/images/linux-singleuser/first.png)


- 编辑原有行，直接移动黑色显示条到改行，然后按“e”，进入对该行的编辑模式

![编辑选中的内核]({{site.baseurl}}/images/linux-singleuser/second.png)

- 输入single或者1后回车，回车后grub回到引导目录的编辑状态

![编辑选中的内核增加single模式]({{site.baseurl}}/images/linux-singleuser/third.png)

- 按“b”，启动系统进入单机模式

![进入single模式]({{site.baseurl}}/images/linux-singleuser/fourth.png)

然后就进入到了单用户模式，可以对操作系统进行一些配置（如打开Telnet,修改root密码等）

#### CentOS7.x

- 开机启动时，选中想要启动的内核，按e进入编辑模式；

![修改内核启动参数]({{site.baseurl}}/images/linux-singleuser/choosekernel.png)

- ro->rw 

原本呢加载内核为ro只读方式，在进入系统后，内核可以被root用户修改
但是单用户模式实际上没有进入系统，而我们要修改参数所以要把权限放开为rw可读可写。

- init=/sysroot/bin/sh 启动后进入shell
- ctl+x启动该内核

![修改内核启动参数]({{site.baseurl}}/images/linux-singleuser/editkernel.png)

- chroot /sysroot
- 在单用户模式下进行操作，例如：修改root密码
- touch /.autorelabel（SELinux开启的时候需要操作这一步）
- exit 退出sysroot
- exec /sbin/reboot或者exec /sbin/init

![修改内核启动参数]({{site.baseurl}}/images/linux-singleuser/singleusermode.png)
