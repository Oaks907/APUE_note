[TOC]

## 系统数据文件与信息

### 口令文件

![图片.png](https://upload-images.jianshu.io/upload_images/1916953-2028a6f9ce3c0bcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
> cat /etc/passwd
nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
root:*:0:0:System Administrator:/var/root:/bin/sh
daemon:*:1:1:System Services:/var/root:/usr/bin/false
_uucp:*:4:4:Unix to Unix Copy Protocol:/var/spool/uucp:/usr/sbin/uucico
```

passwd结构定义：

```c
           struct passwd {
                   char    *pw_name;       /* user name */
                   char    *pw_passwd;     /* encrypted password */
                   uid_t   pw_uid;         /* user uid */
                   gid_t   pw_gid;         /* user gid */
                   time_t  pw_change;      /* password change time */
                   char    *pw_class;      /* user access class */
                   char    *pw_gecos;      /* Honeywell login info */
                   char    *pw_dir;        /* home directory */
                   char    *pw_shell;      /* default shell */
                   time_t  pw_expire;      /* account expiration */
                   int     pw_fields;      /* internal: fields filled in */
           };
```

根据用户id和用户名获取Pass信息。获取的信息结构如上所示：

```c
#include <pwd.h>

struct passwd * getpwuid(uid_t uid);
struct passwd * getpwnam(const char *login);
```



```c
#include <pwd.h>

struct passwd *getpwent(void);
void setpwent(void);
void endpwent(void);
```

`getpwent`返回pass文件的下一条记录。

The setpassent() function causes getpwent() to ``rewind'' to the beginning of the list of entries cached by a previous getpwent() call.

### 阴影文件

加密口令是通过加密算法处理过的用户口令副本，算法是单向的。在某些系统中，将加密口令存放到一个称为阴影口令的文件中，该文件至少要包含用户名和加密口令。

![图片.png](https://upload-images.jianshu.io/upload_images/1916953-80a011c6ee535823.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```c
#include <shadow.h>

struct spwd * getspnam(const char *name);
struct spwd * getspent(void);
void setspent(void);
void endspent(void);
```

### 组文件

![图片.png](https://upload-images.jianshu.io/upload_images/1916953-696ad9a922ebb6df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

group结构:

```c
       struct group {
               char    *gr_name;       /* group name */
               char    *gr_passwd;     /* group password */
               gid_t   gr_gid;         /* group id */
               char    **gr_mem;       /* group members */
       };
```
通过 name 和 gid 来获取组信息

```c
#include <grp.h>

struct group * getgrnam(const char *name);
struct group * getgrgid(gid_t gid);
```


The `getgrent()` function searches all available directory services on it's first invocation.  It caches the returned entries in a list and returns group entries one at a time.
`````c
#include <grp.h>

struct group * getgrent(void);
void setgrent(void);
void endgrent(void);
`````

### 附属组文件

附属组ID是POSIX.1要求的特性。同时规定了它的数量，其值是16。

使用附属组的优点是不必在显式地经常更改组。一个用户会参与多个项目，也会同时属于多个组，此类情况是常有的。

`getgroups()` gets the current group access list of the current user process and stores it in the array grouplist[].  The parameter gidsetsize indicates the number of entries that may be placed in grouplist[].

```c
#include <unistd.h>

int getgroups(int gidsetsize, gid_t grouplist[]);
```

`setgroups()` sets the group access list of the current user process according to the array gidset.  The parameter ngroups indicates the number of entries in the array and must be no more than {NGROUPS_MAX}
```c
#include <sys/param.h>
#include <unistd.h>

int setgroups(int ngroups, const gid_t *gidset);
```

The `initgroups()` function calculates a group access list for the user specified in name.  This group list is then saved in the kernel credentials for the current process.  The basegid is included in the groups  list.  Typically this value is given as the default group associated with the user's account record.
```c
#include <unistd.h>

int initgroups(const char *name, int basegid);
```

### 登陆账户记录

大多数UNIX系统提供了两个数据文件：

* utmp 文件记录当前登陆到系统的各个用户。
* wtmp 文件跟踪各个登陆和注销事件。

### 系统标识

主机与操作系统有关的信息。

```c
#include <sys/utsname.h>

int uname(struct utsname *name);
```

```shell
> uname -a
Darwin haifeis-MacBook-Pro.local 18.6.0 Darwin Kernel Version 18.6.0: Thu Apr 25 23:16:27 PDT 2019; root:xnu-4903.261.4~2/RELEASE_X86_64 x86_64
```

获取主机名

```c
#include <unistd.h>

int gethostname(char *name, size_t namelen);
int sethostname(const char *name, int namelen);
```

### 时间与日期例程

由Linux内核提供的基本时间服务是计算自协调世界时（Coordinated Universal Time, UTC）时间。UNIX与其他系统的区别是：

1. 以协调统一时间而非本地时间记时
2. 可自动进行转换
3. 将时间与日期作为一个量值保存

![图片.png](https://upload-images.jianshu.io/upload_images/1916953-5f71bcd876d773d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 习题

### **1. 如果系统使用阴影文件，那么如何取得加密口令？**

使用口令文件中的函数直接获取`passwd`对象中的`pw_passwd`字段。该字段不能直接与加密口令做比较，因为此字段不是加密的口令。正确的方法是使用阴影口令文件中的加密口令进行比较。

### 2. **假设你有超级用户权限，并且系统使用了阴影口令，重新考虑上一道习题。**

阴影口令的获取，在需要系统中都需要root超级用户权限。如：

Linux3.0 和 solaris 10 ，非超级用户权限，调用`getspnam`将返回`EACCES`错误

Mac OSX 10.6.8 即使拥有超级用户权限`pw_passwd`返回值也会为 * 号;

### 3. **编写一程序，它调用 uname 并且输出 utsname 结构中的所有字段，将该输出与 uname 命令的输出结果进行比较。**

![image.png](https://upload-images.jianshu.io/upload_images/1916953-2265354f176f45c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4. **计算可由 time_t 数据类型表示的最近时间。如果超出了这一时间将会如何？**

根据系统不同 time_t 可能是 long int 或者 int 。如果是 int 存储的话，那么到 2038 年将达到上限，之后将会溢出，时间变成负数。

### 5. **编写一程序，获取当前时间，并使用 strftime 将输出结果转换为类似于 date 命令的默认输出。将环境变量 TZ 设置为不同值，观察输出结果。**

```c
#include "apue.h"
#include <time.h>

int main() {
    time_t caltime;
    struct tm *tm;
    char line[MAXLINE];

    if ((caltime = time(NULL)) == -1) {
        err_sys("time error");
    }
    if ((tm = localtime(&caltime)) == NULL) {
        err_sys("localtime error");
    }
    if (strftime(line, MAXLINE, "%a %b %d %X %Z %Y\n", tm) == 0) {
        err_sys("strftime error");
    }
    fputs(line, stdout);
    exit(0);
}
```

