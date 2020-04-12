## poll()

#### 1.头文件

```c
#include <poll.h>
```

#### 2.函数原型

```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout)
```

#### 3.参数类型

**fds:**指向一个结构体数组的第0个元素的指针，每个数组元素都是一个struct pollfd结构，用于指定测试某个给定的fd的条件

```c
struct pollfd{
	int fd;			//文件描述符
	short events;	//等待的事件
	short revents;	//实际发生的事件
};
```

|    常量    | events | revents |           说明           |
| :--------: | :----: | :-----: | :----------------------: |
|   POLLIN   |   Y    |    Y    |  普通或优先级带数据可读  |
| POLLRDNORM |   Y    |    Y    |       普通数据可读       |
| POLLRDBAND |   Y    |    Y    |     优先级带数据可读     |
|  POLLPRI   |   Y    |    Y    |     高优先级数据可读     |
|  POLLOUT   |   Y    |    Y    |       普通数据可写       |
| POLLWRNORM |   Y    |    Y    |       普通数据可写       |
| POLLWRBAND |   Y    |    Y    |     优先级带数据可写     |
|  POLLERR   |        |    Y    |         发生错误         |
|  POLLHUP   |        |    Y    |         发生挂起         |
|  POLLNVAL  |        |    Y    | 描述字不是一个打开的文件 |

**events：**指定监测fd的事件（输入、输出、错误），每一个事件有多个取值

**revents：**revents 域是文件描述符的操作结果事件，内核在调用返回时设置这个域。events 域中请求的任何事件都可能在 revents 域中返回

**nfds：**用来指定第一个参数数组元素个数

**timeout：**指定等待的毫秒数，无论 I/O 是否准备好，poll() 都会返回



| timeout值 | 说明                      |
| --------- | ------------------------- |
| -1        | 永远等待， 知道时间发生   |
| 0         | 立即返回                  |
| >0        | 等待指定timeout时间后返回 |

#### 4.返回值

成功时，poll() 返回结构体中 revents 域不为 0 的文件描述符个数；如果在超时前没有任何事件发生，poll()返回 0；

失败时，poll() 返回 -1，并设置 errno 为下列值之一：

EBADF：一个或多个结构体中指定的文件描述符无效。
EFAULT：fds 指针指向的地址超出进程的地址空间。
EINTR：请求的事件之前产生一个信号，调用可以重新发起。
EINVAL：nfds 参数超出 PLIMIT_NOFILE 值。
ENOMEM：可用内存不足，无法完成请求。



#### 5.原理

如果当前不可读，那么在sys_poll->do_poll中当前进程就会睡眠在等待队列上，这个等待队列是由驱动程序提供的（就是poll_wait中传入的那个）。当可读的时候，驱动程序可能有一部分代码运行了（比如驱动的中断服务程序），那么在这部分代码中，就会唤醒等待队列上的进程，也就是之前睡眠的那个，当那个进程被唤醒后do_poll会再一次的调用驱动程序的poll函数，这个时候应用程序就知道是可读的了。