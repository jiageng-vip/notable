---
attachments: [Clipboard_2022-05-28-16-51-57.png, Clipboard_2022-05-28-16-56-30.png]
tags: [Docker]
title: Docker 部署nginx、php-fpm错误记录
created: '2022-05-28T08:50:32.189Z'
modified: '2022-05-28T08:57:51.460Z'
---

# Docker 部署nginx、php-fpm错误记录
### 遇到问题
问题：docker 部署完成 nginx，php-fpm后，运行服务，出现：recv() failed (104: Connection reset by peer) while reading response header from upstream
![](@attachment/Clipboard_2022-05-28-16-51-57.png)
### 解决历程
- 提示php-fpm默认监听的是127.0.0.1:9000,如果要提供对外服务，需要修改为：listen = 0.0.0.0:9000或者listen=9000
多次修改修改无果后，开始自己查找此错误：
php-fpm一样有工作进程，工作进程的日志貌似默认并没有打印，找到catch_workers_output 选项打开注释，再次访问php页面，这时，错误日志里出现：
![](@attachment/Clipboard_2022-05-28-16-56-30.png)

问题终于找到了，php-fpm默认只允许127.0.0.1的服务进行访问，以前部署nginx，php-fpm时因为同时部署在一台机器上所以未出现过此问题，而这次nginx和php-fpm是分开部署在docker container中的，所以出现了问题，知道了原因，解决问题就好办了，找到listen.allowed_clients，注释掉（默认为任意IP）
