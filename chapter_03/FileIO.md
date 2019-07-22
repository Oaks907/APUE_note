
[TOC]

## 第三章 文件I/O

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

### 3.5 close 

delete a descriptor

```c
#include <unistd.h>
int close(int fildes);
```

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
