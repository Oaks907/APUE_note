[TOC]

## 守护进程

守护进程daemon 是生存期长的一种进程。它们常常在系统引导装入时启动，仅在系统关闭时终止。

### 守护进程的特征

大多数的守护进程以超级用户root特权运行。所有的守护进程都没有控制终端，其终端名设置为 ？号。

内核守护进程都以无控制终端方式启动。

### 编程规则

编写守护进程需要遵守一些基本规则，防止产生不必要的作用。

1. 首先是调用`umask`将文件模式创建屏蔽字设置为一个已知值。由继承得来的文件模式创建屏蔽字可能会设置为拒绝某些权限。
2. 调用`fork`， 然后使父进程exit
3. 调用 `setsid` 创建一个新的会话。使调用进程
   * 成为新会话的首进程。
   * 成为新进程组的组长进程
   * 没有控制终端
4. 将工作目录更改为根目录。从父进程处继承过来的当前工作目录可能在一个挂载的文件系统中。
5. 关闭不再需要的文件描述符。这使守护进程不再持有从父进程继承来的任何文件描述符。
6. 默写守护进程打开`/dev/null`使其具有文件描述符0、1、2，这样，任何一个试图读标准输入、写标准输出或标准错误的库例程都不会产生任何效果。

### 出错记录

守护进程没有控制终端，所以日志不能写到标准错误中。

守护进程没有不能写入到单独的文件中，因为难以维护。

所以，想要一个集中的地方去记录出错信息。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-eb4c9c6b5a36e243.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由以下三种产生日志的方法：

1. 内核例程调用 log 函数。任何一个用户进程都可以通过打开`/dev/log`设备来读取这些消息。
2. 大多数用户进程（守护进程）调用 `syslog(3)`来产生日志消息。这使消息被发送至UNIX 域数据报套接字 `/dev/log`
3. 无论用户进程在此主机上，还是通过TCP/IP 网络链接到此主机的其他主机上，都可将日志消息发向UDP端口514.

### 单实例守护进程

为了正常运作，某些守护进程会实现为，在任一时刻只运行该守护进程的一个副本。例如这种守护进程可能需要排它地访问一个设备。对cron守护进程而言，如果同时有多个实例运行，那么每个副本都可能试图开始某个预定的操作，于是造成该操作的重复执行，可能导致报错。

### 守护进程的惯例

在UNIX中，守护进程遵循以下惯例：

1. 若守护进程使用锁文件，那么该文件存储在`/var/run`目录中。
2. 若守护进程支持配置选项，那么配置文件通常存放在`/etc`目录中.
3. 守护进程可用命令行启动，但通常是由系统初始化脚本之一（`/etc/rc*或/etc/init.d/`*）启动的
4. 若一个守护进程有一个配置文件，那么当该守护进程启动时会读取该文件，但在此之后一般不会查看它。若某个管理员更改了配置文件，那么该守护进程需要被停止，然后再启动。

### 客户进程-服务器进程模型 

守护进程常常用作服务端进程。一般而言，服务器进程等待客户进程与其联系，提出某种类型的服务要求。