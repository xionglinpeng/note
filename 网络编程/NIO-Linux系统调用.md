Netty
NIO
AIO
Reactor
Proactor
reactor-siemens
Scalable IO in Java

## select

### Name

`select`, `pselect`, `FD_CLR`, `FD_ISSET`, `FD_SET`, `FD_ZERO` - 同步I/O多路复用

### Synopsis

```c
#include <sys/select.h>

int select(int nfds, fd_set *restrict readfds,
           fd_set *restrict writefds, fd_set *restrict exceptfds,
           struct timeval *restrict timeout);

void FD_CLR(int fd, fd_set *set);
int  FD_ISSET(int fd, fd_set *set);
void FD_SET(int fd, fd_set *set);
void FD_ZERO(fd_set *set);

int pselect(int nfds, fd_set *restrict readfds,
            fd_set *restrict writefds, fd_set *restrict exceptfds,
            const struct timespec *restrict timeout,
            const sigset_t *restrict sigmask);

//glibc的特性测试宏需求(见feature_test_宏(7)):

pselect():
	_POSIX_C_SOURCE >= 200112L
```

### Description

**WARNING:** select() can monitor only file descriptors numbers that are less than FD_SETSIZE (1024)—an unreasonably low limit for many modern applications—and this limitation will not change. All modern applications should instead use poll(2) or epoll(7), which do not suffer this limitation.

select() allows a program to monitor multiple file descriptors, waiting until one or more of the file descriptors become "ready" for some class of I/O operation (e.g., input possible).  A file descriptor is considered ready if it is possible to perform a corresponding I/O operation (e.g., read(2), or a sufficiently small write(2)) without blocking.

**File descriptor sets**

The principal arguments of select() are three "sets" of file descriptors (declared with the type fd_set), which allow the caller to wait for three classes of events on the specified set of file descriptors.  Each of the fd_set arguments may be specified as NULL if no file descriptors are to be watched for the corresponding class of events.
**Note well:** Upon return, each of the file descriptor sets is modified in place to indicate which file descriptors are currently "ready".  Thus, if using select() within a loop, the sets must be reinitialized before each call.

The contents of a file descriptor set can be manipulated using the following macros:

- FD_ZERO()
  This macro clears (removes all file descriptors from) set. It should be employed as the first step in initializing a file descriptor set.
- FD_SET()
  This macro adds the file descriptor fd to set.  Adding a file descriptor that is already present in the set is a no-op, and does not produce an error.
- FD_CLR()
  This macro removes the file descriptor fd from set. Removing a file descriptor that is not present in the set is a no-op, and does not produce an error.
- FD_ISSET()
  select() modifies the contents of the sets according to the rules described below.  After calling select(), the FD_ISSET() macro can be used to test if a file descriptor is still present in a set.  FD_ISSET() returns nonzero if the file descriptor fd is present in set, and zero if it is not.

**Arguments**

The arguments of select() are as follows:

- readfds

  		The file descriptors in this set are watched to see if they are ready for reading.  A file descriptor is ready for reading if a read operation will not block; in particular, a file descriptor is also ready on end-of-file.
	
  		After select() has returned, readfds will be cleared of all file descriptors except for those that are ready for reading.

- writefds

  		The file descriptors in this set are watched to see if they are ready for writing.  A file descriptor is ready for writing if a write operation will not block.  However, even if a file descriptor indicates as writable, a large write may still block.
	
  		After select() has returned, writefds will be cleared of all file descriptors except for those that are ready for writing.

- exceptfds

  		The file descriptors in this set are watched for "exceptional conditions".  For examples of some exceptional conditions, see the discussion of POLLPRI in poll(2).
	
  		After select() has returned, exceptfds will be cleared of all file descriptors except for those for which an exceptional condition has occurred.

- nfds

  This argument should be set to the highest-numbered file descriptor in any of the three sets, plus 1.  The indicated file descriptors in each set are checked, up to this limit (but see BUGS).

- timeout

  		The timeout argument is a timeval structure (shown below) that specifies the interval that select() should block waiting for a file descriptor to become ready.  The call will block until either:

  • a file descriptor becomes ready;

  • the call is interrupted by a signal handler; or

  • the timeout expires.

  		Note that the timeout interval will be rounded up to the system clock granularity, and kernel scheduling delays mean that the blocking interval may overrun by a small amount.
  	
  		If both fields of the timeval structure are zero, then select() returns immediately.  (This is useful for polling.)
  	
  		If timeout is specified as NULL, select() blocks indefinitely waiting for a file descriptor to become ready.        

**pselect()**

The pselect() system call allows an application to safely wait until either a file descriptor becomes ready or until a signal is caught.

The operation of select() and pselect() is identical, other than these three differences:

• select() uses a timeout that is a struct timeval (with seconds and microseconds), while pselect() uses a struct timespec (with seconds and nanoseconds).

• select() may update the timeout argument to indicate how much time was left.  pselect() does not change this argument.

