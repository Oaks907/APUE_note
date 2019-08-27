[TOC]

## 第三章 文件I/O

### 3.2 文件描述符

对于内核而言，所有打开的文件都通过文件描述符引用。

按照惯例：文件描述符0与进程的标准输入关联、文件描述符1与标准输出关联、文件描述符3与标准错误关联。

在POSIX.1 中， 幻数0、1、2被标准化替换为符合常量STDIN_FILENO、STDIN_FILENO、STDERR_FILENO

### 3.3 open 与 openat

open or create a file for reading or writing

```c
#include <fcntl.h>
int open(const char *path, int oflag, ...);
int openat(int fd, const char *path, int oflag, ...);
```

### 3.4 create

create a new file

```c
#include <fcntl.h>
int creat(const char *path, mode_t mode);
```

等效于

`open(path, O_WRONLY | O_CREAT | O_TRUNC, mode)`

### 3.5 close 

delete a descriptor

```c
#include <unistd.h>
int close(int fildes);
```

关闭一个文件会释放掉所有加在该文件上的记录锁。

**当一个进程终止时，内核会自动关闭它所有的打开文件。很多程序利用了这一点而不显式的关闭文件**

### 3.6 lseek

reposition read/write file offset

```c
#include <unistd.h>
off_t lseek(int fildes, off_t offset, int whence);
```

> If whence is SEEK_SET, the offset is set to offset bytes.
>
> If whence is SEEK_CUR, the offset is set to its current location plus offset bytes.
>
> If whence is SEEK_END, the offset is set to the size of the file plus offset bytes.
>
> If whence is SEEK_HOLE, the offset is set to the start of the next hole greater than or equal to the supplied offset.  The definition of a hole is provided below.
>
> If whence is SEEK_DATA, the offset is set to the start of the next non-hole file region greater than or equal to the supplied offset.

每个打开的文件都关联一个“当前文件偏移量”。它通常是一个非负整数，用于度量文件开始处计算的字节数。通过，读、写操作都从当前的偏移量开始。

lseek 将当前的偏移量记录在内核中，用于下一次的读/写操作，它并不引起任何I/O操作。

当偏移量大于文件的大小时，会在文件中产生空洞。

### 3.7 read

由打开的文件中读取数据到 buf 中。

```c
#include <unistd.h>
sszie_t read(int fd, void *buf, size_t nbytes);
```

### 3.8 write

向fd对应的打开文件写入buf中nbytes位数据。

```c
#include <unistd.h>
sszie_t read(int fd, const void *buf, size_t nbytes);
```

### 3.12 dup与dup2

duplicate an existing file descriptor

```c
#include <unistd.h>
int dup(int fildes);
int dup2(int fildes, int fildes2);
```

`dup`返回的文件描述符一定是可用描述符的最小值

### 3.13 sync、fsync、fdatasync

sync -- force completion of pending disk writes (flush cache)

fsync -- synchronize a file's in-core state with that on disk

fdatasync -- like fsync, but fdatasync not update the status about file attribute 

```c
#include <unistd.h>
int sync(int fildes);
int fsync(int fildes);
int fdatasync(int fildes);
```

当我们向文件写入数据时，内核通常先将数据输入到缓冲区中，然后排入队列，晚些时候再写入磁盘。这种方式被称为**延迟写**。

`sync`只是将所有修改过的块缓冲区排入写队列，然后返回，并不等待实际写磁盘操作结束。

`fsync`等待写磁盘结束并返回。它会确保所有的数据都已经写会到了磁盘。

`fdatasync`等待写磁盘结束并返回。只影响它的数据部分，不改变文件属性。

### 3.14 fcntl

fcntl() provides for control over descriptors. 可以改变已经打开文件的属性。

```c
#include <fcntl.h>
int fcntl(int fildes, int cmd, ...);
```

### 3.15 ioctl

The ioctl() function manipulates the underlying device parameters of special files.  In particular, many operating characteristics of character special files (e.g. terminals) may be controlled with ioctl() requests.

```c
#include <sys/ioctl.h>
int ioctl(int fildes, unsigned long request, ...);
```

### 3.16 /dev/fd

较新的系统提供名为**/dev/fd**的目录，其目录项是为0、1、2等的文件。打开文件 `/dev/fd/n`等同与复制描述符n

## 习题：

### 1. **当读/写磁盘文件时，本章中描述的函数确实是不带缓冲机制的吗？请说明原因**

读写磁盘文件时，是带有缓冲机制的

1. read的时候会采用某种预读 read ahead 技术。当检测到正在顺序读取时，系统就试图读入比应用所要求的更多数据。
2. write的时候会采用延迟写的方式。向文件写入数据时，通常先将数据复制到缓冲区中，然后排入队列，晚些再同步到磁盘

标准答案：

### 2. **编写一个与 3.12 节中 dup2 功能相同的函数，要求不调用 fcntl 函数，并且要有正确的出错处理**

### 3. **假设一个进程执行下面 3 个函数调用：**

```
fd1 = open(path, oflags);
fd2 = dup(fd1);
fd3 = open(path, oflags);
```

**画出类似于图 3-9 的结果图。对 fcntl 作用于 fd1 来说， F_SETFD 命令会影响哪一个文件描述符？ F_SETFL 呢？**

F_SETFD:设置描述符标志，只影响描述符，只作用fd1

F_SETFL:设置文件状态标志，影响文件本身，影响所有描述符

### 4. **许多程序中都包含下面一段代码：**

```
dup2(fd, 0);
dup2(fd, 1);
dup2(fd, 2);
if (fd > 2)
    close(fd);
```

**为了说明 if 语句的必要性，假设 fd 是 1 ，画出每次调用 dup2 时 3 个描述符项及相应的文件表项的变化情况。然后再画出 fd 为 3 的情况。**

为1时，0，1， 2 三个描述符都指向相同的文件V节点。

为3时，0， 1，2，3 四个描述符指向相同的文件节点。然后关闭一个

### 5. **在 Bourne shell 、 Bourne-again shell 和 Korn shell 中， digit1 > digit2 表示要将描述符 digit1 重定向至描述符 digit2 的同一文件。请说明下面两条命令的区别。**

```
./a.out > outfile 2>&1
./a.out 2>&1 > outfile
```

**在Unix中,2>&1类似于 dup2(fd1,fd2) 函数，2>&1是将标准错误输出重定向到标准输出。**

`./a.out > outfile 2>&1`设置标准输出到outfile，然后执行dup将标准输出复制到描述符2上，设置描述符1、2只想同一个文件表项。

`./a.out 2>&1 > outfile`首先执行dup，描述符2称为终端，标准输出重定向到outfile

### 6. **如果使用追加标志打开一个文件以便读、写，能否仍用 lseek 在任一位置开始读？能否用 lseek 更新文件中任一部分的数据？请编写一段程序验证**

仍然可以使用 lseek 和 read 函数读文件任意一个位置的内容。对于write来说，每次写数据之前会自动将文件偏移量设置为末尾，所以写文件只能从文件尾端开始。