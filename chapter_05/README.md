[TOC]

## 标准I/O库

### 流与FILE对象

当用标准 I/O 库打开或创建一个文件时，我们已经使一个流与一个文件关联。

**流的定向** 决定了所读写的流是单字节还是多字节的。在一个未定向的流上使用多字节I/O函数，那么它是宽字节的，单字节依然。

只有两个函数可以改变一个流的定向。

`freopen`清除一个流的定向；`fwide`函数可用于设置流的定向

```c
#include <stdio.h>
#include <wchar.h>

int fwide(FILE *stream, int mode);
```

If mode is less than zero, stream is set to be byte-oriented.  If mode is greater than zero, stream is set to be wide-oriented.  Otherwise, mode is zero, and stream is unchanged.

### 标准输入、标准输出与标准错误

对于一个进程，预定义了 3 个流，并且这三个流可以自动的被进程使用，它们是 标准输入、标准输出与标准错误。这些流引用的文件与文件描述符`STDIN_FILENO`、`STDOUT_FILENO` 和 `STDERR_FILENO` 所引用相同

### 缓冲

标准I/O库提供缓冲的目的是尽可能的减少I/O调用的次数。它对每个I/O流自动地进行缓冲管理，从而避免应用程序需要考虑这点。

标准I/O提供了3种类型的缓冲：

* 全缓冲

全缓冲意味着，在填满标准I/O缓冲区后才进行实际的 I/O 操作。对于驻留在磁盘上的文件通常由标准I/O库实施全缓冲。

flush冲洗：在标准I/O意味着将缓冲区中的内容写到磁盘中（缓冲区可能部分填满）。

在终端程序方面，flush意味着丢弃已存储在缓冲区中的数据。

* 行缓冲

行缓冲意味着，当在输入和输出中遇到换行符时，标准I/O库执行I/O操作。允许我们一次输出一个字符，但只有在写了一行之后才进行实际的I/O操作。

* 不带缓冲

对于任何一个给定的流，如果我们可以更改其缓冲类型：

```c
#include <stdio.h>
void setbuf(FILE *restrict stream, char *restrict buf);
void setbuffer(FILE *stream, char *buf, int size);
int setlinebuf(FILE *stream);
int setvbuf(FILE *restrict stream, char *restrict buf, int type, size_t size);
```

flush清洗流：

```c
#include <stdio.h>
int fflush(FILE *stream);
int fpurge(FILE *stream);
```

The function fpurge() erases any input or output buffered in the given stream.  For output streams this discards any unwritten output.  For input streams this discards any input read from the underlying object but not yet obtained via getc(3); this includes any text pushed back via ungetc(3).

### 打开流

```c
#include <stdio.h>

FILE * fopen(const char * restrict path, const char * restrict mode);
FILE * fdopen(int fildes, const char *mode);
FILE * freopen(const char *path, const char *mode, FILE *stream);
FILE * fmemopen(void *restrict *buf, size_t size, const char * restrict mode);
```

```c
#include <stdio.h>
int fclose(FILE *stream);
void fcloseall(void);
```

文件被关闭之前，冲洗缓冲中的输出数据。缓冲区中的任何输入数据都会被丢弃。如果标准I/O库自动为该流自动分配了一个缓冲区，则释放该缓冲区。

当一个进程终止时，所有的未写缓冲数据的标准I/O流都被冲洗，所有打开的标准I/O流都会被关闭。

### 读与写流

输入函数。一次读一个字符

```c
#include <stdio.h>

int fgetc(FILE *stream);
int getc(FILE *stream);
int getchar(void);
```

输出函数。每次输入一个字符

```c
#include <stdio.h>

int fputc(int c, FILE *stream);
int putc(int c, FILE *stream);
int putchar(int c);
```

### 每次一行IO

输入函数，每次输入一行数据：

```c
#include <stdio.h>

char * fgets(char * restrict str, int size, FILE * restrict stream);
char * gets(char *str);
```

gets 是一个不推荐的函数，原因是不能制定缓冲区的大小。这样可能造成缓冲区溢出。

没出输出一行的功能：

```c
#include <stdio.h>

int fputs(const char *restrict s, FILE *restrict stream);
int puts(const char *s);
```

### 标准I/O的效率

标准I/O库与直接调用read和write函数相比并不慢多少。对于大多数比较复杂的程序，最主要的是用户CPU时间的消耗是由应用本身的各种处理消耗的，而不是标准I/O例程消耗的。

### 二进制I/O

二进制I/O，一次读取或写一个完整的结构。

```c
#include <stdio.h>
size_t fread(void *restrict ptr, size_t size, size_t nitems, FILE *restrict stream);
size_t fwrite(const void *restrict ptr, size_t size, size_t nitems, FILE *restrict stream);
```

二进制I/O只能用于读在同一系统上已写的数据。现在通过网络相互链接起来的系统，相互转换会有问题。

* 在一个结构中，同一成员的偏移量可能随着编译程序的不同而不同。
* 用来存储多字节整数和浮点数的二进制格式在不同的系统结构间也可能不同

### 定位流

有三种方法定位标准I/O流：

ftell、fseek假定文件可以存在一个长整型中。
```c
#include <stdio.h>

long ftell(FILE *stream);
int fseek(FILE *stream, long offset, int whence);
void rewind(FILE *stream);
```

