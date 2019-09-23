[TOC]

## 高级I/O

### 非阻塞I/O

非阻塞I/O使我们发出open、read、write 这样的I/O操作，并使这些操作不会永远阻塞。如果这种操作不能完成、则调用立即出错返回，表示该操作继续执行将阻塞。

### 记录锁

记录锁 `Recording Lock`的功能是：当第一个进程正在读或者修改文件的某个部分时，使用记录锁可以阻止其他进程修改同一文件区。

#### fcntl 记录锁

```c
#include <fcntl.h>

int fcntl(int fildes, int cmd, ...);
```

第三个参数一般是一个指向flock结构的指针：

```c
struct flock {
	off_t       l_start;    /* starting offset */
	off_t       l_len;      /* len = 0 means until end of file */
	pid_t       l_pid;      /* lock owner */
	short       l_type;     /* lock type: read/write, etc. */
	short       l_whence;   /* type of l_start */
};
```

* `l_type`锁类型：F_RDLCK 共享读锁、F_WRLCK独占性写锁、F_UNLIK解锁一个区域

fcntl 函数的3种命令：

1. `F_GETLK`判断由`flockptr`所描述的锁是否会被另外一把锁排斥。如果存在一把锁，它阻止创建由flockptr所描述的锁，则该现有锁的信息将重写flockptr所指向的信息。如果不存在这种情况，则将l_type 设置为 F_UNLCK之外，flockptr所指向结构中的其他信息保持不变。
2. `F_SETLK`设置由`flockptr`所描述的锁。如果试图后去一把读锁或写锁，而兼容性规则阻止系统给我们这把锁，那么`fcntl`会立即出错返回，此时errorno 设置为`EACCES`或`EAGAIN`。
3. `F_SETLKW`设个命令是`F_SETLK`的阻塞版本。所以所请求的读锁或写锁因另一个进程当前已经对所请求的区域的某部分进行了加锁而不能被授予，那么调用进程会被设置为休眠。如果请求创建的锁已经可用，或者休眠由信号中断，则该进程被唤醒。

##### 锁的隐含继承与释放：

1. 锁与进程和文件两者相关联。它存在两种含义

   * 当一个进程终止时，它的所有建立的锁将被释放
   * 无论一个文件描述符何时关闭，该进程通过这一描述符引用的文件上的任何一把锁都会被释放。
2. 由`fork`产生的子进程不继承父进程设置的锁。

3. 当执行`exec`后，新程序可以继承原执行程序的锁。

### I/O多路转接

**异步I/O**，进程会告诉内核：当描述符可以进程I/O时，用一个信号通知它。

存在的问题：

1. 异步I/O存在标准化问题，可移植性会存在问题。
2. 这种信号对每个进程而言只有一个（`SIGPOLL`与`SIGIO`)。如果使该信号对两个描述符都起作用，那么进程在接收到此信号时将无法判断哪一个描述符准备好了。

**I/O多路转接**，它会先构造一个描述符列表，然后调用一个函数，知道这些描述符中的一个已经准备好了I/O，该函数返回。`poll`

、`pselect`、`select`这三个函数能够使我们执行I/O多路转接。

#### 函数select 和pselect

##### select

```c
#include <sys/select.h>

int select(int nfds, fd_set *restrict readfds, fd_set *restrict writefds, fd_set *restrict errorfds, struct timeval *restrict timeout);

void FD_CLR(fd, fd_set *fdset);
void FD_COPY(fd_set *fdset_orig, fd_set *fdset_copy);
int FD_ISSET(fd, fd_set *fdset);
void FD_SET(fd, fd_set *fdset);
void FD_ZERO(fd_set *fdset);
```

`select()` examines the I/O descriptor sets whose addresses are passed in readfds, writefds, and errorfds to see if some of their descriptors are ready for reading, are ready for writing, or have an exceptional condition pending, respectively.  The first nfds descriptors are checked in each set; i.e., the descriptors from 0 through nfds-1 in the descriptor sets are examined.  (Example: If you have set two file descriptors "4" and "17", nfds should  not be "2", but rather "17 + 1" or "18".)  On return, `select()` replaces the given descriptor sets with subsets consisting of those descriptors that are ready for the equested operation. `select()` returns the total number of ready descriptors in all the sets.

参数`readfds`、`writefds`、`errorfds`

