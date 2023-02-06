

## 基本连接流程

在网络编程中，socket基本上以fd的形式存在

### 客户端流程：

- 通过 `int socket(int __domain, int __type, int __protocol)`，得到一个fd（文件描述符）
- 创建`struct sockaddr_in`来存储要连接的地址
- 通过`connect(int __fd, const sockaddr *__addr, socklen_t __len)`连接到指定的地址
- 之后即可直接调用`write(int __fd, const void *__buf, size_t __n)`往socket中写入数据，也可直接调用`read(int __fd, void *__buf, size_t __nbytes)`从socket中读取数据
- 最后调用`close(int __fd)`关闭socket



### 服务器流程（使用epoll）：

- 同客户端一样，也是通过`socket`调用得到一个fd，再创建一个`sockaddr_in`，主要填写的参数是自身的IP地址，以及要监听的端口号

- 使用`bind(int __fd, const sockaddr *__addr, socklen_t __len)`，将socket与sockaddr_in绑定在一起

- 使用`listen(int __fd, int __n)`，将socket设置为监听状态

- 接下来是创建epoll

- 使用`int epoll_create1(int __flags)`创建一个epoll_fd

- 创建`struct epoll_event`，并对其进行相应设置

- 使用`epoll_ctl(int __epfd, int __op, int __fd, epoll_event *__event)`，将服务器fd（不是epoll_fd！）和epoll_event添加到epoll_fd中

- 创建一个`epoll_event`数组events

  ```c
  // epoll_event的内部结构
  
  typedef union epoll_data
  {
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
  } epoll_data_t;
  
  struct epoll_event
  {
    uint32_t events;	/* Epoll events */
    epoll_data_t data;	/* User data variable */
  } __EPOLL_PACKED;
  ```

- 使用`int epoll_wait(int __epfd, epoll_event *__events, int __maxevents, int __timeout)`，将epoll_fd，events传入，函数返回后，events会包含所有触发的事件，该函数返回一个int值，大小为触发事件的数量

- 遍历events，每个event都包含有对应的fd

  1. 如果这个fd与服务器的fd相同的话，表示服务器上有新的事件，即有新客户端连接。使用`int accept(int __fd, sockaddr *__addr, socklen_t *__addr_len)`，该函数的作用是，当该socketfd被连接时，会生成一个新的socketfd与之通信，并将连接人的地址传入`__addr`中
  2. 继续调用`epoll_ctl`，将新获得fd添加到epoll_fd（途中再生成一个`epoll_event`作为参数传入）
  3. 如果不相同，对event的成员`uint32_t events`做判断，如果处于可读状态，则可以对该fd进行读取操作

- 

## 问题清单：

1.  server.cc 接收新的client连接时，通过指针开辟的堆空间未释放
2. 

