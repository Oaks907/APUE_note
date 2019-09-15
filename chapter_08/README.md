[TOC]

## 进程控制

### 进程标识

每个进程都有一个非负整数的唯一进程ID。因为进程ID标识符总是唯一的。

系统中存在一些专用进程：

* ID为0的通常是调度进程，也称为交换进程。该进程是内核中的一部分，它并不执行任何磁盘上的程序，因此称为系统进程
* ID为1的通常是Init进程，在自举过程结束时由内核调用。
* 在某些UNIX虚拟存储器上，进程ID为2的是页守护进程，此进程负责支持虚拟存储器系统的分页操作。

```c
#include <unistd.h>

pid_t getpid(void);
pid_t getppid(void);
```

### 函数fork

创建一个新进程。

```c
#include <unistd.h>

pid_t fork(void);
```

> fork() causes creation of a new process.  The new process (child process) is an exact copy of the calling process (parent process) except for the following:
>
> o   The child process has a unique process ID.
>    
> o   The child process has a different parent process ID (i.e., the process ID of the parent process).
>    
> o   The child process has its own copy of the parent's descriptors.  These descriptors reference the same underlying objects, so that, for instance, file pointers in file objects are shared between the child and the parent, so that an lseek(2) on a descriptor in the child process can affect a subsequent read or write by the parent.  This descriptor copying is also used by the shell to establish standard input and output for newly created processes as well as to set up pipes.

由fork创建的新进程称为子进程。fork 函数被调用一次，但返回两次。两次返回的区别是子进程的返回值是0，父进程的返回值是子进程的进程ID。

子进程和父进程继续执行 fork 调用之后的指令。子进程是父进程的副本。子进程获取父进程数据空间、堆、栈的副本。注意，这是子进程所拥有的副本。父进程和子进程并不共享这些存储空间部分。

子进程与父进程之间的区别如下：

* fork的返回值不同
* 进程ID不同
* 子进程的`tms_utime``tms_stime``tms_cutime`和`tms_ustime`的值设置为0
* 子进程不能继承父进程设置的文件锁
* 子进程的未处理闹钟会被清除
* 子进程的未处理信号集设置为空集

### 函数vfork

区别：

1. vfork用于创建一个新进程，该新进程的目的是exec一个新程序。
2. vfork会保证子进程先运行，在它调用exec或exit之后父进程才能调度运行。

### 函数exit

在UNIX术语中，一个已经终止，但其父进程尚未对其进行善后处理（获取终止子进程的有关信息，释放它仍占用的资源）的进程称为**僵死进程**。

由init直接产生的进程或者由init收养的进程，init被编写为只要有一个子进程终止，init就会调用一个wait函数获取其终止状态。

### 函数wait和 waitpid

当进程正常或者异常终止时，内核会向父进程发送异步通知。因为子线程终止是个异步事件，所以这种信号也是内核向父进程的异步通知。

`wait`与`waitpid`作用：

* 如果其所有子进程都还未运行，则阻塞
* 如果一个子进程已终止，正等待父进程获取其终止状态，则取得该子进程的终止状态立即返回。
* 如果他没有任何一个子进程，则立即出错返回。

区别：

* 在一个子进程终止前，`wait`会使调用者阻塞，而`waitpid`有一个选项可以不阻塞
* `waitpid`并不等待在其调用之后的第一个终止进程，它有若干个选项，可以通知它所等待的进程。

```c
#include <sys/wait.h>

pid_t wait(int *stat_loc);
pid_t waitpid(pid_t pid, int *stat_loc, int options);
```

### 函数waitid

```c
#include <sys/wait.h>

int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
```

与`waitpid`类似，`waitid`允许一个进程指定要等待的子进程。但它使用的两个单独的参数表示要等待的子进程所属的类型，而不是将此与进程ID或进程组ID组合成一个参数。

* `idtype`
  * `P_PID`等待一特定的进程：id包含要等待子进程的进程ID
  * `P_PGID`等待一特定进程组中的任一子进程：id包含要等待子进程的进程组ID
  * `P_ALL`等待任一子进程，忽略id
* `options`
  * `WCONTINUED`等待一进程，它以前曾被停止，而后又已继续，但其状态尚未报告
  * `WEXITED`等待已退出的进程
  * `WNOHANG`如无可用的子进程退出状态，立即返回而非阻塞
  * `WNOWAIT`不破坏子进程的退出状态。该子进程退出状态可由后续的wait、waitid、waitpid调用取得
  * `WSTOPPED`等待一进程，它已经停止，但其状态尚未报告

### 函数wait3和wait4

```c
#include <sys/wait.h>

pid_t wait3(int *stat_loc, int options, struct rusage *rusage);
pid_t wait4(pid_t pid, int *stat_loc, int options, struct rusage *rusage);
```

与wait、waitpid、waitid提供的功能不同，根据`rusage`参数，允许内核返回由终止进程及其所有子进程使用的资源概况。

### 竞争条件

多个进程都企图对共享数据进程某种处理，而最后的结果又依赖于进程运行的顺序，我们认为发生的`竞争条件`。

### 函数exec

当调用fork函数创建新的子进程后，子进程往往要调用一种exec函数来执行另一个程序。当进程调用`exec`函数时，该进程执行的程序完全替换为新程序，而新程序则从main函数开始执行。因为调用`exec`并不创建新进程，所以进程ID不变。**`exec`只是用磁盘上的一个新程序替换了当前进程的正文段、数据段、堆段和栈段。**