• select() has no sigmask argument, and behaves as pselect() called with NULL sigmask.

sigmask is a pointer to a signal mask (see sigprocmask(2)); if it is not NULL, then pselect() first replaces the current signal mask by the one pointed to by sigmask, then does the "select" function, and then restores the original signal mask.  (If sigmask is NULL, the signal mask is not modified during the pselect() call.)

Other than the difference in the precision of the timeout argument, the following pselect() call:

```c
ready = pselect(nfds, &readfds, &writefds, &exceptfds, timeout, &sigmask);
```

is equivalent to atomically executing the following calls:

```c
sigset_t origmask;

pthread_sigmask(SIG_SETMASK, &sigmask, &origmask);
ready = select(nfds, &readfds, &writefds, &exceptfds, timeout);
pthread_sigmask(SIG_SETMASK, &origmask, NULL);
```

The reason that pselect() is needed is that if one wants to wait for either a signal or for a file descriptor to become ready, then an atomic test is needed to prevent race conditions. (Suppose the signal handler sets a global flag and returns.  Then a test of this global flag followed by a call of select() could hang indefinitely if the signal arrived just after the test but just before the call.  By contrast, pselect() allows one to first block signals, handle the signals that have come in, then call pselect() with the desired sigmask, avoiding the race.)

**The timeout**

The timeout argument for select() is a structure of the following type:

```c
struct timeval {
    time_t      tv_sec;         /* seconds */
    suseconds_t tv_usec;        /* microseconds */
};
```

The corresponding argument for pselect() has the following type:

```c
struct timespec {
    time_t      tv_sec;         /* seconds */
    long        tv_nsec;        /* nanoseconds */
};
```

On Linux, select() modifies timeout to reflect the amount of time not slept; most other implementations do not do this.  (POSIX.1 permits either behavior.)  This causes problems both when Linux code which reads timeout is ported to other operating systems, and when code is ported to Linux that reuses a struct timeval for multiple select()s in a loop without reinitializing it.  Consider timeout to be undefined after select() returns.

### RETURN VALUE

On success, select() and pselect() return the number of file descriptors contained in the three returned descriptor sets (that is, the total number of bits that are set in readfds, writefds, exceptfds).  The return value may be zero if the timeout expired before any file descriptors became ready.

On error, -1 is returned, and errno is set to indicate the error; the file descriptor sets are unmodified, and timeout becomes undefined.

### Errors

| 编码   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| EBADF  | An invalid file descriptor was given in one of the sets. (Perhaps a file descriptor that was already closed, or one on which an error has occurred.)  However, see BUGS. |
| EINTR  | A signal was caught; see signal(7).                          |
| EINVAL | nfds is negative or exceeds the RLIMIT_NOFILE resource limit (see getrlimit(2)). |
| EINVAL | The value contained within timeout is invalid.               |
| ENOMEM | Unable to allocate memory for internal tables.               |

### Versions

pselect() was added to Linux in kernel 2.6.16.  Prior to this, pselect() was emulated in glibc (but see BUGS).

### Conforming To

select() conforms to POSIX.1-2001, POSIX.1-2008, and 4.4BSD (select() first appeared in 4.2BSD).  Generally portable to/from non-BSD systems supporting clones of the BSD socket layer (including System V variants).  However, note that the System V variant typically sets the timeout variable before returning, but the BSD variant does not.

pselect() is defined in POSIX.1g, and in POSIX.1-2001 and POSIX.1-2008.

### Notes

An fd_set is a fixed size buffer.  Executing FD_CLR() or FD_SET() with a value of fd that is negative or is equal to or larger than FD_SETSIZE will result in undefined behavior.  Moreover, POSIX requires fd to be a valid file descriptor.

The operation of select() and pselect() is not affected by the O_NONBLOCK flag.

On some other UNIX systems, select() can fail with the error EAGAIN if the system fails to allocate kernel-internal resources, rather than ENOMEM as Linux does.  POSIX specifies this error for poll(2), but not for select().  Portable programs may wish to check for EAGAIN and loop, just as with EINTR.

**The self-pipe trick**

On systems that lack pselect(), reliable (and more portable) signal trapping can be achieved using the self-pipe trick.  In this technique, a signal handler writes a byte to a pipe whose other end is monitored by select() in the main program.  (To avoid possibly blocking when writing to a pipe that may be full or reading from a pipe that may be empty, nonblocking I/O is used when reading from and writing to the pipe.)

**Emulating usleep(3)**

Before the advent of usleep(3), some code employed a call to select() with all three sets empty, nfds zero, and a non-NULL timeout as a fairly portable way to sleep with subsecond precision.

**Correspondence between select() and poll() notifications**

Within the Linux kernel source, we find the following definitions which show the correspondence between the readable, writable, and exceptional condition notifications of select() and the event notifications provided by poll(2) and epoll(7):

