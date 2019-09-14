[TOC]

## 线程

### 线程标识

像进程一样，每个线程都有一个线程ID。进程ID在整个系统中时唯一的，但线程不同，线程ID只有在它所属的进程上下文中才有意义。

比较两个线程：The `pthread_equal()` function will return non-zero if the thread IDs t1 and t2 correspond to the same thread, otherwise it will return zero.

```c
#include <pthread.h>

int pthread_equal(pthread_t t1, pthread_t t2);
```

获取线程信息：

```c
#include <pthread.h>

pthread_t pthread_self(void);
```

### 线程创建

```c
#include <pthread.h>

int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void *), void *arg);
```

* The `pthread_create()` function is used to create a new thread, with attributes specified by attr, within a process.  

* If `attr` is NULL, the default attributes are used.  If the attributes specified by attr are modified later, the thread's attributes are not affected.  

* Upon successful completion pthread_create() will store the ID of the created thread in the location specified by `thread`.
* The thread is created executing `start_routine` with `arg` as its sole argument.  If the start_routine returns, the effect is as if there was an implicit call to `pthread_exit()` using the return value of start_routine as the exit status.

### 线程终止

线程终止，通过三种方式退出：

1. 线程可以简单从启动例程中返回，返回值时线程的退出码。
2. 线程可以被同一进程中的其他线程取消。
3. 线程调用pthread_exit;

```c
#include <pthread.h>

void pthread_exit(void *value_ptr);
```

* The `pthread_exit()` function terminates the calling thread and makes the value `value_ptr` available to any successful join with the terminating thread.  
* Any cancellation cleanup handlers that have been pushed and are not yet popped are popped in the reverse order that they were pushed and then executed.  After all cancellation handlers have been executed, if the thread has any thread-specific data, appropriate destructor functions are called in an unspecified order.  
* Thread termination does not release any application visible process resources, including, but not limited to, mutexes and file descriptors, nor does it  perform any process level cleanup actions, including, but not limited to, calling atexit() routines that may exist.

`value_ptr`参数时一个无类型指针，与传给启动例程的单个参数类似。进程中的其他线程可以通过调用`pthread_join`访问到这个指针。`pthread_join`会将调用线程一直阻塞，直到 thread 调用返回。

```c
#include <pthread.h>

int pthread_join(pthread_t thread, void **value_ptr);
```

线程可以通过调用`pthread_cancel`函数来请求取消同一进程中的其他线程。

```c
#include <pthread.h>

int pthread_cancel(pthread_t thread);
```

* The `pthread_cancel()` function requests that thread be canceled.  The target thread's cancelability state and type determines when the cancellation takes effect.  

* When the cancellation is acted on, the cancel-lation cleanup handlers for thread are called.  When the last cancellation cleanup handler returns, the thread-specific data destructor functions will be called for thread.  When the last destructor function returns, thread will be terminated.

线程可以安排它退出时需要调用的函数，这与进程在退出时可以用**atexit**函数安排退出类似。这种函数称为**线程清理程序(thread clean handler)**。

```c
#include <pthread.h>

void pthread_cleanup_push(void (*cleanup_routine)(void *), void *arg);
void pthread_cleanup_pop(int execute);
```



### 线程同步

#### 互斥量

通过对数据加锁的方式，限制同一时间只有一个线程访问数据。

初始化操作：

```c
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

加解锁操作：

```c
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

#### 函数pthread_mutex_timedlock

带有超时时间的加锁：

```c
#include <pthread.h>

int pthread_mutex_timedlock(pthread_mutex_t *mutex, const struct timespec *restrict tsptr);
```

#### 读写锁

写锁互斥，读锁共享

```c
#include <pthread.h>

int pthread_rwlock_init(pthread_rwlock_t *lock, const pthread_rwlockattr_t *attr);
int pthread_rwlock_destroy(pthread_rwlock_t *lock);

int pthread_rwlock_rdlock(pthread_rwlock_t *lock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t *lock);
int pthread_rwlock_wrlock(pthread_rwlock_t *lock);
int pthread_rwlock_trywrlock(pthread_rwlock_t *lock);
int pthread_rwlock_unlock(pthread_rwlock_t *lock);
```

#### 带有超时的读写锁