是指向描述符集的指针。这3个描述符集说明了我们关心的可读，可写或处于异常的描述符集合。每个描述符集说明我们关心的可读、可写与处于异常条件的描述符结合。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-cfb3834088aeccdc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

返回值说明：

* -1 表示出错
* 返回值为 0 ，表示描述符没有准备好。若指定的描述符都没有准备好，且过了超时时间，就会出现这种情况。
* 返回值为正。说明存在已准备好的描述符之和，所以如果同一描述符已准备好了读和写，那么其返回值会对其计两次数。

##### pselect

```c
#include <sys/select.h>

int pselect(int nfds, fd_set *restrict readfds, fd_set *restrict writefds, fd_set *restrict errorfds, const struct timespec *restrict timeout, const sigset_t *restrict sigmask);
```

pselect 与 select 的不同之处：

* select的超时值用timeval结构指定，但 pselect 使用 timespec结构。timespec 以秒和纳秒表示超时值。
* pselect 的超时值被声明为const，这保证了调用pselect不会改变此值。
* pselect 可使用可选信号屏蔽字。

#### 函数poll

```c
 #include <poll.h>

int poll(struct pollfd fds[], nfds_t nfds, int timeout);
```

`poll()` examines a set of file descriptors to see if some of them are ready for I/O or if certain events have occurred on them.  The fds argument is a pointer to an array of pollfd structures, as defined in` <poll.h>` (shown below).  The nfds argument specifies the size of the fds array.

```c
struct pollfd {
	int    fd;       /* file descriptor */
	short  events;   /* events to look for */
	short  revents;  /* events returned */
};
```

`poll`函数类似于`select`，但是poll函数能够支持任何类型的文件描述符。

### 异步I/O



### 函数 readv 和 writev

`readv` 与`writev`函数用于在一次函数中调用中读、写多个非连续缓冲区。有时也将这个两个函数称为散布读(scatter read)和聚集写(gather write)。

```c
#include <sys/uio.h>

ssize_t readv(int d, const struct iovec *iov, int iovcnt);
ssize_t writev(int fildes, const struct iovec *iov, int iovcnt);
```

`iov` 是指向 `iovec`结构的一个数组指针。iov的大小由第三个参数 iovcnt确定。 

```c
struct iovec {
	char   *iov_base;  /* Base address. */
	size_t iov_len;    /* Length. */
};
```



`writev`函数从缓冲区中聚集输出数据的顺序是 iov[0] iov[1]直到 iov[iovcnt - 1]. writev 返回输出的字节总数，通常应该是等于所有缓冲区长度之和。

`readv`函数则将读入的数据按照上面的顺序散布到缓冲区中。readv 总是先填满一个缓冲区，然后再填入下一个。readv返回读到的字节总数。如果遇到文件尾端，已无数据可读，则返回0。

### 函数readn和writen

管道、FIFO、以及某些设备有下列两种性质：

1. 一次read操作返回的数据可能少于指定输入的字节数。这可能是由某种因素造成的，例如：内核输出缓冲区满了。这也不是错误，应该继续读该设备。
2. 一次write操作的返回值少于指定输入的字节数。这也不是错误，应该继续写余下的数据。



`readn` 与`writen`的功能是分别是读、写指定的N字节数据，并处理返回值可能小于要求值的情况。这个两个函数只是按需多次调用 read 和write 直至读、写了N字节数据。

```c
#include "apue.h"

ssize_t readn(int fd, void *buf, size_t nbytes);
ssize_t writen(int fd, void *buf, size_t nbytes);
```

### 存储映射I/O

存储映射I/O(memory-mapped I/O) 能将一个磁盘文件映射到存储空间中的一个缓冲区上。于是，当从缓冲区中取数据时，就相当于读文件中的相应字节。与此类似，将数据写入到缓冲区时，相应字节就自动写入文件了。

实现文件映射：	

```c
#include <sys/mman.h>

void * mmap(void *addr, size_t len, int prot, int flags, int fd, off_t offset);
```

![image.png](https://upload-images.jianshu.io/upload_images/1916953-799e946c921b2c19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

更改现有映射的权限：

```c
#include <sys/mman.h>

int mprotect(void *addr, size_t len, int prot);
```

将页冲洗到缓冲区中：

```c
#include <sys/mman.h>

int msync(void *addr, size_t len, int flags);
```

解除映射区。

```c
#include <sys/mman.h>

int munmap(void *addr, size_t len);
```