```c
#define POLLIN_SET  (EPOLLRDNORM | EPOLLRDBAND | EPOLLIN | EPOLLHUP | EPOLLERR)
/* Ready for reading */
#define POLLOUT_SET (EPOLLWRBAND | EPOLLWRNORM | EPOLLOUT | EPOLLERR)
/* Ready for writing */
#define POLLEX_SET  (EPOLLPRI)
/* Exceptional condition */
```

 **Multithreaded applications**

If a file descriptor being monitored by select() is closed in another thread, the result is unspecified.  On some UNIX systems, select() unblocks and returns, with an indication that the file descriptor is ready (a subsequent I/O operation will likely fail with an error, unless another process reopens file descriptor between the time select() returned and the I/O operation is performed).  On Linux (and some other systems), closing the file descriptor in another thread has no effect on select().  In summary, any application that relies on a particular behavior in this scenario must be considered buggy.

**C library/kernel differences**

The Linux kernel allows file descriptor sets of arbitrary size, determining the length of the sets to be checked from the value of nfds.  However, in the glibc implementation, the fd_set type is fixed in size.  See also BUGS.

The pselect() interface described in this page is implemented by glibc.  The underlying Linux system call is named pselect6(). This system call has somewhat different behavior from the glibc wrapper function.

The Linux pselect6() system call modifies its timeout argument. However, the glibc wrapper function hides this behavior by using a local variable for the timeout argument that is passed to the system call.  Thus, the glibc pselect() function does not modify its timeout argument; this is the behavior required by POSIX.1-2001.

The final argument of the pselect6() system call is not a sigset_t * pointer, but is instead a structure of the form:

```c
struct {
    const kernel_sigset_t *ss;   /* Pointer to signal set */
    size_t ss_len;               /* Size (in bytes) of object
                                               pointed to by 'ss' */
};
```

This allows the system call to obtain both a pointer to the signal set and its size, while allowing for the fact that most architectures support a maximum of 6 arguments to a system call. See sigprocmask(2) for a discussion of the difference between the kernel and libc notion of the signal set.

**Historical glibc details**

Glibc 2.0 provided an incorrect version of pselect() that did not take a sigmask argument.

In glibc versions 2.1 to 2.2.1, one must define _GNU_SOURCE in order to obtain the declaration of pselect() from <sys/select.h>.

### Bugs

POSIX allows an implementation to define an upper limit, advertised via the constant FD_SETSIZE, on the range of file descriptors that can be specified in a file descriptor set.  The Linux kernel imposes no fixed limit, but the glibc implementation makes fd_set a fixed-size type, with FD_SETSIZE defined as 1024, and the FD_*() macros operating according to that limit.  To monitor file descriptors greater than 1023, use poll(2) or epoll(7) instead.

The implementation of the fd_set arguments as value-result arguments is a design error that is avoided in poll(2) and epoll(7).

According to POSIX, select() should check all specified file descriptors in the three file descriptor sets, up to the limit nfds-1.  However, the current implementation ignores any file descriptor in these sets that is greater than the maximum file descriptor number that the process currently has open.  According to POSIX, any such file descriptor that is specified in one of the sets should result in the error EBADF.

Starting with version 2.1, glibc provided an emulation of pselect() that was implemented using sigprocmask(2) and select(). This implementation remained vulnerable to the very race condition that pselect() was designed to prevent.  Modern versions of glibc use the (race-free) pselect() system call on kernels where it is provided.

On Linux, select() may report a socket file descriptor as "ready for reading", while nevertheless a subsequent read blocks.  This could for example happen when data has arrived but upon examination has the wrong checksum and is discarded.  There may be other circumstances in which a file descriptor is spuriously reported as ready.  Thus it may be safer to use O_NONBLOCK on sockets that should not block.

On Linux, select() also modifies timeout if the call is interrupted by a signal handler (i.e., the EINTR error return). This is not permitted by POSIX.1.  The Linux pselect() system call has the same behavior, but the glibc wrapper hides this behavior by internally copying the timeout to a local variable and passing that variable to the system call.     

### Examples

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/select.h>