````c
#include <unistd.h>
extern char **environ;

int execl(const char *path, const char *arg0, ... /*, (char *)0 */);
int execle(const char *path, const char *arg0, ... /*, (char *)0, char *const envp[] */);
int execlp(const char *file, const char *arg0, ... /*, (char *)0 */);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvP(const char *file, const char *search_path, char *const argv[]);
````

![image.png](https://upload-images.jianshu.io/upload_images/1916953-d147136d80c67c2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 更改用户ID和更改组ID

`setuid`函数设置实际用户ID和有效用户ID。与此类似，可以用`setgid`函数设置实际组ID与有效组ID

```c
#include <unistd.h>

int setgid(gid_t gid);
int setuid(uid_t uid);
```

* 进程拥有超级用户权限，则`setuid`函数将实际用户ID、有效用户ID以及保存的设置用户ID保存为uid
* 进程没有超级用户权限，但是`uid`等于实际用户ID或保存的设置用户ID，则`setuid`只是将有效用户id设置为uid。不更实际用户ID和保存的设置用户ID
* 上面条件都没满足，则errno设置为EPERM，并返回 -1

![image.png](https://upload-images.jianshu.io/upload_images/1916953-79cba04a1322df0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

交换实际用户ID和有效用户ID的值：

```c
 #include <unistd.h>

int setreuid(uid_t ruid, uid_t euid);
int setregid(gid_t rgid, gid_t egid);
```

更改有效用户ID和有效组ID

```c
#include <unistd.h>

int setegid(gid_t egid);
int seteuid(uid_t euid);
```

![image.png](https://upload-images.jianshu.io/upload_images/1916953-9fd856a27a47c6eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 解释器文件

解释器文件是文本文件，其起始形式是

```shell
#! pathname [optional-argument]
```

例如：

```shell
#! /bin/bash
```

解释器文件确实使用户得到效率方面的好处，其代价是**内核的额外开销**。由于下面，解释器文件是有用的：

* 有些程序是用某种语言写的脚本，解释器文件可将这一事实隐藏起来。
* 解释器文件在效率方面提供了好处
* 解释器使我们可以使用除`\bin\sh`以外的其他shell来编写shell脚本

### 函数system

```c
#include <stdlib.h>

int system(const char *command);
```

The system() function hands the argument command to the command interpreter sh(1).  The calling process waits for the shell to finish executing the command, ignoring SIGINT and SIGQUIT, and blocking SIGCHLD.

### 进程会计

大多数UNIX系统都会提供一个选项进行**进程会计(process accounting)**。启用后，当进程结束后，就会写入一个会计记录。记录一般包括命令名、所使用的CPU时间等。

### 进程标识

用户可以有多个登陆名，这些用户名又对应同一个用户ID，为了获取运行时用户的登录名？系统通常会记录用户登录时使用的名字，可以以此通过`getlogin`获取登录名。

```c
#include <unistd.h>

char * getlogin(void);
int setlogin(const char *name);
```

### 进程调度

UNIX 系统历史上对于进程提供的只是基于优先级的粗粒度的控制。调度策略与调度优先级是由内核决定的。进程可以通过调整nice值来选择以更低优先级运行。

The nice() function obtains the scheduling priority of the process from the system and sets it to the priority value specified in incr.  The priority is a value in the range -20 to 20.  The default priority is  0; lower priorities cause more favorable scheduling.  Only the super-user may lower priorities.

```c
#include <unistd.h>

int nice(int incr);
```

The scheduling priority of the process, process group, or user as indicated by which and who is obtained with the getpriority() call and set with the setpriority() call.  Which is one of PRIO_PROCESS, PRIO_PGRP, or PRIO_USER, and who is interpreted relative to which (a process identifier for PRIO_PROCESS, process group identifier for PRIO_PGRP, and a user ID for PRIO_USER).  A zero value of who denotes the current process, process group, or user.  prio is a value in the range -20 to 20.  The default priority is 0; lower priorities cause more favorable scheduling.

The getpriority() call returns the highest priority (lowest numerical value) enjoyed by any of the specified processes.  The setpriority() call sets the priorities of all of the specified processes to the specified value.  Only the super-user may lower priorities.

```c
#include <sys/resource.h>

int getpriority(int which, id_t who);
int setpriority(int which, id_t who, int prio);
```

### 进程时间

```c
struct tms {
	clock_t tms_utime;
	clock_t tms_stime;
	clock_t tms_cutime;
	clock_t tms_cstime;
};
```

* tms_utime   The CPU time charged for the execution of user instructions.
* tms_stime   The CPU time charged for execution by the system on behalf of the process.
* tms_cutime  The sum of the tms_utimes and tms_cutimes of the child processes.
* tms_cstime  The sum of the tms_stimes and tms_cstimes of the child processes.

```c
include <sys/times.h>

clock_t times(struct tms *tp);
```

## 习题

### 1. **在图 8-3 程序中，如果用 exit 调用代替 _exit 调用，那么可能会使标准输出关闭，使 printf 返回 -1 。修改该程序以验证在那你所使用的系统上是否会产生此种效果。如果并非如此，你怎样处理才能得到类似结果呢？**