ftello、fseeko 使文件偏移量不必是一个长整形。它们使用off_t来代替长整型。
```c
#include <stdio.h>

off_t ftello(FILE *stream);
int fseeko(FILE *stream, off_t offset, int whence);
```

使用抽象数据结构，记录文件位置。
```c
#include <stdio.h>

int fgetpos(FILE *restrict stream, fpos_t *restrict pos);
int fsetpos(FILE *stream, const fpos_t *pos);
```

### 格式化流

格式化输出

```c
#include <stdio.h>

int printf(const char * restrict format, ...);
int fprintf(FILE * restrict stream, const char * restrict format, ...);
int sprintf(char * restrict str, const char * restrict format, ...);
int snprintf(char * restrict str, size_t size, const char * restrict format, ...);
int asprintf(char **ret, const char *format, ...);
int dprintf(int fd, const char * restrict format, ...);
```

```c
#include <stdarg.h>

int vprintf(const char * restrict format, va_list ap);
int vfprintf(FILE * restrict stream, const char * restrict format, va_list ap);
int vsprintf(char * restrict str, const char * restrict format, va_list ap);
int vsnprintf(char * restrict str, size_t size, const char * restrict format, va_list ap);
int vasprintf(char **ret, const char *format, va_list ap);
int vdprintf(int fd, const char * restrict format, va_list ap);
```

格式化输入

```c
#include <stdio.h>

int fscanf(FILE *restrict stream, const char *restrict format, ...);
int scanf(const char *restrict format, ...);
int sscanf(const char *restrict s, const char *restrict format, ...);
```

```c
#include <stdarg.h>
#include <stdio.h>

int vfscanf(FILE *restrict stream, const char *restrict format, va_list arg);
int vscanf(const char *restrict format, va_list arg);
int vsscanf(const char *restrict s, const char *restrict format, va_list arg);
```

### 临时文件

```c
#include <stdio.h>

FILE * tmpfile(void);
char * tmpnam(char *s);
```

`tmpnam`函数会产生一个与现有文件名不同的有效路径名字符串。每次调用它，都会产生一个不同的路径名。

`tmpfile`函数会创建一个临时二进制文件，在关闭该文件或者程序终止时，自动删除该文件。

### 内存流

标准I/O库把数据缓存在内存中，因此，每次一字符和每次一行的I/O更有效。

有三个函数可以用于内存流的创建：

```c
#include <stdio.h>

FILE * fmemopen(void *restrict *buf, size_t size, const char * restrict mode);
```

`fmemopen`允许开发者提供缓冲区用于内存流。

```c
#include <stdio.h>

FILE * open_memstream(char **bufp, size_t *sizep);

#include <wchar.h>

FILE * open_wmemstream(wchar_t **bufp, size_t *sizep);
```

`open_memstream`创建的流是面向字节的

`open_wmemstream`创建的流是面向宽字节的。

这两个函数与`fmemopen`的区别：

* 创建的流只能写打开
* 不能指定自己的缓冲区，但可以分别通过bufp和sizep参数访问缓冲区地址和大小
* 关闭流后需要自行释放缓冲区
* 对流添加字节会增加缓冲区大小

## 课后题答案

### 1. **用 setvbuf 实现 setbuf 。**

见p1。

### 2. **图 5-5 中的程序利用每次一行 I/O （fgets 和 fputs 函数）复制文件。若将程序中的 MAXLINE 改为 4 ，当复制的行超过该最大值时会出现什么情况？请对此进行解释**

fgets在读取数据的时候，每次调用会读取指定大小的数据或者是文件剩余的数据到缓冲区。当设定的读取MAXLINE时，多余的数据会再次进行函数的调用

### 3. **printf 返回 0 值表示什么？**

printf 函数的返回值是输出的字符的长度，因此返回 0 值代表着输出了空白。

### 4. **下面的代码在一些机器上运行正确，而在另外一些机器运行时出错，解释原因所在**

```c
#include <stdio.h>

int main() {
    char c;
    while ((c = getchar()) != EOF) {
        putchar(c);
    }
}
```

这是一个比较常见的错误。 getc 以及 getchar 的返回值是 int 类型，而不是 char 类型。由于 EOF 经常定义为 -1 ，那么如果系统使用的是有符号的字符类型，程序可以正常工作。但如果使用的是无符号字符类型，那么返回的 EOF 被保存到字符 c 后将不再是 -1 ，所以，程序会进入死循环。

### 5. **对标准 I/O 流如何使用 fsync 函数（见 3.13 节）？**

sync 函数只对由文件描述符 fd 指定的一个文件起作用，并且等待写磁盘操作结束才返回。 fsync 可用于数据库这样的应用程序，这种应用程序需要确保修改过的块立即写到磁盘上。

对标准 I/O 流首先调用 fflush ，然后调用 fsync 。 fsync 的参数由 fileno 函数获得。如果不调用 fflush ，所有的数据仍然在内存缓冲区中，此时调用 fsync 将没有任何效果。

### 6. **在图 1-7 和图 1-10 程序中，打印的提示信息没有包含换行符，程序也没有调用 fflush 函数，请解释输出提示信息的原因是什么。**

在标准输入输出流连接终端的时候，这些流是行缓冲的，调用 fgets 的时候标准输出流将自动冲洗。



### 7. **基于 BSD 的系统提供了 funopen 的函数调用使我们可以拦截读、写、定位以及关闭一个流的调用。使用这个函数为 FreeBSD 和 MacOS X 实现 fmemopen。**