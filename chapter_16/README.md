## 网络IPC：套接字

### 套接字描述符

套接字是通信端点的抽象。正如使用文件描述符访问文件，应用程序用套接字描述符访问套接字。套接字描述符是UNIX系统中被当作是一种文件描述符。事实上，许多处理文件描述符的函数（如Read和Write）可以用于处理套接字描述符。

```c
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

虽然套接字描述符本质上是一个文件描述符，但不是所有的参数为文件描述符的函数都可以接受套接字描述符时的行为。

![image.png](https://upload-images.jianshu.io/upload_images/1916953-79ce7c9da50078d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 寻址

#### 字节序

字节序是一个处理器架构特性，用于指示像整数这样的大数据类型内部的字节如何排序。

```c
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);

uint16_t htons(uint16_t hostshort);

uint32_t ntohl(uint32_t netlong);

uint16_t ntohs(uint16_t netshort);
```

h ： host

n： network

l： long

s：short

#### 地址格式

一个地址标识一个特定通信域的套接字端点，地址格式与这个特定的通信域相关。为使不同格式地址能够传入套接字函数，地址会被强制转换为一个通用的地址结构 sockaddr。

```c
#include <arpa/inet.h>

const char * inet_ntop(int af, const void * restrict src, char * restrict dst, socklen_t size);

int inet_pton(int af, const char * restrict src, void * restrict dst);
```

* The function `inet_ntop()` converts an address *src from network format (usually a struct in_addr or some other binary form, in network byte order) to presentation format (suitable for external display pur-poses).  The size argument specifies the size, in bytes, of the buffer *dst. ` INET_ADDRSTRLEN` and `INET6_ADDRSTRLEN` define the maximum size required to convert an address of the respective type.  It returns NULL if a system error occurs (in which case, errno will have been set), or it returns a pointer to the destination string.  This function is presently valid for `AF_INET` and `AF_INET6`.

* The routine `inet_ntoa()` takes an Internet address and returns an ASCII string representing the address in '.' notation.  The routine inet_makeaddr() takes an Internet network number and a local network address and constructs an Internet address from it.  The routines `inet_netof()` and `inet_lnaof()` break apart Internet host addresses, returning the network number and local network address part, respectively.

#### 地址查询

获取给定计算机系统的主机信息：

```c
#include <netdb.h>

struct hostent * gethostent(void);

void sethostent(int stayopen);
void endhostent(void);
```

获取网络名称与网络编号

```c
#include <netdb.h>

struct netent * getnetbyname(const char *name);
struct netent * getnetbyaddr(uint32_t net, int type);

void setnetent(int stayopen);
void endnetent(void);
```

在协议名字和协议编号之间进行映射：

```c
#include <netdb.h>

struct protoent * getprotoent(void);
struct protoent * getprotobyname(const char *name);
struct protoent * getprotobynumber(int proto);

void setprotoent(int stayopen);
void endprotoent(void);
```

服务是由地址的端口号部分表示的。每个服务由一个唯一的总所周知的端口号来支持。

`getservbyname`将一个服务名映射到一个端口号。

`getservbyport`将一个端口号映射1到一个服务名。

`getservent`顺序扫描服务数据库。

```c
#include <netdb.h>

struct servent * getservent();
struct servent * getservbyname(const char *name, const char *proto);
struct servent * getservbyport(int port, const char *proto);

void setservent(int stayopen);
void endservent(void);
```

`getaddrinfo`将一个主机名和一个服务名映射到一个地址

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *hostname, const char *servname, const struct addrinfo *hints, struct addrinfo **res);

void freeaddrinfo(struct addrinfo *ai);
```

```c
struct addrinfo {
             int ai_flags;           /* input flags */
             int ai_family;          /* protocol family for socket */
             int ai_socktype;        /* socket type */
             int ai_protocol;        /* protocol for socket */
             socklen_t ai_addrlen;   /* length of socket-address */
             struct sockaddr *ai_addr; /* socket-address for socket */
             char *ai_canonname;     /* canonical name for service location */
             struct addrinfo *ai_next; /* pointer to next in list */
};
```

`getnameinfo`将一个地址转换为一个主机名和一个服务名

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getnameinfo(const struct sockaddr *sa, socklen_t salen, char *host, socklen_t hostlen, char *serv, socklen_t servlen, int flags);
```

#### 将套接字与地址关联

`bind`函数关联地址和套接字

```c
#include <sys/socket.h>

