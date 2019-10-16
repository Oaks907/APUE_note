[TOC]

## 终端I/O

### 综述

终端I/O默认存在两种工作模式：

1. 规范模式输入处理。在这种模式下，对终端输入以行为单位进行处理。对于每个读请求，终端驱动程序最多返回一行。
2. 非规范模式输入处理。输入字符不装配成行。

如果不做特殊处理，则默认的模式是规范模式。

大多数UNIX系统在一个称为终端行规范（terminal line discipline）的模块中进行全部的规范处理。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-5642a95513943711.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 特殊输入字符

POSIX.1 定义了11个在输入时要处理的特殊处理的字符。

### 获得和设置终端属性

检测和修改各种终端选项标志和特殊字符。

```c
#include <termios.h>

int tcgetattr(int fildes, struct termios *termios_p);
int tcsetattr(int fildes, int optional_actions, const struct termios *termios_p);
```

optional_actions参数：

* **TCSANOW**    The change occurs immediately.
* **TCSADRAIN**  The change occurs after all output written to fildes has been transmitted to the terminal.  This value of optional_actions should be used when changing parameters that affect output.
* **TCSAFLUSH**  The change occurs after all output written to fildes has been transmitted to the terminal.  Additionally, any input that has been received but not read is discarded.
* **TCSASOFT**   If this value is or'ed into the optional_actions value, the values of the c_cflag, c_ispeed, and c_ospeed fields are ignored.

### 终端选项标志

略

### stty命令

在程序中，通过使用`tcgetattr`和`tcsetattr`函数进行检查和更改。在命令行中用`stty`命令进行检查和更改。

### 波特率函数

波特率是指“位/秒”（bit per second）。虽然大多数终端设备对输入和输出使用同一波特率，但是只要硬件允许，可以把他们设置为不同值。

```c
#include <termios.h>

speed_t cfgetispeed(const struct termios *termios_p);
speed_t cfgetospeed(const struct termios *termios_p);

int cfsetispeed(struct termios *termios_p, speed_t speed);
int cfsetospeed(struct termios *termios_p, speed_t speed);
```

### 行控制函数

下列四个函数提供了终端设备的行控制能力：

```c
#include <termios.h>

int tcdrain(int fildes);
int tcflow(int fildes, int action);
int tcflush(int fildes, int action);
int tcsendbreak(int fildes, int duration);
```

### 终端标识

历史上，在大多数UNIX系统版本中，控制终端的名字一直是/dev/tty。POSIX.1 提供了一个运行时函数，可用来确定控制终端的名字。

```c
#include <stdio.h>

char * ctermid(char *buf);
```

另外存在的函数：

`isatty`用来判断文件描述符是否是一个终端设备。

`ttyname`返回的是该文件描述符上打开的终端设备的路径名。

```c
#include <unistd.h>

char * ttyname(int fd);
int isatty(int fd);
```

### 规范模式

规范模式：当发起一个读请求时，当一行已经输入后，终端驱动程序即返回。以下几个条件都会造成读返回：

1. 所请求的字节数已读到时，读返回，无需读到一个完整的行。如果读到部分行，那么不会丢失任何信息，下一次读从前一次读的停止处开始
2. 当读到一个行定界符时，读返回。在规范模式中，下列字符被解释为“行结束”：NL、EOL、EOL2、EOF
3. 如果捕获到信号，并且该函数不再自动重启，则读也返回。

### 非规范模式

可以通过关闭termios结构中c_lfag 字段中ICANON标志来指定非规范模式。

在非规范模式中，输入数据不装配成行，不处理下列特殊字符：ERASE、KILL、EOF、NL、EOL、EOL2、CR、REPRINT、STATUS、WERASE。

非规范模式下，当已读指定量的数据，或者已经超过给定量的时间后，即通知系统返回。

### 终端窗口大小

大多数UNIX系统都提供了一种跟踪当前终端窗口大小的方法，当窗口大小发生变化时，使内核通知前台进程组。内核为每一个终端和伪终端都维护了一个winsize结构:

```c
struct winsize {
  unsigned short ws_row;
  unsigned short ws_col;
  unsigned short ws_xpixel;
  unsigned short ws_ypixel;
}
```

此结构的规则如下：

* 用`ioctl`的TIOCGWINSA命令取得该结构的当前值。

* 用`ioctl`的TIOCSWINSZ命令可以将此结构的新值存储在内核中。如果此新值与存储在内核中的当前值不同，则前台进程组会收到SIGWINCH信号。
* 除了存储此结构的当前值以及在此值改变后产生一个信号以外，内核对该结构不进行任何其他操作。对结构中的值进行解释，完全是应用程序的工作。

### termcap、terminfo、curses

略