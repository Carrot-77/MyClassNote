## **mmap函数**

### **原型** 

```c
#include <sys/mman.h>                
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset); 
```

### **功能**

将文件所在的磁盘空间映射到进程空间。

### **参数**

**addr**：人为指定映射的起始虚拟地址

如果设置为NULL，表示由内核决定映射的起始虚拟地址，这也是最常见的设置方式，这与我们调用shmat映射共享内存时指定NULL是一样的。

如果设置不为NULL，就表示由自己指定，指定的起始虚拟地址必须是虚拟页（4k）的整数倍，这与自己指定shmat的映射起始虚拟地址也是一样的。

**length**：映射长度，也就是你想对文件映射多长。

**prot**：指定对映射区的操作权限，可指定如下宏：

```c
PROT_EXEC 
//映射区的内容可执行。

//如果你映射的是普通文件是一个可执行文件的话，将映射权限指定为PROT_EXEC后，是能够通过映射后的虚拟地址去执行文件中的“指令”。

PROT_READ 
//映射区的内容可读。

ROT_WRITE 
//映射区的内容可写
//以上三种选项可相互 | 操作。比如：PROT_EXEC | PROT_READ

PROT_NONE 
//映射区不允许访问（不允许执行、读、写），一般不会指定这个，如果指定为不可访问的话，映射就没有意义了。

```



**flags**：向映射区写入了数据，是否将数据立即更新到文件中。

```c
//对映射区域的写入数据会复制回文件, 且允许其他映射该文件的进程共享.
MAP_SHARED

//对映射区域的写入操作会产生一个映射的复制(copy-on-write), 对此区域所做的修改不会写回原文件
MAP_PRIVATE
```



**fd**：需要被映射文件的描述符。

**offset**：表示从文件头的offset处开始映射。一般都指定为0，表示从文件头开始映射。

### **返回值**

调用成功，返回映射的起始虚拟地址，失败则返回(void*)-1，errno被设置。

## **munmap函数**

### **原型**

```
#include <sys/mman.h>                    
int munmap(void *addr, size_t length); 
```

### **功能**

取消映射

### **参数**

**addr**：映射的起始虚拟地址

**length**：需要取消的长度

### **返回值**

调用成功返回0, 失败则-1, errno被设置。调用成功返回0, 失败则-1, errno被设置。

### **代码演示**



**man手册代码**



```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define handle_error(msg) \
    do { perror(msg); exit(EXIT_FAILURE); } while (0)

int main(int argc, char *argv[]) {
    char *addr;
    int fd;
    struct stat sb;
    off_t offset, pa_offset;
    size_t length;
    ssize_t s;

    if (argc < 3 || argc > 4) {
        fprintf(stderr, "%s file offset [length]\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    fd = open(argv[1], O_RDONLY);
    if (fd == -1)
        handle_error("open");

    if (fstat(fd, &sb) == -1)           /* To obtain file size */
        handle_error("fstat");

    offset = atoi(argv[2]);
	printf("size: %ld\n", sysconf(_SC_PAGE_SIZE));
    pa_offset = offset & ~(sysconf(_SC_PAGE_SIZE) - 1);
        /* offset for mmap() must be page aligned */

    if (offset >= sb.st_size) {
        fprintf(stderr, "offset is past end of file\n");
        exit(EXIT_FAILURE);
    }

    if (argc == 4) {
        length = atoi(argv[3]);
        if (offset + length > sb.st_size)
            length = sb.st_size - offset;
                /* Can't display bytes past end of file */

    } else {    /* No length arg ==> display to end of file */
        length = sb.st_size - offset;
    }

    addr = mmap(NULL, length + offset - pa_offset, PROT_READ,
                MAP_PRIVATE, fd, pa_offset);
    if (addr == MAP_FAILED)
        handle_error("mmap");

    s = write(STDOUT_FILENO, addr + offset - pa_offset, length);
    if (s != length) {
        if (s == -1)
            handle_error("write");

        fprintf(stderr, "partial write");
        exit(EXIT_FAILURE);
    }

    munmap(addr, length + offset - pa_offset);
    close(fd);

    exit(EXIT_SUCCESS);
}

```



写一个例子程序，将A文件的大量数据复制到B文件中。如果采用传统方式，使用read函数从A文件读出数据，然后向B文件write，如果数据量很大的话，复制的效率会非常低，此时我们就可以使用存储映射来实现。

mmap映射文件size为0的文件时，会映射失败，映射失败时内核会向进程发送一个SIGBUS信号，提示mmap失败了，这个信号的默认处理方式是终止，所以当进程收到这个信号时就被异常终止了。如果你不想被这个信号终止，你可以自己忽略或者屏蔽这个信号，一般来说我们不需要忽略和屏蔽该信号。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <strings.h>
#include <errno.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>



void print_err(char *str, int line, int err_no)
{
        printf("%d, %s: %s\n", line, str, strerror(err_no));
        exit(-1);
}

int main(void)
{
    int srcfd = -1;
    int dstfd = -1;
    void *srcaddr = NULL;
    void *dstaddr = NULL;

    
    /* 打开源文件 */
    srcfd = open("./file_lock.h", O_RDWR);
    if(srcfd == -1) print_err("open file_lock.h fial", __LINE__, errno);     
    
    /* 打开目标文件 */
    dstfd = open("./file", O_RDWR|O_CREAT|O_TRUNC, 0664);
    if(dstfd == -1) print_err("open file fial", __LINE__, errno);     
    
    /* mmap映射源文件 */    
    struct stat src_stat = {0};
    fstat(srcfd, &src_stat);//获取文件属性(主要是想得到文件的size)
    srcaddr = mmap(NULL, src_stat.st_size, PROT_READ, MAP_SHARED, srcfd, 0);
    if(srcaddr == (void *)-1) print_err("mmap srcfile fail", __LINE__, errno);
    
    /* mmap映射目标文件 */    
    ftruncate(dstfd, src_stat.st_size);
    /*
    参数1:指定映射的起始虚拟地址,如果制定NULL表示由mmap指定
    参数2: 要映射的长度
    参数3:指定映射后的操作权限,PROT_WRITE/PROT_READ/PROT_EXEC/PROT_NONE
    参数4:是否立即更新到文件中,指定MAP_SHARED,表示理解更新
    参数5:你要映射的那个文件的fd
    参数6:指定一个偏移,表示你要从文件的什么位置开始映射 */
    dstaddr = mmap(NULL, src_stat.st_size, PROT_WRITE, MAP_SHARED, dstfd, 0);
    if(dstaddr == (void *)-1) print_err("mmap dstfile fail", __LINE__, errno);    
    
    
    /* 想办法讲源文件的数据复制到目标文件中 */
    memcpy(dstaddr, srcaddr, src_stat.st_size);

    
    return 0;    
}
```

