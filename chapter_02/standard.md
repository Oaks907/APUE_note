[TOC]

### UNIX标准及实现

#### UNIX标准化

##### 1. ISO C（International Organization for Standardization）美国标准国际化组织

##### 2. IEEE POSIX

1. IEEE（Institute of Electrical and Electronics Engineers）电子与电气工程师协会
2. POSIX（Portable Operating System Interface）可移植操作系统接口

##### 3. Single UNIX Specification 单一UNX规范

Single UNIX Specification 是POSIX.1标准的一个超集，它定义了一些附加接口扩展了POSIX.1 规范提供了功能。

POSIX.1 中的X/open系统接口（X/open System Interface XSI）描述了可选的接口， 也定义了XSI的实现必须支持POSIX的哪些可选部分。

#### UNIX系统的实现

##### 1. SVR4

SVR4（UNIX System V Release 4）是AT&T的UNIX的系统实验室。

##### 2. 4.4BSD

BSD（Berkeley Software Distribution）是由加州大学伯克利分校的计算机系统研究组研发开发和分发的。

##### 3. FreeBSD

FreeBSD是基于4.4BSD-lite操作系统。

##### 4. Linux

Linux是一种提供类似于UNIX的丰富编程环境的操作系统，在GNU公用许可证的指导下，Linux是免费使用的。

##### 5. Mac OS X

其核心操作系统称为“Darwin”，它基于Mach内核、FreeBSD操作系统以及面向对象框架的驱动和其他内核扩展的结合。

##### 6. Solaris

Solaris是由 Sun Microsystems（现为Oracle）开发的UNIX系统版本。

#### 限制

UNIX系统实现了定义了很多幻数与常量，其中有很多已经被编码到程序中，或用特定的技术确定。由于大量标准化工作的努力，已有若干可移植的方法用以确定这些幻数和具体实现定义。

以下两种类型的限制是必须的。

1. 编译时限制（例如：短整形的最大值是多少？）
2. 运行时限制（例如：文件名的限制字符）

为了解决问题，提供了下面3种限制：

* 编译时限制（头文件）
* 与文件或目录无关的运行时限制（sysconf函数）
* 与文件或目录有关的运行时限制（pathconf和fpathconf）

#### 函数 sysconf、fpathconf、pathconf

```c
#include <unistd.h>
long sysconf(int name);
long fpathconf(int fildes, int name);
long pathconf(const char *path, int name);
```

#### 不确定的运行时限制

如果某些限制值没有在头文件中 `<limits.h>` 定义,那么在编译时也就不能使用它们。但是如果他们的值是不确定的，在运行时他们也可能是未定义的。

1. 路径名
2. 最大打开文件数

#### 选项

要编写可移植的应用程序，而这些程序可能会依赖于这些可选的支持的功能，那么就需要一种可移植的方法来判断实现是否支持一个特定的选项。

POSIX.1定义了3种处理选项的方法：

* 编译时选项定义在`<unistd.h>`中
* 与文件或目录无关的运行时选项用sysconf函数
* 与文件或目录有关的运行时选项通过调用pathconf和fpathconf函数判断