int main(void) {
    fd_set rfds;
    struct timeval tv;
    int retval;

    /* Watch stdin (fd 0) to see when it has input. */

    FD_ZERO(&rfds);
    FD_SET(0, &rfds);

    /* Wait up to five seconds. */

    tv.tv_sec = 5;
    tv.tv_usec = 0;

    retval = select(1, &rfds, NULL, NULL, &tv);
    /* Don't rely on the value of tv now! */

    if (retval == -1)
        perror("select()");
    else if (retval)
        printf("Data is available now.\n");
    /* FD_ISSET(0, &rfds) will be true. */
    else
        printf("No data within five seconds.\n");

    exit(EXIT_SUCCESS);
}
```

### See also

[accept(2)](https://www.man7.org/linux/man-pages/man2/accept.2.html), [connect(2)](https://www.man7.org/linux/man-pages/man2/connect.2.html), [poll(2)](https://www.man7.org/linux/man-pages/man2/poll.2.html), [read(2)](https://www.man7.org/linux/man-pages/man2/read.2.html), [recv(2)](https://www.man7.org/linux/man-pages/man2/recv.2.html), [restart_syscall(2)](https://www.man7.org/linux/man-pages/man2/restart_syscall.2.html), [send(2)](https://www.man7.org/linux/man-pages/man2/send.2.html), [sigprocmask(2)](https://www.man7.org/linux/man-pages/man2/sigprocmask.2.html), [write(2)](https://www.man7.org/linux/man-pages/man2/write.2.html), [epoll(7)](https://www.man7.org/linux/man-pages/man7/epoll.7.html), [time(7)](https://www.man7.org/linux/man-pages/man7/time.7.html)

For a tutorial with discussion and examples, see [select_tut(2)](https://www.man7.org/linux/man-pages/man2/select_tut.2.html).

## [epoll_create](https://www.man7.org/linux/man-pages/man2/epoll_create.2.html)

### NAME

`epoll_create`，`epoll_create1` - 打开一个epoll文件描述符

### SYNOPSIS

```c
#include <sys/epoll.h>

int epoll_create(int size);
int epoll_create1(int flags);
```

### DESCRIPTION

`epoll_create()`创建一个新的`epoll(7)`实例。因为Linux 2.6.8，`size`参数被忽略，但是必须大于零；参阅NOTES。

`epoll_create()`返回指向新`epoll`实例的文件描述符。此文件描述符用于对`epoll`接口的所有后续调用。当不再需要时，`epoll_create()`返回的文件描述符应该使用`close(2)`关闭。当所有引用`epoll`实例的文件描述符都被关闭时，内核将销毁该实例并释放相关的资源以供重用。

`epoll_create1()`

如果`flags`为0，那么除了删除过时的size参数外，`epoll_create1()`与`epoll_create()`是相同的。以下值可以包含在flags中以获得不同的行为:

		 ` EPOLL_CLOEXEC`：在新的文件描述符上设置执行时关闭(`FD_CLOEXEC`)标志。请参阅`open(2)`中`O_CLOEXEC`标志的描述，以了解为什么它可能有用。


### RETURN VALUE

如果成功，这些系统调用将返回一个文件描述符(非负整数)。如果发生错误，则返回-1，并设置errno来指示错误。

### ERRORS

- `EINVAL` `size`不是正数（不大于0）。

- `EINVAL` (`epoll_create1()`)指定的值无效。

- `EMFILE` 超过`/proc/sys/fs/epoll/max_user_instances`对每个用户施加的`epoll`实例数量的限制。详见`epoll(7)`。
- `EMFILE` 已达到每个进程对打开文件描述符数量的限制。
- `ENFILE `已达到系统范围内打开文件总数的限制。
- `ENOMEM` 内存不足，无法创建内核对象。

### VERSIONS

`epoll_create()`在2.6版本中被添加到内核中。从2.3.2版本开始，glibc提供了库支持。

`epoll_create1()`在2.6.27版本中被添加到内核中。从2.9版本开始，glibc提供了库支持。

### CONFORMING TO

`epoll_create()`和`epoll_create1()`是Linux特有的。

### NOTES

在初始`epoll_create()`实现中，size参数告知内核调用者希望添加到epoll实例的文件描述符的数量。内核使用该信息作为在描述事件的内部数据结构中初始分配空间量的提示。(如果有必要，如果调用者的使用量超过给出的大小提示，内核将分配更多的空间。)现在，不再需要这个提示了(内核不需要提示就可以动态地调整所需数据结构的大小)，但是size仍然必须大于零，以便在旧内核上运行新的epoll应用程序时确保向后兼容性。

### SEE ALSO

[`close(2)`](https://www.man7.org/linux/man-pages/man2/close.2.html), [`epoll_ctl(2)`](https://www.man7.org/linux/man-pages/man2/epoll_ctl.2.html), [`epoll_wait(2)`](https://www.man7.org/linux/man-pages/man2/epoll_wait.2.html), [`epoll(7)`](https://www.man7.org/linux/man-pages/man7/epoll.7.html)

## epoll_ctl(2)

### Name

epoll_ctl - epoll文件描述符的控制接口

### Synopsis

```c
#include <sys/epoll.h>
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

### Description

这个系统调用用于在`epoll(7)`实例的interest列表中添加、修改或删除条目，`epoll(7)`实例由文件描述符`epfd`引用。它请求对目标文件描述符`fd`执行操作操作。

op参数的有效值如下:

- `EPOLL_CTL_ADD`：向epoll文件描述符`epfd`的interest列表添加一个条目。该条目包括文件描述符`fd`、对相应打开文件描述的引用(参见`epoll(7)`和`open(2)`)以及事件中指定的设置。
- `EPOLL_CTL_MOD`：将interest列表中与`fd`关联的设置更改为事件中指定的新设置。
- `EPOLL_CTL_DEL`：从interest列表中删除(取消注册)目标文件描述符`fd`。event参数被忽略，可以为NULL(但请参阅下面的bug)。

