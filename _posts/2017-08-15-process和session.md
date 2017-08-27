---
layout: post
title: process和session
category: oracle
---
什么是process  
process：这个参数限制了能够连接到SGA的操作系统进程数(或者是Windows 系统中的线程数)，这个总数必须足够大，从而能够适用于后台进程与所有的专用服务器进程。此外，共享服务器进程与调度进程的数目也被计算在内。此外，共享服务器进程与调度进程的数目也被计算在内。因此，在专用服务器环境中，这是一种限制并发连接数的方法。  
什么是session  
通俗来讲，session 是通信双方从开始通信到通信结束期间的一个上下文(context)。这个上下文是一段位于服务器端的内存：记录了本次连接的客户端机器、通过哪个应用程序、哪个用户在登录等信息[在pl/sql developer中，通过Tools-->Sessions可以查看当前数据库的session]。session 是和connection同时建立的，两者是对同一件事情不同层次的描述。简单讲，connection是物理上的客户机同服务器段的通信链路，session是逻辑上的用户同服务器的通信交互。session被应用于Oracle层次而非操作系统层次.在不考虑通过专用服务器或共享服务器进行登录的情况下,这个参数限制了对指定实例的并发登陆数。  
由上面的分析可知，一个后台进程可能同时对应对个会话，因此通常sessions的值是大于processes的值，通常的设置公式：  
        sessions = 1.1 * processes + 5 

如何修改session与process  

v$session  每一个连接到数据库实例中的session都拥有一条记录。包括用户session及后台进程如DBWR，LGWR，arcchiver等等。

V$process 本视图包含当前系统oracle运行的所有进程信息。常被用于将oracle或服务进程的操作系统进程ID与数据库session之间建立联系。

show parameter sessions  查看当前session配置

show parameter processes   查看当前process配置

alter system set processes=1000 scope=spfile  更改配置，更改完后需要重启数据库。

![process-session的关系]({{site.baseurl}}/images/process-session/process-session.png)
 