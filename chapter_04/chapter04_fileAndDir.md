[TOC]

## 第四章 文件与目录

### 4.2 stat、fstat、fstatat 与lstat

```c
#include <sys/stat.h>
int fstat(int fildes, struct stat *buf);
int lstat(const char *restrict path, struct stat *restrict buf);
int stat(const char *restrict path, struct stat *restrict buf);
int fstatat(int fd, const char *path, struct stat *buf, int flag);
```

The **stat()** function obtains information about the file pointed to by path.  Read, write or execute permission of the named file is not required, but all directories listed in the path name leading to the file must be searchable.

The **lstat()** function is like stat() except in the case where the named file is a symbolic link; lstat() returns information about the link, while stat() returns information about the file the link references.
For symbolic links, the st_mode member contains meaningful information when used with the file type macros, and the st_size member contains the length of the pathname contained in the symbolic link. File mode bits and the contents of the remaining members of the stat structure are unspecified. The value returned in the st_size member is the length of the contents of the symbolic link, and does not count any trail-ing null.

The **fstat()** obtains the same information about an open file known by the file descriptor fildes.

The **fstatat()** system call is equivalent to stat() and lstat() except in the case where the path specifies a relative path.  In this case the status is retrieved from a file relative to the directory associated with the file descriptor fd instead of the current working directory.

### 4.7 access 和 faccessat

判断文件的访问权限。

```c
#include <unistd.h>
int access(const char *path, int mode);
int faccessat(int fd, const char *path, int mode, int flag);
```

The **access()** system call checks the accessibility of the file named by the path argument for the access permissions indicated by the mode argument.  The value of mode is either the bitwise-inclusive OR of the access permissions to be checked (R_OK for read permission, W_OK for write permission, and X_OK for execute/search permission), or the existence test (F_OK).

The **faccessat()** system call is equivalent to access() except in the case where path specifies a relative path.  In this case the file whose accessibility is to be determined is located relative to the direc-tory associated with the file descriptor fd instead of the current working directory.  If faccessat() is passed the special value AT_FDCWD in the fd parameter, the current working directory is used and the behavior is identical to a call to access().

### 4.8 umask

创建屏蔽字，并返回之前的值。

```c
#include <sys.stat.h>
mode_t umask(mode_t cmask);
```

### 4.9 chmod、fchmod、fchmodat

更改现有文件的访问权限

```c
#include <sys/types.h>
#include <sys/stat.h>
int chmod(const char *path, mode_t mode);
int fchmod(int fildes, mode_t mode);
int fchmodat(int fd, const char *path, mode_t mode, int flag);
```

The function **chmod()** sets the file permission bits of the file specified by the pathname path to mode. 

**fchmod()** sets the permission bits of the specified file descriptor fildes.  chmod() verifies that the process owner (user) either owns the file specified by path (or fildes), or is the super-user.

The **fchmodat()** is equivalent to chmod() except in the case where path specifies a relative path.  In this case the file to be changed is determined relative to the directory associated with the file descriptor fd instead of the current working directory.

### 4.11 chown、fchown、fchownat、lchown

改变文件的用户ID与组ID

```c
#include <unistd.h>
int chown(const char *path, uid_t owner, gid_t group);
int fchown(int fildes, uid_t owner, gid_t group);
int lchown(const char *path, uid_t owner, gid_t group);
int fchownat(int fd, const char *path, uid_t owner, gid_t group, int flag);
```

The **chown()** system call clears the set-user-id and set-group-id bits on the file.  The chown() system call follows symbolic links to operate on the target of the link rather than the link itself.

The **fchown()** system call is particularly useful when used in conjunction with the file locking primitives (see flock(2)).

The** lchown()** system call is similar to chown() but does not follow symbolic links.

The **fchownat()** system call is equivalent to the chown() and lchown() except in the case where path specifies a relative path.  In this case the file to be changed is determined relative to the directory asso-ciated with the file descriptor fd instead of the current working directory.

### 4.13 truncate、ftruncate

截断文件

```c
#include <unistd.h>
int ftruncate(int fildes, off_t length);
int truncate(const char *path, off_t length);
```

