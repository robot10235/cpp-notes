## 第五章 Linux 网络编程基础API

### socket address API

#### big endian, little endian

网络字节序，大端字节序：big endian	network byte order

主机字节序，小端字节序：little endian	host byte order	

`htonl`: host to network long

```c++
#include <netinet/in.h>
unsigned long int htonl(unsigned long int hostlong);
unsigned short int htons(unsigned short int hostshort);
unsigned long int ntohl(unsigned long int netlong);
unsigned short int ntohs(unsigned short int netshort);
```



#### struct sockaddr and others

| protocol family | address family | description |
| --------------- | -------------- | ----------- |
| PF_UNIX         | AF_UNIX        | UNIX        |
| PF_INET         | AF_INET        | TCP/IPv4    |
| PF_INET6        | AF_INET6       | TCP/IPv6    |



##### general address

```c++
#include <bits/socket.h>
struct sockaddr {
    sa_family_t sa_family;
    char sa_data[14];
}
```



##### special address

```c++
#include <sys/un.h>
// UNIX
struct sockaddr_un {
    sa_family_t sin_family;
    char sun_path[108];
};

// TCP/IP
struct sockaddr_in {
   sa_family_t sin_family;
   u_int16_t sin_port;			// network byte order
   struct in_addr sin_addr;		// network byte order
};
```



#### ip address

convert char* to network byte order long

```c++
#include <arpa/inet.h>
in_addr_t inet_addr(const char* strptr);
int inet_aton(const char* cp, struct in_addr* inp);
char* inet_ntoa(struct in_addr in);
```



### socket

domain: PF_INET	PF_UNIX

type: SOCK_STREAM	SOCK_DGRAM

protocol: 0 (default protocol)

文件描述符：**File descriptor，简称fd，当应用程序请求内核打开/新建一个文件时，内核会返回一个文件描述符用于对应这个打开/新建的文件**，其fd本质上就是一个非负整数。 实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。 当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。 在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。

`bind`将my_addr所指的socket地址分配给未命名的`sockfd`文件描述符，`addrlen`参数指出该socket地址的长度。

```c++
#include <sys/types.h>
#include <sys/socket.h>
int socket(int domain, int type, int protocol);	// return socket file descriptor
int bind(int sockfd, const struct sockaddr* my_addr, socklen_t addrlen);	// return 0 success -1 error
int listen(int sockfd, int backlog);	// backlog:内核监听队列最大长度
int accept(int sockfd, struct sockaddr* addr, socklen_t *addrlen);	// sockfd是执行过listen的socket，addr获取client addr
int connect(int sockfd, const struct sockaddr* serv_addr, socklen_t addrlen); // client connect to server, sockfd used to communicate with the server

#include <unistd.h>
int close(int fd);

// TCP read/write
#include <sys/types.h>
#include <sys/socket.h>
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
ssize_t send(int sockfd, const, void* buf, size_t len, int flags);
int accept(int sockfd, struct sockaddr* addr, socklen_t* addrlen);

// UDP
ssize_t recvfrom(int sockfd, void* buf, size_t len, int flags, struct sockaddr* src_addr, socklen_t* addrlen);
ssize_t sendto(int sockfd, void* buf, size_t len, int flags, struct sockaddr* dest_addr, socklen_t addrlen);
```



#### example1 send msg TCP

```c++
struct sockaddr_in address;
bzero(&address, sizeof(address));
address.sin_family = AF_INET;
inet_pton(AF_INET, ip, &address.sin_addr);
address.sin_port = htons(port);

// use default protocol TCP
int sock = socket(PF_INET, SOCK_STREAM, 0);
assert(sock >= 0);

int ret = bind(sock, (struct sockaddr*)&address, sizeof(address));
assert(ret != -1);

ret = listen(sock, backlog);
assert(ret != -1);

struct sockaddr_in client;
socklen_t client_addrlength = sizeof(client);
int connfd = accpet(sock, (struct sockaddr*)&client, &client_addrlength);
if (connfd < 0) {
    printf("errno is %d\n", errno);
}
else {
    char buffer[BUF_SIZE];
    
    memset(buffer, '\0', BUF_SIZE);
    ret = recv(connfd, buffer, BUF_SIZE-1, 0);
    printf("got %d bytes of normal data '%s'\n", ret, buffer)
}
```



#### example2 send msg UDP

