[TOC]

## 进程间通信

![image.png](https://upload-images.jianshu.io/upload_images/1916953-74b55e22e80a8f1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 管道

管道是UNIX系统IPC的最古老的形式，所有的UNIX系统都提供这种通信方式。它存在两种局限性：

1. 它们是半双工通信。现在，某些系统上已经提供全双工管道
2. 管道只能在两个进程之间使用。通常，一个管道由一个进程创建，在进程调用fork之后，这个管道就能在父进程与子进程之间使用。

```c
#include <unistd.h>

int pipe(int fd[2]);
```

fd[0]为读而打开，fd[1]为写而打开

![image.png](https://upload-images.jianshu.io/upload_images/1916953-f107e7983b6a0098.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 函数popen和pclose

这两个函数含义是：创建一个管道，fork一个子进程，关闭未使用的管道端，执行一个shell命令，然后等待命令终止。

```c
#include <stdio.h>

FILE * popen(const char *command, const char *mode);
int pclose(FILE *stream);
```

popen 会先执行 fork， 然后调用 exec 执行 cmdstring， 并且返回一个标准I/O文件指针。

pclose 函数会关闭标准I/O流，等待命令终止，然后返回shell的终止状态。

### 协同进程

当一个过滤程序，即产生某个过滤程序的输入，又读取该过滤程序的输出时，它就变成了协同进程。

### FIFO

FIFO有时被称为命名管道。未命名的管道只能在两个相关的进程之间使用，而且这个两个相关的进程还要有一个共同创建它们的祖先进程。但是通过 FIFO，不想关的进程也能交换数据。

```c
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *path, mode_t mode);
int mkfifoat(int fd, const char *path, mode_t mode);
```

FIFO一般有两种用途：

1. shell命令使用FIFO将数据从一条管道传送到另一条，无需创建中间临时文件。
2. 客户进程-服务器进程应用程序中，FIFO用作汇聚点，在客户进程和服务器进程之间传递数据。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-8e687cf1e56b5b24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### XSI IPC

#### 标识符与键

每个内核中的IPC结构（消息队列、信号量、共享存储段）都用一个非负整数的标识符加以引用。例如，要向一个消息队列发送消息或从一个消息队列中取出消息，只需要知道其队列标识符。与文件标识符不同、IPC标识符不是小的整数，当一个IPC结构被创建，然后又被删除时，与这种结构相关的标识符连续加1， 直至达到一个整形数的最大正值，然后又会转到0。

#### 权限结构

XSI IPC为每一个IPC结构关联了一个ipc_perm 结构。该结构规定了权限与所有者，它至少包括下列成员：

```c
struct ipc_perm {
	uid_t uid;	// owner's effective user id
  gid_t gid;	// owner's effective group id
  uid_t cuid;	// creator's effective user id
  gid_t cgid; // creator's effective group id
  mod_t mode;	// access modes
  ...
}
```

### 消息队列

消息队列时消息的链接表，存储在内核中，由消息队列标识符标识。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-5b6ccffbbb8a64c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**创建消息队列**

```c
#include <sys/msg.h>

int msgget(key_t key, int flag);
```

对队列执行操作

```c
#include <sys/msg.h>

int msgctl(int msgid, int cmd, struct msgid_ds *buf);
```

发送消息

```c
#include <sys/msg.h>

int msgsnd(int msgid, const void *ptr, size_t nbytes, int flag);
```

接收消息

```c
#include <sys/msg.h>

int msgsnd(int msgid, const void *ptr, long type, size_t nbytes, int flag);
```

### 信号量

信号量时一个计数器，用于为多个进程提供对共享数据对象的访问。

获得一个新信号量ID

```c
#include <sys/sem.h>

int segment(key_t key, int nsems, int flag);
```

操作信号量

```c
#include <sys/sem.h>

int semctl(int semid, int semnum, int cmd, ...);
```

自动执行信号量集合上的操作数组

```c
#include <sys/sem.h>

int semop(int semid, struct sembuf semoparray[], size_t nops);
```

### 共享存储

共享存储允许两个或者多个进程共享一个指定的存储区。因为数据不需要在客户进程和服务器进程之间复制，所以这是最快的一种IPC。

内核为每个共享存储段都维护着一个结构，该结构至少要为每个共享存储段包含以下成员：

```c
struct shmid_ds {
         struct ipc_perm  shm_perm;     /* operation permissions */
         int              shm_segsz;    /* size of segment in bytes */
         pid_t            shm_lpid;     /* pid of last shm op */
         pid_t            shm_cpid;     /* pid of creator */
         short            shm_nattch;   /* # of current attaches */
         time_t           shm_atime;    /* last shmat() time*/
         time_t           shm_dtime;    /* last shmdt() time */
         time_t           shm_ctime;    /* last change by shmctl() */
         void            *shm_internal; /* sysv stupidity */
     };
```



创建获取一个新共享存储段：

```c
#include <sys/shm.h>

int shmget(key_t key, size_t size, int shmflg);
```

对存储段执行操作：

```c
#include <sys/shm.h>

int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

将共享的存储段，进程链接到它的地址空间中：

````c
#include <sys/shm.h>

void * shmat(int shmid, const void *shmaddr, int shmflg);
````

### POSIX信号量

POSIX 信号量是 3 种 IPC 机制之一。

POSIX 信号量接口意在解决XSI信号量接口的几个缺陷：

* 相比于XSI接口，POSIX 信号量接口考虑到了更高性能的实现。
* POSIX 信号量接口使用更简单：没有信号量集，在熟悉的文件系统操作后一些接口被格式化。
* POSIX 信号量在删除时表现更完美。当XSI信号量被删除时，使用这个信号量标识符的操作会失败，并将errorno设置为 EIDRM。使用POSIX 信号量时，操作能继续正常工作指导该信号量的最后一次引用被释放。

### 客户进程-服务器进程属性

略