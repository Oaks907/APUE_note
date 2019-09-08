[TOC]

## 进程环境

### main函数

main函数原型：

```c
int main(int argc, char const *argv[])
```

`argc`是命令行参数的个数，`argv`指向参数的各个指针所构成的数组。

在内核执行 C 程序之前，在调用 main 之前先调用一个特殊的**启动例程**。可执行程序文件将此启动例程指定为程序的起始地址，这是由连接编辑器设置的，而连接编辑器由 C 编译器调用。启动例程从内核中取得的命令行参数和环境变量参数值，然后按照上面的方式调用main函数做好安排。

### 进程终止

有 8 种方式能够使进程终止，其中五种为正常终止，三种为异常终止：

* 正常终止
  1. 从main返回
  2. 调用 exit
  3. 调用\_exit 或_Exit
  4. 最后一个线程由例程中返回
  5. 最后一个线程对取消请求作出了响应
* 异常终止
  1. 调用 abort
  2. 接到一个信号
  3. 最后一个线程对取消请求做出响应

![image-20190908085543925](/Users/haifei/Library/Application Support/typora-user-images/image-20190908085543925.png)

### 命令行参数

当执行一个程序时，调用 exec 的进程可将命令行参数传递给新程序。

### 环境表

每个程序都有一张环境表。与参数表一样，环境表也是一个字符指针数组，其中每个指针包含一个以 null 结束的 C 字符串的地址。全局变量environ则包含了该指针数组的地址：

```c
extern char **environ;
```

![image-20190908090052196](/Users/haifei/Library/Application Support/typora-user-images/image-20190908090052196.png)

### C程序的存储空间布局

![image-20190908090152247](/Users/haifei/Library/Application Support/typora-user-images/image-20190908090152247.png)

* 正文段

  这是由CPU执行的机器指令部分。通常，正文段是可以共享的，所以即使是一个频繁执行的程序在存储器中也只是需要一个副本。

* 未初始化数据段

  bss段（block started by symbol），在程序开始之前，内核将该段中的数据初始化为 0 或空指针。函数外的声明：`long sum[1000]`使此变量存放在非初始化数据段中。

* 栈

  自动变量以及每次函数调用时所需要保存的信息都存放在此段中。每次函数调用时，其返回地址以及调用者的环境信息都存放在栈中。然后，最近被调用的函数为其自动和临时变量分配存储空间。

* 堆

  通常在堆中进行动态存储分配

### 共享库

共享库使得可执行文件中不再包含公用的库函数，而只需要所有进程都可引用的存储区中保存这种库例程的一个副本。程序第一次执行或者第一次调用某个库函数时，用动态连接方法将程序与共享库函数相链接。者较少了每个执行文件的长度，但增加了一些运行时间的开销。

### 存储空间分配

ISO C 说明了3个用于存储空间动态分配的函数。

1. malloc，分配指定字节数的存储区。此存储区中的初始值不确定。
2. calloc，为指定数量指定长度的对象分配存储空间。该空间的每一位bit都初始化为0
3. realloc，增加或者减少之前分配区的长度。当增加长度时，可能需将以前分配区的内容迁移到另一个足够大的区域，以便在尾端增加存储区，而新增区域中的值是不确定的。

```c
#include <stdlib.h>

void * malloc(size_t size);
void * calloc(size_t count, size_t size);
void * realloc(void *ptr, size_t size);
```

### 环境变量

```c
#include <stdlib.h>

char * getenv(const char *name);

int setenv(const char *name, const char *value, int overwrite);
int putenv(char *string);
int unsetenv(const char *name)
```

### 函数 setjmp 和 longjmp

在 C 中，goto 语句是不能跨越函数的，而执行这种类型跳转功能的函数 `setjmp`和`longjmp`,这两个函数对于处理发生在很深层嵌套函数调用中的出错情况是非常有用的。

```c
#include <setjmp.h>

void longjmp(jmp_buf env, int val);
int setjmp(jmp_buf env);
```

### 函数 getrlimt 和 setrlimit

每个进程都有一组资源控制，其中一些可以用 getrlimit 和 setrlimit 函数查询和更改。

```c
#include <sys/resource.h>

int getrlimit(int resource, struct rlimit *rlp);
int setrlimit(int resource, const struct rlimit *rlp);
```

## 习题

### 1.**在 Intel x86 系统上，使用 Linux ，如果执行一个输出 “hello, world” 的程序但不调用 exit 或 return ，则程序的返回代码为 13 （用 shell 检查），解释其原因。**

三种情况可以导致返回的代码不为0：

* 调用这些函数时没有带终止状态
* main执行了一个无返回值的return语句
* main没有声明返回类型为整型，则该进程的终止状态是未定义的

如果 main 函数返回值为 int 的话，那么无论如何返回的都是 0 ；如果 main 函数返回值为 void 的话，那么将会返回 printf 的返回值，也就是输出的字符个数 13 。

### 2. **图 7-3 中的 printf 函数的结果何时才被真正输出。**

* 当程序处于交互运行方式时，标准输出通常处于行缓冲方式，所以当输出换行符时，上次的结果才被真正输出。

* 如果标准输出被定向到一个文件，而标准输出处于全缓冲方式，则当标准 I/O 清理操作执行时，结果才真正被输出。

### 3. **是否有方法不适用（a）参数传递、（b）全局变量这两种方法，将 main 中的参数 argc 和argv 传递给它所调用的其他函数？**

没有

### 4. **在有些 UNIX 系统实现中执行程序时访问不到其数据段的 0 单元，这是一种有意的安排，为什么？**

当C程序解引用一个空指针出错时，执行该程序的进程将终止。可以利用这种方式终止进程。

### 5. **用 C 语言的 typedef 为终止处理程序定义了一个新的数据类型 Exitfunc ，使用该类型修改 atexit 的原型。**

atexit 的原型：

```c
#include <stdlib.h>
int atexit(void (*func)(void));
```

定义如下：

```c
typedef void Exitfunc(void);
int atexit(Exitfunc *func);
```

### 6. **如果用 calloc 分配一个 long 型的数组，数组的初始值是否为 0 ？如果用 calloc 分配一个指针数组，数组的初始值是否为空指针？**

ISO C 说明：

> - calloc，为指定数量指定长度的对象分配存储空间。该空间中的每一位（bit）都初始化为 0 。

因此数组的空间初始为0。但是ISO C并不保证0值与浮点型或空指针的值相同。 

### 7. **在 7.6 节结尾处 size 命令的输出结果中，为什么没有给出堆和栈的大小？**

只有通过exec执行一个程序时，才会分配堆和栈

### 8. **为什么 7.7 节中两个文件的大小（879443 和 8378）不等于它们各自文本和数据大小的和？**

可执行文件 a.out 包含了用于调试 core 文件的符号表信息。用 strip 命令可以删除这些信息，对两个 a.out 文件执行这条命令，它们的大小减为 798760 和 6200 字节。

### 9. **为什么 7.7 节中对于一个简单的程序，使用共享库以后其可执行文件的大小变化如此巨大？**

没有使用共享库，可执行文件的大部分都被标准I/O库所占用

### 10. **在 7.10 节中我们已经说明为什么不能将一个指针返回给一个自动变量，下面的程序是否正确？**

```c
nt f1(int val) {
  int num = 0;
  int *ptr = &num;
  if (val == 0) {
    int val;
    val = 5;
    ptr = &val;
  }
  return (*ptr + 1);
}
```

不正确，if 语句里面的 val 是一个自动变量，当离开 if 的作用域时，这个变量就不存在了。