int bind(int socket, const struct sockaddr *address, socklen_t address_len);
```

`getsockname`函数来发现绑定到套接字上的地址。

```c
#include <sys/socket.h>

int getsockname(int socket, struct sockaddr *restrict address, socklen_t *restrict address_len);
```

如果套接字已经和对等方连接，可以调用`getpeername`获取对方的地址。

```c
#include <sys/socket.h>

int getpeername(int socket, struct sockaddr *restrict address, socklen_t *restrict address_len);
```
### 建立连接

使用connect 在服务请求方与服务提供方之间建立连接

```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int socket, const struct sockaddr *address, socklen_t address_len);
```

服务器使用`listen`函数来宣告它愿意接受连接请求：

```c
#include <sys/socket.h>

int listen(int socket, int backlog);
```

一旦服务器调用了 listen， 所有的套接字就都能收到连接请求。使用`accept`函数获得请求并建立连接

```c
#include <sys/socket.h>

int accept(int socket, struct sockaddr *restrict address, socklen_t *restrict address_len);
```

### 数据传输

发送消息的三个函数：

```c
#include <sys/socket.h>

ssize_t send(int socket, const void *buffer, size_t length, int flags);

ssize_t sendmsg(int socket, const struct msghdr *message, int flags);

ssize_t sendto(int socket, const void *buffer, size_t length, int flags, const struct sockaddr *dest_addr, socklen_t dest_len);
```

接收消息的三个函数：

```c
#include <sys/socket.h>

ssize_t recv(int socket, void *buffer, size_t length, int flags);

ssize_t recvfrom(int socket, void *restrict buffer, size_t length, int flags, struct sockaddr *restrict address, socklen_t *restrict address_len);

ssize_t recvmsg(int socket, struct msghdr *message, int flags);
```

### 套接字选项

套接字机制提供了两个套接字选项接口来控制套接字的行为。

一个用来设置选项，一个用来查询选项的状态。

```c
#include <sys/socket.h>

int getsockopt(int socket, int level, int option_name, void *restrict option_value, socklen_t *restrict option_len);

int setsockopt(int socket, int level, int option_name, const void *option_value, socklen_t option_len);
```



The following options are recognized at the socket `level`.  Except as noted, each may be examined with getsockopt() and set with setsockopt().

```c
           SO_DEBUG        enables recording of debugging information
           SO_REUSEADDR    enables local address reuse
           SO_REUSEPORT    enables duplicate address and port bindings
           SO_KEEPALIVE    enables keep connections alive
           SO_DONTROUTE    enables routing bypass for outgoing messages
           SO_LINGER       linger on close if data present
           SO_BROADCAST    enables permission to transmit broadcast messages
           SO_OOBINLINE    enables reception of out-of-band data in band
           SO_SNDBUF       set buffer size for output
           SO_RCVBUF       set buffer size for input
           SO_SNDLOWAT     set minimum count for output
           SO_RCVLOWAT     set minimum count for input
           SO_SNDTIMEO     set timeout value for output
           SO_RCVTIMEO     set timeout value for input
           SO_TYPE         get the type of the socket (get only)
           SO_ERROR        get and clear error on the socket (get only)
           SO_NOSIGPIPE    do not generate SIGPIPE, instead return EPIPE
           SO_NREAD        number of bytes to be read (get only)
           SO_NWRITE       number of bytes written not yet sent by the protocol (get only)
           SO_LINGER_SEC   linger on close if data present with timeout in seconds
```

### 带外数据

带外数据（out-of-band data）是通信协议所支持的可选功能，与普通数据相比，它允许更高优先级的数据传输。带外数据先行传输，即使传输队列中已有数据。TPC支持带外数据，但是UDP不支持。套接字接口对外带数据的支持很大程度上受TCP带外数据具体实现的影响。

### 非阻塞和异步I/O

通常，`recv`函数没有数据时可用时会阻塞等待。同样地，当套接字输出队列没有足够空间来发送消息时，send 函数会阻塞。在套接字非阻塞模式下，行为会改变。

套接字有自己的处理异步I/O的方式。一些文献把经典的基于套接字的异步I/O机制称为“基于信号的I/O”，区别于Single UNIX Epecification的通用异步I/O机制。