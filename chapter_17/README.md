## 高级进程间通信

### UNIX 域套接字

UNIX 域套接字用于在同一台计算机上运行的进程之间的通信。虽然因特网套接字可用于同一目的，但UNIX域套接字的效率更高。UNIX 域套接字仅仅复制数据，它们并不运行协议处理，不需要添加或删除网络报头，无须计算检验和，不要产生顺序号，无须发送确认报文。

```c
#include <sys/socket.h>

int socketpair(int domain, int type, int protocol, int socket_vector[2]);
```

The socketpair() call creates an unnamed pair of connected sockets in the specified domain domain, of the specified type, and using the optionally specified protocol.  The descriptors used in referencing the new sockets are returned in socket_vector[0] and socket_vector[1].  The two sockets are indistinguishable.

**RETURN VALUES**
The socketpair() function returns the value 0 if successful; otherwise the value -1 is returned and the global variable errno is set to indicate the error.

### 唯一连接

### 传送文件描述符