event参数描述了链接到文件描述符fd的对象。结构epoll_event被定义为:

```c
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};
```

`epoll_event`结构的data成员指定了当这个文件描述符准备好时，内核应该保存并返回(通过`epoll_wait(2)`)的数据。

`epoll_event`结构的events成员是一个位掩码，由以下零个或多个可用事件类型ORing组成:

- `EPOLLIN`：关联文件可用于`read(2)`操作。

- `EPOLLOUT`：关联文件可用于`write(2)`操作。

- `EPOLLRDHUP`：流套接字对等关闭连接，或关闭写入连接的一半。(当使用边缘触发监视时，这个标志对于编写简单的代码来检测对等关闭特别有用。)

- `EPOLLPRI`：在文件描述符上有一个异常条件。参见`poll(2)`中对`POLLPRI`的讨论。

- `EPOLLERR`：关联文件描述符发生错误条件。当管道的读端已关闭时，也会报告该事件。`epoll_wait(2)`将始终报告此事件；在调用`epoll_ctl()`时，不需要在事件中设置它。

- `EPOLLHUP`：关联文件描述符上发生了挂起。`epoll_wait(2)`将始终等待此事件；在调用`epoll_ctl()`时，不需要在事件中设置它。请注意，当从通道(如管道或流套接字)读取数据时，此事件仅表明对等端关闭了通道的末端。

  只有在通道中所有未完成的数据都被消耗掉之后，后续对通道的读取才会返回0(文件结束)。

- `EPOLLET`：为关联的文件描述符请求边缘触发的通知。epoll的默认行为是级别触发的。请参阅epoll(7)了解更多关于边缘触发和级别触发通知的详细信息。该标志是事件的输入标志。事件字段调用epoll_ctl();epoll_wait(2)永远不会返回它。

- `EPOLLONESHOT` (since Linux 2.6.2)：请求关联文件描述符的一次性通知。这意味着，在epoll_wait(2)为文件描述符通知了一个事件之后，该文件描述符将在兴趣列表中被禁用，epoll接口将不会报告其他事件。用户必须使用EPOLL_CTL_MOD调用epoll_ctl()来使用新的事件掩码重新武装文件描述符。

  该标志是事件的输入标志。事件字段调用epoll_ctl();epoll_wait(2)永远不会返回它。

- `EPOLLWAKEUP` (since Linux 3.5)：

  如果EPOLLONESHOT和EPOLLET是清除的，并且进程有CAP_BLOCK_SUSPEND能力，请确保当该事件挂起或正在处理时，系统不会进入“suspend”或“hibernate”。从调用epoll_wait(2)返回事件到下一次调用epoll_wait(2)对同一epoll(7)文件描述符、关闭该文件描述符、使用EPOLL_CTL_DEL删除事件文件描述符或使用EPOLL_CTL_MOD清除事件文件描述符的EPOLLWAKEUP，事件被认为是被“processed”的。参见BUGS。

  该标志是事件的输入标志。事件字段调用epoll_ctl();epoll_wait(2)永远不会返回它。

- `EPOLLEXCLUSIVE` (since Linux 4.5)：

  		  为附加到目标文件描述符fd的epoll文件描述符设置独占唤醒模式。当一个唤醒事件发生，并且使用EPOLLEXCLUSIVE将多个epoll文件描述符附加到同一个目标文件时，一个或多个epoll文件描述符将使用epoll_wait(2)接收一个事件。在此场景中(未设置EPOLLEXCLUSIVE)，默认情况下所有epoll文件描述符都会接收一个事件。因此，EPOLLEXCLUSIVE对于在某些情况下避免羊群问题是有用的。

    		  如果相同的文件描述符在多个epoll实例中，有些带有EPOLLEXCLUSIVE标志，而有些没有，那么事件将提供给所有没有指定EPOLLEXCLUSIVE的epoll实例，以及至少一个指定了EPOLLEXCLUSIVE的epoll实例。
    	
    		  POLLEXCLUSIVE一起指定:EPOLLIN, EPOLLOUT, EPOLLWAKEUP和EPOLLET。EPOLLHUP和EPOLLERR也可以指定，但这不是必需的:通常，如果发生了这些事件，无论它们是否在事件中指定，都将报告它们。如果试图在事件中指定其他值，则会产生EINVAL错误。
    	
    		  EPOLLEXCLUSIVE只能在EPOLL_CTL_ADD操作中使用;试图在EPOLL_CTL_MOD中使用它会产生错误。如果使用epoll_ctl()设置了EPOLLEXCLUSIVE，那么在同一epfd, fd对上的后续EPOLL_CTL_MOD会产生一个错误。在事件中指定EPOLLEXCLUSIVE并指定目标文件描述符fd作为epoll实例的epoll_ctl()调用同样会失败。所有这些情况下的误差都是EINVAL。
    	
    		  EPOLLEXCLUSIVE标志是事件的输入标志。事件字段调用epoll_ctl();epoll_wait(2)永远不会返回它。

