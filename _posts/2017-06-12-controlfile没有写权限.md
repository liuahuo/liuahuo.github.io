---
layout: post
title: controlfile 没有写权限
category: oracle
---

某数据库down了(服务器操作系统为linux)，查看trace文件，发现报错controlfile没有写权限。在操作系统查看controlfile权限，发现oracle用户确实没有写权限，使用chmod命令赋予权限，报错：read-only file system。  
冷备份数据库文件后，重启操作系统，重启数据库，一切恢复。  
这种问题一般是因为服务器外界被迫掉电，操作系统对掉电的某种保护措施。应避免服务器被迫掉电情况。
