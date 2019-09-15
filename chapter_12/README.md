[TOC]

## 线程控制

### 线程限制

Single UNIX Specification 定义了与线程操作相关的一些操作。

| 限制名称                      | 描述                                                 | name参数                         |
| ----------------------------- | ---------------------------------------------------- | -------------------------------- |
| PTHREAD_DESTRUCTOR_ITERATIONS | 线程退出时操作系统实现试图销毁线程特定数据的最大次数 | _SC_THREAD_DESTRUCTOR_ITERATIONS |
| PTHREAD_KEYS_MAX              | 进程可以创建键的最大数目                             | _SC_THREAD_KEYS_MAX              |
| PTHREAD_STACK_MIN             | 一个线程的栈可用的最小字节数                         | _SC_PTHREAD_STACK_MIN            |
| PTHREAD_THREADS_MAX           | 进程可以创建的最大线程数                             | _SC_PTHREAD_THREADS_MAX          |

![image.png](https://upload-images.jianshu.io/upload_images/1916953-4c4ebfb29b13d3f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 线程属性

pthread 接口允许我们通过设置每个对象关联的不同属性来细调线程和同步对象的行为。

```c
#include <pthread.h>

int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
int pthread_attr_setstack(pthread_attr_t *attr, void *stackaddr, size_t stacksize);
int pthread_attr_getstack(const pthread_attr_t * restrict attr, void ** restrict stackaddr, size_t * restrict stacksize);
int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
int pthread_attr_getstacksize(const pthread_attr_t *attr, size_t *stacksize);
int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
int pthread_attr_getguardsize(const pthread_attr_t *attr, size_t *guardsize);
int pthread_attr_setstackaddr(pthread_attr_t *attr, void *stackaddr);
int pthread_attr_getstackaddr(const pthread_attr_t *attr, void **stackaddr);
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
int pthread_attr_setinheritsched(pthread_attr_t *attr, int inheritsched);
int pthread_attr_getinheritsched(const pthread_attr_t *attr, int *inheritsched);
int pthread_attr_setschedparam(pthread_attr_t *attr, const struct sched_param *param);
int pthread_attr_getschedparam(const pthread_attr_t *attr, struct sched_param *param);
int pthread_attr_setschedpolicy(pthread_attr_t *attr, int policy);
int pthread_attr_getschedpolicy(const pthread_attr_t *attr, int *policy);
int pthread_attr_setscope(pthread_attr_t *attr, int contentionscope);
int pthread_attr_getscope(const pthread_attr_t *attr, int *contentionscope);
```

Thread attributes are used to specify parameters to `pthread_create()`.  One attribute object can be used in multiple calls to pthread_create(), with or without modifications between calls.

* The `pthread_attr_init()` function initializes attr with all the default thread attributes.
* The `pthread_attr_destroy()` function destroys attr.
* The `pthread_attr_set()` functions set the attribute that corresponds to each function name.
* The` pthread_attr_get()` functions copy the value of the attribute that corresponds to each function name to the location pointed to by the second function parameter.

### 同步属性

#### 互斥量属性	

初始化与销毁

```c
#include <pthread.h>

int pthread_mutexattr_init(pthread_mutexattr_t *attr);
int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);
```

互斥量具有三种属性：**进程共享属性**、**健壮属性**、**类型属性**。

获取和修改**进程共享属性**：

* `PTHREAD_PROCESS_PRIVATE`
* `PTHREAD_PROCESS_SHARED`

```c
#include <pthread.h>

int pthread_mutexattr_getpshared(const pthread_mutexattr *attr, *restrict attr, int *restrict pshared);
int phtread_mutexattr_setpshared(pthread_mutexattr_t *attr, int pshared);
```

获取与设置**健壮属性**的互斥量属性：

* `PTHRAD_MUTEX_STALLED`默认
* `PTHRAD_MUTEX_ROBUST`

```c
#include <pthread.h>

int pthread_mutexattr_getrobust(const pthread_mutexattr *attr, *restrict attr, int *restrict robust);
int phtread_mutexattr_setrobust(pthread_mutexattr_t *attr, int robust);
```

判断与该互斥量相关的状态在互斥量之前是一致的。

```c
#include <pthread.h>

int pthread_mutex_consistent(pthread_mutex_t *mutex);
```

**类型互斥量属性**控制着互斥量的锁定特性。

| 类型                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `PTHREAD_MUTEX_NORMAL`     | 一种标准的互斥量属性，不做任何特殊的错误检查或死锁检测。     |
| `PTHREAD_MUTEX_ERRORCHECK` | 此类型提供错误检查。                                         |
| `PTHREAD_MUTEX_RECURSIVE`  | 允许同一线程在互斥量解锁之前对该互斥量进程多次加锁。递归互斥量维护锁的计数，在解锁次数和加锁次数不同的情况下，不会释放锁。 |
| `PTHREAD_MUTEX_DEFAULT`    | 提供默认的特性与属性。                                       |

```c
#include <pthread.h>

int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type);
int pthread_mutexattr_gettype(pthread_mutexattr_t *attr, int *type);
```

#### 读写锁属性

```c
#include <pthread.h>

int pthread_rwlockattr_init(pthread_rwlockattr_t *attr);
int pthread_rwlockattr_destroy(pthread_rwlockattr_t *attr);
```

读写锁支持的唯一属性是**进程共享**。

读取与设置读写锁的进程共享属性：

```c
#include <pthread.h>

int pthread_rwlockattr_getshared(pthread_rwlockattr_t *attr, int *restrict pshared);
int pthread_rwlockattr_setshared(pthread_rwlockattr_t *attr, int *restrict pshared);
```

#### 条件变量属性

条件变量定义了两个属性：**进程共享属性**和**时钟属性**。

初始化：

```c
#include <pthread.h>

int pthread_condattr_init(pthread_condattr_t *attr);
int pthread_condattr_destroy(pthread_condattr_t *attr);
```

获取与设置进程共享属性：

```c
#include <pthread.h>

int pthread_condattr_getpshared(pthread_condattr_t *attr, int *restrict pshared);
int pthread_condattr_setpshared(pthread_condattr_t *attr, int *restrit pshared);
```

时钟属性控制计算`pthread_cond_timedwait`函数的超时参数时使用哪个时钟。

设置与获取时钟属性：

```c
#include <pthread.h>

int pthread_condattr_getclock(pthread_condattr_t *attr, int *restrict clock_id);
int pthread_condattr_setclock(pthread_condattr_t *attr, int *restrit clock_id);
```

#### 屏障属性

屏障属性只有**进程共享属性**。

```c
#include <pthread.h>

int pthread_barrierattr_init(pthread_barrierattr_t *attr);
int pthread_barrierattr_destroy(pthread_barrierattr_t *attr);
```

### 重入

如果一个函数在相同的时间点可以被多个线程安全的调用，就称为该函数是**线程安全的**。

如果一个函数对于多个线程是可重入的，就说这个函数是线程安全的。但这并不说明对信号处理程序来说该函数也是可重入的。如果函数对于异步信号处理程序的重入是安全的，那么说明函数是**异步信号安全的**。

### 线程特定数据

**线程特定数据 thread-specific data**也称为线程私有数据，是存储和查询某个特定线程相关数据的一种机制。我们把这种数据称为线程特定数据或线程私有数据的原因是，我们希望每个线程可以访问它自己单独的数据副本，而不需要担心与其他线程的同步访问问题。

创建与删除键

第二个参数为析构函数，当线程退出时，如果数据地址已经被置为非空值，那么析构函数就会被调用。

```c
#include <pthread.h>

int pthread_key_create(pthread_key_t *key, void (*destructor)(void *));
int pthread_key_delete(pthread_key_t *key);
```

* The `pthread_key_create()` function creates a thread-specific data key visible to all threads in the process.  Key values provided by pthread_key_create() are opaque objects used to locate thread-specific data.

* Although the same key value may be used by different threads, the values bound to the key by pthread_setspecific() are maintained on a per-thread basis and persist for the life of the calling thread. 

```c
#include <pthread.h>

pthread_once_t once_control = PTHREAD_ONCE_INIT;

int pthread_once(pthread_once_t *once_control, void (*init_routine)(void));
```

init_routine只有在第一次才会被调用

The first call to `pthread_once()` by any thread in a process, with a given once_control, will call the `init_routine()` with no arguments.  Subsequent calls to `pthread_once()` with the same once_control will not call the `init_routine()`.  On return from `pthread_once()`, it is guaranteed that init_routine() has completed.  The once_control parameter is used to determine whether the associated initialization routine has been called.

关联键与线程特定数据：

```c
#include <pthread.h>

void * pthread_getspecific(pthread_key_t key);
int pthread_setspecific(pthread_key_t key, const void *value);
```

The **pthread_setspecific()** function associates a thread-specific value with a key obtained via a previous call to pthread_key_create().  Different threads can bind different values to the same key.  These val-ues are typically pointers to blocks of dynamically allocated memory that have been reserved for use by the calling thread.

### 取消选项

有两个属性没有包含在`pthread_attr_t`结构中，它们时**可取消状态**和**可取消类型**。这两个属性影响着线程在响应**pthread_cancel**函数调用时所呈现的行为。

```c
#include <pthread.h>

int pthread_setcancelstate(int state, int *oldstate);
int pthread_setcanceltype(int type, int *oldtype);
void pthread_testcancel(void);
```

* The `pthread_setcancelstate()` function atomically both sets the calling thread's cancelability state to the indicated state and, if oldstate is not NULL, returns the previous cancelability state at the location referenced by oldstate.  Legal values for state are `PTHREAD_CANCEL_ENABLE` and `PTHREAD_CANCEL_DISABLE`.
* The `pthread_setcanceltype()` function atomically both sets the calling thread's cancelability type to the indicated type and, if oldtype is not NULL, returns the previous cancelability type at the location ref-erenced by oldtype.  Legal values for type are `PTHREAD_CANCEL_DEFERRED` and `PTHREAD_CANCEL_ASYNCHRONOUS`.
* The `pthread_testcancel()` function creates a cancellation point in the calling thread.  The pthread_testcancel() function has no effect if cancelability is disabled.

调用`pthread_setcancelstate`设置为PTHREAD_CANCEL_ENABLE，在取消请求被调用后，线程会在下个取消点，对所有挂起的取消请求进程处理。

`pthread_testcancel`用来添加自己的取消点。

调用`pthread_cancel`以后，在线程到达取消点之前，并不会出现真正的取消。可以通过调用`pthread_setcanceltype`来修改取消类型。异步取消与推迟取消不同，因为使用异步取消时，线程可以在任意时间撤销。

### 线程与信号

每个线程都有自己的信号屏蔽字，但是信号的处理时进程中所有线程共享的。这意味着单个线程可以阻止某些信号，但当某个线程修改了与某个给定信号相关的处理行为后，所有的线程都必须共享这个行为的改变。

```c
#include <signal.h>

int pthread_sigmask(int how, const sigset_t * restrict set, sigset_t * restrict oset);
```

The `pthread_sigmask()` function examines and/or changes the calling thread's signal mask.

线程可以调用`sigwait`等待一个或多个信号的出现。

```c
#include <signal.h>

int sigwait(const sigset_t *restrict set, int *restrict sig);
```

set指定线程等待的信号集，sig 指向的整数包含了发送信号的数量。

要把信号发送给进程，可以调用kill。要把信号发送给线程，可以调用 pthread_kill。

```c
#include <signal.h>

int pthread_kill(pthread_t thread, int sig);
```

The `pthread_kill()` function sends a signal, specified by sig, to a thread, specified by thread.  If sig is 0, error checking is performed, but no signal is actually sent.

### 线程和fork

当线程调用`fork`时，就为子进程创建了整个个进程地址空间的副本。

子进程通过继承整个地址空间的副本，还从父进程那儿继承了每个互斥量、读写锁，和条件变量的状态。如果父进程包含一个以上的线程，子进程在fork返回后，如果紧接着不是马上调用 exec 时，就需要清理锁状态。

要清理锁状态，可以通过pthread_atfork函数建立fork处理程序。

```c
#include <pthread.h>

int pthread_atfork(void (*prepare)(void), void (*parent)(void), void (*child)(void));
```

he `pthread_atfork()` function declares fork handlers to be called before and after fork(2), in the context of the thread that called fork(2).

The handlers registered with `pthread_atfork()` are called at the moments in time described below:

* `prepare`  Before fork(2) processing commences in the parent process.  If more than one prepare handler is registered they will be called in the opposite order they were registered.

* `parent`   After fork(2) completes in the parent process.  If more than one parent handler is registered they will be called in the same order they were registered.

* `child`    After fork(2) processing completes in the child process.  If more than one child handler is registered they will be called in the same order they were registered.

### 线程与I/O

3.11 节介绍的`pread`和`pwrite`函数。这些函数在多线程环境中时非常有用的，因为进程中的所有线程共享相同的文件描述符。

## 习题：

### 12.1**在 Linux 系统中运行图 12-17 中的程序,但把输出结果重定向到一个文件中,并解释结果.**

在 IO 章节中提到，在输出到控制台时，标准输出为行缓存的，也就是说每一行输出都会冲洗缓冲区；而输出到文件时，标准输出为全缓冲，因此，在 fork 子进程的时候，缓冲区还没有被冲洗，因此子进程也带有父进程的缓冲区中尚未输出的部分。

### 12.2**实现 putenv_r ，即 putenv 的可重入版本，确保你的实现既是线程安全的，也是异步信号安全的。**
### 12.3**是否可以通过在 getenv 函数开始的时候阻塞信号,并在 getenv 函数返回之前回复原来的信号屏蔽字这种方法,让图 12-13 中的 getenv 函数变成异步安全的? 解释其原因.**
理论上来说,如果在信号处理程序运行时阻塞所有的信号, 那么就能使函数成为异步信号安全的. 问题是我们并不能知道调用的某个函数可能并没有屏蔽已经被阻塞的信号, 这样通过另一个信号处理程序可能会使该函数变成可重入的.

### 12.4**写一个程序练习图 12-13 中的 getenv 版本, 在 FreeBSD 上编译并运行程序, 会出现什么结果? 解释其原因.**


### 12.5**假设可以在一个程序中创建多个线程执行不同的任务, 为什么还是有可能会需要用 fork? 解释其原因.**
如果希望在一个程序中运行另一个程序, 还需要 fork (即在调用 exec 之前)

### 12.6**重新实现图 12-29 中的程序, 在不使用 nanosleep 或 clock_nanosleep 的情况下使它成为线程安全的.**


### 12.7**调用 fork 以后, 是否可以通过首先用 pthread_cond_destroy 销毁条件变量,然后用 pthread_cond_init 初始化条件变量这种方法安全地在子进程中对条件变量进行重新初始化?**


### 12.8**图 12-8 中的 timeout 函数可以大大简化, 解释其原因.**

