[TOC]

## 伪终端

### 概述

伪终端是指，对于一个应用程序而言，它看上去是一个终端，但事实上它并不是一个真正的终端。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-7e0d222c2dfc2619.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 通常，一个进程打开伪终端主设备，然后调用 fork。子进程建立一个会话，打开一个相应的伪终端从设备，将其文件描述符复制到标准输入、标准输出和标准错误，然后调用 exec。伪终端从设备成为子进程的控制终端。
* 对于伪总端从设备上的用户进程来说，其标准输入、标准输出和标准错误都是终端设备。通过这些描述符，用户进程能够处理所有终端I/O函数。但是因为伪终端从设备不是真正的终端设备，所以无意义的函数调用，如改变波特率将被忽略。
* 任何写入到伪终端主设备的都会作为从设备的输入，反之亦然。事实上，所有从设备端的输入都来自于伪终端主设备上的用户进程。

### 打开终端设备

打开可用伪终端的主设备。

```c
#include <stdlib.h>
#include <fcntl.h>

int posix_openpt(int oflag);
```

grantpt 函数用于设置权限。

unlockpt 函数用于准予对伪终端从设备的访问。

```c
#include <stdlib.h>

int grantpt(int fildes);
int unlockpt(int fildes);
```

 The `grantpt()` function is used to establish ownership and permissions of the slave device counterpart to the master device specified with fildes.  The slave device's ownership is set to the real user ID of the calling process; its permissions are set to user readable-writable and group writable.  The group owner of the slave device is also set to the group 'tty if it exists on the system; otherwise, it is left untouched.



ptsname 用于找到伪终端从设备的路径名。

```c
#include <stdlib.h>

char * ptsname(int fildes);
```

`ptym_open`打开下一个可用的 PTY 主设备，`ptys_open`打开相应的从设备。

```c
#include "apue.h"

int ptym_open(char *pts_name, int pts_namest);
int ptys_open(char *pts_name);
```