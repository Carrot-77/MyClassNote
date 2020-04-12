## epoll() 

#### 1.头文件

```c
#include <sys/epoll.h>
```

#### 2.函数原型

```c
int epoll_create(int size) //创建一个epoll实例，文件描述符
    
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event) //将监听的文件描述符添加到epoll实例中，实例代码为将标准输入文件描述符添加到epoll中

int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout) //等待epoll事件从epoll实例中发生， 并返回事件以及对应文件描述符
```

#### 3.参数类型

**`epoll_create(int size)`**

​	创建一个epoll的 句柄，size用来告诉内核这个监听的数目一共有多大。这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值。需要注意的是，当创建好epoll句柄后，它就是会占用一个fd值，在linux下如果查看/proc/进程id/fd/，是能够看到这个fd的，所以在使用完epoll 后，必须调用close()关闭，否则可能导致fd被耗尽。

**`epoll_create1()`**

如果flags为0，epoll_create1（）和删除了过时size参数的epoll_create（）相同。

如果flags中包含以下值就有不同的表现：

EPOLL_CLOEXEC

在文件描述符上面设置执行时关闭（FD_CLOEXEC）标志描述符。

**`int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)`**

​	epoll的事件注册函数，它不同与select()是在监听事件时告诉内核要监听什么类型的事件，而是在这里先注册要监听的事件类型。

- 第一个参数是epoll_create()的返回值
- 第二个参数表示动作，用三个宏来表示：

```c
EPOLL_CTL_ADD //注册新的fd到epfd中；

EPOLL_CTL_MOD //修改已经注册的fd的监听事件；

EPOLL_CTL_DEL //从epfd中删除一个fd；
```

- 第三个参数是需要监听的fd
- 第四个参数是告诉内核需要监听什么事件

struct epoll_event结构体

```c
typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;

struct epoll_event {
    __uint32_t events; /* Epoll events */
    epoll_data_t data; /* User data variable */
};
```

events

```c
EPOLLIN //表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；

EPOLLOUT //表示对应的文件描述符可以写；

EPOLLPRI //表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；

EPOLLERR //表示对应的文件描述符发生错误；

EPOLLHUP //表示对应的文件描述符被挂断；

EPOLLET //将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的；

EPOLLONESHOT //只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里。
```

**`int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout)`**

- epfd:由epoll_create 生成的epoll专用的文件描述符；
- epoll_event:用于回传代处理事件的数组；
- maxevents:每次能处理的事件数；
- timeout:等待I/O事件发生的超时值（ms）；-1永不超时，直到有事件产生才触发，0立即返回。
  返回发生事件数。-1有错误。

#### 4.使用

```c
#include <sys/epoll.h>
#define MAX_EVENTS 10
#include "../common/common.h"
#include "../common/head.h"
#include "../common/tcp_server.h"
#define BUFFSIZE 512

int main(int argc, char **argv) {
	char buff[BUFFSIZE] = {0};
    struct epoll_event ev, events[MAX_EVENTS];
    int listen_sock, conn_sock, nfds, epollfd;
	if (argc != 2) exit(1);
	listen_sock = socket_create(atoi(argv[1]));
    /* Code to set up listening socket, 'listen_sock',
       (socket(), bind(), listen()) omitted */

    epollfd = epoll_create1(0);
    if (epollfd == -1) {
        perror("epoll_create1");
        exit(EXIT_FAILURE);
    }

    ev.events = EPOLLIN;
    ev.data.fd = listen_sock;
    if (epoll_ctl(epollfd, EPOLL_CTL_ADD, listen_sock, &ev) == -1) {
        perror("epoll_ctl: listen_sock");
        exit(EXIT_FAILURE);
    }
	ev.events = EPOLLIN;
	ev.data.fd = listen_sock;

	if (epoll_ctl(epollfd, EPOLL_CTL_ADD, listen_sock, &ev) == -1) {
		perror("epoll_ctl:");
		exit(0);
	}

    for (;;) {
        nfds = epoll_wait(epollfd, events, MAX_EVENTS, -1);
        if (nfds == -1) {
            perror("epoll_wait");
            exit(EXIT_FAILURE);
        }

        for (int n = 0; n < nfds; ++n) {
            if (events[n].data.fd == listen_sock) {
                conn_sock = accept(listen_sock, NULL, NULL);
                if (conn_sock == -1) {
                    perror("accept");
                    exit(EXIT_FAILURE);
                }
                make_nonblock(conn_sock);
                ev.events = EPOLLIN | EPOLLET;
                ev.data.fd = conn_sock;
                if (epoll_ctl(epollfd, EPOLL_CTL_ADD, conn_sock, &ev) == -1) {
                    perror("epoll_ctl: conn_sock");
                    exit(EXIT_FAILURE);
                }
            } else {
                //do_use_fd(events[n].data.fd);
				if (events[n].events & (EPOLLIN | EPOLLHUP | EPOLLERR)) {
					memset(buff, 0, sizeof(buff));
					if (recv(events[n].data.fd, buff, BUFFSIZE, 0) > 0) {
						printf("recv: %s", buff);
						for (int i = 0; i < strlen(buff); i++) {
							if (buff[i] >= 'a' && buff[i] <= 'z') buff[i] -= 32;
						}
						send(events[n].data.fd, buff, strlen(buff), 0);
					} else {
						if (epoll_ctl(epollfd, EPOLL_CTL_DEL, events[n].data.fd, NULL) < 0) {
							perror("epoll_ctrl");
						}
						printf("Logout!\n");
						close(events[n].data.fd);
					}
				}
            }
        }
    }
	return 0;
}

```

