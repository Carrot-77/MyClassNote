## select()

#### 1.头文件

```c
#include <sys/select.h>
```

#### 2.函数原型

```c
int select(int nfdp, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```



```c
void FD_CLR(inr fd,fd_set* set) //用来清除描述词组set中相关fd的位

int FD_ISSET(int fd,fd_set* set) // 用来测试描述词组set中相关fd的位是否为真

void FD_SET（int fd,fd_set* set）// 用来设置描述词组set中相关fd的位

void FD_ZERO（fd_set* set）// 用来清除描述词组set的全部位
```

#### 3.参数类型

**timeout**

timeout表示select返回之前的时间上限。

如果timeout==NULL，无期限等待下去，将select置于阻塞状态，一定等到监视文件描述符集合中某个文件描述符发生变化为止。如果是捕获到信号，select返回-1，并将变量errno设置成EINTR。

如果timeout->tv_sec == 0 && timeout->tv_sec == 0 ，不等待直接返回，纯粹的非阻塞函数，不管文件描述符是否有变化，都立刻返回继续执行，文件无变化返回0，有变化返回一个正值。

如果timeout->tv_sec！=0 || timeout->tv_sec！=0 ，等待指定的时间。当有描述符复合条件或者超过超时时间的话，函数返回。

timeval结构体

```c
struct timeval {
    long tv_sec; //秒
    long tv_usec; //微秒
};
```



#### 4.返回值

成功时：返回三中描述符集合中”准备好了“的文件描述符数量。
超时：返回0
错误：返回-1，并设置 errno



#### 5.原理

* select睡眠过程

select会循环遍历它所监测的fd_set（一组文件描述符(fd)的集合）内的所有文件描述符对应的驱动程序的poll函数。驱动程序提供的poll函数首先会将调用select的用户进程插入到该设备驱动对应资源的等待队列(如读/写等待队列)，然后返回一个bitmask告诉select当前资源哪些可用。当select循环遍历完所有fd_set内指定的文件描述符对应的poll函数后，如果没有一个资源可用(即没有一个文件可供操作)，则select让该进程睡眠，一直等到有资源可用为止，进程被唤醒(或者timeout)继续往下执行。

select的调用path如下：sys_select -> core_sys_select -> do_select

* select唤醒过程

select会循环遍历它所监测的fd_set内的所有文件描述符对应的驱动程序的poll函数。驱动程序提供的poll函数首先会将调用select的用户进程插入到该设备驱动对应资源的等待队列(如读/写等待队列)，然后返回一个bitmask告诉select当前资源哪些可用。



#### 6.使用

伪代码

```c
fd_set readfds;
int fd;
FD_ZERO(&readfds)//新定义的变量要清空一下,相当于初始化。
FD_SET(fd,&readfds);//把文件描述符fd加入到readfds中。
//do select 返回
if(FD_ISSET(fd,&readset))//判断是否成功监视
{
    //dosomething
}
```



man手册使用例子

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>


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
    else if (retval) {
  	  char buff[512] = {0};
        printf("Data is available now.\n");
  	  scanf("%s", buff);
    }
        /* FD_ISSET(0, &rfds) will be true. */
    else
        printf("No data within five seconds.\n");

    exit(EXIT_SUCCESS);
}

```

使用select完成TCP非堵塞链接

```c
#include "../common/color.h"
#include "../common/common.h"
#include "../common/tcp_server.h"
#include "../common/head.h"

#define	CLIENTSIZE 10
#define BUFFSIZE 512
char ch_char(char c) {
	if (c >= 'a' && c <= 'z')
		return c - 32;
	return c;
}


int main(int argc, char **argv) {
	if (argc != 2) {
		fprintf(stderr, "Usage: %s port!\n", argv[0]);
		exit(1);
	}
	int server_listen, fd, max_fd;
	int client[CLIENTSIZE] = {0};
	memset(client, -1, CLIENTSIZE);
	if ((server_listen = socket_create(atoi(argv[1]))) < 0) {
		perror("socket_create");
		exit(1);
	}

	make_nonblock(server_listen);
	
	fd_set rfds, wfds, efds;
	max_fd = server_listen;
    client[server_listen] = server_listen;

	while (1) {
		FD_ZERO(&rfds);
		FD_ZERO(&wfds);
		FD_ZERO(&efds);

		FD_SET(server_listen, &rfds);
		
		for (int i = 0; i < CLIENTSIZE; i++) {
			if (client[i] == server_listen) continue;
			if (client[i] > 0) {
				if (max_fd < client[i]) max_fd = client[i];
				FD_SET(client[i], &rfds);
			}
		}

		if (select(max_fd + 1, &rfds, NULL, NULL, NULL) < 0) {
			perror("select");
			return 1;
		}

		if (FD_ISSET(server_listen, &rfds)) {
			printf("Connect ready on serverlisten\n");
			if ((fd = accept(server_listen, NULL, NULL)) < 0) {
				perror("accept");
				return 1;
			}
			if (fd > CLIENTSIZE) {
				printf("too many clients\n");
				close(fd);
			} else {
				make_nonblock(fd);
				if (client[fd] == -1)
					client[fd] = fd;
			}
		}
		for (int i = 0; i < CLIENTSIZE; i++) {
			if (i == server_listen) continue;
			if (FD_ISSET(i, &rfds)) {
				char buff[BUFFSIZE] = {0};
				int retval = recv(i, buff, BUFFSIZE, 0);
				if (retval <= 0) {
					printf("logout!\n");
					client[i] = -1;
					close(i);
					continue;
				}
				printf("recv: %s", buff);
				for (int j = 0; j < strlen(buff); j++) {
					buff[j] = ch_char(buff[j]);
				}
				send(i, buff, strlen(buff), 0);
			}
		}
	}
	return 0;
}


```