ftruncate() and truncate() cause the file named by path, or referenced by fildes, to be truncated (or extended) to length bytes in size. If the file size exceeds length, any extra data is discarded. If the file size is smaller than length, the file is extended and filled with zeros to the indicated length.  The ftruncate() form requires the file to be open for writing.

### 4.15 link、linkat、unlink、unlinkat、remove

创建文件硬链接

```c
#include <unistd.h>
int link(const char *path1, const char *path2);
int linkat(int fd1, const char *name1, int fd2, const char *name2, int flag);
```

The **link()** function call atomically creates the specified directory entry (hard link) path2 with the attributes of the underlying object pointed at by path1.  If the link is successful, the link count of the underlying object is incremented; path1 and path2 share equal access and rights to the underlying object.

The **linkat()** system call is equivalent to link except in the case where either name1 or name2 or both are relative paths.  In this case a relative path name1 is interpreted relative to the directory associated with the file descriptor fd1 instead of the current working directory and similarly for name2 and the file descriptor fd2.

删除目录项，并将由pathname所引用的文件链接计数减一。

```c
#include <unistd.h>
int unlink(const char *path);
int unlinkat(int fd, const char *path, int flag);
```

对于文件，remove功能与unlink相同；对于目录，remove功能与rmdir相同

```c
#include <stdio.h>
int remove(const char *path);
```

### 4.16 rename、renameat

对文件进行重命名

```c
#include <stdio.h>
int rename(const char *old, const char *new);
int renameat(int fromfd, const char *from, int tofd, const char *to);
int renamex_np(const char *from, const char *to, unsigned int flags);
int renameatx_np(int fromfd, const char *from, int tofd, const char *to, unsigned int flags);
```

The **rename()** system call causes the link named old to be renamed as new.  If new exists, it is first removed.  Both old and new must be of the same type (that is, both must be either directories or non-direc-tories) and must reside on the same file system.

### 4.18 symlink、symlinkat、readlink、readlinkat

创建符号链接

```c
#include <unistd.h>
int symlink(const char *path1, const char *path2);
int symlinkat(const char *name1, int fd, const char *name2);
```

A symbolic link path2 is created to path1 (path2 is the name of the file created, path1 is the string used in creating the symbolic link).  Either name may be an arbitrary path name; the files need not be on the same file system.

read value of a symbolic link

```c
#include <unistd.h>
ssize_t readlink(const char *restrict path, char *restrict buf, size_t bufsize);
ssize_t readlinkat(int fd, const char *restrict path, char *restrict buf, size_t bufsize);
```

### 4.20 futimens、utimensat、utimes

更改文件的访问与修改时间。

```c
#include <sys/stat.h>
int futimens(int fd, const struct timespec times[2]);
int utimensat(int fd, const char *path, const struct timespec times[2], int flag);
```

```c
#include <sys/time.h>
int futimes(int fildes, const struct timeval times[2]);
int utimes(const char *path, const struct timeval times[2]);
```

### 4.21 mkdir、mkdirat、rmdir

创建与删除目录

```c
#include <sys/stat.h>
int mkdir(const char *path, mode_t mode);
int mkdirat(int fd, const char *path, mode_t mode);
```

删除一个空目录

```c
#include <unistd.h>
int rmdir(const char *path);
```

# 4.22 读目录

```c
#include <dirent.h>

DIR * opendir(const char *filename);
DIR * fdopendir(int fd);

struct dirent * readdir(DIR *dirp);
int readdir_r(DIR *dirp, struct dirent *entry, struct dirent **result);

long telldir(DIR *dirp);
void seekdir(DIR *dirp, long loc);
void rewinddir(DIR *dirp);
int closedir(DIR *dirp);
int dirfd(DIR *dirp);
```

### 4.23 chdir、fchdir、getcwd

更改当前进程的工作目录

```c
#include <unistd.h>
int chdir(const char *path);
int fchdir(int fildes);
```

获取当前的工作目录

```c
#include <unistd.h>
char * getcwd(char *buf, size_t size);
char * getwd(char *buf);
```
