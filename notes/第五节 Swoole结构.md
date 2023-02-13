---
attachments: [Clipboard_2022-05-03-17-39-11.png, Clipboard_2022-05-03-17-39-53.png]
tags: [Swoole]
title: 第五节 Swoole结构
created: '2022-05-03T09:39:08.326Z'
modified: '2022-05-03T09:42:06.036Z'
---

# 第五节 Swoole结构

![](@attachment/Clipboard_2022-05-03-17-39-11.png)
```
如上分为四层:
1. master:主进程
2. Manger:管理进程
3. worker:工作进程
4. task:异步任务工作进程
```
![](@attachment/Clipboard_2022-05-03-17-39-53.png)
##### 说明
```
Master进程，这个是swoole的主进程,这个进程是用于处理swoole的核心事件驱动的，那么在这个进程当中可以看到它拥有一个MainReactor[线程]以及若干个Reactor[线程]，swoole所有对于事件的监听都会在这 些线程中实现，比如来自客户端的连接，信号处理等。

manager进程，Swoole想要实现最好的性能必须创建出多个工作进程帮助处理任务，但Worker进程就必须fork操作，但是fork操作是不安全的，如果没有管理会出现很多的僵尸进程，进而影响服务器性能，同时worker进程被误杀或者由于程序的原因会异常退出，为了保证服务的稳定性，需要重新创建worker进程。 
Swoole在运行中会创建一个单独的管理进程，所有的worker进程和task进程都是从管理进程Fork出来的。管理进程会监视所有子进程的退出事件，当worker进程发生致命错误或者运行生命周期结束时，管理进程会回收此进程，并创建新的进程。换句话也就是说，对于worker、task进程的创建、回收等操作全权有“保姆”Manager进程进行管理。

worker进程，属于swoole的主逻辑进程，用户处理客户端的一系列请求，接受由Reactor线程投递的请求数据包，并执行PHP回调函数处理数据生成响应数据并发给Reactor线程，由Reactor线程发送给TCP客户端可以是异步非阻塞模式，也可以是同步阻塞模式

taskWorker进程，这一进城是swoole提供的异步工作进程，这些进程主要用于处理一些耗时较长的同步任务，在worker进程当中投递过来。
```
##### 进程查看&流程梳理
```
当启动一个Swoole应用时，一共会创建2 + n + m个进程，2为一个Master进程和一个Manager进程，其中n为Worker进程数。m为TaskWorker进程数。 默认如果不设置，swoole底层会根据当前机器有多少CPU核数，启动对应数量的Reactor线程和Worker进程。我机器为1核的。Worker为1。 所以现在默认我启动了1个Master进程，1个Manager进程，和1个worker进程，TaskWorker没有设置也就是为0，当前server会产生3个进程。 在启动了server之后，在命令行查看当前产生的进程
```
##### 理解
```
Swoole的Reactor、Worker、TaskWorker之间可以紧密的结合起来，提供更高级的使用方式。
一个更通俗的比喻，假设Server就是一个工厂，那Reactor就是销售，接受客户订单。而Worker就是工人，当销售接到订单后，Worker去工作生产出客户要的东西。而TaskWorker可以理解为行政人员，可以帮助Worker干 些杂事，让Worker专心工作。
```