### Return Value

当成功时，epoll_ctl()返回0。当发生错误时，epoll_ctl()返回-1，并设置errno来指示错误。

### Errors

| 编码     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| `EBADF`  | epfd或fd不是有效的文件描述符。                               |
| `EEXIST` | op为EPOLL_CTL_ADD，并且提供的文件描述符fd已经注册到这个epoll实例。 |
| `EINVAL` | epfd不是epoll文件描述符，或者fd和epfd相同，或者请求的操作操作不支持本接口。 |
| `EINVAL` | 事件中指定EPOLLEXCLUSIVE的事件类型是无效的。                 |
| `EINVAL` | op为EPOLL_CTL_MOD，事件包括EPOLLEXCLUSIVE。                  |
| `EINVAL` | op为EPOLL_CTL_MOD, EPOLLEXCLUSIVE标志之前已经应用于这个epfd, fd对。 |
| `EINVAL` | EPOLLEXCLUSIVE在event中指定，fd引用epoll实例。               |
| `ELOOP`  | fd指的是一个EPOLL_CTL_ADD操作，该EPOLL_CTL_ADD操作将导致epoll实例相互监视的循环循环或epoll实例的嵌套深度大于5。 |
| `ENOENT` | op为EPOLL_CTL_MOD或EPOLL_CTL_DEL，并且fd没有注册到这个epoll实例。 |
| `ENOMEM` | 内存不足，无法处理请求的操作控制操作。                       |
| `ENOSPC` | 在试图在epoll实例上注册(EPOLL_CTL_ADD)一个新的文件描述符时遇到了/proc/sys/fs/epoll/max_user_watches施加的限制。详见epoll(7)。 |
| `EPERM`  | 目标文件fd不支持epoll。如果fd引用常规文件或目录等，则可能发生此错误。 |

### Versions

epoll_ctl()在2.6版本中被添加到内核中。从2.3.2版本开始，glibc提供了库支持。

### Conforming To

epoll_ctl()是Linux特有的。

### Notes

epoll接口支持所有支持poll(2)的文件描述符。

### Bugs

在2.6.9之前的内核版本中，`EPOLL_CTL_DEL`操作在事件中需要一个非空指针，即使这个参数被忽略。从Linux 2.6.9开始，当使用`EPOLL_CTL_DEL`时，事件可以被指定为NULL。在2.6.9之前需要移植到内核的应用程序应该在事件中指定一个非空指针。

如果`EPOLLWAKEUP`在标志中指定，但是调用者没有`CAP_BLOCK_SUSPEND`功能，那么`EPOLLWAKEUP`标志将被静默忽略。这种不幸的行为是必要的，因为在原始实现中没有对`flags`参数执行有效性检查，并且如果调用方不具备`CAP_BLOCK_SUSPEND`功能，那么`EPOLLWAKEUP`和检查将导致调用失败，导致至少一个现有用户空间应用程序中断，该应用程序碰巧随机(且无用)指定了这个位。因此，一个健壮的应用程序如果试图使用`EPOLLWAKEUP`标志，应该再次检查它是否具有`CAP_BLOCK_SUSPEND`功能。

### See Also

