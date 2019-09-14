[TOC]

## 信号

### 信号概念

信号时一种软件中断，它提供了一种处理异步事件的方法。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-2b2dd3fb57d02764.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 函数`signal`

```c
 #include <signal.h>

void (*signal(int sig, void (*func)(int)))(int);
```

`sig`参数是信号名。

`func`的值为：

* SIG_IGN: 忽略信号
* SIG_DFL: 收到信号后，执行系统默认动作
* 接收到信号后的，函数调用地址。

`kill(1)`与`kill(2)`函数只是将一个信号发送给一个进程或进程组。该信号是否终止进程则取决于该信号的类型。

随书代码：

![](https://github.com/Oaks907/APUE_note/tree/master/chapter_10/img/10.3.png)

### 不可靠的信号

早期的UNIX版本中，信号是不可靠的。不可靠指的是，信号可能会丢失。

早期版本存在的问题：

1. 进程在每次接收到信号对齐处理时，随即将该信号动作重置为默认值。
2. 当进程不希望某种信号发生时，它不能关闭该信号。进程能做的一切是忽略该信号。

### 中断的系统调用

早期的UNIX系统的一个特性：如果进程在执行一个低速系统调用而阻塞期间捕获到一个信号，则该系统调用就被中断不再继续执行。该系统调用返回出错，其 errno 设置为 EINTR。

必须区分系统调用与函数，当捕捉到某个信号时，被中断的是内核中执行的系统调用。

为了支持这种特性，该系统调用分为两类：**低速系统调用**和**其他系统调用**。低速系统调用时可能会使进程永远阻塞的一类系统调用。

### 可重入函数

进程捕获到信号并对其进行处理时，进程正常执行的正常指令序列会被信号处理程序临时中断，它首先执行该信号处理程序中的指令。如果从信号处理程序中返回，则继续执行在捕获到信号时进程正在执行的指令序列。

但是正常的指令序列执行过程中，不能判断捕捉到信号时进程执行到了何处。如果进程正在执行malloc，因为内含多个步骤，会出现数据不一致的问题。

Single UNIX Specification 说明了信号处理程序中保证调用安全的函数。这些函数是**可重入的**并被称为**异步信号安全**。除了可重入之外，在信号处理操作期间，它会阻塞任何会引起不一致的信号发送。

### SIGCLD语义

对于SIGCLD信号的早期处理方式：

* 如果进程明确地将该信号的配置设置为SIG_IGN，则调用进程的子进程将不产生僵死进程。子进程在终止时，将其状态丢弃。如果调用进程随后调用了一个wait函数，那么它将阻塞知道所有子进程都终止。随后该wait会返回 -1，并将其errno设置为ECHILD。
* 如果将SIGCLD设置为捕捉，则内核立即检查是否有子进程准备好被等待，如果是这样，则调用SIGCLD处理程序。

### 函数`kill`和`raise`

```c
 #include <signal.h>

int kill(pid_t pid, int sig);
int raise(int sig);
```

`kill`函数将信号发送给进程或进程组。`raise`函数则允许进程向自身发送信号。

`reise(signo)`等价于 `kill(getpid(), signo)`

pid参数的不同情况：

* `pid > 0`信号发送给进程ID为 pid 的进程
* `pid == 0`信号发送给与发送进程属于同一进程组的所有进程，而且发送进程必须具有权限发送信号向这些进程。
* `pid< 0`信号发送给其进程组ID等于pid绝对值，而且发送进程必须具有权限发送信号向这些进程。
* `pid == -1`信号发送给发送进程有权限向它们发送信号的所有进程。

### 函数`alarm`和`pause`

`alarm`函数会设置一个定时器，在将来某个事件，该定时器在某个时刻会超时，对外发送 `SIGALRM`信号。如果忽略或不捕捉该信号，默认是终止调用该alarm函数的进程。

```c
#include <unistd.h>

unsigned alarm(unsigned seconds);
```

`pass`挂起进程，直到捕捉到一个信号。

```c
#include <unistd.h>

int pause(void);
```

### 信号集

信号集用来表示一个或者多个信号。UNIX通过下面函数维持了一个信号集，以便告诉内核不允许发生该信号集中的信号。

```c
#include <signal.h>

int sigaddset(sigset_t *set, int signo);
int sigdelset(sigset_t *set, int signo);
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigismember(const sigset_t *set, int signo);
```

### 函数`sigprocmask`

设置当前阻塞而不能传递给该进程的信号集。

```c
#include <signal.h>

int sigprocmask(int how, const sigset_t *restrict set, sigset_t *restrict oset);
```

how参数取值

* `SIG_BLOCK`    The new mask is the union of the current mask and the specified set.
* `SIG_UNBLOCK`  The new mask is the intersection of the current mask and the complement of the specified set.
* `SIG_SETMASK`  The current mask is replaced by the specified set.

`oset`当前信号屏蔽字通过该指针返回。

### 函数`sigpending`

```c
#include <signal.h>

int sigpending(sigset_t *set);
```

返回同一信号集，对于调用进程而言，其中的各个信号是不能递送的。

### 函数`sigaction`

```c
#include <signal.h>


     struct  sigaction {
             union __sigaction_u __sigaction_u;  /* signal handler */
             sigset_t sa_mask;               /* signal mask to apply */
             int     sa_flags;               /* see signal options below */
     };

     union __sigaction_u {
             void    (*__sa_handler)(int);
             void    (*__sa_sigaction)(int, siginfo_t *,
                            void *);
     };

     #define sa_handler      __sigaction_u.__sa_handler
     #define sa_sigaction    __sigaction_u.__sa_sigaction


int sigaction(int sig, const struct sigaction *restrict act, struct sigaction *restrict oact);
```

检查和修改与指定信号相关的动作。

### 函数`sigsetjmp`和`siglongjmp`

信号处理程序中，进行非局部转移时，应该使用这两个函数。

```c
#include <setjmp.h>

void siglongjmp(sigjmp_buf env, int val);
int sigsetjmp(sigjmp_buf env, int savemask);
```

The `sigsetjmp()/siglongjmp()` function pairs save and restore the signal mask if the argument savemask is non-zero; otherwise, only the register set and the stack are saved

### 函数`sigsuspend`

```c
#include <signal.h>

int sigsuspend(const sigset_t *sigmask);
```

`sigsuspend()` temporarily changes the blocked signal mask to the set to which sigmask points, and then waits for a signal to arrive; on return the previous set of masked signals is restored.  The signal mask set is usually empty to indicate that all signals are to be unblocked for the duration of the call.

屏蔽新的信号，原来的屏蔽字失效。当信号发生，并是非屏蔽信号时，suspend返回，并且恢复旧的屏蔽字。

### 函数 `abort`

The `abort()` function causes abnormal program termination to occur, unless the signal `SIGABRT` is being caught and the signal handler does not return.

```c
#include <stdlib.h>

void abort(void);
```

### 函数`sleep`、`nanosleep`、`clock_nanosleep`

```c
#include <unistd.h>

unsigned int sleep(unsigned int seconds);
```

The sleep() function suspends execution of the calling thread until either seconds seconds have elapsed or a signal is delivered to the thread and its action is to invoke a signal-catching function or to terminate the thread or process.  System activity may lengthen the sleep by an indeterminate amount.

```c
#include <time.h>

int clock_nanosleep(clockid_t clock_id, int flags, const struct timespec * reqtp, struct timespec *remtp);
```

`clock_id`指定了计算延迟时间基于的时钟。

`flags`用于控制延时是相对的，还是绝对的。

除了出错返回，`nanosleep`与`clock_nanosleep`效果是相同的。

### 函数`sigqueue`

对信号排队的支持。使用排队信号需要做一下几个操作：

1. 使用`sigaction`函数安装信号处理程序时指定`SA_SIGINFO`标志。如果没有给出这个标志，信号会延迟，但信号是否进入队列要取决于具体的实现。
2. 在`sigaction`结构中的 `sa_sigaction`成员中提供信号处理程序。实现可能允许用户使用sa_handler字段，但不能获取sigqueue函数发出的额外信息。
3. 使用`sigqueue`发送信号 

```c
#include <signal.h>

intt sigqueue(pid_t pid, int signo, const union sigval value);
```

sigqueue只能向单个进程发送信号，是可以使用value向信号处理程序传递整数和指针值，除此之外，sigueue与kill相同。

### 作业控制信号

### 信号名和编号

在信号编号与信号名之间进行映射。

`psignal`函数可移植地打印与信号编号对应的字符串。

```c
#include <signal.h>

void psignal(int signo, const char *msg);
```

`sigaction`信号处理程序中包含signifo结构，可以使用psiginfo函数打印信号信息。

```c
#include <signal.h>

void psiginfo(const siginfo_t *info, const char *msg);
```

## 习题

### 1. **删除图 10-2 程序中的 for(;;) 语句，结果会怎样？为什么？**

`pause` 在捕捉到信号的时候就返回了，没有循环调用 `pause` ，因此程序在第一次传递信号后结束了。

### 2. **实现 10.22 节中说明的 sig2str 函数**

见[p2.c](https://github.com/Oaks907/APUE_note/tree/master/chapter_10/p2.c)

### 3. **画出运行图 10-9 程序时的栈帧情况**

![image.png](https://upload-images.jianshu.io/upload_images/1916953-88ace77892950e8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4. **图 10-11 程序中利用 setjmp 和 longjmp 设置 I/O 操作的超时，下面的代码也常见用于此种目的：**

```
signal(SIGALRM, sig_alrm);
alarm(60);
if (setjmp(env_alrm) != 0) {
    /* handle timeout */
}
```

**这段代码有什么错误？**

在第一次调用 `alarm` 和 `setjmp` 之间有一次竞争条件。如果进程在调用 `alarm` 和 `setjmp` 之间被内核阻塞了，闹钟时间超过后就调用信号处理程序，然后调用 `longjmp` 。但是由于没有调用过 `setjmp` ，所以没有设置 `env_alrm` 缓冲区。如果 `longjmp` 的跳转缓冲区没有被 `setjmp` 初始化，则说明 `longjmp` 的操作是未定义的。

### 5. **仅适用一个定时器（alarm 或较高精度的 setitimer），构造一组函数，使得进程在该单一定时器基础上可以设置任意数量的定时器。**

见[p5.c](https://github.com/Oaks907/APUE_note/tree/master/chapter_10/p5.c)

### 6. **编写一段程序测试图 10-24 中父进程和子进程的同步函数，要求进程创建一个文件并向文件写一个整数 0, ，然后，进程调用 fork ，接着，父进程和子进程交替增加文件中的计数器值，每次计数器值增加 1 时，打印是哪一个进程（子进程或父进程）进行了该增加 1 操作。**

见[p6.c](https://github.com/Oaks907/APUE_note/tree/master/chapter_10/p6.c)

### 7. **在图 10-25 中，若调用者捕捉了 SIGABRT 并从该信号处理程序返回，为什么不是仅仅调用 _exit ，而要恢复其默认设置并再次调用 kill？**

如果仅仅调用 _exit ，则进程终止状态不能表示该进程是由于 SIGABRT 信号而终止的。

### 8. **为什么在 siginfo 结构（见 10.14 节）的 si_uid 字段中包括实际用户 ID 而非有效用户 ID ？**

如果信号是由其他用户的进程发出的，进程必须设置用户 ID 为根或者是接受进程的所有者，否则 kill 不能执行。所以实际用户 ID 为信号的接受者提供了更多的信息。

### 9. **重写图 10-14 中的函数，要求它处理图 10-1 中的所有信号，每次循环处理当前信号屏蔽字中的一个信号（并不是对每一个可能的信号都循环一遍）。**

### 10. **编写一段程序，要求在一个无限循环中调用 sleep(60) 函数，每 5 分子（即 5 次循环）取当前的日期和时间，并打印 tm_sec 字段。将程序执行一晚上，请解释其结果。有些程序，如 cron 守护进程，每分钟运行一次，它是如何处理这类工作的？**

对于本书作者所用的一个系统，每 60~90 分钟增加一秒，这个误差是由于每次调用 `sleep` 都要调度一次将来的时间事件，但是由于 CPU 调度，有事并没有在事件发生时立即被唤醒。另外一个原因是进程开始运行和再次调用 `sleep` 都需要一定的时间。

cron 守护进程这样的程序每分钟都要获取当前时间，它首先设置一个休眠周期，然后在下一分钟开始时唤醒。（将当前时间转换为本地时间并查看 tm_sec 值）。每一分钟，设置下一个休眠周期，使得在下一分钟开始时可以唤醒。大多数调用是 `sleep(60)` ，偶尔有一个 `sleep(59)` 用于在下一分钟同步。但是，若在进程中花费了很多时间执行命令或者系统的负载重、调度慢，这时休眠值可能远小于 60 。

### 11.**修改图 3-5 的程序，要求：（a）将 BUFFSIZE 改为 100 ；（b）用 signal_intr 函数捕捉 SIGXFSZ 信号量并打印消息，然后从信号处理程序中返回；（c）如果没有写满请求的字节数，则打印 write 的返回值。将软资源限制 RLIMIT_FSIZE （见 7.11 节）更改为 1024 字节（在 shell 中设置软资源限制，如果不行就直接在程序中调用 setrlimit），然后复制一个大于 1024 字节的文件，在各种不同的系统上运行新程序，其结果如何？为什么？**

在 Linux 3.2.0 、 Mac OS X 10.6.8 和 Solaris 10 中，从来没有调用过 SIGXFSZ 的信号处理程序，一旦文件的大小打到 1024 时， write 就返回 24.

在 FreeBSD 8.0 中，当文件大小已达到 1000 字节，在下一次准备写入 100 字节时调用该信号处理程序， write 返回 -1 ，并且将 errno 设置为 EFBIG （文件太大）。

在所有的 4 中平台上，如果在当前文件偏移量处（文件尾端）尝试再一次 write ，将收到 SIGXFSZ 信号， write 将失败，返回 -1 ，并将 errno 设置为 EFBIG 。

### ### 12. **编写一段调用 fwrite 的程序，它使用一个较大的缓冲区（约 1GB ），调用 fwrite 钱调用 alarm 使得 1s 以后产生信号。在信号处理程序中打印捕捉到的信号，然后返回。 fwrite 可以完成吗？结果如何？**

结果依赖于标准 I/O 库的实现： fwrite 函数如何处理一个被中断的 write 。

例如，在 Linux 3.2.0 上，当使用 fwrite 函数写一个大的缓冲区时， fwrite 以相同的字节数直接调用 write 。在 write 系统调用中，闹钟时间到，但我们直到写结束才看到信号。**看上去就好像在 write 系统调用进行当中内核阻塞了信号**。

与此不同的是，在 Solaris 10 中， fwrite 函数调用以 8 KB 的增量调用 write ，直到写完整个要求的字节数。当闹钟时间到，会被捕捉到，中断 write 回到 fwrite 。当从信号处理程序返回时，返回到 fwrite 函数内部的循环，并继续以 8KB 的增量写。