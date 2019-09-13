[TOC]


- [进程关系](#----)
  * [终端登录](#----)
  * [网络登录](#----)
    + [BSD网络登录](#bsd----)
  * [进程组](#---)
  * [会话](#--)
  * [控制终端](#----)
  * [函数`tcgetpgrp`、`tcsetpgrp`和`tcgetsid`](#---tcgetpgrp---tcsetpgrp---tcgetsid-)
  * [作业控制](#----)
  * [`shell`执行程序](#-shell-----)
  * [孤儿进程组](#-----)
- [习题：](#---)
  * [9.1 **考虑 6.8 节中说明的 utmp 和 wtmp 文件，为什么 logout 记录是由 init 进程写的？对于网络登录的处理与此相同吗？**](#91------68-------utmp---wtmp--------logout------init-----------------------)
  * [9.2 **编写一段程序调用 fork 并使子进程建立一个新的会话。验证子进程变成了进程组组长且不再有控制终端。**](#92------------fork---------------------------------------)


## 进程关系

### 终端登录

* 系统先创建通常名为`/etc/ttys`的文件，其中，每个终端设备都有一行，每一行说明设备名和传到`gettty`程序的参数。当系统自举时，内核创建进程ID为 1 的进程，也就是`init`进程。init进程使系统进入**多用户模式**。**init读取/etc/ttys,**对每个允许登录的终端设备，init调用一次fork，它所生成的子进程则`exec getty`程序。所有进程的实际用户ID与有效用户ID都是0（也就是说，它们具有超级用户权限）。init 以空环境`exec getty`程序
* `getty`对终端设备调用open函数，以读、写方式打开终端。如果设备时调制解调器，则open可能会在设备驱动程序中滞留，知道用户拨号调制解调器，并且线路被接通。一旦设备被打开，则文件描述符0、1、2就被设置到该设备。然后`getty`输出`login:`之类的信息，并等待用户键入用户名。
* 当用户键入用户名后，`getty`的工作就完成了，然后他以下面的方式调用login程序.

```c
execle("/bin/login", "login", "-p", username, (char *) 0, envp)
```

init 以一个空环境调用getty。getty 以终端名和在gettytab中说明的环境字符串为login创建一个环境（envp参数）。-p表示通知login保留传递给它的环境，也可将其他环境字符串加到该环境中，但是不是替换它。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-9ec054ee28d5b032.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果用户能够正确登录，login将完成以下工作：

1. 将当前的工作目录更改为该用户的起始目录
2. 调用`chown`更改该终端的所有权，使登录的用户成为它的所有者。
3. 将对该终端设备的访问权限改变成“用户读和写”
4. 调用`setgid`和`initgroups`设置进程的组ID
5. 用login得到的所有信息初始化环境：起始目录、shell、用户名和系统默认路径
6. login进程更改为登录用户的用户ID(`setuid`)并调用该用户的登录shell

### 网络登录

网络登录时，在终端与计算机之间的连接不再是点到点的。在网络登录的情况下，login 仅仅是一种可用的服务，这与其他网络服务（如FTP、SMTP）的性质相同。

为了使同一个软件既能处理终端登录，又能处理网络登录，系统使用一种称为**伪终端**的软件驱动程序，它仿真串行终端的运行行为，并将终端操作映射为网络操作。

#### BSD网络登录

* 作为系统启动的一部分，init 调用一个 shell，使其执行脚本 `/etc/rc`。由此shell脚本启动一个守护进程inetd，一旦该shell脚本终止，inetd的父进程就变为了init。
* `inetd`等待TCP/IP连接到达主机，而当一个连接请求达到时，它进程一次 fork，然后生成的子进程`exec`适当的程序。
* `talnetd`进程打开一个伪终端设备，并用fork 分为两个进程。父进程处理通过网络连接的通信，子进程则执行login程序

![image.png](https://upload-images.jianshu.io/upload_images/1916953-08c72abbd3549c4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 进程组

进程组是一个或多个进程的集合。通常，它们是同一作业结合起来的。每个进程除了有一进程ID之外，还属于一个进程组。

获取进程组ID

```c
#include <unistd.h>

pid_t getpgid(pid_t pid);
pid_t getpgrp(void);
```

每个进程组有一个组长进程。组长进程的进程组ID等于其进程ID。组长进程可用创建一个进程组、创建一个组内的进程，然后终止。

创建或者加入一个进程组：

```c
#include <unistd.h>

int setpgid(pid_t pid, pid_t pgid);
pid_t setpgrp(void);
```

### 会话

会话是一个或多个进程组的集合。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-66591765b1673148.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 控制终端

会话和进程组还有一些其他的特性：

* 一个会话可以有一个控制终端。这通常是一个终端设备（终端登录情况下）或伪终端设备（网络登录情况下）
* 建立与控制终端链接的会话首进程被称为**控制进程**
* 一个会话中的几个进程组可被分为**前台进程组**以及一个或多个后台进程组。
* 如果一个会话有一个控制终端，则它有一个前台进程组，其他进程组为后台进程组。
* 无论何时键入终端的中断建，都会将中断信号发送至前台进程组中的所有进程。
* 无论何时键入终端的退出建，都会将退出信号发送至前台进程组中的所有进程。
* 如果终端接口检测到调制解调器或网络已经断开链接，则将挂断信号发送至控制进程（会话首进程）

### 函数`tcgetpgrp`、`tcsetpgrp`和`tcgetsid`

 The tcgetpgrp() function returns the value of the process group ID of the foreground process group associated with the terminal device.  If there is no foreground process group, tcgetpgrp() returns an invalid process ID.

```c
#include <unistd.h>

pid_t tcgetpgrp(int fildes);
```

If the process has a controlling terminal, the tcsetpgrp() function sets the foreground process group ID associated with the terminal device to pgid_id.  The terminal device associated with fildes must be the  controlling terminal of the calling process and the controlling terminal must be currently associated with the session of the calling process.  The value of pgid_id must be the same as the process group ID of a process in the same session as the calling process.

```c
#include <unistd.h>

int tcsetpgrp(int fildes, pid_t pgid_id);
```

The tcgetsid() function obtains the process group ID of the session for which the terminal specified by fildes is the controlling terminal.

```c
#include <termios.h>

pid_t tcgetsid(int fildes);
```

### 作业控制

作业控制允许一个终端上启动多个作业，它控制哪一个作业可以访问该终端以及哪些作业可以在后台运行。

### `shell`执行程序

![image.png](https://upload-images.jianshu.io/upload_images/1916953-e866b34aa4c3fc4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 孤儿进程组

一个其父进程已经终止的进程称为孤儿进程，这种进程由init进程收养。现在整个进程组也可能称为“孤儿”。

## 习题：

### 9.1 **考虑 6.8 节中说明的 utmp 和 wtmp 文件，为什么 logout 记录是由 init 进程写的？对于网络登录的处理与此相同吗？**

因为init是登录 shell 的父进程，当登录shell终止时它收到SIGCHLD信号量，所以init进程知道什么时候终端用户注销。

网络登录没有 init，在utmp与wtmp文件中的登录项和相应的注销项是由一个处理登录并检测注销的进程写的。

### 9.2 **编写一段程序调用 fork 并使子进程建立一个新的会话。验证子进程变成了进程组组长且不再有控制终端。**

```shell
> a.out
parent: pid = 46410, ppid = 70803, pgrp = 46410, tpgrp = 46410
child: pid = 46411, ppid = 46410, pgrp = 46411, tpgrp = -1
```