[epoll_create(2)](https://www.man7.org/linux/man-pages/man2/epoll_create.2.html), [epoll_wait(2)](https://www.man7.org/linux/man-pages/man2/epoll_wait.2.html), [poll(2)](https://www.man7.org/linux/man-pages/man2/poll.2.html), [epoll(7)](https://www.man7.org/linux/man-pages/man7/epoll.7.html)  

## epoll_wait

### Name

`epoll_wait`, `epoll_pwait`, `epoll_pwait2` - 等待epoll文件描述符上的I/O事件

### Synopsis

```c
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout,
const sigset_t *sigmask);
int epoll_pwait2(int epfd, struct epoll_event *events, int maxevents, const struct timespec *timeout, const sigset_t *sigmask);
```

### Description

epoll_wait()系统调用等待文件描述符epfd引用的epoll(7)实例上的事件。事件指向的缓冲区用于从就绪列表中返回关于有一些事件可用的兴趣列表中的文件描述符的信息。epoll_wait()最多返回maxevents。maxevents参数必须大于零。

timeout参数指定epoll_wait()将阻塞的毫秒数。时间是用CLOCK_MONOTONIC时钟来测量的。

调用epoll_wait()将被阻塞，直到:

- 文件描述符传递一个事件;
- 调用被信号处理程序中断;或
- 超时过期。

请注意，超时时间将被四舍五入到系统时钟粒度，而内核调度延迟意味着阻塞时间间隔可能会超出一小部分。指定-1的超时将导致epoll_wait()无限阻塞，而指定等于0的超时将导致epoll_wait()立即返回，即使没有可用的事件。

结构epoll_event被定义为:

```c
typedef union epoll_data {
    void    *ptr;
    int      fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;    /* Epoll events */
    epoll_data_t data;      /* User data variable */
};
```

每个返回的epoll_event结构的数据字段包含与最近调用epoll_ctl(2) (EPOLL_CTL_ADD, EPOLL_CTL_MOD)中指定的对应打开文件描述符相同的数据。

events字段是一个位掩码，指示对应打开的文件描述已经发生的事件。请参阅epoll_ctl(2)以获得可能出现在此掩码中的位的列表。

<font style="background-color:#02aaf4">epoll_pwait()</font>

epoll_wait()和epoll_pwait()之间的关系类似于select(2)和pselect(2)之间的关系:与pselect(2)类似，epoll_pwait()允许应用程序安全地等待，直到文件描述符准备就绪或捕获信号。

下面的epoll_pwait()调用:

```c
ready = epoll_pwait(epfd, &events, maxevents, timeout, &sigmask);
```

等价于原子地执行以下调用:

```c
sigset_t origmask;

pthread_sigmask(SIG_SETMASK, &sigmask, &origmask);
ready = epoll_wait(epfd, &events, maxevents, timeout);
pthread_sigmask(SIG_SETMASK, &origmask, NULL);
```

sigmask参数可以指定为NULL，在这种情况下epoll_pwait()等价于epoll_wait()。

<font style="background-color:#02aaf4">epoll_pwait2()</font>

epoll_pwait2()系统调用等价于epoll_pwait()，除了timeout参数。它需要一个timespec类型的参数才能指定纳秒分辨率超时。该参数的作用与pselect(2)和ppoll(2)相同。如果timeout为NULL，那么epoll_pwait2()可以无限阻塞。

### Return Value

如果成功，epoll_wait()返回为请求的I/O准备好的文件描述符的数量，如果在请求的超时毫秒内没有文件描述符准备好，则返回0。如果失败，epoll_wait()返回-1，并设置errno来指示错误。

### Errors

| 编码   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| EBADF  | epfd不是一个有效的文件描述符。                               |
| EFAULT | 事件指向的内存区域无法用写权限访问。                         |
| EINTR  | 在(1)任何被请求的事件发生之前或(2)超时之前，调用被信号处理程序中断;看到signal(7)。 |
| EINVAL | epfd不是epoll文件描述符，或者maxevents小于等于0。            |

### Versions

epoll_wait()在2.6版本中被添加到内核中。从2.3.2版本开始，glibc提供了库支持。

epoll_pwait()在2.6.19内核中被添加到Linux中。从版本2.6开始，glibc提供了库支持。

epoll_pwait2()在5.11内核中被添加到Linux中。

### Conforming To

epoll_wait(), epoll_pwait()和epoll_pwait2()是Linux特有的。

### Notes

当一个线程在调用epoll_wait()时被阻塞时，另一个线程可以向等待的epoll实例添加文件描述符。如果新的文件描述符准备好了，它将导致epoll_wait()调用解除阻塞。

如果在调用epoll_wait()时，就绪的maxevents文件描述符多于maxevents文件描述符，那么连续的epoll_wait()调用将轮询整个就绪文件描述符集。这种行为有助于避免出现供不应求的情况，在这种情况下，进程不会注意到额外的文件描述符已经准备好了，因为它关注的是一组已知已经准备好的文件描述符。

注意，可以对兴趣列表当前为空的epoll实例调用epoll_wait()(或者因为文件描述符关闭或从另一个线程的兴趣列表中删除而使其兴趣列表变为空)。这个调用将会阻塞，直到某个文件描述符稍后被添加到兴趣列表(在另一个线程中)并且该文件描述符已经准备好。

C library/kernel差异

原始epoll_pwait()和epoll_pwait2()系统调用有第六个参数，size_t sigsetsize，它指定sigmask参数的大小(以字节为单位)。glibc epoll_pwait()包装器函数将此参数指定为固定值(等于sizeof(sigset_t))。

### Bugs

在2.6.37之前的内核中，超过LONG_MAX / HZ毫秒的超时值被视为-1(即无穷大)。因此，例如，在sizeof(long)为4、内核HZ值为1000的系统中，这意味着大于35.79分钟的超时被视为无穷。

### See Also

[epoll_create(2)](https://www.man7.org/linux/man-pages/man2/epoll_create.2.html), [epoll_ctl(2)](https://www.man7.org/linux/man-pages/man2/epoll_ctl.2.html), [epoll(7)](https://www.man7.org/linux/man-pages/man7/epoll.7.html)

## splice

### Name

splice - 将数据与管道拼接

### Synopsis

```C
#define _GNU_SOURCE         /* See feature_test_macros(7) */
#include <fcntl.h>

ssize_t splice(int fd_in, off64_t *off_in, int fd_out, off64_t *off_out, 
               size_t len, unsigned int flags);
```

### DESCRIPTION

```
       splice() moves data between two file descriptors without copying
       between kernel address space and user address space.  It
       transfers up to len bytes of data from the file descriptor fd_in
       to the file descriptor fd_out, where one of the file descriptors
       must refer to a pipe.

       The following semantics apply for fd_in and off_in:

       *  If fd_in refers to a pipe, then off_in must be NULL.

       *  If fd_in does not refer to a pipe and off_in is NULL, then
          bytes are read from fd_in starting from the file offset, and
          the file offset is adjusted appropriately.

       *  If fd_in does not refer to a pipe and off_in is not NULL, then
          off_in must point to a buffer which specifies the starting
          offset from which bytes will be read from fd_in; in this case,
          the file offset of fd_in is not changed.

       Analogous statements apply for fd_out and off_out.

       The flags argument is a bit mask that is composed by ORing
       together zero or more of the following values:

       SPLICE_F_MOVE
              Attempt to move pages instead of copying.  This is only a
              hint to the kernel: pages may still be copied if the
              kernel cannot move the pages from the pipe, or if the pipe
              buffers don't refer to full pages.  The initial
              implementation of this flag was buggy: therefore starting
              in Linux 2.6.21 it is a no-op (but is still permitted in a
              splice() call); in the future, a correct implementation
              may be restored.

       SPLICE_F_NONBLOCK
              Do not block on I/O.  This makes the splice pipe
              operations nonblocking, but splice() may nevertheless
              block because the file descriptors that are spliced
              to/from may block (unless they have the O_NONBLOCK flag
              set).

       SPLICE_F_MORE
              More data will be coming in a subsequent splice.  This is
              a helpful hint when the fd_out refers to a socket (see
              also the description of MSG_MORE in send(2), and the
              description of TCP_CORK in tcp(7)).

       SPLICE_F_GIFT
              Unused for splice(); see vmsplice(2).
```

### RETURN VALUE

```
       Upon successful completion, splice() returns the number of bytes
       spliced to or from the pipe.

       A return value of 0 means end of input.  If fd_in refers to a
       pipe, then this means that there was no data to transfer, and it
       would not make sense to block because there are no writers
       connected to the write end of the pipe.

       On error, splice() returns -1 and errno is set to indicate the
       error.
```

### ERRORS

| 标签     | 描述 |
| -------- | ---- |
| `EAGAIN` |      |
| `EBADF`  |      |
| `EINVAL` |      |
| `EINVAL` |      |
| `EINVAL` |      |
| `EINVAL` |      |
| `EINVAL` |      |
| `ENOMEM` |      |
| `ESPIPE` |      |



```
       EAGAIN SPLICE_F_NONBLOCK was specified in flags or one of the
              file descriptors had been marked as nonblocking
              (O_NONBLOCK), and the operation would block.

       EBADF  One or both file descriptors are not valid, or do not have
              proper read-write mode.

       EINVAL The target filesystem doesn't support splicing.

       EINVAL The target file is opened in append mode.

       EINVAL Neither of the file descriptors refers to a pipe.

       EINVAL An offset was given for nonseekable device (e.g., a pipe).

       EINVAL fd_in and fd_out refer to the same pipe.

       ENOMEM Out of memory.

       ESPIPE Either off_in or off_out was not NULL, but the
              corresponding file descriptor refers to a pipe.
```

### VERSIONS

`splice()`系统调用首次出现在Linux 2.6.17中；glibc在2.5版本中添加了库支持。

### CONFORMING TO

这个系统调用是Linux特有的。

### NOTES

```
       The three system calls splice(), vmsplice(2), and tee(2), provide
       user-space programs with full control over an arbitrary kernel
       buffer, implemented within the kernel using the same type of
       buffer that is used for a pipe.  In overview, these system calls
       perform the following tasks:

       • splice() moves data from the buffer to an arbitrary file
         descriptor, or vice versa, or from one buffer to another.

       • tee(2) "copies" the data from one buffer to another.

       • vmsplice(2) "copies" data from user space into the buffer.

       Though we talk of copying, actual copies are generally avoided.
       The kernel does this by implementing a pipe buffer as a set of
       reference-counted pointers to pages of kernel memory.  The kernel
       creates "copies" of pages in a buffer by creating new pointers
       (for the output buffer) referring to the pages, and increasing
       the reference counts for the pages: only pointers are copied, not
       the pages of the buffer.

       In Linux 2.6.30 and earlier, exactly one of fd_in and fd_out was
       required to be a pipe.  Since Linux 2.6.31, both arguments may
       refer to pipes.
```

### EXAMPLES

See tee(2).   

### SEE ALSO

`copy_file_range(2)`, `sendfile(2)`, `tee(2)`, `vmsplice(2)`, `pipe(7)`

## COLOPHON

这个页面是Linux手册页项目5.13版本的一部分。可以在https://www.kernel.org/doc/man-pages/上找到项目的描述、报告错误的信息以及该页面的最新版本。