```c++
struct sockaddr_in server_address;
bzero(&server_address, sizeof(server_address));
server_address.sin_family = AF_INET;
inet_pton(AF_INET, ip, &server_address.sin_addr);
server_address.sin_port = htons(port);

// use default protocol TCP
int sock = socket(PF_INET, SOCK_STREAM, 0);
assert(sock >= 0);
if (connect(sockfd, (struct sockaddr*)&server_address, sizeof(server_address)) < 0) {
    printf("connection failed\n");
}
else {
   	const char* oob_data = "abc";
    const char* normal_data = "123";
    send(sockfd, normal_data, strlen(normal_data), 0);
    // send out of band data
    send(sockfd, normal_data, strlen(oob_data), MSG_OOB);
    send(sockfd, normal_data, strlen(normal_data), 0);
}
```





## 第6章 高级I/O函数

### pipe

a pipe is a connection between two processes

```c++
#include <unistd.h>
int pipe(int fd[2]);
```

**fd[0] is used for read, fd[1] is used for write**

![Lightbox](https://media.geeksforgeeks.org/wp-content/uploads/Process.jpg)

#### socketpair

both fd can be used for read and write

```c++
#include <sys/types.h>
#include <sys/socket.h>
int socketpair(int domain, int type, int protocol, int fd[2]);
```



### readv writev

**readv 将数据从fd读到分散的内存块 (vector) 中。**

**writev 将多块分散的内存数据 (vector) 一并写入文件描述符中**

用例：当web服务器解析完一个HTTP请求后，如果目标文档存在且客户具有读取该文档的权限，那么它就需要发送一个HTTP应答来传输该文档。这个HTTP应答包含1个状态行、多个头部字段、1个空行和文档的内容。其中前三部分的内容可能在同一块内存，但是文档的内容通常被读入到另外一块单独的内存中。我们可以不需要将它们拼接在一起，而是直接发送。

```c++
#include <sys/uio.h>
ssize_t readv(int fd, const struct iovec* vector, int count);
ssize_t writev(int fd, const struct iovec* vector, int count);
```



### mmap munmap

mmap用于申请内存，可用于进程间通信的共享内存，munmap释放mmap创建的内存。



### fcntl

file control，可以将fd设置为非阻塞





## 第9章 I/O复用

### epoll

epoll使用一组函数来完成任务，而不是单个函数。

其次，epoll把用户关心的fd的事件放在内核里的一个事件表中，不需要像select和poll那样每次调用都要重复传入fd的事件

epoll需要额外的fd表示事件表

```c++
#include <sys/epoll.h>

// return epoll list fd
int epoll_create(int size);

// do fd action to epoll list
// op = EPOLL_CTL_ADD / EPOLL_CTL_MOD / EPOLL_CTL_DEL
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

struct epoll_event {
    __uint32_t events; /* epoll function */
    epoll_data_t data; /* user data */
}

// maxevents 监听最多事件数量
// epoll_wait 如果检测到事件，就将所有就绪事件从内核事件表复制到数组events里
int epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout);
```



#### epoll poll diff

```c++
int ret = poll(fds, MAX_EVENT_NUMBER, -1);
for (int i = 0; i < MAX_EVENT_NUMBER, ++i) {
	if (fds[i].revents & POLLIN) {
        int sockfd = fds[i].fd;
        /* deal with sockfd */
    }
}

int ret = epoll_wait(epollfd, events, MAX_EVENT_NUMBER, -1);
// only ret times
for (int i = 0; i < ret; ++i) {
    int sockfd = events[i].data.fd;
    /* deal with sockfd */
}
```



### Level Trigger / Edge Trigger EPOLLET

LT：当epoll_wait检测到有事件发生并将此事件通知应用程序时，应用程序可以不立即处理该事件。这样下一次通知时还会有之前通知过的事件

ET：应用程序必须立即处理该事件，减少同一epoll事件重复触发的次数

工作差异：LT可以不用直接将该事件（client向server发数据，buffer size 比较小，需要分批读）读完。但是ET需要全部读完

ET的fd必须都是非阻塞的，不然一直在recv。。。



### EPOLLONESHOT

希望一个socket accept在任意时刻只能被一个thread处理，即使是不同的事件（读写异常）

EPOLLONESHOT只触发一次该事件

注册了EPOLLONESHOT的socket一旦被某个线程处理完毕，该线程就应该立即重置这个socket的EPOLLONESHOT事件。

但是监听listenfd是不能oneshot，否则程序只能处理一个客户连接，后续连接不再触发EPOLLIN，因为listenfd显然还没有完成任务。