```c
#include <pthread.h>
#include <time.h>

int pthread_rwlock_timedrdlock(pthread_rwlock_t *lock, const struct timespec *restrict tsptr);
int pthread_rwlock_timedwrlock(pthread_rwlock_t *lock, const struct timespec *restrict tsptr);
```

#### 条件变量

条件变量给多个线程提供了一个会和的场所。条件变量和互斥量一起使用，允许线程以无竞争的方式等待特定的条件发生。

```c
#include <pthread.h>

int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr);
int pthread_cond_destroy(pthread_cond_t *cond);

int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime);
```

#### 自旋锁

自旋锁不会通过休眠使线程阻塞，而是在获取锁之前一直处于盲等的状态阻塞状态。自旋锁一般用于下面的场景：锁被持有的时间短，而且线程不希望在重新调度上花费太多的成本。

#### 屏障

屏障允许多个线程等待，知道所有合作的线程到达某一点，然后由该点开始运行。

```c
#include <pthread.h>

int pthread_barrier_init(pthread_barrier_t *restrict barrier, const pthread_barrierattr_t *restrict attr, unsigned int count);
int pthread_barrier_wait(pthread_barrier_t *barrier);
```

## 习题

### **1. 修改图 11-4 所示的实例代码，正确的在两个线程之前传递结构**

使用动态内存分配，使用时赋值。见[p1.c](https://github.com/Oaks907/APUE_note/tree/master/chapter_11/p1.c)

### 2. **在图 11-14 所示的实例代码中，需要另外添加什么同步（如果需要的话）可以使得主线程改变与挂起作业关联的线程 ID ？这会对 job_remove 函数产生什么影响？**

要改变挂起作业的线程 ID ，必须持有写模式下的读写锁，防止 ID 在改变过程中有其他线程在搜索该列表。目前定义该接口的方式存在的问题在于：调用 job_find 找到该作业以及调用 job_remove 从列表中删除该作业这两个时间之间作业 ID 可以改动。这个问题可以通过在 job 结构中嵌入引用计数和互斥量，然后让 job_find 增加引用计数的方法来解决。这样修改 ID 的代码就可以避免对列表中非零引用计数的任何作业进行 ID 改动的情况。

### 3. **把图 11-15 中的技术运用到工作线程实例（图 11-1 和图 11-14 ）中实现工作线程函数。不要忘记更新 queue_init 函数对条件变量进行初始化，修改 job_insert 和 job_append 函数给工作线程发信号。会出现什么样的困难？**

首先，列表是由读写锁保护的，但条件变量需要互斥量对条件进行保护。其次，每个线程等待满足的条件应该是有某个作业进行处理时需要的条件，所以需要创建每线程数据结构来表示这个条件。或者，可以把互斥量和条件变量嵌入到 queue 结构中，但这意味着所有的工作线程将等待相同的条件。如果有很多工作线程存在，当唤醒了许多线程但由没有工作可做时，就可能出现*惊群效应*问题，最后导致 CPU 资源的浪费，并且增加了锁的争夺。

### 4. **下面哪个步骤序列是正确的？**

>（1） 对互斥量加锁（pthread_mutex_lock）
>（2） 改变互斥量保护的条件
>（3） 给等待条件的线程发信号（pthread_cond_broadcast）
>（4） 对互斥量解锁（pthread_mutex_unlock）
>或者
>（1） 对互斥量加锁（pthread_mutex_lock）
>（2） 改变互斥量保护的条件
>（3） 对互斥量解锁（pthread_mutex_unlock）
>（4） 给等待条件的线程发信号（pthread_cond_broadcast）

这根据具体情况而定。总的来说，两种情况都可能是正确的，但每一种方法都有不足之处。在第一种情况下，等待线程会被安排在调用 pthread_cond_broadcast 之后运行。如果程序运行在多处理器上，由于还持有互斥锁（pthread_cond_wait 返回持有的互斥锁），一些线程就会运行而且马上阻塞。在第二章情况下，运行线程可以在第 3 步和第 4 步之间获得互斥锁，然后使条件失效，最后释放互斥锁。接着，当调用 pthread_cond_broadcast 时，条件不再为真，线程无需执行。这就是为什么唤醒线程必须重新检查条件，不能仅仅因为 pthread_cond_wait 返回就假定条件就为真。

### 5. **实现屏障需要什么原语？给出 pthread_barrier_wait 函数的一个实现**