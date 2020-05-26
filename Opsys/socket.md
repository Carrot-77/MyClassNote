# socket

### socket()

```c
int socket(int protofamily, int type, int protocol);
```

sockfd是描述符。socket函数对应于普通文件的打开操作。普通文件的打开操作返回一个文件描述字，而socket()用于创建一个socket描述符（socket descriptor），它唯一标识一个socket。这个socket描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作。
创建socket的时候，可以指定不同的参数创建不同的socket描述符，socket函数的三个参数分别为：

`protofamily`：即协议域，又称为协议族（family）。常用的协议族有，AF_INET(IPV4)、AF_INET6(IPV6)、AF_LOCAL（或称AF_UNIX，Unix域socket）、AF_ROUTE等等。协议族决定了socket的地址类型，在通信中必须采用对应的地址，如AF_INET决定了要用ipv4地址（32位的）与端口号（16位的）的组合、AF_UNIX决定了要用一个绝对路径名作为地址。

`type`：指定socket类型。常用的socket类型有，SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等。

`protocol`：故名思意，就是指定协议。常用的协议有，IPPROTO_TCP、IPPTOTO_UDP、IPPROTO_SCTP、IPPROTO_TIPC等，它们分别对应TCP传输协议、UDP传输协议、STCP传输协议、TIPC传输协议

**返回值**

int sockfd

### bind()

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

通常服务器在启动的时候都会绑定一个众所周知的地址（如ip地址+端口号），用于提供服务，客户就可以通过它来接连服务器；而客户端就不用指定，有系统自动分配一个端口号和自身的ip地址组合。这就是为什么通常服务器端在listen之前会调用bind()，而客户端就不会调用，而是在connect()时由系统随机生成一个。

`sockfd`：即socket描述字，它是通过socket()函数创建了，唯一标识一个socket。bind()函数就是将给这个描述字绑定一个名字。
`addr`：一个const struct sockaddr *指针，指向要绑定给sockfd的协议地址。
`addrlen`：对应的是地址的长度。

### listen()

```c
int listen(int sockfd, int backlog);
```

`sockfd`：要监听的socket描述字

`backlog`：相应socket可以排队的最大连接个数。

socket()函数创建的socket默认是一个主动类型的，listen函数将socket变为被动类型的，等待客户的连接请求。

### accept()

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen); //返回连接connect_fd
```

`sockfd`：参数sockfd就是上面解释中的监听套接字，这个套接字用来监听一个端口，当有一个客户与服务器连接时，它使用这个一个端口号，而此时这个端口号正与这个套接字关联。当然客户不知道套接字这些细节，它只知道一个地址和一个端口号。
`addr`：这是一个结果参数，它用来接受一个返回值，这返回值指定客户端的地址，当然这个地址是通过某个地址结构来描述的，用户应该知道这一个什么样的地址结构。如果对客户的地址不感兴趣，那么可以把这个值设置为NULL。
`addrlen`：它也是结果的参数，用来接受上述addr的结构的大小的，它指明addr结构所占有的字节个数。同样的，它也可以被设置为NULL。
　　如果accept成功返回，则服务器与客户已经正确建立连接了，此时服务器通过accept返回的套接字来完成与客户的通信。
注意：accept默认会阻塞进程，直到有一个客户连接建立后返回，它返回的是一个新可用的套接字，这个套接字是连接套接字。

### recv()

### connect()

```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

`sockfd`：客户端的socket描述字

`addr`：服务器的socket地址

`addrlen`： socket地址的长度

客户端通过调用connect函数来建立与TCP服务器的连接。

### send()

### 多进程实现server端

socket

bind

listen

accept

recv

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char **argv) {
	if (argc != 2) {
		fprintf(stderr, "Usage: %s port\n", argv[0]);
		exit(1);
	}
	int port, server_listen;
	port = atoi(argv[1]);

	if ((server_listen = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		perror("socket");
		exit(1);
	}

	printf("Socket create.\n");
	struct sockaddr_in server;
	server.sin_family = AF_INET;
	server.sin_port = htons(port);
	server.sin_addr.s_addr = INADDR_ANY;

	if (bind(server_listen, (struct sockaddr *)&server, sizeof(server)) < 0) {
		perror("bind");
		exit(1);
	}

	printf("Socket listen.\n");
	if (listen(server_listen, 20) < 0) {
		perror("server");
		exit(1);
	}

	while (1) {
		int sockfd;
		printf("Socket before.\n");
		if ((sockfd = accept(server_listen, NULL, NULL)) < 0) {
			perror("accept");
			close(sockfd);
			continue;
		}

		pid_t pid;
		if ((pid = fork()) < 0) {
			perror("fork");
			continue;
		}
		if (pid == 0) {
			close(server_listen);
			char name[20] = {0};
			if (recv(sockfd, name, sizeof(name), 0) <= 0) {
				perror("recv");
				close(sockfd);
			}
			printf("Name = %s\n", name);
			exit(0);
		} 		
	}
	return 0;
}


```

### 多线程实现server端

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <pthread.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>

struct Msg {
    int server_listen, sockfd;
};

void *recv_data(void *arg) {
    struct Msg *tmp;
    tmp = (struct Msg *)arg;
    char name[512] = {0};
    if (recv(tmp->sockfd, name, sizeof(name), 0) <= 0) {
        perror("recv");
        close(tmp->sockfd);
    }
    printf("Name = %s\n", name);
    pthread_exit(NULL);
}

int main(int argc, char **argv) {
	if (argc != 2) {
		fprintf(stderr, "Usage: %s port\n", argv[0]);
		exit(1);
	}
	int port, server_listen;
	port = atoi(argv[1]);

	if ((server_listen = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		perror("socket");
		exit(1);
	}

	printf("Socket create.\n");
	struct sockaddr_in server;
	server.sin_family = AF_INET;
	server.sin_port = htons(port);
	server.sin_addr.s_addr = INADDR_ANY;

	if (bind(server_listen, (struct sockaddr *)&server, sizeof(server)) < 0) {
		perror("bind");
		exit(1);
	}

	printf("Socket listen.\n");
	if (listen(server_listen, 20) < 0) {
		perror("server");
		exit(1);
	}

    while (1) {
        pthread_t thread;
        int sockfd;
        if ((sockfd = accept(server_listen, NULL, NULL)) < 0) {
            perror("accept");
            close(sockfd);
            continue;
        }
        struct Msg msg;
        msg.sockfd = sockfd;
        msg.server_listen = server_listen;
        printf("%d %d\n", msg.sockfd, msg.server_listen);
        if (pthread_create(&thread, NULL, recv_data, (void *)&msg) != 0) {
            perror("pthread create");
            break;
        }
    }
    return 0;
}


```

### client端

socket

connect

send

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char **argv) {
	int sockfd, port;
    struct sockaddr_in server;
    if (argc != 3) {
		fprintf(stderr, "Usage: %s ip port \n", argv[0]);
        exit(1);
	}
    port = atoi(argv[2]);

    server.sin_family = AF_INET;
    server.sin_port = htons(port);
    server.sin_addr.s_addr = inet_addr(argv[1]);

    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket");
        exit(1);
    }
    printf("Socket create. \n");
    if (connect(sockfd, (struct sockaddr *)&server, sizeof(server)) < 0) {
        perror("connect");
        exit(1);
    }
    if (send(sockfd, "lll", sizeof("llll"), 0) < 0) {
        perror("send");
        exit(1);
    }
    close(sockfd);
	return 0;
}


```