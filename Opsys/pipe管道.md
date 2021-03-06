## pipe管道

pipe函数的定义如下：

```c
#include<unistd.h>
int pipe(int fd[2]);
```

 pipe函数定义中的fd参数是一个大小为2的一个数组类型的指针。该函数成功时返回0，并将一对打开的文件描述符值填入fd参数指向的数组。失败时返回 -1并设置errno。

通过pipe函数创建的这两个文件描述符 fd[0] 和 fd[1] 分别构成管道的两端，往 fd[1] 写入的数据可以从 fd[0] 读出。并且 fd[1] 一端只能进行写操作，fd[0] 一端只能进行读操作，不能反过来使用。要实现双向数据传输，可以使用两个管道。

默认情况下，这一对文件描述符都是阻塞的。此时，如果我们用read系统调用来读取一个空的管道，则read将被阻塞，知道管道内有数据可读；如果我们用write系统调用往一个满的管道中写数据，则write也将被阻塞，直到管道有足够的空闲空间可用(read读取数据后管道中将清除读走的数据)。当然，用户可自行将 fd[0] 和 fd[1] 设置为非阻塞的。    

