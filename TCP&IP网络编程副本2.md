TCP/IP网络编程

# 第一部分：开始网络编程

# 第一章：理解网络编程和套接字

## 1.1理解网络编程和套接字

### 1.1.1构建接电话套字节

调用socket函数：安装电话机

```C++
int socket(int domain,int type,int protocol);
//成功返回文件描述符，失败返回-1；
```

调用bind函数：分配号码

```C++
int bind(int sockfd,struct sockaddr *myaddr, socklen_t addrlen);
//成功返回0，失败返回-1；
```

调用listen函数：连接电话线

```C++
int listen(int sockfd, int backlog);
//成功返回0，失败返回-1；
```

调用accept函数：拿起话筒

```C++
int accept(int sockfd, struct sockaddr *addr，socklen_t *addrlen);
//成功返回文件描述符，失败返回-1；
```

网络编程中接受连接请求的套接字创建过程可整理如下 ：

1. 调用socket函数创建套接字
2. 调用bind函数分配ip地址和端口号
3. 调用listen函数转换为可接收请求状态
4. 调用accept函数接受请求

### 1.1.2编写helloworld服务端

```C++
#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc,char *argv[]){
    int servSock;//服务端的服务端套接字是服务端创建的
    int clntSock;//客户端套接字是通过accept函数获取的客户端的套接字

    struct sockaddr_in servAddr;//服务器地址
    struct sockaddr_in clntAddr;//客户端地址
    socklen_t clntAddrSize;//客户端地址长度

    char message[]="helloworld!";

    if(argc!=2){
        std::cout<<"usage:"<<argv<<"port";
        exit(1);
    }

    servSock=socket(PF_INET,SOCK_STREAM,0);//调用socket函数创建服务端套接字
    if(servSock==-1)
        error_handling("socket error");

    memset(&servAddr,0,sizeof(servAddr));
    servAddr.sin_family=AF_INET;
    servAddr.sin_addr.s_addr= htonl(INADDR_ANY);
    servAddr.sin_port= htons(atoi(argv[1]));

    if (bind(servSock,(struct sockaddr*)&servAddr,sizeof (servAddr))==-1){//调用bind函数给服务端套接字分配ip地址和端口
        error_handling("binding error");
    }

    if (listen(servSock,5)==-1){//调用listen函数将套接字转换为可接收连接状态
        error_handling("liten error");
    }

    clntAddrSize=sizeof(clntAddr);
  	//调用accept函数受理连接请求，如果在没有请求的情况下调用该函数，则阻塞，直到有连接请求为止，返回一个客户端套接字
    clntSock= accept(servSock,(struct sockaddr*)&clntAddr,&clntAddrSize);
    if (clntSock==-1)
        error_handling("accept error");

    write(clntSock,message, sizeof(message));//向客户端套接字中写入消息
    close(clntSock);
    close(servSock);
    return 0;
}

void error_handling(char *message){
    std::cout<<*message<<std::endl;
    exit(1);
}
```

### 1.1.3构建打电话套接字

打电话：请求连接

```C++
int connect(int sockfd,struct sockaddr *servAddr,socklen_t addrlen);
//成功返回0，失败返回-1；
```

客户端程序只有“调用socket函数创建套接字”和“调用connect函数向服务器端发送连接请求”这两个步骤；

```C++
#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc,char *argv[]){
   int sock;
   struct sockaddr_in servAddr;//只需要服务器的地址，自己的地址在调用connect函数的时候会自动绑定的
   char message[30];
   int strLen;

    if (argc!=3){
        std::cout<<"usage:"<<argv[0];
        exit(1);
    }
    
    sock=socket(PF_INET,SOCK_STREAM,0);//创建套接字，但此时套接字并不马上分为服务器端和客户端，如果接下来调用bind、listen函数，则它将成为服务器套接字，如果调用connect函数，将成为客户端套接字。
    if (sock==-1){
        error_handling("socket error");
    }
    
    memset(&servAddr,0,sizeof (servAddr));
    servAddr.sin_family=AF_INET;
    servAddr.sin_addr.s_addr= inet_addr(argv[1]);
    servAddr.sin_port=htons(atoi(argv[2]));

    if (connect(sock,(struct sockaddr*)&servAddr,sizeof (servAddr))==-1){//调用connect函数向服务器发出连接请求，并且此时会自动绑定客户端地址
        error_handling("connect error");
    }
    
    strLen= read(sock,message,sizeof (message)-1);//读出客户端套接字中的消息
    if (strLen==-1){
        error_handling("read error");
    }

    std::cout<<"message from server"<<message;
    close(sock);
    return 0;
}

void error_handling(char *message){
    std::cout<<*message<<std::endl;
    exit(1);
}
```

## 1.2基于Linux的文件操作

对于linux而言，socket操作和文件操作没有区别，因此有必要详细了解文件。在linux下，socket也被认为是文件的一种，因此在网络数据传输过程中可以使用文件I/O的相关函数。

### 1.2.1底层文件访问和文件描述符

底层可以理解为“与标准无关的操作系统独立提供的”。

此处的文件描述符是系统分配给文件或者套接字的**整数**，学习C语言中用过的标准输入输出以及标准错误在Linux中也被分配了文件描述符：

| 文件描述符 | 对象                      |
| ---------- | ------------------------- |
| 0          | 标准输入：Standard Input  |
| 1          | 标准输出：Standard Output |
| 2          | 标准错误：Standard Error  |

文件和套接字一般经过创建过程才会被分配文件描述符，表格中的三种对象即使没有特殊创建过程，程序开始运行后也会被自动分配文件描述符。

文件描述符（文件句柄）：学校附近有个打印店，张三每次都去复印同一片论文的不同部分，店主说“从现在开始，这篇论文编号18，以后就说复印18号论文的哪一页到哪一页”；以后每次张三要来复印新的论文这篇论文都被分到一个新的标号。这里店主就是操作系统，张三就是程序员，论文编号相当于文件描述符，论文相当于文件或者套接字。也就是每次有新论文，操作系统都会返回分配给他们的整数，成为了程序员和操作系统之间良好沟通的渠道。windows下叫文件句柄，linux下叫文件描述符。

### 1.2.2打开文件

打开文件读写数据的函数，第一个参数：打开文件名及路径信息，第二个参数：文件打开模式：

```C++
int open (const char * pathe,int flag);
```

flag可能的值以及意义。

| 打开模式 | 含义               |
| -------- | ------------------ |
| O_CREAT  | 必要时创建文件     |
| O_TRUNC  | 删除全部现在数据   |
| O_APPEND | 维持现有数据，追加 |
| O_RDONLY | 只读打开           |
| O_WRONLY | 只写打开           |
| O_RDWR   | 读写打开           |

### 1.2.3关闭文件

```C++
int close(int fd);
```

### 1.2.4将数据写入文件

向文件输出数据

```C++
ssize_t write(int fd, const void * buf, size_t nbytes);
//成功时返回写入的字节数，失败时返回-1
```

fd：数据传输对象的文件描述符；

buf：写入数据的缓冲地址值；

nbytes：要写入数据的字节数

size_t是通过typedef声明的unsigned int类型，ssize_t是通过typedef声明的signed int类型；

以t为后缀的数据类型：size_t、ssize_t等等这些都是元数据类型，在sys/types.h文件中一般由typedef定义，算是给熟悉的数据类型起了别名，为什么？：int类型由以前的16位变成现在的32位，如果之前在需要4字节的地方使用size_t，则只需要修改并编译size_t的typedef声明即可。

Write.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

int main(void){
  int fd;
  char buf[]="Lets go!\n";
  
  fd=open("data.txt",O_CREAT|O_WRONLY|O_TRUNC);//打开文件，必要时创建，只写，删除现在
  if(fd==-1)
    error_handling("open error");
  std::cout<<"file descriptor:"<<fd;
  if(write(fd,buf,sizeof(buf))==-1)
    error_handling("write error");
  close(fd);
  return 0;
}
```

### 1.2.5读取文件中的数据

```C++
ssize_t read(int fd,void * buf, size_t nbytes);
//成功时返回接收的字节数，失败时返回-1
```

fd：显示数据传输对象的文件描述符；

buf：保存要接收数据的缓冲地址值；

nbytes：要接受数据的最大字节数

read.c：

```C++
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#define BUF_SIZE 100

int main(void){
  int fd;
  char buf[BUF_SIZE];
  
  fd=open("data.txt",O_RDONLY);//打开文件，只读
  if(fd==-1){
    error_handling("open error");
  }
  std::cout<<"file descriptor:"<<fd;
  
  if(read(fd,buf,sizeof(buf))==-1){
    error_handling("read error");
  }
  std::cout<<"file data"<<buf;
  close(fd);
  return 0;
}
```

## 1.3基于windows平台实现

### 1.3.1为windows套接字准备头文件和库

导入头文件winsock2.h；

链接ws2_32.lib库：打开项目属性页，选择配置属性->输入->附加依赖项->在附加依赖项空白处直接写入ws2_32.lib；现在只需要在头文件中导入winsock2.h即可调用winsock相关函数。

### 1.3.2winsock初始化(WSA相关)

进行winsock编程时，先调用WSAStartup函数，设置程序中用到的winsock版本，并初始化相应版本的库；

```C++
int WSAStartup(WORD wVersionRequested, LPWSADATA lpWSAData);
//成功时返回0，失败返回相应的错误码
```

wVersionRequested：winsock存在多个版本，应准备WORD类型的（typedef声明的unsigned short类型）套接字版本信息，若版本为1.2，应传递0x0201；借助MARKWORD宏函数则能轻松构建 WORD型版本信息 。

```C++
MARKWORD(1,2);//1.2
MARKWORD(2,2);//2.2
```

lpWSAData:此参数中需传入WSADATA型结构体变量地址(LPWSADATA是WSADATA的指针类型 )，

WSAStartup函 数调用过程：

```C++
int main(int argc,char *argv[])
{
  WSADATA wsaData;
  ...;
  if(WSAStartup(MAKEWORD(2,2),&wsaData)!=0)
  {
    ErrorHandling("WSAStartup error");
  }
  ...;
  return 0;
}
```

注销winsock相关库：

```C++
int WSAClean(void);
//成功返回0，失败返回SOCKET_ERROR
```

调用该函数时，winsock相关库将归还windows系统，无法再调用。

## 1.4基于 Windows 的套接字相关函数及示例

### 1.4.1基于 Windows 的套接字相关函数

与 Linux下的 socket函数提供相同功能：

```C++
SOCKET socket(int af， int type, int protocol);
//成功时返回套接字句柄，失败时返回INVALID-SOCKET
```

与Linux的bind函数相同，调用其分配IP地址和端口号 ：

```C++
int bind(SOCKET s, const struct sockaddr *name, int namelen); 
//成功时返回0，失败时返回SOCKET_ERROR
```

与Linux的listen函数相同，调用其使套接字可接收客户端连接 :

```C++
int listen(SOCKET s,int backlog);
//成功时返回0，失败时返回 SOCKET_ERROR
```

与 Linux的accept函数相同，调用其受理客户端连接请求 :

```C++
SOCKET accept(SOCKET s, struct sockaddr * addr, int * addrlen);
//成功时返回套接字句柄，失败时返回INVALID-SOCKET
```

与Linux的connect函数相同，调用其从客户端发送连接请求 :

```C++
int connect(SOCKET s, const struct sockaddr* name, int namelen);
//成功时返回0，失败时返回 SOCKET_ERROR
```

这个函数在关闭套接字时调用。 Linux中，关闭文件和套接字时都会调用close函数; 而 Windows中有专门用来关闭套接字的函数 :

```C++
int closesocket(SOCKET s);
//成功时返回0，失败时返回 SOCKET_ERROR
```

### 1.4.2Windows 中的文件旬柄和套接字句柄

Windows中通过调用系统函数创建文件时，返回"句柄" ( handle)，Windows中的句柄相当于Linux中的文件描述符。只不过Windows中要区分文件句柄和套接字句柄。虽然都称为"句柄"，但不像Linux那样完全一致，文件句柄相关函数与套接字句柄相关函数是有区别的，这一点不同于Linux文件描述符 。

SOCKET类型就是为了保存套接字句柄整型值的新数据类型， 它由 typedef声明定义 。

### 1.4.3创建基于 Windows 的服务器端和客户端

服务端实例：

```C++
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
#include <iostream>
void ErrorHandling(char *message);

int main(int argc, char* argv[]){
    WSADATA wsaData;
    SOCKET hServSock,hClntSock;
    SOCKADDR_IN servAddr, clntAddr;
    int szClntAddr;
    char message[]="Hello world";
    if(argc!=2){
        std::cout<<"usage:"<<argv[0]<<"port";
        exit(1);
    }

    if(WSAStartup(MAKEWORD(2,2),&wsaData)!=0)//初始化套接字库
        ErrorHandling("WSAStartup error");

    hServSock=socket(PF_INET,SOCK_STREAM,0);//创建服务端套接字
    if(hServSock==INVALID_SOCKET)
        ErrorHandling("socket error");

    memset(&servAddr,0,sizeof(servAddr));
    servAddr.sin_family=AF_INET;
    servAddr.sin_addr.s_addr=htonl(INADDR_ANY);
    servAddr.sin_port=htons(atoi(argv[1]));

    if(bind(hServSock,(SOCKADDR*) &servAddr,sizeof(servAddr))==SOCKET_ERROR)//给套接字分配ip地址和端口号
        ErrorHandling("bind error");

    if(listen(hServSock,5)==SOCKET_ERROR)//调用listen函数开始监听
        ErrorHandling("listen error");

    szClntAddr=sizeof(clntAddr);
    hClntSock=accept(hServSock,(SOCKADDR*) &clntAddr,&szClntAddr);//调用accept函数受理客户端连接请求，并返回客户端套接字句柄
    if(hClntSock==INVALID_SOCKET)
        ErrorHandling("accept error");

    send(hClntSock,message,sizeof(message),0);//调用send函数向连接的客户端传输数据
    closesocket(hClntSock);
    closesocket(hServSock);
    WSACleanup();//注销初始化的套接字库
    return 0;
}

void ErrorHandling(char *message){
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

客户端代码：

```C++
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
#include <iostream>
void ErrorHandling(char *message);

int main(int argc, char* argv[]){
    WSADATA wsaData;
    SOCKET hSocket;
    SOCKADDR_IN servAddr;

    char message[30];
    int strLen;
    if(argc!=3){
        std::cout<<"usage:"<<argv[0]<<"port";
        exit(1);
    }

    if(WSAStartup(MAKEWORD(2,2),&wsaData)!=0)//初始化套接字库
        ErrorHandling("WSAStartup error");

    hSocket=socket(PF_INET,SOCK_STREAM,0);//创建套接字
    if(hSocket==INVALID_SOCKET)
        ErrorHandling("socket error");

    memset(&servAddr,0,sizeof(servAddr));
    servAddr.sin_family=AF_INET;
    servAddr.sin_addr.s_addr=htonl(INADDR_ANY);
    servAddr.sin_port=htons(atoi(argv[2]));

    if(connect(hSocket,(SOCKADDR*) &servAddr,sizeof(servAddr))==SOCKET_ERROR)//通过套接字向服务端发出连接请求
        ErrorHandling("connect error");

    strLen=recv(hSocket,message,sizeof(message)-1,0);//接收服务器发来的数据
    if(strLen==-1)
        ErrorHandling("read error");
    std::cout<<"message from server:"<<message;

    closesocket(hSocket);
    WSACleanup();//注销库
    return 0;
}

void ErrorHandling(char *message){
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

### 1.4.4基于 Windows 的I/O函数

windows 中则有些不同 。Windows严格区分文件I/O函数和套接字I/O函数 。

 Winsock数据传输函数 ：与linux的write函数相比只多处flags，send函数并非Windows 独有 。 Linux中也有同样的函数，它也来自于BSD套接字。只不过我在Linux相关示例中暂时只使 用read、 write函数，为了强调Linux环境下文件IO和套接字IO相同。

```C++
int send(SOCKET s,const char * buf, int len;int flags);
//成功时返回传输字节数，失败时返回 SOCKET_ERROR
```

 与send函数对应的recv函数 ：

```C++
int recv(SOCKET s, const char * buf, int len, int flags);
//成功时返回接收字节数，失败时返回 SOCKET_ERROR
```

请不要误认为 Linux中的read、 write函数就是对应于Windows的 send、 recv函数；

## 1.5习题

1. 套接字在网络编程中的作用是什么?为何称它为套接字?

   建立网络连接、进行数据传递、支持多种通信协议、允许多连接处理，socket是插座、连接点的意思，套接字可以被视为计算机之间通信的插座。

2. 在服务器端创建套接字后，会依次调用listen函数和accept函数。 请比较并说明二者作用。

   listen是开始监听客户端的连接请求，而accept才是接收客户端的连接请求

3. Linux中，对套接字数据进行I/O时可以直接使用文件I/O相关函数；而在Windows中则不可以。原因为何?

   在Linux中，对套接字数据进行I/O时可以直接使用文件I/O相关函数，因为在Linux中套接字被视为文件描述符（File Descriptor），这意味着套接字可以像文件一样进行读取和写入。这种设计的背后是Linux的"一切皆文件"哲学，即将设备、套接字、文件等抽象为文件描述符，使得它们可以使用相同的I/O函数进行操作，提供了一种统一的编程接口。

   而在Windows中，情况有所不同。Windows的套接字编程模型和文件I/O模型是分开的，套接字被视为一种特殊的对象，不同于文件。因此，Windows提供了专门用于套接字通信的一组函数，如`send()`和`recv()`，而不像Linux那样可以直接使用文件I/O函数。

4. 创建套接字后一般会给它分配地址，为什么?为了完成地址分配需要调用哪个函数 ?

   bind函数

5. Linux中的文件描述符与Windows的句柄实际上非常类似。请以套接字为对象说明它们的含义。

6. 底层文件I/O函数与ANSI标准定义的文件I/O函数之间有何区别?

   **功能和抽象级别**：

   - 底层文件I/O函数提供了对文件的更底层、更直接的访问。它们通常是操作系统提供的系统调用，允许你执行基本的读取和写入操作，以及移动文件指针等底层操作。这些函数不处理缓冲、格式化或高级文件操作。
   - ANSI标准定义的文件I/O函数（如`fopen()`、`fwrite()`、`fread()`等）提供了更高级别、更抽象的文件操作。它们对文件进行缓冲、提供了格式化输入输出的功能，允许你以更方便的方式处理文件数据。

   **使用方式**：

   - 底层文件I/O函数通常需要你手动管理缓冲区、文件指针的位置和文件打开模式。你需要显式地打开文件、读取数据、写入数据，并确保在完成操作后关闭文件。
   - ANSI标准定义的文件I/O函数封装了很多底层细节，提供了更简单的文件操作接口。你可以使用`fopen()`来打开文件，然后使用`fprintf()`和`fscanf()`等函数来进行格式化的输入输出。这些函数会自动处理缓冲和文件指针位置，而且在文件关闭时通常也会自动关闭文件。

   **跨平台性**：

   - ANSI标准定义的文件I/O函数通常是跨平台的，因为它们是C语言标准库的一部分，提供了一致的接口，可以在不同的操作系统上使用。这使得你的代码更具可移植性。
   - 底层文件I/O函数在不同操作系统上可能会有不同的实现和语法。因此，如果你使用底层函数，可能需要在不同的操作系统上进行适当的修改和调整。

7. 参考本书给出的示例low_open.c和low_read.c，分别利用底层文件I/O和ANSI标准I/O编写文件复制程序。可任意指定复制程序的使用方法。

# 第二章：套接字类型与协议设置

## 2.1套接字协议及其数据传输特性

### 2.1.1关于协议

协议是对话中使用的通信规则，是为了完成数据交换而定好的约定。

### 2.1.2创建套接字

```C++
int socket(int domain,int type,int protocol);
//成功时返回文件描述符，失败时返回-1
```

### 2.1.3协议族

通过socket函数的第一个参数传递套接字中使用的协议分类信息，此协议分类信息称为协议族，可分成如下几类：

| 名称      | 协议族               |
| --------- | -------------------- |
| PF_INET   | ipv4互联网协议族     |
| PF_INET6  | ipv6互联网协议族     |
| PF_LOCAL  | 本地通信的UNIX协议族 |
| PF_PACKET | 底层套接字协议族     |
| PF_IPX    | IPX Novell协议族     |

事实上套接字中实际使用的协议信息是通过socket函数的第三个参数传递的，在指定的协议族范围内通过第一个参数决定第三个参数。

### 2.1.4套接字类型

套接字类型指的是套接字的数据传输方式，决定了协议族并不能同时决定数据传输方 式， socket函数第一个参数PF_INET协议族中也存在多种数据传输方式。

#### 2.1.4.1面向连接套接字

第二个参数传递SOCK_STREAM将创建面向连接的套接字：

- 传输过程中数据不会丢失
- 按序传输数据
- 传输的数据不存在数据边界

收发数据的套接字内部有缓冲 (buffer)字节数组。通过套接字传输的数据将保存到该数组。因此收到数据并不意味着马上调用read函数。 只要不超过数组容量，则有可能在数据填充满缓冲后通过1次read函数调用读取全部，也有可能分成多次read函数调用进行读取。也就是说，在面向连接的套接字中，read函数和write函数的调用次数并无太大意义。所以说面向连接的套接字不存在数据边界。稍后将给出示例以查看该特性。

面向连接的套接字只能与另外一个同样特性的套接字连接

"可靠的、按序传递的、基于字节的面向连接的数据传输方式的套接字"

#### 2.1.4.2面向消息套接字

第二个参数传递SOCK_DGRAM，则将创建面向消息的套接字：

- 快速传输而非传输顺序 
- 传输的数据可能丢失也可能损毁 
- 传输的数据有数据边界 
- 限制 每次传输 的数据大小 

### 2.1.5协议的最终选择

第三个参数，该参数决定最终采用的协议，大部分情况下可以向第三个参数传递0，除非遇到以下这种情况:

"同一协议族中存在多个数据传输方式相同的协议"

### 2.1.6面向连接套接字：TCP套接字示例



## 2.2windows平台下的实现及验证

### 2.2.1windows操作系统的socket函数

```C++
SOCKET socket(int af， int type, int protocol);
//成功时返回套接字句柄，失败时返回INVALID-SOCKET
```

参数完全相同，只讨论返回值，SOCKET结构体用来保存整数型套接字句柄值，也可以使用int，发生错误时返回INALID_SOCKET，实际

值为-1

### 2.2.2基于windows的TCP套接字示例

## 2.3习题

- 什么是协议，在收发数据中定义协议有何意义

- 面向连接的TCP套接字传输特性有哪3点？
- 下列哪些是面向消息的套接字的特性?
  - a. 传输数据可能丢失
  - b.没有数据边界 (Boundary)
  - c. 以快速传递为目标
  - d. 不限制每次传递数据的大小
  - e. 与面向连接的套接字不同，不存在连接的概念
- 下列数据适合用哪类套接字传输?并给出原因
  - a. 演唱会现场直播的多媒体数据 ( )
  - b.某人压缩过的文本文件 ( )
  - c. 网上银行用户与银行之间的数据传递 ( )
- 何种类型的套接字不存在数据边界?这类套接字接收数据时需要注意什么 ?
- tcp_server.c和tcp_client.c中需多次调用read函数读取服务器端调用1次write函数传递的字符串。更改程序，使服务器端多次调用 ( 次数自拟 ) write函数传输数据，客户端调用1次read函数进行读取。为达到这一目的，客户端需延迟调用read函数，因为客户端要等待服务器端传输所有数据。Windows和Linux都通过下列代码延迟read或recv函数的调用 。
  - for(i=0; i<3000; i++)
     printf("W ait time %d \n" , i);
  - 让CPU执行多余任务以延迟代码运行的方式称为 "Busy W组ting气 使用得当即可推迟函数调用。

# 第三章 地址族与数据序列

## 3.1分配给套接字的IP地址与端口号

### 3.1.1网络地址

IPv4标准的4字节地址分为网络地址和主机地址，且分为A、 B、 C、 D 、 E等 类型。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230807110100268.png" alt="image-20230807110100268" style="zoom:50%;" />

### 3.1.2网络地址分类与主机地址边界

只需通过地址的第一个字节即可判断网络地址占用的字节数，因为我们根据地址的边界区分网络地址：

- A类地址的首字节范围: 0-- l27 
- B类地址的首字节范围: 128--191
- C类地址的首字节范围: 192--223

还有如下这种表述方式 

- A类地址的首位以0开始 
- B类地址的前2位以10开始
- C类地址的前3位以110开始

### 3.1.3用于区分套接字的端口号

端口号就是在同一操作系统内为区分不同套接字而设置的，因此无法将1个端口号分配给不同套接字，另外端口号由 16位构成，可分配的端口号范围是 0-65535，但 0-1023是知名端口( Well-known PORT )， 一般分配给特定应用程序，所以应当分配此范围之外的值。另外，虽然端口号不能重复，但TCP套接字和UDP套接字不会共用端口号，所以允许重复

## 3.2地址信息的表示

### 3.2.1表示IPV4地址的结构体

此结构体将作为地址信息传递给bind函数：

```C++
struct sockaddr_in{	//sockaddr_in是sockaddr内部细化后的结构体
  sa_family_t 		sin_family;	//地址族
  uint16_t				sin_port;		//16位tcp/udp端口号
  struct in_addr	sin_addr;		//32位ip地址
  char						sin_zero[8];//不使用
}
```

in_addr定义如下，它用来存放32位IP地址

```C++
struct in_addr{
  In_addr_t	s_sddr;//32位ip地址
}
```

### 3.2.2结构体sockaddr_in的成员分析

**sin_family：**

每种协议族适用的地址族均不同 。 比如，IPv4使用4字节地址族：

| 地址族   | 含义                             |
| -------- | -------------------------------- |
| AF_INET  | IPV4地址族                       |
| AF_INET6 | IPV6地址族                       |
| AF_LOCAL | 本地通信中采用的UNIX协议的地址族 |

**sin_port：**

该成员保存 16位端口 号，重点在于，它以网络字节序保存

**sin_addr：**

该成员保存32位IP地址信息，且也以网络字节序保存。 为理解好该成员，应同时观察结构体 in_addr。 但结构体in_addr声明为uint32 t，因此只需当作32位整数型即可。

**sin_zero：**

无特殊含义。 只是为使结构体sockaddr in的大小与sockaddr结构体保持一致而插入的成员。必需填充为0，否则无法得到想要的结果 。 后面会另外讲解sockaddr。

```C++
struct sockaddr_in serv_addr;
。。。
if(bind(serv_sock, (struct sockaddr * ) &serv_addr, sizeof(serv_addr)) == -1)
erro_handling( "bind() error");
。。。
```

bind函数的第二个参数期望得到sockaddr结构体变量地址值，包括地址族、端口号、 IP地址等，但是：

```C++
struct sockaddr{
  sa_family_t 		sin_family;	//地址族
  char						sa_data[14];//地址信息
}
```

此结构体成员 sa_data保存的地址信息中需包含IP地址和端口号，剩余部分应填充0，而这对于包含地址信息来讲非常麻烦，继而就有了新的结构体sockaddr_in

## 3.3网络字节序与地址变换

不同CPU中， 4字节整数型值1在内存空间的保存方式是不同的。 4字节整数型值1可用2进制表示如下：

00000000 00000000 00000000 000000001

有些CPU以这种顺序保存到内存， 另外一些CPU则以倒序保存：

00000001 00000000 00000000 000000000

若不考虑这些就收发数据则会发生问题，因为保存顺序的不同意意味着对接收数据的解析顺序也不同。

### 3.3.1字节序与网络字节序

CPU向内存保存数据的方式有2种，这意味着CPU解析数据的方式也分为2种 ：

- 大端序 (BigEndian): 高位字节存放到低位地址。
- 小端序 ( Little Endian ):高位字节存放到高位地址 。

代表 CPU数据保存方式的主机字节序( Host Byte Order )在不同 CPU中也各不相同，正因如此，在通过网络传输数据时约定统一方式，这种约定称为网络字 节序( Network Byte Order )，非常简单一一统一 为大端序，先把数据数组转化成大端序格式再进行网络传输。 因此，所有计算机接收数据时应识别该数据是网络字节序格式，小端序系统传输数据时应转化为大端序排列方式 

### 3.3.2字节序转换

帮助转换字节序的函数 ：

- unsigned short htons(unsigned short);
- unsigned short ntohs(unsigned short);
- unsigned long htonl(unsigned long);
- unsigned long ntohl(unsigned long);

只需了解以下细节 :htons中的h代表主机 (host)字节序，htons中的n代表网络 (network) 字节序。

## 3.4网络地址的初始化与分配

### 3.4.1将字符串信息转换为网络字节序的整数型

sockaddr_in中保存地址信息为32位整数型 。 因此，为了分配IP地址，需要将其表示为32位整数型数据，

函数会帮我们将字符串形式的IP地址转换成32位整数型数据。 此函数在转换类型的同时进行网络字节序转换：

```C++
in_addr_t inet_addr(const char * string);
//成功返回32位大端序整数型，失败时返回INADDR_NONE
```

如果向该函数传递类似 "211.214.107.99" 的点分十进制格式的字符串，它会将其转换为 32 位整数型数据并返回。 当然，该整数型值满足网络字节序：

```C++
#include <stdio.h>
#include <arpa/inet.h>

int main(int argc,char *argv[]){
  char *addr1="1.2.3.4";
  char *addr2="1.2.3.256";
  
  unsigned long convAddr=inet_addr(addr1);
  if(convAddr==INADDR_NONE){
    std::cout<<"error";
  }
  else{
    std::cout<<"Network ordered integer addr:"<<convAddr;
  }
  
  convAddr=inet_addr(addr2);
  if(convAddr==INADDR_NONE){
    std::cout<<"error";
  }
  else{
    std::cout<<"Network ordered integer addr:"<<convAddr;
  }
}
```

结果可以看出，inet_addr函数不仅可以把ip地址转成32位整数型，而且可以检测无效的IP地址。另外，从输出结果可以验证确实转换为网络字节序 。

inet_aton函数与inet_addr函数在功能上完全相同，也**将字符串形式IP地址转换为32位网络字节序整数**并返回。只不过该函数利用了in_addr结构体，且其使用频率更高：

```C++
int inet_aton(const char * string, struct in_addr * addr);
//成功时返回1，失败时返回0
```

介绍一个与 inet_aton函数正好相反的函数，此函数可以把**网络字节序整数型IP地址转换成我们熟悉的字符串**形式：

```C++
char * inet_ntoa(struct in_addr adr);
//成功时返回字符串地址，失败时返回-1
```

但调用时需小心，返回值类型为char指针。返回字符串地址意味着字符串已保存到内存空间，但该函数未向程序员要求分配内存，而是在内部申请了内存并保存了字符串。也就是说，调用完该函数后，应立即将字符串信息复制到其他内存空间。因为，若再次调用ine_ntoa函数，则有可能覆盖之前保存的字符串信息 。

### 3.4.2网络地址初始化

套接字创建过程中常见的网络地址信息初始化方法：

```C++
struct sockaddr_in addr;
char * serv_ip = "211.117.168.13";	//ip地址字符串
char * serv_port = "9190";					//端口字符串
memset(&addr,0,sizeof(addr));				//结构体变量addr所有成员初始化为0
addr.sin_family = AF_INET;					//指定地址族
addr.sin_addr.s_addr = inet_addr(serv_ip);	//基于字符串的ip地址初始化
addr.sin_port = htons(atoi(serv_port));			//基于字符串的端口号初始化
```

memset函数将每个字节初始化为同一值，这么做是为了将sodaddzjn结构体的成员 sin-zero初始化为0。

另外，最后一行代码调用的 atoi函数把字符串类型的值转换成整数型 。 总之曾上述代码利用字符串格式的ip地址和端口号初始化了sockaddr_in结构体。

### 3.4.3客户端地址信息初始化

上述网络地址信息初始化过程主要针对服务器端而非客户端，"请把进入IP211.217.168.13、 9190端口的数据传给我 !"

反观客户端中连接请求如下 :"请连接到IP211.217.168.13、 9190端口 !"

服务器端的准备工作通过bind函数完成，而客户端 则通过connect函数完成 。因此，函数调用前需准备的地址值类型也不同 。服务器端声明 sockaddr_in结构体变量将其初始化为赋予服务器端 IP和套接字的端口号，然后调用 bind函数，而客户端则声明 sockaddr_in结构体， 并初始化为要与之连接的服务器端套接字的IP和端口号然后调用connect函数 。

### 3.4.4INADDR_ANY

```C++
struct sockaddr_in addr;
char *serv_port="9190";
memset(&addr,0,sizeof(addr));
addr.sin_family=AF_INET;
addr.sin_addr.s_addr=htonl(INADDR_ANY);
addr.sin_port=htons(atoi(serv_port));
```

用常数lNADDR_ANY分配服务器端的ip地址。若采用这种方式，可自动获取运行服务器端的计算机IP地址，不必亲自输入

### 3.4.5第1章的hello_server.c、hello_client.c运行过程

### 3.4.6向套接字分配网络地址

把初始化的地址信息分配给套接字，bind函数负责这项操作。

```C++
int bind(int sockfd,struct sockaddr *myaddr, socklen_t addrlen);
//成功返回0，失败返回-1；
```

如果此函数调用成功，则将第二个参数指定的地址信息分配给第一个参数中的相应套接字，下面给出服务器端常见套接字初始化过程

```C++
int servSock;
struct sockaddr_in sockAddr;
char * sockPort="9190";

/*创建服务器套接字（监听套接字）*/
servSock=socket(PF_INET,SOCK_STREAM,0);

/*地址信息初始化*/
memset(&sockAddr,0,sizeof(sockAddr));
sockAddr.sin_family=AF_INET;
sockAddr.sin_addr.s_addr=htonl(INADDR_ANY);//unsigned long
sockAddr.sin_port=htons(atoi(sockPort));//unsigned short

/*分配地址信息*/
bind(servSock,(struct *sockaddr)&sockAddr,sizeof(sockAddr));
```

## 3.5基于windows的实现

windows中同样存在 sockaddr_in结构体及各种变换函数，而且名称、使用方法及含义都相同 。 也就无需针对Windows平台进行太多修改或改用其他函数。

### 3.5.1函数htons、htonl在windows中的应用



### 3.5.2函数inet_addr、inet_ntoa在windows中的使用



### 3.5.3在windows环境下给套接字分配网络地址

windows中向套接字分配网络地址的过程与Linux中完全相同，因为bind函数的含义、 参数及返回类型完全一致 。

```C++
SOCKET servSock;
struct sockaddr_in servAddr;
char * sockPort="9190";

/*创建服务器套接字（监听套接字）*/
servSock=socket(PF_INET,SOCK_STREAM,0);

/*地址信息初始化*/
memset(servAddr,0,sizeof(servAddr));
servAddr.sin_family=AF_INET;
servAddr.sin_addr.s_addr=htonl(INADDR_ANY);
servAddr.sin_port=htons(atoi(sockPort));

/*分配地址信息*/
bind(servSock,(struct *sockaddr)&sockAddr,sizeof(sockAddr));
```



### 3.5.4WSAStringToAddress和WSAAddressToString

## 3.6习题

IP地址族IPv4和IPv6有何区别? 在何种背景下诞生了lPv6?

通过IPv4网络ID 、 主机ID及路由器的关系说明向公司局域网中的计算机传输数据的过程。

套接字地址分为IP地址和端口号。 为什么需要IP地址和端口号?或者说，通过IP可以区分哪些对象? 通过端口号可以区分哪些对象?

请说明IP地址的分类方法，并据此说出下面这些IP地址的分类：

214.121.212.102；120.101.122.89；129.78.102.211；

计算机通过路由器或交换机连接到互联网，请说出路由器和交换机的作用。

什么是知名端口?其范围是多少?知名端口中具有代表性的Http和FTP端口号各是多少?

向套接字分配地址的 bind函数原型如下:

请解释大端序、小端序、网络字节序，并说明为何需要网络字节序 。

大端序计算机希望把4字节整数型数据12传递到小端序计算机。请说出数据传输过程中发 生的字节序变换过程。

怎样表示回送地址?其含义是什么?如果向回送地址传输数据将发生什么情况?

# 第四章 基于TCP的服务器端/客户端

## 4.1理解TCP和UDP

网络协议的套接字一般分为TCP套接字和UDP套接字。 因为TCP套接字是面向连接的，因此又称基于流 (stream)的套接字。

### 4.1.1TCP/IP协议栈

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230808100216919.png" alt="image-20230808100216919" style="zoom:50%;" />

### 4.1.2TCP/IP 协议的诞生背景

### 4.1.3链路层

链路层是物理链接领域标准化的结果，也是最基本的领域专门定义LAN、 WAN、 MAN等网络标准。 若两台主机通过网络进行数据交换， 则需要图 4-4所示的物理连接，链路层就负责这些标准。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230808100730465.png" alt="image-20230808100730465" style="zoom:50%;" />

### 4.1.4ip层

在复杂的网络中传输数据，需要考虑路径的选择。向目标传输数据需要经过哪条路径?解决此问题就是IP层。

IP本身是面向消息的 、 不可靠的协议。每次传输数据时会帮我们选择路径，但并不一致。如果传输中发生路径错误，则选择其他路径；但如果发生数据丢失或错误，则无法解决。lP协议无法应对数据错误。

### 4.1.5TCP/UDP层

IP层解决数据传输中的路径选择问题，只需照此路径传输数据即可。TCP和UDP层以IP层提供的路径信息为基础完成实际的数据传输，故该层又称传输层( Transport )。

TCP和UDP存在于IP层之上，决定主机之间的数据传输方式， TCP协议确认后向不可靠的IP协议赋予可靠性。

### 4.1.6应用层

上述内容是套接字通信过程中自动处理的。选择数据传输路径、数据确认过程都被隐藏到套接字内部。提供的工具就是套接字，只需利用套接字编出程序即可。 编写软件的过程中，需要根据程序特点决定服务器端和客户端之间的数据传输规则(规定)，这便是应用层协议。

## 4.2实现基于TCP的服务器端/客户端

### 4.2.1TCP服务器端的默认函数调用顺序

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230808102541219.png" alt="image-20230808102541219" style="zoom:50%;" />

### 4.2.2进入等待连接请求状态

调用 listen函数进入等待连接请求状态。只有调用了listen函数，客户端才能进入可发出连接请求的状态。换言之，这时客户端才能调用 connect函数(若提前调用将发生错误)。

```C++
int listen(int sock,int backlog);
//成功时返回0，失败时返回-1
```

sock 进入等待连接请求状态的套接字文件描述符，传递的描述符套接字参数成为服务器端套接字(监听套接字)。

backlog 连接请求等待队列(Queue)的长度，若为6，则队列长度为5，表示最多使5个连接请求进入队列。

客户端如果向服务器端询问 : "请问我是否可以发起连接?"服务器端套接字就会亲切应答 : "您好! 当然可以，但系统正忙，请到等候室排号等待，准备好后会立即受理您的连接。"同时将连接请求请到等候室。调用 listen函数即可生成这种门卫(服务器端套接字)， listen函数的第 二个 参数决定了等候室的大小。等候室称为连接请求等待队列，准备好服务器端套接字和连接请求等待队列后，这种可接收连接请求的状态称为等待连接请求状态。

listen函数的第二个参数值与服务器端的特性有关，像频繁接收请求的Web服务器端至少应为15。

### 4.2.3受理客户端连接请求

调用 listen函数后，若有新的连接请求，则应按序受理，下面这个函数将自动创建套接字，并连接到发起请求的客户端。

```C++
int accept(int sock,struct sockaddr* addr,socklen_t * addrlen);
//成功返回套接字文件描述符，失败返回-1
```

sock：服务器文件套接字描述符

addr：保存发起连接请求的客户端地址信息的变量地址值，调用函数后向传递来的地址变量参数充客户端地址信息 

addrlen：第二个参数addr结构体的长度量但是存有长度的变量地址。函数调用完成后，该变量即被填入客户端地址长度

accept函数受理连接请求等待 队列中待处理的客户端连接请求。函数调用成功时，accept函数内部将产生用于数据I/O的套接字，并返回其文件描述符 。

### 4.2.4回顾HelloWorld服务器端

```C++
#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc,char *argv[]){
    int servSock;//服务端的服务端套接字是服务端创建的
    int clntSock;//客户端套接字是通过accept函数获取的客户端的套接字

    struct sockaddr_in servAddr;
    struct sockaddr_in clntAddr;
    socklen_t clntAddrSize;

    char message[]="helloworld!";

    if(argc!=2){
        std::cout<<"usage:"<<argv<<"port";
        exit(1);
    }

    servSock=socket(PF_INET,SOCK_STREAM,0);//调用socket函数创建服务端套接字
    if(servSock==-1){
        error_handling("socket error");
    }

    memset(&servAddr,0,sizeof(servAddr));
    servAddr.sin_family=AF_INET;
    servAddr.sin_addr.s_addr= htonl(INADDR_ANY);
    servAddr.sin_port= htons(atoi(argv[1]));

    if (bind(servSock,(struct sockaddr*)&servAddr,sizeof (servAddr))==-1){//调用bind函数给服务端套接字分配ip地址和端口
        error_handling("binding error");
    }

    if (listen(servSock,5)==-1){//调用listen函数将套接字转换为可接收连接状态连接请求等待队列的长度设置为5。此时的套接字才是服 务器端套接字。
        error_handling("liten error");
    }

    clntAddrSize=sizeof(clntAddr);
  //调用accept函数受理连接请求，如果在没有请求的情况下调用该函数，则不会返回，直到有连接请求为止，并且返回一个客户端套接字
  //accept函数从队头取 1个连接请求与客户端建立连接，并返回创建的文件描述符。另外，调用accept函数时若等待队列为空，则accept函数不会返回，直到队列中出现新的客户端连接。
    clntSock= accept(servSock,(struct sockaddr*)&clntAddr,&clntAddrSize);
    if (clntSock==-1)
    {
        error_handling("accept error");
    }

    write(clntSock,message, sizeof(message));//向客户端套接字中写入消息
    close(clntSock);
    close(servSock);
    return 0;
}

void error_handling(char *message){
    std::cout<<*message<<std::endl;
    exit(1);
}
```

### 4.2.5TCP客户端的默认函数调用顺序

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230808111557978.png" alt="image-20230808111557978" style="zoom:50%;" />

服务器端调用listen函数后创建连接请求等待队列，之后客户端即可请求连接。通过调用如下函数完成。

```C++
int connect(int sock,struct sockaddr* servaddr,socklen_t addrlen);
//成功时返回0，失败时返回-1
```

sock：客户端套接字文件描述符

servaddr：保存目标服务器端地址信息的变量地址值

addrlen：以字节为单位传递已传递给第二个结构体参数servaddr的地址变量长度

客户端调用 connect函数后，发生以下情况之一 才会返回(完成函数调用)：

服务器端接收连接请求；发生断网等异常情况而中断连接请求；

**知识补给站**

客户端实现过程中并未出现套接字地址分配，而是创建套接字后调用 connect函数。 难道客户端套接字无需分配IP和端口?当然不是!网络数据交换必须分配IP和端口。 既然如此，那客户端套接字何时、 何地、如何分配地址呢?

何时 ?调用 connect函数时

何地 ?操作系统，更准确地说是在内核中

如何 ? IP用计算机(主机)的 IP，端口随机

### 4.2.6回顾HelloWorld客户端

```C++
#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
void error_handling(char *message);

int main(int argc,char *argv[]){
   int sock;
   struct sockaddr_in servAddr;
   char message[30];
   int strLen;

    if (argc!=3){
        std::cout<<"usage:"<<argv[0];
        exit(1);
    }
    
    sock=socket(PF_INET,SOCK_STREAM,0);//创建套接字，但此时套接字并不马上分为服务器端和客户端，如果接下来调用bind、listen函数，则它将成为服务器套接字，如果调用connect函数，将成为客户端套接字。
    if (sock==-1){
        error_handling("socket error");
    }
    
    memset(&servAddr,0,sizeof (servAddr));//结构体servaddr中初始化IP和端口信息。初始化值为目标服务器端套接字的IP和端口信息。
    servAddr.sin_family=AF_INET;
    servAddr.sin_addr.s_addr= inet_addr(argv[1]);
    servAddr.sin_port=htons(atoi(argv[2]));

    if (connect(sock,(struct sockaddr*)&servAddr,sizeof (servAddr))==-1){//调用connect函数向服务器发出连接请求
        error_handling("connect error");
    }
    
    strLen= read(sock,message,sizeof (message)-1);//完成连接后 ，接收服务器端传输的数据
    if (strLen==-1){
        error_handling("read error");
    }

    std::cout<<"message from server"<<message;
    close(sock);
    return 0;
}

void error_handling(char *message){
    std::cout<<*message<<std::endl;
    exit(1);
}
```

### 4.2.7基于TCP的服务端/客户端函数调用关系

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230808134328388.png" alt="image-20230808134328388" style="zoom:50%;" />

服务器端创建套接字后连续调用bind、listen函数进入等待状态， 客户端通过调用 connect函数发起连接请求。需要注意的是，客户端只 能等到服务器端调用 listen函数后才能调connect函数。同时要清楚，**客户端调用connect函数前，服务器端有可能率先调用accept函数**。当然，此时服务器端在调用accept函数时进入阻塞 (blocking) 状态， 直到客户端调connect函数为止。

## 4.3实现迭代服务器端/客户端

回声 (echo)服务器端/客户端。服务器端将客户端传输的字符串数据原封不动地传回客户端。在此之前，需要先解释一下迭代服务器端。

### 4.3.1实现迭代服务器端

设置好等待队列的大小后，应向所有客户端提供服务 。简单的办法就是插入循环语句反复调用 accept函数

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230808140634441.png" alt="image-20230808140634441" style="zoom:50%;" />

这并非针对服务器端套接字 ，而是针对accept函数调用时创建的套接字 。

### 4.3.2迭代回声服务器端/客户端

- 服务器端在同一时刻只与一个客户端相连，并提供回声服务 。
- 服务器端依次向 5个客户端提供服务并退出 。
- 客户端接收用户输入的字符串并发送到服务器端 。
- 服务器端将接收的字符串数据传回客户端，即"回声"。
- 服务器端与客户端之间的字符串回声一直执行到客户端输入Q为止。

echo_server.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arþa/inet.h> 
#include <sys/socket.h>

#define BUF_SIZE 1024
void error_handling(char *message){
    std::cout<<*message<<std::endl;
    exit(1);
}

int main(int argc,char *argv[]){
  int servSock,clntSock;
  char message[BUF_SIZE];
  int strLen,i;
  
  struct sockaddr_in servAddr,clntAddr;
  socklen_t clntAddrSz;
  
  if(argc!=2){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  servSock=socket(PF_INET,SOCKET_STREAM,0);
  if(servSock==-1)
    error_handling("socket error");
  
  memset(&servAddr,0,sizeof(servAddr));
  servAddr.sin_family=AF_INET;
  servAddr.sin_addr.s_addr=htonl(INADDR_ANY);
  servAddr.sin_port=htons(atoi(argv[1]));
  
  if(bind(servSock,(struct sockaddr*)&servAddr,sizeof(servAddr))==-1){
    error_handling("binding error");
  }
  
  if(listen(servSock,5)==-1){
    error_handling("listen error");
  }
  
  clntAddrSz=sizeof(clntAddr);
  for(int i=0;i<5;i++){
    clntSock=accept(servSock,(struct sockaddr*)&clntAddr,&clntAddrSz);
    if(clntSock==-1){
      error_handling("accept error");
    }
    else{
      std::cout<<"Connected client:"<<i+1:
    }
    while(strLen=read(clntSock,message,BUF_SIZE)!=0){//传输读取到的字符串
      write(clntSock,message,strLen);
    }
    close(clntSock);
  }
  close(servSock);
  return 0;
}
```

echo_client.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arþa/inet.h> 
#include <sys/socket.h>

#define BUF_SIZE 1024
void error_handling(char *message){
    std::cout<<*message<<std::endl;
    exit(1);
}

int mian(int argc,char *argv[]){
  int sock;
  char message[BUF_SIZE];
  int strLen;
  struct sockaddr_in servAddr;
  
  if(argc!=3){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  sock=socket(PF_INET,SOCK_STREAM,0);
  if(sock==-1){
    error_handling("socket error");
  }
  
  memset(&servAddr,0,sizeof(servAddr));
  servAddr.sin_family=AF_INET;
  servAddr.sin_addr.s_addr=inet_addr(argv[1]);
  servAddr.sin_port=htons(atoi(argv[2]));
  
  //调用connect函数，若调用该函数引起的连接请求被注册到服务器端等待队列，则connect函数将正常完成，如果服务器尚未调用accept函数，也不会真正建立服务关系。
  if(connect(sock,(struct sockAddr*)&servAddr,sizeof(servAddr))==-1){
    error_handling("connect error");
  }
  else{
    std::cout<<"Connected......";
  }
  
  while(1){
    std::cout<<"input message(Q to quit)";
    fgets(message,BUF_SIZE,stdin);//获取用户输入的字符串
    
    if(!strcmp(message,"q\n")||!strcmp(message,"Q\n"))
      break;
    
    write(sock,message,strlen(message));//向sock中写入
    strLen=read(sock,message,BUF_SIZE-1);
    message[strLen]=0;
    std::cout<<"Message from server:"<<message;
  }
  close(sock);
  return 0;
}
```

### 4.3.3回声客户端存在的问题

```C++
		write(sock,message,strlen(message));//向sock中写入
    strLen=read(sock,message,BUF_SIZE-1);
    message[strLen]=0;
    std::cout<<"Message from server:"<<message;
```

以上代码有个错误假设：每次调用read、write函数都会以字符串为单位进行一次I/O操作；

TCP不存在数据边界，多次调用write函数传递的字符串可能一次性传递给服务器，也有可能字符串太长，一次write函数传递的字符串分成两次传递给服务器。

## 4.4基于windows的实现

### 4.4.1基于 Windows的回声服务器端

将linux平台转为windows平台需要记住：

- WSAStartup、WSACleanup函数初始化并清除套接字相关库；
- 将数据类型和数据名转换成windows风格；
- 数据传输中用recv、send函数；
- 关闭套接字使用closesocket函数。

echo_server_win.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <winsock2.h>

#define BUF_SIZE 1024
void ErrorHandling(char *message){
  
}

int main(int argc,char *argv[]){
  WSADATA wsaData;
  SOCKET hServSock,hClntSock;
  char message[BUF_SIZE];
  int strLen,i;
  SOCKADDR_IN servAddr,clntAddr;
  int clntAddrSize;
  
  if(argc!=2){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  if(WSAStartup(MAKEWORD(2,2),&wsaData)!=0){
    ErrorHandling("WSAStartup error");
  }
  
  hServSock=socket(PF_INET,SOCK_STREAM,0);
  if(hServSock==INVALID_SOCKET){
     ErrorHandling("socket error");
  }
  
  memset(&servAddr,0,sizeof(servAddr));
  servAddr.sin_family=AF_INET;
  servAddr.sin_addr.s_addr=htonl(INADDR_ANY);
  servAddr.sin_port=htons(atoi(argv[1]));
  
  if(bind(hServSock,(struct SOCKADDR*)&servAddr,sizeof(servAddr))==SOCKET_ERROR){
    ErrorHandling("bind error");
	}
  
  if(listen(hServSock,5)==SOCKET_ERROR){
    ErrorHandling("listen error");
  }
  
  clntAddrSize=sizeof(clntAddr);
  for(i=0;i<5;i++){
    hClntSock=accept(hServSock,(SOCKADDR*)&clntAddr,&clntAddrSize);
    if(hClntSock==-1){
      ErrorHandling("accept error");
    }
    else{
      std::cout<<"Connected client:"<<i+1;
    }
    
    while(strLen=recv(hClntSock,message,BUF_SIZE,0)!=0){
      send(hClntSock,message,strLen,0);
    }
    clsoesocket(hClntSock);
  }
  closesocket(hServSock);
  WSACleanup();
  return 0;
}
```

4.4.2基于windows的回声客户端

echo_client_win.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <winsock2.h>

#define BUF_SIZE 1024
void ErrorHandling(char *message){
  
}

int main(int argc,char *argv[]){
  WASDATA wasData;
  SOCKET hSocket;
  char message[BUF_SIZE];
  int strLen;
  SOCKADDR_IN servAddr;
  
  if(argc!=3){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  if(WSAStartup(MAKEWORD(2,2),&wasData)!=0){
    ErrorHandling("WSAStartup error");
  }
  
  hSocket=socket(PF_INET,SOCK_STREAM,0);
  if(hSocket==INVALID_SOCKET){
    ErrorHandling("socket error");
	}
  
  memset(&servAddr,0,sizeof(servAddr));
  servAddr.sin_family=AF_INET;
  servAddr.sin_addr.s_addr=inet_addr(argv[1]);
  servAddr.sin_port=htons(atoi(argv[2]));
  
  if(connect(hSocket,(SOCKADDR *)&servAddr,sizeof(servAddr))==SOCKET_ERROR){
    ErrorHandling("connect error");
  }
  else{
    std::cout<<"connected......";
  }
  
  while(1){
    std::cout<<"input message(Q to quit):";
    fgets(message,BUF_SIZE,stdin);
    
    if(!strcmp(message,"q\n")||!strcmp(message,"Q\n")){
      break;
    }
    
    send(hSocket,message,strlen(message),0);
    strLen=recv(hSocket,message,BUF_SIZE-1,0);
    message[strLen]=0;
    std::cout<<"message from server:"<<message;
  }
  closesocket(hSocket);
  WSACleanup();
  return 0;
}
```

## 4.5习题

请说明 TCP/IP的4层协议栈，并说明 TCP和UDP套接字经过的层级结构差异 ;

请说出 TCP/IP协议栈中链路层和IP层的作用，并给出二者关系；

为何需要把TCP/IP协议校分成4层(或7层) ?结合开放式系统回答；

客户端调用 connect函数向服务器端发送连接请求。 服务器端调用哪个函数后，客户端可以调用 connect函数 ?

什么时候创建连接请求等待队列?它有何作用?与 accept有什么关系?

客户端中为何不需要调用bind函数分配地址?如果不调用bind函数，那何时、如何向套接字分配ip地址和端口号?

把第1章的hello_server.c和hello_server_win.c改成迭代服务器端，并利用客户端测试更改是否准确 。

# 第五章 基于TCP的服务器端/客户端（2）

## 5.1回声客户端的完美实现

### 5.1.1回声服务器端没有问题，只有回声客户端有问题?

服务器端I/O代码

```C++
while(strLen=read(clntSock,message,BUF_SIZE)!=0){
  write(clntSock, message, strLen);
}
```

客户端I/O代码

```C++
write(sock,message,strlen(message));
strLen=read(sock,message,BUF_SIZE-1);
```

二者循环调用write和read函数，回声客户端将100%收到自己发送的数据，只不过在接受数据时单位有问题，

### 5.1.2回声客户端问题解决方法

提前确定接受数据的大小，若之前传输了20字节长的数据，则在接受时读取20个字节即可

Echo_client2.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arþa/inet.h> 
#include <sys/socket.h>

#define BUF_SIZE 1024
void error_handling(char *message){
    std::cout<<*message<<std::endl;
    exit(1);
}

int mian(int argc,char *argv[]){
  int sock;
  char message[BUF_SIZE];
  int strLen,recvLen,recvCnt;
  struct sockaddr_in servAddr;
  
  if(argc!=3){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  sock=socket(PF_INET,SOCK_STREAM,0);
  if(sock==-1){
    error_handling("socket error");
  }
  
  memset(&servAddr,0,sizeof(servAddr));
  servAddr.sin_family=AF_INET;
  servAddr.sin_addr.s_addr=inet_addr(argv[1]);
  servAddr.sin_port=htons(atoi(argv[2]));
  
  //调用connect函数，若调用该函数引起的连接请求被注册到服务器端等待队列，则connect函数将正常完成，如果服务器尚未调用accept函数，也不会真正建立服务关系。
  if(connect(sock,(struct sockAddr*)&servAddr,sizeof(servAddr))==-1){
    error_handling("connect error");
  }
  else{
    std::cout<<"Connected......";
  }
  
  while(1){
    std::cout<<"input message(Q to quit)";
    fgets(message,BUF_SIZE,stdin);//获取用户输入的字符串
    
    if(!strcmp(message,"q\n")||!strcmp(message,"Q\n"))
      break;
    
    strLen=write(sock,message,strlen(message));//向sock中写入
    recvLen=0;
    //修改部分
    while(recvLen<strLen){
      recvCnt=read(sock,,&message[recvLen],BUF_SIZE-1);
      if(recvCnt==-1){
        error_handling("read error");
      }
      recvLen+=recvCnt;
    }
    message[recvLen]=0;
    //修改部分
    std::cout<<"Message from server:"<<message;
  }
  close(sock);
  return 0;
}
```

### 5.1.3如果问题不在于回声客户端:定义应用层协议

若无法预知接收数据长度时应如何收发数据?此时需要的就是应用层协议的定义。之前的回声服务器端/客户端中定义了如下协议：

收到q就立即终止

收发数据过程中也需要定好规则(协议)以表示数据的边界，或提前告知收发数据的大小。服务器端/客户端实现过程中逐步定义的这些规则集合就是应用层协议。

### 5.1.4计算器服务器端/客户端示例

设计如下应用层协议，但这只是为实现程序而设计的最低协议，实际的应用程序实现中需要的协议更详细、准确 。

- 客户端连接到服务器端后以 1字节整数形式传递待算数字个数；
- 客户端向服务器端传递的每个整数型数据占用 4字节；
- 传递整数型数据后接着传递运算符。运算符信息占用1字节；
- 选择字符+、-、*之一传递；
- 服务器端以 4字节整数型向客户端传回运算结果；
- 客户端得到运算结果后终止与服务器端的连接；

op_client.c

```C++
#include 省略
#define BUF_SIZE 1024
#define RLT_SIZE 4
#define OPSZ 4

void error_handling(char *message){
  
}

int main(int argc,char *argv[]){
  int sock;
  char opmsg[BUF_SIZE];
  int result,opnd_cnt,i;
  struct sockaddr_in serv_addr;
  if(argc!=3){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  sock=socket(PF_INET,SOCK_STREAM,0);
  if(sock==-1){
    error_handling("socket error");
  }
  
  memset(&serv_addr,0,sizeof(serv_addr));
  serv_addr.sin_family=AF_INET;
  serv_addr.sin_addr.s_addr=htonl
}
```



## 5.2TCP原理

### 5.2.1TCP套接字中的 I/O缓冲

TCP套接字的数据收发无边界。 服务器端即使调用 1次write函数传输40字节的数据，客户端也有可能通过4次read函数调用每次读取10字节。

write函数调用后并非立即传输数据， read函数调用后也并非马上接收数据。write函数调用瞬间，数据将移至输出缓冲; read函数调用瞬间，从输入缓冲读取数据。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230810145642911.png" alt="image-20230810145642911" style="zoom:50%;" />

调用write函数时数据将移到输出缓冲，在适当的时候(不管是分别传送还是一次性传送)传向对方的输入缓冲 。 这时对方将调用 read函数从输入缓冲读取数据 。 这些 I/O 缓冲特性可整理如下：

- I/O缓冲在每个TCP套接字中单独存在；
- I/O缓冲在创建套接字时自动生成；
- 即使关闭套接字也会继续传递输出缓冲中遗留的数据；
- 关闭套接字将丢失输入缓冲中的数据 ；

"客户端输入缓冲为 50字节，而服务器端传输了 100字节 。"怎么办：

因为TCP会控制数据流。 TCP中有滑动窗口( SlidingWindow) 协议，因此TCP中不会因为缓冲溢出而丢失数据 。

### 5.2.2TCP 内部工作原理 1:与对方套接字的连接

TCP套接字从创建到消失所经过程分为如下 3步 ：

- 与对方套接字建立连接；
- 与对方套接字进行数据交换 ；
- 断开与对方套接字的连接；

建立连接：

A:"你好，套接字B。 我这儿有数据要传给你，建立连接吧 。"

B:"好的，我这边己就绪。"

A:"谢谢你受理我的请求。"

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230810160317247.png" alt="image-20230810160317247" style="zoom:50%;" />

A向B传递：[SYN] SEQ: 1000, ACK: -；同步请求，本数据包序号为1000，如果接收无误，请通知我向您传递 1001号数据包；

B向A传递：[SYN+ACK] SEQ: 2000，ACK: 1001；同步与确认请求，本数据包序号为 2000，如果接收无误，请通知我向您传递2001号数据包；刚才传输的 SEQ为1000的数据包接收无误，现在请传递 SEQ为 1001的数据包；

A向B传递：[ACK] SEQ: 1001, ACK: 2001；确认请求，刚才传输的 SEQ为2000的数据包接收无误，现在请传递 SEQ为 2001的数据包；

至此，主机A和主机B确认了彼此均就绪。

### 5.2.3TCP 内部工作原理2: 与对方主机的数据交换

图5-4给出了主机A分2次(分2个数据包)向主主机B传递200字节的过程

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230810163413245.png" alt="image-20230810163413245" style="zoom:50%;" />

A通过1个数据包发送 100个字节的数据.数据包的SEQ为1200；

B为了确认这一点，向A发送ACK1301消息；此时的ACK号为 1301而非 1201 ，原因在于ACK号的增量为传输的数据字节数；ACK 号 → SEQ 号+ 传递的字节数+ 1

### 5.2.4TCP 内部工作原理3: 断开与套接字的连接

如果对方还有数据需要传输时直接断掉连接会出问题， 所以断开连接时需要双方协商 。 断开连接时双方对话如下：

A: "我希望断开连接。"

B: "哦是吗?请稍候。"

B: "我也准备就绪，可以断开连接。"

A:"好的，谢谢合作 。"

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230810164149216.png" alt="image-20230810164149216" style="zoom:50%;" />



## 5.3基于windows实现

## 5.4习题

请说明TCP套接字连接设置的三次握手过程。 尤其是3次数据交换过程每次收发的数据内容；

TCP是可靠的数据传输协议 ， 但在通过网络通信的过程中可能丢失数据。 请通过ACK和 SEQ说名TCP通过何种机制保证丢失数据的可靠传输；

TCP套接字中调用write和read函数时数据如何移动?结合I/O缓冲进行说明；

对方主机的输入缓冲剩余50字节空间时，若本方主机通过write函数请求传输70字节，请 问 TCP如何处理这种情况?

# 第六章 基于UDP的服务器端/客户端

## 6.1理解UDP

### 6.1.1UDP 套接字的特点

寄信前应先在信封上填好寄信人和收信人的地址，之后贴上邮票放进邮筒即可 。

为 了提供可靠的数据传输服务， TCP在不可靠的 IP层 进行流控制，而UDP就缺少这种流控制机制 。

### 6.1.2UDP 内部工作原理

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230810165149008.png" alt="image-20230810165149008" style="zoom:50%;" />

IP的作用就是让离开主机B的UDP数据包准确传递到主机A。 但把UDP 包最终交给主机A的某一UDP套接字的过程则是由UDP完成的。 UDP最重要的作用就是根据端口号将传到主机的数据包交付给最终的UDP套接字 。

### 6.1.3UDP 的高效使用

## 6.2实现基于UDP的服务器端/客户端

### 6.2.1UDP 中的服务器端和客户端没有连接

UDP中只有创建套接字的过程和数据交换过程 。

### 6.2.2UDP 服务器端和客户端均只需1个套接字

### 6.2.3基于 UDP 的数据I/O函数

创建好TCP套接字后，传输数据时无需再添加地址信息。 因为TCP套接字将保持与对方套接字的连接。但UDP套接字不会保持连接状态 (UDP套接字只有简单的邮筒功能)，因此每次传输数据都要添加目标地址信息。

```C++
ssize_t sendto(int sock, void *buff, size_t nbytes, int flags, struct sockaddr *to, socklen_t addrlen);
//成功时返回传输的字节数，失败时返回-1；
```

sock：用于传输数据的 UDP套接字文件描述符；

buff：保存待传输数据的缓冲地址值；

nbytes：待传输的数据长度，以字节为单位；

flags：可选项参数，若没有则传递 0；

to：存有目标地址信息的 sockaddr结构体变量的地址值；

addrlen：传递给参数to的地址值结构体变量长度；

```C++
ssize_t recvfrom(int sock, void *buff, size_t nbytes, int flags,struct sockaddr * from, socklen_t *addrlen);
//成功时返回接收的字节数，失败时返回-1；
```

sock：用于接收数据的 UDP套接字文件描述符；

buff：保存接收数据的缓冲地址值；

nbytes：可接收的数据长度，以字节为单位；

flags：可选项参数，若没有则传递 0；

from：存有发送端地址信息的 sockaddr结构体变量的地址值；

addrlen：保存参数 from的结构体变量长度的变量地址值 ；

### 6.2.4基于 UDP 的回声服务器端/客户端

UDP不同于TCP，不存在请求连接和受理过程，因此在某种意义上无法明确区分服务器端和客户端。

uecho_server.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message){
  
}

int main(int argc,char *argv[]){
  int servSock;
  char message[BUF_SIZE];
  int strLen;
  socklen_t clntAdrSz;
  struct sockaddr_in servAddr,clntAddr;//服务器地址，客户端地址结构体
  if(argc!=2){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  servSock=socket(PF_INET,SOCK_DGRAM,0);
  if(servSock==-1){
    error_handling("socket error");
  }
  
  memset(&servAddr,0,sizeof(servAddr));
  servAddr.sin_family=AF_INET;
  servAddr.sin_addr.s_addr=htonl(INADDR_ANY);
  servAddr.sin_port=htons(atoi(argv[1]));
  
  if(bind(servSock,(struct sockaddr*)&servAddr,sizeof(servAddr))==-1){
    error_handling("bind error");
  }
  
  while(1){
    clntAdrSz=sizeof(clntAddr);
    strLen=recvfrom(servSock,message,BUF_SIZE,0,(struct sockaddr*)&clntAddr,&clntAdrSz);
    sendto(servSock,message,strLen,0,(struct sockaddr*)&clntAddr,clntAdrSz);
  }
  close(servSock);
  return 0;
}
```

uecho_client.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message){
  
}

int main(int argc,char *argv[]){
  int sock;
  char message[BUF_SIZE];
  int strLen;//字符串长度
  socklen_t addrSz;//存储接受消息地址长度而已
  
  struct sockaddr_in serv_addr,from_addr;//服务器地址，消息来源结构体
  if(argc!=3){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  memset(&serv_addr,0,sizeof(serv_addr));
  serv_addr.sin_family=AF_INET;
  serv_addr.sin_addr.s_addr=inet_addr(argv[1]);
  serv_addr.sin_port=htons(atoi(argv[2]));
  
  while(1){
    std::cout<<"input message(q to quite):";
    fgets(message,sizeof(message),stdin);
    if(!strcmp(message,"q\n")||!strcmp(message,"Q\n"))
      break;
    
    sendto(sock,message,strlen(message),0,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
    addrSz=sizeof(from_addr);
    strLen=recvfrom(sock,message,BUF_SIZE,0,(struct sockaddr*)&from_addr,&addrSz);
    message[strLen]=0;
    std::cout<<"message from server:"<<message;
  }
  close(sock);
  return 0;
}
```

TCP客户端套接字在调用 connect函数时自动分配IP地址和端 口号，既然如此， UDP 客户端何时分配IP地址和端口号 ?

### 6.2.5UDP 客户端套接字的地址分配

UDP程序中，调用sendto函数传输数据前应完成对套接字的地址分配工作，因此调用bind函数 。 当然， bind函数在TCP程序中出现过，但bind函数不区分TCP和UDP，也就是说，在UDP程序中同样可以调用。如果调用sendto函数时发现尚未分配地址信息，则在首次调用sendto函数时给相应套接字向动分配IP和端口。

## 6.3UDP的数据传输特性和调用connect函数

### 6.3.1存在数据边界的UDP套接字

TCP数据传输中不存在边界，这表示"数据传输过程中调用I/O函数的次数不具有任何意义。"

UDP是具有数据边界的协议，输入函数的调用次数应和输出函数的调用次数完全一致，这样才能保证接收全部已发送数据。 

bound_host1.c

```C++
#include 省略
#define BUF_SIZE 30
void error_handling(char *message){
  
}

int main(int argc,char *argv[]){
  int sock;
  char message[BUF_SIZE];
  int strLen,i;
  socklen_t addrSz;//存储接受消息地址长度而已
  struct sockaddr_in myAddr,yourAddr;//服务器地址，客户端地址结构体
  if(argc!=2){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  sock=socket(PF_INET,SOCK_DGRAM,0);
  if(sock==-1){
    error_handling("socket error");
  }
  
  memset(&myAddr,0,sizeof(myAddr));
  myAddr.sin_family=AF_INET;
  myAddr.sin_addr.s_addr=htonl(INADDR_ANY);
  myAddr.sin_port=htons(atoi(argv[1]));
  
  if(bind(sock,(struct sockaddr*)&myAddr,sizeof(myAddr))==-1){
    error_handling("bind error");
  }
  
  for(i=0;i<3;i++){
    sleep(5);
    addrSz=sizeof(yourAddr);
    strLen=recvfrom(sock,message,BUF_SIZE,0,(struct sockaddr*)&yourAddr,&addrSz);
   	std::cout<<"message:"<<message;
  }
  close(sock);
  return 0;
}
```

bound_host2.c

```C++
#include 省略
#define BUF_SIZE 30
void error_handling(char *message){
  
}

int main(int argc,char *argv[]){
  int sock;
  char msg1[]="你好";
  char msg2[]="我是另一台udp主机";
  char msg3[]="很高兴认识你";
  
  struct sockaddr_in yourAddr;//目标地址哦
  socklen_t yourAddrSz;
  if(argc!=3){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  sock=socket(PF_INET,SOCK_DGRAM,0);
  if(sock==-1){
    error_handling("socket error");
  }
  
  memset(&yourAddr,0,sizeof(yourAddr));
  yourAddr.sin_family=AF_INET;
  yourAddr.sin_addr.s_addr=inet_addr(argv[1]);
  yourAddr.sin_port=htons(atoi(argv[2]));
  
  sendto(sock,msg1,sizeof(msg1),0,(struct sockaddr*)&yourAddr,sizeof(yourAddr));
  sendto(sock,msg2,sizeof(msg1),0,(struct sockaddr*)&yourAddr,sizeof(yourAddr));
  sendto(sock,msg3,sizeof(msg1),0,(struct sockaddr*)&yourAddr,sizeof(yourAddr));
  close(sock);
  return 0;
}
```

udp数据报：

UDP套接字传输的数据包又称数据报，实际上数据报也属于数据包的一种。只是与 TCP 包不同，其本身可以成为1个完整数据。这与 UDP 的数据传输特性有关，UDP 中存在数据边界，1个数据包即可成为1个完整数据，因此称为数据报。

### 6.3.2已连接(connected)UDP套接字与未连接(unconnected)UDP套接字

TCP套接字中需注册待传输数据的目标IP和端口号，而UDP中则无需注册。 通过sendto函数传输数据的过程大致可分为以下 3个阶段 ：

第 1阶段:向UDP套接字注册目标E和端口号；

第 2阶段:传输数据；

第 3阶段:删除UDP套接字中注册的目标地址信息 ；

因此可以重复利用同一UDP套接字向不同目标传输数据。这种未注册目标地址信息的套接字称为未连接套接字，反之，注册了目标地址的套接字称为连接connected套接字。

“IP为 211.210.147.82的主机 82号端口共准备了 3个数据，调用 3次sendto函数进行传输 。”

此时需重复3次上述三阶段。因此，要与同一主机进行长时间通信时，将UDP套接字变成已连接套接字会提高效率。

### 6.3.3创建己连接UDP套接字

 只需针对UDP套接字调用 connect函数 

```C++
connect(sock, (struct sockaddr *) &adr, sizeof(adr));
```

针对UDP套接字调用connect函数并不意味着要与对方UDP套接字连接，这只是向UDP套接字注册目标IP和端口信息；

之后就与TCP套接字一样，每次调用sendto函数时只需传输数据。 因为已经指定了收发对象，所以不仅可以使用sendto、 recvfrom函数 ， 还可以使用write、 read函数进行通信。

Todo

uecho_con_client.c

```C++
#include 省略
#define BUF_SIZE 30
void error_handling(char *message){
  
}

int main(int argc,char *argv[]){
	int sock;
  char message[BUF_SIZE];
  int strLen;
  
  struct sockaddr_in servAddr;
  if(argc!=3){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  sock=socket(PF_INET,SOCK_DGRAM,0);
}
```

## 6.4基于windows实现

## 6.5习题

UDP为什么TCP速度快?为什么TCP数据传输可靠而UDP数据传输不可靠?

下列不属于UDP特点的是?

- a.UDP不同于TCP; 不存在连接的概念，所以不像TCP那样只能进行一对一的数据传输。
- b.利用UDP传输数据时， 如果有2个目标， 则需要2个套接字。
- c. UDP套接字中无法使用 已分配给TCP 的同一端口号。
- d.UDp套接字和TCP套接字可以共存 。若需要，可以在同一主机进行TCP和UDP数据传输 。
- e.针对UDP函数也可以调用connect函数， 此时UDP套接字跟TCP套接字相同，也需要经过3次握手过程 。

UDP数据报向对方主机的UDP套接字传递过程中IP和UDP分别负责哪些部分?

UDP一般比TCP快。但根据交换数据的特点，其差异可太可小。 请说明何种情况下UDP的性能优于TCP?

客户端TCP套接字调用connect函数时 自动分配IP和端 口号。UDP中不调用bind函数， 那合适分配IP和端口号?

TCP客户端必需调用connect函数，而UDP中可以选择性调用 。请问，在UDP中调用connect函数有哪些好处 ?

请参考本章给出的uecho-server-c和uecho-client-c，编写示例使服务器端和客户端轮流收发消息。收发的消息均要输出到控制台窗口。

# 第七章 优雅的断开套接字连接

## 7.1基于TCP的半关闭

### 7.1.1单方面断开连接带来的问题

Linux的close函数和Windows的closesocket函数意味着完全断开连接。 完全断开不仅指无法传 输数据，而且也不能接收数据。

### 7.1.2套接字和流 (Stream)

两台主机通过套接字建立连接后进入可交换数据的状态，又称"流形成的状态"。也就是把建立套接字后可交换数据的状态看作一种流。为了近行双向通信，需要如图7-2所示的2个流。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230815132746286.png" alt="image-20230815132746286" style="zoom:50%;" />



### 7.1.3针对优雅断开的 shutdown函数

shutdown函数就用来关闭其中1个流

```C++
int shutdown(int sock, int howto);
//成功时返回0，失败时返回-1
```

sock：需要断开的套接字文件描述符；

howto：传递断开方式信息。

第二个参数决定断开连接的方式，其可能值如：

- SHUT_RD：断开输入流：断开输入流，套接字无法接收数据，即使输 入缓冲收到数据也会抹去，而且无法调用输入相关函数。
- SHUT_WR：断开输出流：中断输出流，也就无法传输数据。 但如果输出缓冲还留有未传输的数据，则将传递至目标主机。 
- SHUT_RDWR：同时断开I/O流：同时中断1/0流。 这相当于分2次调用shutdown。

### 7.1.4为何需要半关闭

"究竟为什么需要半关闭?是否只要留出足够长的连接时间，保证完成数据交换即可?只要不急于断开连接，好像也没必要使用半关闭 。"

但要考虑如下情况 :

"一旦客户端连接到服务器端，服务器端将约定的文件传给客户端，客户端收到后发送字符串‘Thankyou'给服务器端 。"

此处字符串 "Thank you" 的传递实际是多余的，这只是用来模拟客户端断开连接前还有数据需要传递的情况。此时程序实现的难度并不小，因为传输文件的服务器端只需连续传输文件数据即可，而客户端则无法知道需要接收数据到何时。 客户端也没办法无休止地调用输入函数，因为这有可能导致程序阻塞(调用的函数未返回)。

### 7.1.5基于半关闭的文件传输程序

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230815134919986.png" alt="image-20230815134919986" style="zoom:50%;" />

file_server.c

```C++

```



## 7.2基于windows实现

## 7.3习题

# 第八章 域名及网络地址

## 8.1域名系统

DNS是对IP地址和域名进行相互转换的系统 ，其核心是DNS服务器。

### 8.1.1什么是域名

### 8.1.2DNS服务器

## 8.2IP和域名之间的转换

### 8.2.1程序中有必要使用域名吗?

IP地址比域名发生变更的概率要高，所以利用E地址编写程序并非上策，肉此利用域名编写程序会好一些。 这样，每次运行程序时根据域名获取IP地址，再接人服务器，这样程序就不会依赖于服务器IP地址了 。

### 8.2.2利用域名获取IP地址

```C++
struct hostent * gethostbyname(const char * hostname);
//成功时返回hostent结构体地址 ，失败时返回 NULL 指针
```

只要传递域名字符串，就会返回域名对应的 IP地址 。 信息装入hosten结构体。 此结构体定义如下：

```C++
struct hostent{
  char * h_name;			//官方名
  char ** h_aliases;	//别名列表
  int h_addrtype;			//主机地址类型
  int h_length;				//地址长度
  char ** h_addr_list;//地址列表
}
```

域名转IP时只需关注h_addr_list，

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230815141243192.png" alt="image-20230815141243192" style="zoom:50%;" />

gethostbyname.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <netdb.h>
void error_handling(char * message){
  
}

int main(int argc,char *argv[]){
  int i;
  struct hostent *host;
  if(argc!=2){
    std::cout<<"usage:"<<argv[0];
    exit(1);
  }
  
  host=gethostbyname(argv[1]);
  if(!host){
    error_handling("gethostbyname error");
  }
  
  std::cout<<"offical name:"<<host->hostname;
  for(i=0;host->h_aliases[i];i++){
    std::cout<<""
  }
}
```



### 8.2.3利用IP地址获取域名

## 8.3基于windows实现

## 8.4习题

# 第九章 套接字的多种可选项

## 9.1套接字可选项和I/O缓冲大小

### 9.1.1套接字多种可选项

### 9.1.2getsockopt& setsockopt

### 9.1.3SO_SNDBUF & SO_RCVBUF

## 9.2SO_REUSEADDR

### 9.2.1发生地址分配错误 (Binding Error)

### 9.2.2Time-wait状态

### 9.2.3地址再分配

## 9.3TCP_NODELAY

### 9.3.1Nagle算法

### 9.3.2禁用 NagJe算法

## 9.4基于windows实现

## 9.5习题

下列关于 Time-wait状态的说法错误的是?

# 第十章 进程服务器端

## 10.1进程概念及应用

### 10.1.1两种类型的服务器端

"第一个连接请求的受理时间为 0秒，第 50个连接请求的受理时间为 50秒，第 100个连接请求的受理时间为100秒!但只要受理，服务只需 1秒钟 。"

"所有连接请求的受理时间不超过1秒，但平均服务时间为 2--3秒 。"

### 10.1.2并发服务器端的实现方法

有必要改进服务器端，使其同时向所有发起请求的客户端提供服务，网络程序中数据通信时间比CPU运算时间占比更大，因此，向多个客户端提供服务是一种有效利用CPU的方式。 

下面列出的是具有代表性的并发服务器端实现模型和方法：

- 多进程服务器 ：通过创建多个进程提供服务 。
- 多路复用服务器：通过捆绑并统一管理I10对象提供服务 。
- 多线程服务器：通过生成与客户端等量的线程提供服务。

多进程服务器。这种方法不适合在Windows平台下 (Windows不支持) 讲解，因此将重点放在Linux平台。

### 10.1.3理解进程 (Process)

"占用内存空间的正在运行的程序"

### 10.1.4进程 ID

无论进程是如何创建的，所有进程都会从操作系统分配到ID。此ID称为"进程ID"，其值为大于2的整数，1要分配给操作系统启动后（用于协助操作系统）首个进程，因此用户进程无法得到lD值1。

通过ps指令可以查看当前运行的所有进程。

### 10.1.5通过调用 fork 函数创建进程

创建进程的方法很多，此处只介绍用于创建多进程服务器端的 fork函数 。

```C++
pid_t fork(void);
//成功时返回进程工0，失败时返回-1
```

fork函数将创建调用的进程副本，也就是说是复制正在运行的、调用fork函数的进程，两个进程都将执行fork函数调用后的语句(准确地说是在fork函数返回后)利用fork函数的如下特点区分程序执行流程：

- 父进程: fork函数返回子进程ID
- 子进程: fork函数返回0

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230815165713185.png" alt="image-20230815165713185" style="zoom:50%;" />

父进程调用fork函数的同时复制出子进程，并分别得到fork函数的返回值，但复制前，父进程将全局变量gval增加到 11， 将局部变量lval的值增加到25

fork.c

```C++
#include <stdio.h>
#include <unistd.h>

int gval=10;
int main(int argc,char *argv[]){
  pid_t pid;
  int lval=20;
  gval++;
  lval+=5;
  
  pid=fork();
  if(pid==0){
    gval+=2;
    lval+=2;
  }
  else{
    gval-=2;
    lval-=2;
  }
  
  if(pid==0){
    std::cout<<"child process:"<<gval<<lval;
  }else{
    std::cout<<"parent process:"<<gval<<lval;
  }
  return 0;
}
```

## 10.2进程和僵尸进程 

文件操作中，关闭文件和打开文件同等重要。同样，进程销毁也和进程创建同等重要。如果未认真对待进程销毁，它们将变成僵尸进程困扰各位。 

### 10.2.1僵尸 (Zombie) 进程

进程完成工作后 (执行完main函数中的程序后)应被销毁，但有时这 些进程将变成僵尸进程，占用系统中的重要资源。这种状态下的进程称作"僵尸进程"。

### 10.2.2产生僵尸进程的原因

利用如下两个示例展示调用 fork函数产生子进程的终止方式：

- 传递参数并调用exit函数；
- mam函数中执行return语句并返回值；

向exit函数传递的参数值和mam函数的return语句返回的值都会传递给操作系统，而操作系统 不会销毁子进程，直到把这些值传递给产生该子进程的父进程，处在这种状态下的进程就是僵尸进程。也就是说，将子进程变成僵尸进程的正是操作系统。 既然如此，此僵尸进程何时被销毁呢?

"应该向创建子进程的父进程传递子进程的 exit参数值或 return语句的返回值 。"

操作系统不会主动把这些值传递给父进程。只有父进程主动发起请求(函数调用)时，操作系统才会传递该值，如果父进程未主动要求获得子进程的 结束状态值，操作系统将一直保存，父母要负责收回自己生的孩子。

zombie.c

```C++
#include <stdio.h>
#include <unistd.h>

int main(int argc,char * argv[]){
  pid_t pid=fork();
  
  if(pid==0){//子进程
    std::cout<<"你好，我是子线程";
  }
  else{
    std::cout<<"子线程id为"<<pid;
    sleep(30);//父进程暂停30秒。如果父进程终止，处于僵尸状态的子进程将同时销毁。延缓父进程的执行以验证僵尸进程。
  }
  
  if(pid==0){
    std::cout<<"子线程结束";
  }
  else{
    std::cout<<"父线程结束";
  }
  return 0;
}
```

PID为10977的进程状态为僵尸进程 (z+)。 另外，经过30秒的等待时间后，PID 为10976的父进程和之前的僵尸子进程同时销毁。

**后台处理：**

将控制台中的指令放到后台运行的方式：

root@my_linux:/tcpip# ./zombie &

如果采用这种方式运行程序，即可在同一控制台输入下列命令，无需另外 打开新控制台。

root@my_linux:/tcpip# ps au

### 10.2.3销毁僵尸进程 1:利用 wait函数

父进程应主动请求获取子进程的返回值：

```C++
pid_t wait(int * statloc);
//成功时退回终止的子进程ID，失败时返回-1
```

调用此函数时如果已有子进程终止 ，那么子进程终止时传递的返回值( exit函数的参数值、 main函数的 retum返回值)将保存到该函数的参数所指内存空间。

参数指向的单元中还包含其他信息，因此需要通过下列宏进行分离：

- WIFEXITED子迸程正常终止时返回true；
- WEXITSTATUS返回子进程的返回值；

调用wait函数后应编写如下代码：

```C++
if(WIFEXITED(status)){//是否正常终止了？
  std::cout<<"正常结束";
  std::cout<<"子进程ID为"<<WEXITSTATUS(status);
}
```

wait.c

```C++
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>

int main(int argc,char * argv[]){
  int status;
  pid_t pid=fork();
  
  if(pid==0){
    return 3;//终止第一次创建的子进程；
  }
  else{
    std::cout<<"子线程id="<<pid;
    pid=fork();
    if(pid==0){
      exit(7);//终止第二次创建的子进程；
    }
    else{
      std::cout<<"子线程id="<<pid;
      wait(&status);//之前终止的子进程相关信息保存到status，相关子进程被完全销毁；
      //此时子线程已经终止，返回值也已经由父进程主动获取到了
      if(WIFEXITED(status)){//验证第一个子进程是否正常终止，如果正常退出则输出子进程返回值
        std::cout<<"子线程发送1"<<WEXITSTATUS(status);
      }
      
      wait(&status);
      if(WIFEXITED(status)){//验证第二个子进程是否正常终止，如果正常退出则输出子进程返回值
        std::cout<<"子线程发送2"<<WEXITSTATUS(status);
      }
      sleep(30);
    }
  }
  return 0;
}
```

调用wait函数时，如果没有己终止的子进程，那么程序将阻塞( Blocking) 直到有子进程终止，因此需谨慎调用该函数。

### 10.2.4销毁僵尸进程 2: 使用 waitpid 函数

是防止阻塞的方法

```C++
pid_t waitpid(pid_t pid, int * statloc, int options);
//成功时返回终止的子进程ID(或0)，失败时返回-1
```

Pid：等待终止的目标子进程的ID，若传递-1，则与wait函数相同，可以等待任意子进程终止；

statloc：与wait函数的 statloc参数具有相同含义；

options：传递头文件sys/wait.h中声明的常量WNOHANG，即使没有终止的子进程也不会进入阻塞状态，而是返回0并退出函数；

Waited.c

```C++
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc,char *argv[]){
  int status;
  pid_t pid=fork();
  
  if(pid==0){
    sleep(15);
    return 24;
  }
  else{
    while(!waitpid(-1,&status,WNOHANG)){
      sleep(3);
      std::cout<<"停止三秒";
    }
    
    if(WIFEXITED(status)){
      std::cout<<"子线程发送"<<WEXITSTATUS(status);
    }
  }
  return 0;
}
```

 std::cout<<"停止三秒"; 共执行了 5次

## 10.3信号处理

"子进程究竟何时终止?难道要调用waitpid函数后要无休止地等待吗?"

### 10.3.1向操作系统求助

若操作系统能告诉正忙于工作的父进程“你所创建的子进程终止了” ，将有助于构建高效的程序；

为了实现该想法，我们引人信号处理( Signal Handling ) 机制。此处的"信号"是在特定事件发生时由操作系统向进程发送的消息。 另外，为了响应该消息，执行与消息相关的自定义操作的过程称为"处理"或"信号处理"。

### 10.3.2关于 JAVA 的题外话:保持开放思维

JAVA在编程语言层面支持进程或线程，但C语言及C++语言并不支持，ANSI标准并未定义支持进程或线程的函数(JAVA的方法)。但仔细想想，这也是合理的：进程或线程应该由操作系统提供支持，因此，Windows中按照Windows的方式，Linux中按照Linux 的方式创建进程或线程，但JAVA为了保持平台移植性，以独立于操作系统的方式提供进程和线程的创建方法。

### 10.3.3信号与signal函数

- 进程:"嘿，操作系统!如果我之前创建的子进程终止，就帮我调用 zombie_handier函数。"
- 操作系统:"好的!如果你的子进程终止，我会帮你调用 zombie-handler函数，你先把该函数要执行的语句编好!"

"注册信号"过程，即进程发现自己的子进程结束时，请求操作系统调用特定函数。该请求通过如下函数调用完成(因此称此函数为信号注册函数)：

```C++
void (*signal(int signo, void (*func)(int)))(int);
//为了在产生信号时调用，返回之前注册的函数指针，返回值类型为函数指针
```

函数名：signal；

参数：int signo, void (* func)(int)；

返回类型：参数为int型，返回void型函数指针；

第一个参数为特殊情况信息：SIGALRM: 已到通过调用alarm函数注册的时间；SIGINT: 输入CTRL+C；SIGCHLD: 子进程终止。

第二个参数为特殊情况下将要调用的函数的地址值(指针)，发生第一个参数代表的情况时，调用第二个参数所指的函数。

编写调用signal函数的语句完成如下请求 :"子进程终止则调用 mychild函数。"此时mychild函数的参数应为int，返回值类型应为void，才能成为signal函数的第二个参数：

```C++
signal(SIGCHLD, mychild);
```

“已到通过alarm函数注册的时间，请调用 timeout函数。“：

```C++
signal(SIGALRM, timeout);
```

"输入CTRL+C时调用 keycontrol函数。"：

```C++
signal(SIGINT, keycontrol);
```

注册好信号后，发生注册信号时 (注册的情况发生时 )，操作系统将调用该信号对应的函数 。 下面通过示例验证，先介绍 alann函数 ：

```C++
unsigned int alarm(unsigned int seconds);
//返回以秒为单位的距SIGALRM信号发生所剩时间。
```

传递一个正整型参数，相应时间后(以秒为单位)将产生 SIGALRM信号。 若向该函数传递0，则之前对SIGALRM信号的预约将取消。

signal.c

```C++
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void timeout(int sig){//分别定义信号处理函数。这种类型的函数称为信号处理器(Handler)。
  if(sig==SIGALRM){
    std::cout<<"Time out";
  }
  alarm(2);
}
void keycontrol(int sig){
  if(sig==SIGINT){
    std::cout<<"CTRL+C被按住了";
  }
}

int main(int argc char * argv[]){
  int i;
  signal(SIGALRM,timeout);//注册SIGALRM、SIGINT信号及相应处理器。
  signal(SIGINT,keycontrol);
  alarm(2);
  
  for(i=0;i<3;i++){
    std::cout<<"wait...";
    sleep(100);
  }
  return 0;
}
```

"发生信号时将唤醒由于调用 sleep函数而进入阻塞状态的进程 。"

调用函数的主体的确是操作系统，但进程处于睡眠状态时无法调用函数。 因此，产生信号时， 为了调用信号处理器，将唤醒由于调用sleep函数而进入阻塞状态的进程。而且，进程一旦被唤醒， 就不会再进入睡眠状态。 即使还未到sleep函数中规定的时间也是如此。

### 10.3.4利用sigaction函数进行信号处理

类似于 signal函数，而且完全可以代替后者，也更稳定：“signal函数在UNIX系列的不同操作系统中可能存在区别，但sigaction函数完全 相同 。"

```C++
int sigaction(int signo, const struct sigaction * act,struct sigaction * oldact);
//成功时返回0，失败时返回-1
```

signo：传递信号信息；

act：对应于第一个参数的信号处理函数（信号处理器）信息；

oldact：通过此参数获取之前注册的信号处理函数指针，若不需要则返回0；

```C++
struct sigaction{
  void (*sa_handler)(int);//保存信号处理函数的指针值
  sigset_t sa_mask;//sa-mask和sa_flags的所有位均初始化为0即可。
  int as_flags;
}
```

sigaction.c

```C++
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void timeout(int sig){
  if(sig==SIGALRM){
    std::cout<<"超时"；
  }
  alarm(2);
}

int main(int argc,char * argv[]){
  int i;
  struct sigaction act;//声明sigaction结构体变量并在sa-handler成员中保存函数指针值。
  act.sa_handler=timeout;
  sigemptyset(&act.sa_mask);
  act.as_flags=0;
  sigaction(SIGALRM,&act,0);//注册SIGALRM信号的处理器。调用alarm函数预约2秒后发生SIGALRM信号。
  
  alarm(2);
  for(i=0;i<3;i++){
    std::cout<<"wait...";
    sleep(100);
  }
  return 0;
}
```

### 10.3.5利用信号处理技术消灭僵尸进程

子进程终止时将产生 SIGCLD信号，知道这一点就很容易完成。

Remove_zombie.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <sys/wait.h>

void read_childproc(int sig){
  int status;
  pid_t id=waitpid(-1,&status,WNOHANG);
  if(WIFEXITED(status)){
    std::cout<<"移除进程："<<id;
    std::cout<<"子线程发送"<<WEXITSTATUS(status);
  }
}

int main(int argc,char * argv[]){
  pid_t pid;
  //注册SIGCHLD信号对应的处理器。若子进程终止，调用read_childproc。处理函数中调用了waitpid函数，所以子进程将正常终止，不会成为僵尸进程。
  struct signaction act;
  act.sa_handler=read_childproc;
  sigemptyset(&act.sa_mask);
  act.sa_flags=0;
  sigaction(SIGCHLD,&act,0);//注册信号处理器
  
  pid=fork();
  if(pid==0){//子线程执行区域
    std::cout<<"你好，我是子线程";
    sleep(10);
    return 12;
  }
  else{//父线程执行区域
    std::cout<<"子线程id"<<pid;
    pid=fork();
    if(pid==0){//另一子线程执行区域
      std::cout<<"你好，我是子线程";
    	sleep(10);
    	return 24;
    }
    else{//父线程执行区域
      int i;
      std::cout<<"子线程id"<<pid;
      for(i=0;i<5;i++){
        std::cout<<"wait";
        sleep(5);
      }
    }
  }
  return 0;
}
```

## 10.4基于多任务的并发服务器

已做好了利用fork涵数编写并发服务器的准备

### 10.4.1基于进程的并发服务器模型

扩展回声服务器端 ，使其可以同时向多个客户端提供服务：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230817173025455.png" alt="image-20230817173025455" style="zoom:50%;" />

每当有客户端请求服务(连接请求)时，回声服务器端都创建子进程以提供服务。 请求服务的客户端若有5个， 则将创建5个子进程提供服务。 为了完成这些任务，需要经过如下过程：

- 回声服务器端(父进程)通过调用accept函数受理连接请求；
- 此时获取套接字文件描述符创建并传递给子进程；
- 子进程利用传递来的文件描述符提供服务；

子进程会复制父进程拥有的所有资源。实际上根本不用另外经过传递文件描述符的过程。

### 10.4.2实现并发服务器

Echo_mpserv.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <signal.h>
#include <sys/wait.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_SIZE 30;
void error_handling(char *message){
  
}
void read_childproc(int )
```



### 10.4.3通过 fork函数复制文件描述符

## 10.5分割TCP的I/O程序

## 10.6习题

# 第十一章 进程间通信

## 11.1进程间通信的基本概念

为了完成这一点，操作系统中应提供两个进程可以同时访问的内存空间。

### 11.1.1对进程间通信的基本理解

"如果我有1个面包，变量bread的值就变为1。如果吃掉这个面包，bread的值又变回0。因此，你可以通过变量bread值判断我的状态。"

基于管道 (PIPE) 的进程间通信结构模型：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230817231613315.png" alt="image-20230817231613315" style="zoom:50%;" />

管道并非属于进程的资源，而是**和套接字一样，属于操作系统(**也就不是fork函数的复制对象)。 所以，两个进程通过操作系统提供的内存空间进行通信。 下面介绍创建管道的函数：

```C++
int pipe(int filedes [2]) ;
//成功时返回0，失败时返回-1
```

filedes[0]：通过管道接收数据时使用的文件描述符 ，即管道出口 。

filedes[1]：通过管道传输数据时使用的文件描述符，即管道入口 。

父进程调用该函数时将创建管道，同时获取对应于出入口的文件描述 符， 此时父进程可以读写同一管道。但父进程的目的是与子进程进行数据交换，因此需要将入口或出口中的1个文件描述符传递给子进程。 如何完成传递呢?答案就是调用fork函数：

Pipe1.c

```C++
#include <stdio.h>
#include <unistd.h>
#define BUF_SIZE 30

int mian(int argc,char * argv[]){
  int fds[2];
  char str[]="你是谁";
  char buf[BUF_SIZE];
  pid_t pid;
  
  pipe(fds);
  pid=fork();//子进程将同时拥有通过pipe函数调用获取的2个文件描述符，注意！复制的并非管道，而是用于管道I/O的文件描述符，至此父子进程同时拥有I/O文件描述符
  if(pid==0){
    write(fds[1],str,sizeof(str));
  }
  else{
    read(fds[0],buf,BUF_SIZE);
    std::cout<<buf;
  }
  return 0;
}
```

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230817232537296.png" alt="image-20230817232537296" style="zoom:50%;" />

### 11.1.2通过管道进行进程间双向通信

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230818102610692.png" alt="image-20230818102610692" style="zoom:50%;" />

Pipe2.c

```C++
#include <stdio.h>
#include <unistd.h>
#define BUF_SIZE 30

int main(int argc,char *argv[]){
  int fds[2];
  char str1[]="你是谁";
  char str2[]="谢谢你的消息";
  char buf[BUF_SIZE];
  pid_t pid;
  
  pipe(fds);
  pid=fork();
  if(pid==0){
    write(fds[1],str1,sizeof(str1));
    sleep(2);
    read(fds[0],buf,BUF_SIZE);
    std::cout<<"子进程输出"<<buf;
  }
  else{
    read(fds[0],buf,BUF_SIZE);
    std::cout<<"父进程输出"<<buf;
    write(fds[1],str2,sizeof(str2));
    sleep(3);
  }
  return 0;
}
```

如果注释了sleep(2);将会引发错误：

"向管道传递数据时，先读的进程会把数据取走 。"数据进入管道后成为无主数据。也就是通过read函数先读取数据的进程将得到数据，即使该进程将数据传到了管道。 

"创建2个管道。"

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230818154225336.png" alt="image-20230818154225336" style="zoom:50%;" />

Pipe3.c

```C++
#include <stdio.h>
#include <unistd.h>
#define BUF_SIZE 30

int main(int argc,char *argv[]){
  int fds1[2],fds2[2];
  char str1[]="你是谁";
  char str2[]="谢谢你的消息";
  char buf[BUF_SIZE];
  pid_t pid;
  
  pipe(fds1);pipe(fds2);
  pid=fork();
  if(pid==0){
    write(fds1[1],str1,sizeof(str1));
    read(fds2[0],buf,BUF_SIZE);
    std::cout<<"子进程输出"<<buf;
  }
  else{
    read(fds1[0],buf,BUF_SIZE);
    std::cout<<"父进程输出"<<buf;
    write(fds2[1],str2,sizeof(str2));
    sleep(3);
  }
  return 0;
}
```

## 11.2运用进程间通信

## 11.3习题

# 第十二章 I/O复用

## 12.1基于I/O复用的服务器端

### 12.1.1多进程服务器端的缺点和解决方法

创建进程时需要付出极大代价。这需要大量的运算和内存空间，由于每个进程都具有独立的内存空间，所以相互间的数据交换也要求采用相对复杂的方法 (IPC 属于相对复杂的通信方法)。 

"那有何解决方案?能否在不创建进程的同时向多个客户端提供服务?"

### 12.2.2理解复用

"在1个通信频道中传递多个数据 (信号 ) 的技术 。"

### 12.2.3复用技术在服务器端的应用

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230818154725974.png" alt="image-20230818154725974" style="zoom:50%;" />

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230818154754159.png" alt="image-20230818154754159" style="zoom:50%;" />

## 12.2理解select函数并实现服务器端

select函数可以帮助程序员同时监听多个文件描述符（比如网络套接字），并在其中任意一个文件描述符上发生可读、可写或异常事件时立即进行相应的处理。这样可以让程序员更高效地处理多个客户端的请求，避免了每个客户端都需要单独开一个线程或进程来处理的问题。同时，select函数还可以阻塞进程，直到有一个文件描述符上发生了指定的事件，或者超时时间到达，这样可以避免程序的空循环等待，提高程序的效率。

### 12.2.1select 函数的功能和调用顺序

使用select函数时可以将多个文件描述符集中到一起统一监视，项目如下：

- 是否存在套接字接收数据?
- 无需阻塞传输数据的套接字有哪些?
- 哪些套接字发生了异常?

select函数的调用方法和顺序：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230818163128871.png" alt="image-20230818163128871" style="zoom:50%;" />

### 12.2.2设置文件描述符

需要将要监视的文件描述符集中到一起。 集中时也要按照监视项(接收、传输、异常)进行区分，即按照上述3种监视项分成3类。使用fd_set数组执行此项操作，该数组是存有0和1的位数组。位设置为 1，则表示该文件描述符是监视对象。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230818164723935.png" alt="image-20230818164723935" style="zoom:50%;" />

图中哪些文件描述符是监视对象呢?很明显，是文件描述符1和3。

在fd-set变量中注册或更改值的操作都由下列宏完成 ：

- FD_ZERO(fd_set *fdset) :将fd_set变量的所有位初始化为0；
- FD_SET(int fd，fd_set *fdset): 在参数fd_set指向的变量中注册文件描述符fd的信息；即fd文件描述符位变成1
- FD_CLR(int fd,fd_set *fdset): 从参数fd_set指向的变量中清除文件描述符fd的信息；
- FD_ISSET(int fd,fd_set *fdset) : 若参数fdset指向的变量中包含文件描述符fd的信息， 则返回真；即fd文件描述符是否变成了1

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230818170726212.png" alt="image-20230818170726212" style="zoom:50%;" />

### 12.2.3设置检查 (监视）范围及超时

```C++
int select(int maxfd, fd_set * readset, fd_set * writeset，fd_set * exceptset, const struct timeval * timeout);
//成功时返回大于0的值，失败时返回-1
```

maxfd：监视对象文件描述符数量；

readset：将所有关注"是否存在待读取数据"的文件描述符注册到fd_set型变量，并传递其地址值；

writeset：将所有关注"是否可传输无阻塞数据"的文件描述符注册到fd_set型变量，并传递其地址值；

exceptset：将所有关注"是否发生异常"的文件描述符注册到fd_set型变量，并传递其地址值；

timeout：调用select函数后，为防止陷入无限阻塞的状态，传递超时 ( time-out )信息；

返回值：发生错误时返回-1，超时返回时返回0。因发生关注的事件返回时，返回大于0的值，该值是发生事件的文件描述符数。

select函数用来验证3种监视项的变化情况。 根据监视项声明3个fd_set型变量，分别向其注册文件描述符信息，并把变量的地址值传递到上述函数的第二到第四个参数 。 但在此之前(调用select函数前)需要决定下面2件事：

"文件描述符的监视(检查)范围是?"

"如何设定 select函数的超时时间?"

第一 ，文件描述符的监视范围与select函数的第一个参数有关。 实际上， select函数要求通过第一个参数传递监视对象文件描述符的数量 。一般只需将最大的文件描述符值加 1再传递到select 函数即可。意味着监视范围是所有文件描述符。

第二 ， select函数的超时时间与select函数的最后一个参数有关，其中timeval结构体定义如下。

```C++
struct timeval{
  long tv_sec;//秒数
  long tv_usec;//毫秒数
}
```

本来select函数只有在监视的文件描述符发生变化时才返回。如果未发生变化，就会进入阻塞状态。指定超时时间就是为了防止这种情况的发生。如果不想设置超时，则传递NULL参数 。

### 12.2.4调用 select函数后查看结果

如果返回大于0的整数，说明相应数量的文件描述符发生变化。

**文件描述符的变化**

文件描述符变化是指监视的文件描述符中发生了相应的监视事件。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230824103453430.png" alt="image-20230824103453430" style="zoom:50%;" />

select函数调用完成后，向其传递的fd_set变量中将发生变化。 原来为1的所有位均变为0，但发生变化的文件描述符对应位除外。 因此，可以认为值仍为1的位置上的文件描述符发生了变化 。

### 12.2.5select函数调用示例

Select.c

```C++
#include <stdio.h>
#include <unistd.h>
#include <sys/time.h>
#include <sys/select.h>
#define BUF_SIZE 30

int main(int argc,char * argv[]) {
    fd_set reads, temps;
    int result, str_len;
    char buf[BUF_SIZE];
    struct timeval timeout;
    FD_ZERO(&reads);//初始化fd_set变量
    FD_SET(0, &reads);//将文件描述符0的对应位设置为1，也就是需要监视标准输入的变化，标准输入文件描述符是0

  	//为了设置select函数的超时，但不能在此时设置，因为调用了select函数后，结构体timeval的成员tv_sec和tv_usec的值将被替换成超时前剩余时间，因此掉哟个select函数前，每次都要初始化timeval变量
  	/*
  	timeout.tv_sec=5;
  	timeout.tv_usec=5000;
  	*/
  
    while (1) {
      //将准备好的fd_set变量reads内容复制到temps变量中，因为调用select函数后，除发生变化的文件描述符对应位之外，剩下位都变成0，也就是只有变化了的文件描述符对应位，其他都变成了不需要监视的了，为了记住我们设置的需要监视的文件描述符需要先复制初始值，这是调用select函数的通用方法
        temps = reads;
        timeout.tv_sec = 5;
        timeout.tv_usec = 0;
      //调用select函数，如果有控制台输入数据，则返回大于0的数，没有数据引发超时，则返回0
        result = select(1, &temps, 0, 0, &timeout);
        if (result == -1) {
            puts("select error");
            break;
        } else if (result == 0) {
            puts("Time out");
        } else {
            if (FD_ISSET(0, &temps)) {//0位是否是1，如果是表示发生对应事件了；
              //select函数返回大于0的值时运行的区域。验证发生变化的文件描述符是否为标准输入。若是，则从标准输入读取数据并向控制台输出。
                str_len = read(0, buf, BUF_SIZE);
                buf[BUF_SIZE] = 0;
                printf("message from console:%s", buf);
            }
        }
    }
}
```

### 12.2.6实现I/O复用服务器端

echo_selectserv.c

```C++
#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/time.h>
#include <sys/select.h>
#define BUF_SIZE 100
void errorHandling(char *message){

}

int main(int argc,char * argv[]) {
    int servSock, clntSock;
    struct sockaddr_in servAddr, clntAddr;
    struct timeval timeout;
    fd_set reads, copyReads;

    socklen_t adr_sz;
    int fdMax, strLen, fdNum, i;
    char buf[BUF_SIZE];
    if (argc != 2) {
        std::cout << "usage :" << argv[0];
        exit(1);
    }

    servSock = socket(PF_INET, SOCK_STREAM, 0);//获取服务器套接字
    memset(&servAddr, 0, sizeof(servAddr));
    servAddr.sin_family = AF_INET;
    servAddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servAddr.sin_port = htons(atoi(argv[1]));

    if (bind(servSock, (struct sockaddr *) &servAddr, sizeof(servAddr)) == -1) {
        errorHandling("bind error");
    }
    if (listen(servSock, 5) == -1) {
        errorHandling("listen error");
    }

    FD_ZERO(&reads);
    FD_SET(servSock, &reads);//向select函数的第二个参数reads注册服务端套接字，接受数据情况的监视对象就包含了服务器端套接字，服务器端套接字中有接受的数据意味着有连接请求，
    fdMax = servSock;

    while (1) {
        copyReads = reads;
        timeout.tv_sec = 5;
        timeout.tv_usec = 5000;
				//while无限循环调用select，
        if ((fdNum = select(fdMax + 1, &copyReads, 0, 0, &timeout)) == -1) {
            break;
        }
        if (fdNum == 0) {
            continue;
        }
				//select函数返回大于等于1时的循环，遍历所有文件描述符
        for (i = 0; i < fdMax + 1; i++) {
            if (FD_ISSET(i, &copyReads)) {//是否是发生变化了的文件描述符（即有接受数据的套接字）
              //有接受数据的套接字是否是服务器套接字，如果是则是连接请求，不是则是接受数据
                if (i == servSock) {//连接请求
                    adr_sz = sizeof(clntSock);
                    clntSock = accept(servSock, (struct sockaddr *) &clntAddr, &adr_sz);
                  //连接请求接受后要把客户端套接字注册reads
                    FD_SET(clntSock, &reads);
                    if (fdMax < clntSock)
                        fdMax = clntSock;
                    printf("connected client:%d\n", clntSock);
                } else {//读取消息
                    strLen = read(i, buf, BUF_SIZE);
                    if (strLen == 0) {//接收的数据是EOF时，关闭套接字，并在reads中删除该套接字
                        FD_CLR(i, &reads);
                        close(i);
                        printf("close client:%d\n", i);
                    } else {
                        write(i, buf, strLen);//回音
                    }
                }
            }
        }
    }
    close(servSock);
    return 0;
}
```

## 12.3基于windows实现

### 12.3.1在Windows平台调用select函数

windows平台select的第一个参数是为了保持与linux一致，但是没有任何意义

```C++
#include <winsock2.h>
int select(int nfds,fd_set *readfds,fd_set *writefds,fd_set *excpfds,const struct timeval *timeout);
//成功时返回0，失败时返回-1
```

Windows的fd_set不是像Linux中那样采用了位数组：

```C++
typedef struct fd_set{
  u_int fd_count;
  SOCKET fd_array[FD_SETSIZE];
}fd_set;
```

fd_count：套接字句柄数；fd_array：保存套接字句柄；

Linux的文件描述符从0开始递增，因此可以找出当前文件描述符数量和最后生成的文件描述符之间的关系。但Windows的套搜字句柄并 非从0开始，而且句柄的整数值之间并无规律可循，因此需要直接保存句柄的数组和记录句柄数的变量。幸好处理创set结构体的FD一XX型的4个宏的名称、 功能及使用方法与Linux完全相同 (故省略)，这也许是微软为了保证兼容性所做的考量。

### 12.3.2基于windows实现I/O复用服务器

```C++

```



## 12.4习题

1. 请解释复用技术的通用含义，并说明何为I/O复用 。
2. 多进程并发服务器的缺点有哪些? 如何在I/O复用服务器端中弥补?
3. 复用服务器端需要 select函数。下列关于select函数使用方法的描述错误的是?
4. - a. 调用select函数前需要集中I/O监视对象的文件描述符
   - b.若己通过select函数注册为监视对象，则后续调用select函数时无需重复注册
   - c. 复用服务器端同一时间只能服务于1个客户端，因此，需要服务的客户端接入服务器端后只能等待
   - d.与多进程服务器端不同，基于select的复用服务器端只需要1个进程。 因此，可以减少因创建进程产生的服务器端的负担
5. select函数的观察对象中应包含服务器端套接字(监听套接字);那么应将其包含到哪一类监听对象集合?请说明原因 。
6. select函数中使用的fdset结构体在Windows和Linux中具有不同声明 。 请说明区别，同时解释存在区别的必然性 。

# 第十三章 多种I/O函数

## 13.1send&recv函数

## 13.2readv &writev 函数

## 13.3基于windows实现

## 13.4习题

# 第十四章 多播与广播

## 14.1多播

多播( Multicast)方式的数据传输是基于UDP完成的，与udp服务器端/客户端的实现方式非常接近 ，区别在于，用多播方式时，可以同时向多个主机传递数据。

### 14.1.1多播的数据传输方式及流量方面的优点

- 多播服务器端针对特定多播组，只发送1次数据
- 即使只发送1次数据，但该组内的所有客 户端都会接收数据
- 多播组数可在IP地址范围内任意增加
- 加入特定组即可接收发往该多播组的数据

多播组是D类IP地址 (224.0.0.0--239.255.255.255 )，加入多播组可以理解为通过程序完成如下声明："在D类IP地址中，我希望接收发往目标 239.234.218.234的多播数据 。"

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230824172903652.png" alt="image-20230824172903652" style="zoom:50%;" />

### 14.1.2路由（Routing) 和TTL（Time to Live，生存时间) ，以及加入组的方法

为了传递多播数据包，必需设置TTL，TTL是决定"数据包传递距离"的主要因素。TTL用整数表示，并且每经过 1个路由器就减1，TTL变为0时，该数据包无法再被传递，只能销毁。 因此， TTL的值设置过大将影响网络流量。 当然，设置过小也会无法传递到目标。

通过第九章的套接字可选项设置，与设置TTL相关的协议层是IPPROTO_IP，可以用如下代码把TTL设置为64：

```C++
int sendSock;
int timeLive=64;
...
sendSock=socket(PF_INTE,SOCK_DGRAM,0);
setsockopt(SendSock,IPPROTO_IP,IP_MULTICAST_TTL,(void *)&timeLive,sizeof(timeLive));
```

加入多播组也通过设置套接字选项完成。 加入多播组相关的协议层为IPPROTO_IP, 选项名为 IP_ADD_MEMBERSHIP。可通过如下代码加入多播组：

```C++
int recvSock;
struct ip_mreq join_addr;
....
recvSock=socket(PF_INET,SOCK_DGRAM,0);
....
join_addr.imr_multiaddr.s_addr="多播组地址信息";
join_addr.imr_interface.s_addr="加入多播组的主机地址信息";
setsockopt(recvSock,IPPROTO_IP,IP_ADD_MEMBERSHIP,(void *)&join_addr,sizeof(join_addr));
```



## 14.2广播

## 14.3基于windows实现

## 14.4习题

# 第二部分 基于Linux的编程

# 第十八章 多线程服务端的实现

由于 Web服务器端协议本身具有的特点，经常需要同时向多个客户端提供服务。因此，人们逐渐舍弃进程，转而开始利用更高效的线程实现 Web服务器端。

## 18.1理解线程的概念

### 18.1.1引入线程的背景

第10章介绍了多进程服务器端的实现方法。多进程模型与select或epoll相比的确有自身的优点，但同时也有问题，创建进程 (复制 ) 的工作本身会给操作系统带来相当沉重的负担。而且，每个进程具有独立的内存空间，所以进程间通信的实现难度也会随之提高(参考第11 章)。 换言之，多进程模型的缺点可概括如下：

- 创建进程的过程会带来一定的开销。
- 为了完成进程间数据交换，需要特殊的IPC技术。

"每秒少则数十次、多则数千次的上下文切换(Context Switching)是创建进程时最大的开销。

只有 1个CPU (准确地说是CPU的运算设备CORE) 的系统中不是也可以同时运行多个进程吗?这是因为系统将CPU时间分成多个微小的块后分配给了多个进程。为了分时使用CPU，需要"上下文切换"过程：运行程序前需要将相应进程信息读入内存，如果运行进程A后需要紧接着运行进程B，就应该将进程A相关信息移出内存，并读入进程B相关信息。这就是上下文切换。但此时进程A的数据将被移动到硬盘，所以上下文切换需要很长时间。即使通过优化加快速度，也会存在一定的局限。 

线程( Thread )是为了将进程的各种劣势降至最低限度(不是直接消除)而设计的一种轻量级进程，线程相比于进程具有如下优点：

- 线程的创建和上下文切换比进程的创建和上下文切换更快。
- 线程间交换数据时无需特殊技术。

### 18.1.2线程和进程的差异

每个进程的内存空间都由保存全局变量的"数据区"、向 malloc等函数的动态分配提供空间的堆 (Heap)、函数运行时使用的核 (Stack) 构成。 每个进程都拥有这种独立空间，多个进程的内存结构如图所示：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230904155725114.png" alt="image-20230904155725114" style="zoom:50%;" />

但如果以获得多个代码执行流为主要目的，则不应该像图那样完全分离内存结构，而只需分离栈区域。通过这种方式可以获得如下优势：

- 上下文切换时不需要切换数据区和堆。
- 可以利用数据区和堆交换数据。

线程为了保持多条代码执行流而隔开了栈区域，因此具有如图所示的内存结构：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230904162503368.png" alt="image-20230904162503368" style="zoom:50%;" />

进程和线程可以定义为如下形式：

- 进程 : 在操作系统构成单独执行流的单位。
- 线程 : 在进程构成单独执行流的单位。

操作系统、进程、线程之间的关系可以通过图表示：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230904162922950.png" alt="image-20230904162922950" style="zoom:50%;" />

## 18.2线程创建及运行

POSIX是Portable Operating System lnterface for Computer Environment (适用于计算机环境的可移植操作系统接口)的简写，是为了提高UNIX系列操作系统间的移植性而制定的API规范。下面要介绍的线程创建方法也是以POSIX标准为依据的。因此，它不仅适用Linux，也适用于大部分UNIX系列的操作系统。

### 18.2.1线程的创建和执行流程

线程具有单独的执行流，因此需要单独定义线程的main函数，还需要请求操作系统在单独的执行流中执行该函数，完成该功能的函数如下：

```C++
int pthread_create(pthread_t * restrict thread, const pthread_attr_t * restrict attr, void * (* start_routine),(void *),void * restirct arg);
//成功时返回0，失败时返回其他值
```

thread：保存新创建线程ID的变量地址值。线程与进程相同，也需要用于区分不同线程的ID；

attr：用于传递线程属性的参数，传递 NULL时，创建默认属性的线程；

start_routine：相当于线程 main函数的、在单独执行流中执行的函数地址值(函数指针) 。

arg：通过第三个参数传递调用函数时包含传递参数信息的变量地址值。

下面通过简单示例了解该函数的功能：

Thread1.c

```C++
#include <stdio.h>
#include <pthread.h>
void * thread_main(void * arg);

int main(int argc,char *arhv[]){
    pthread_t t_id;
    int thread_param=5;
  	//请求创建一个线程，从thread_main函数调用开始，在单独的执行流中运行，同时在调用thread_main函数时向其传递thread_param变量的地址值。
    if(pthread_create(&t_id,NULL,thread_main,(void*)&thread_param)!=0){
        puts("pthread_create error");
        return -1;
    }
    sleep(100);//调用sleep函数使main函数停顿10秒，这是为了延迟进程的终止时间。
    puts("end of main");
    return 0;//终止内部创建的线程
}

void * thread_main(void * arg){//传入arg参数的是第10行pthread_create函数的第四个参数。
    int i;
    int cnt=*((int*)arg);
    for (i = 0; i < cnt; ++i) {
        sleep(1);
        puts("running thread");
    }
    return NULL;
}
```

gcc threadl.c -0 trl -lpthread

./trl

线程相关代码在编译时需要 添加一lpthread选项声明需要连接线 程库，只有这样才能调用头文件pthread.h中声明的函数。执行流程如图

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230904172718761.png" alt="image-20230904172718761" style="zoom:50%;" /> 

"那线程相关程序中必须适当调用 sleep函数!"

并非如此!通过调用sleep函数控制线程的执行相当于预测程序的执行流程，但实际上这是不可能完成的事情。而且稍有不慎，很可能干扰程序的正常执行流。例如，怎么可能在上述示例中准确预测出线程函数的运行时间，并让main函数恰好等待这么长时间呢?因此，我们不用sleep函数，而是通常利用下面的函数控制线程的执行流。通过下列函数可以更有效地解决现讨论的问题，还可同时了解线程ID的用法 。

```C++
int pthread_join(pthread_t thread, void** status);
//成功时返回0，失败时返回其他值
```

- thread：该参数值ID的线程终止后才会从该函数返回；
- status：保存线程的main函数返回值的指针变量地址值；

调用该函数的进程(或线程)将进入等待状态，直到第一个参数为ID的线程终止为止。而且可以得到线程的main函数返回值，所以该函数比较有用。下面通过示例了解该函数的功能：

Thread2.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
void * thread_main(void * arg);

int main(int argc,char *arhv[]){
    pthread_t t_id;
    int thread_param=5;
    void * thr_ret;//掌握获取线程的返回值的方法

    if(pthread_create(&t_id,NULL,thread_main,(void*)&thread_param)!=0){
        puts("pthread_create error");
        return -1;
    }
		//main函数中，针对创建的线程调用pthread_join函数，main函数将等待id保存在t_id变量中的线程终止
  //掌握获取线程的返回值的方法，thr_ret将保存(void*)msg
    if (pthread_join(t_id,&thr_ret)!=0){
        puts("pthread_join error");
        return -1;
    }

    printf("Thread return message:%s\n",(char*)thr_ret);
    free(thr_ret);
    return 0;
}

void * thread_main(void * arg){
    int i;
    int cnt=*((int*)arg);
    char * msg=(char *)malloc(sizeof(char)*50);
    strcpy(msg,"Hello,I am thread\n");
    for (i = 0; i < cnt; ++i) {
        sleep(1);
        puts("running thread");
    }
  //该返回值是内部动态分配的内存空间地址值。
    return (void*)msg;//掌握获取线程的返回值的方法
}
```

root@my_linux:/tcpip# gcc thread2.c -o tr2 -lpthread

root@my_linux:/tcpip# ./tr2

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230905095804379.png" alt="image-20230905095804379" style="zoom:50%;" />

### 18.2.2可在临界区内调用的函数

关于线程的运行需要考虑"多个线程同时调用函数时(执行时) 可能产生问题" 。 这类函数内部存在临界区 (Critical Section) 也就是说多个线程同时执行这部分代码时，可能引起问题。

根据临界区是否引起问题，函数可分为以下2类：

- 线程安全函数 (Thread-safe function )：
- 非线程安全函数( Thread-unsafe如nction)：

大多数标准函数都是线程安全的函数。我们不用自己区分线程安全 的函数和非线程安全的函数(在Windows程序中同样如此)。 因为这些平台在定义非线程安全函 数的同时，提供了具有相同功能的线程安全的函数。第8章介绍过的如下函数就不是线程安全的函数:

```C++
struct hostent * gethostbyname(const char * hostname);
```

同时提供线程安全的同一功能的函数：

```C++
struct hostent * gethostbyname_r(const char* name,struct hostent* result,char* buffer,intbuflen,int* h_errnop);
```

可以通过如下方法自动将gethostbyname函数调用改为gethostbyname_r函数调用：

"声明头文件前定义一REENTRANT宏 。"

无需为了上述宏定义特意添加#define语句，可以在编译时通过添加 -D REENTRANT选项定义宏。

root@my_linux:/tcpip# gcc -D_REENTRANT mythread.c -0 mthread -lpthread

### 18.2.3工作(Worker) 线程模型

示例将计算 1到 10的和，但并不是在main函数中进行累加运算，而是创建2个线程，其中一个线程计算1到5的和，另一个线程计算6到10的和，main函数只负责输出运算结果。这种方式的编程模型称为"工作线程( Workerthread)模型"。计算1到5之和的线程与计算6到10之和的线程将成为main线程管理的工作 (Worker)。 

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230905100831255.png" alt="image-20230905100831255" style="zoom:50%;" />

Thread3.c

```C++
#include <stdio.h>
#include <pthread.h>
void * thread_summation(void * arg);
int sum=0;

int main(int argc,char *arhv[]){
    pthread_t id_t1,id_t2;
    int range1[]={1,5};
    int range2[]={6,10};
    pthread_create(&id_t1,NULL,thread_summation,(void*)range1);
    pthread_create(&id_t2,NULL,thread_summation,(void*)range2);

    pthread_join(id_t1,NULL);
    pthread_join(id_t2,NULL);
    printf("result:%d\n",sum);
    return 0;
}

void * thread_summation(void * arg){
    int start=((int*)arg)[0];
    int end=((int*)arg)[1];

    while(start<=end){
        sum+=start;
        start++;
    }
    return NULL;
}
```

"2个线程直接访问全局变量sum!"

但示例本身存在问题。此处存在临界区相关问题，因此再介绍另一示例。该示例与上述示例相似，只是增加了发生临界区相关错误的可能性：

```C++
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>
#define NUM_THREAD 100
void * thread_inc(void * arg);
void * thread_des(void * arg);
long long num=0;//64位整型

int main(int argc,char *argv[]) {
    pthread_t thread_id[NUM_THREAD];
    int i;

    printf("sizeof long long:%d \n", sizeof(long long));
    for (i = 0; i < NUM_THREAD; ++i) {
        if (i % 2)
            pthread_create(&thread_id[i], NULL, thread_inc, NULL);
        else
            pthread_create(&thread_id[i], NULL, thread_des, NULL);
    }
    for (i = 0; i < NUM_THREAD; ++i) {
        pthread_join(thread_id[i], NULL);
    }

    printf("result : %lld \n", num);
    return 0;
}

void * thread_inc(void * arg){
    int i;
    for (i = 0; i < 5000000; ++i) {
        num+=1;
    }
    return NULL;
}

void * thread_des(void * arg) {
    int i;
    for (i = 0; i < 5000000; ++i) {
        num-=1;
    }
    return NULL;
}
```

创建了100个线程，其中一半执行inc函数中的代码，另一半则执行des函数中的代码。全局变量num经过增减过程后应存有0，

root@my_linux:/tcpip# gcc thread4.c -D_REENTRANT -0 th4 -lpthread root@my_linux:/tcpip# *./th4*

sízeof long long: result: 4284144869

运行结果并不是0! 而且每次运行的结果均不同。这对于线程的应用是个大问题。

## 18.3线程存在的问题和临界区

### 18.3.1多个线程访问同一变量问题

thread4.c的问题如下:"2个线程正在同时更改全局变量num"

任何内存空间一一只要被同时访问一一都可能发生问题。

"不是说线程会分时使用CPU吗 ? 那应该不会出现同时访问变量的情况啊。"

下面通过示例解释"同时访问"的含义，并说明为何会引起问题。 假设2个线程要执行将变量值逐次加1的工作：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230905110240701.png" alt="image-20230905110240701" style="zoom:50%;" />

在此状态下，线程1将变量num的值增加到100后，线程2再访问num时，num中将按照我们的预想保存101：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230905110343069.png" alt="image-20230905110343069" style="zoom:50%;" />

上图中需要注意值的增加方式，值的增加需要CPU运算完成，变量num中的值不会自动增加，线程1首先读该变量的值并将其传递到 CPU，获得加1之后的结果 100，最后再把结构写回变量num。

线程2的执行过程：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230905110555089.png" alt="image-20230905110555089" style="zoom:50%;" />

变量num中将保存101，但这是最理想的情况，线程1完全增加num值之前，线程2完全有可能通过切换得到CPU资源。

线程1读取变量num的值并完成加1运算时的情况，只是加1后的结果尚未写入变量num：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230905112350001.png" alt="image-20230905112350001" style="zoom:50%;" />

接下来就要将100保存到变量num中，但执行该操作前，执行流程跳转到了线程2，线程2完成了加 1运算，并将加 1之后 的结果写入变量num：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230905112441335.png" alt="image-20230905112441335" style="zoom:50%;" />

变量num的值尚未被线程1加到 100，因此线程2读到的变量num的值为99，结果是线程2将num值改成100。 还剩下线程1将运算后的值写入变量num的操作：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230905112532621.png" alt="image-20230905112532621" style="zoom:50%;" />

此时线程1将自己的运算结果100再次写入变量num， 结果变量num变成100。 虽然线程1和线程2各做了1次加1运算，却得到了意想不到的结果。 因此，线程访问变量num时应该阻止其他线程访问，直到线程1完成运算。 这就是同步 (Synchronization )。 

### 18.3.2临界区位置

“函数内同时运行多个线程时引起问题的多条语句构成的代码块。”

```C++
void * thread_inc(void * arg){
    int i;
    for (i = 0; i < 5000000; ++i) {
        num+=1;//临界区
    }
    return NULL;
}
void * thread_des(void * arg) {
    int i;
    for (i = 0; i < 5000000; ++i) {
        num-=1;//临界区
    }
    return NULL;
}
```

临界区并非num本身，而是访问 num的2条语句，这2条语句可能由多个线程同时运行，也是引起问题的直接原因。 产生的问题可以整理为如下3种情况：

- 2个线程同时执行inc函数；
- 2个线程同时执行des函数；
- 2个线程分别执行inc函数和des函数；

2条不同语句由不同线程同时执行时，也有可能构成临界区。前提是这2条语句访问同一内存空间。

## 18.4线程同步

### 18.4.1同步的两面性

线程同步用于解决线程访问顺序引发的问题。需要同步的情况可以从如下两方面考虑：

- 同时访问同一内存空间时发生的情况。
- 需要指定访问同一内存空间的线程执行顺序的情况。

第二种情况 。 这是"控制( Control )线程执行顺序" 的相关内容 。 假设有A 、 B两个线程，线程A负责向指定内存空间写入(保存)数据，线程B负责取走该数据 。 这种情况下，线程A首先应该访问约定的内存空间并保存数据 。 万一线程B先的问并取走数据，将导致错误结果 。 像这种需要控制执行顺序的情况也需要使用同步技术 。

### 18.4.2互斥量

表示不允许多个线程同时访问；

现实世界中的临界区就是洗手间 。 洗手间无法同时容纳多人(比作线程)，因此可以将临界区比喻为洗手间。洗手间使用规则如下：

- 为了保护个人隐私，进洗手间时锁上门，出来时再打开 。
- 如果有人使用洗手间，其他人需要在外面等待 。
- 等待的人数可能很多，这些人需排队进入洗手间 。 

互斥量的创建及销毁函数：

```C++
int pthread_mutex_init(pthread_mutex_t * mutex,const pthread_mutex attr_t * attr);
int pthread_mutex_destroy(pthread_mutex_t * mutex);
//成功时返回0，失败时返回其他值
```

mutex：保存互斥量的变量地址值

attr：互斥量属性，没有特别需要指定的属性时传递 NULL

利用互斥量锁住或释放临界区时使用的函数：

```C++
int pthread_mutex_lock(pthread_mutex_t * mutex);
int pthread_mutex_unlock(pthread_mutex_t * mutex);
//成功时返回0，失败时返回其他值
```

进入临界区前调用的函数就是ptbread_mutex_lock，调用该函数时，发现有其他线程已进入临界区，则lock函数不会返回，直到里面的线程调用 pthread_mutex_unlock函数退出临界区为止，其他线程让出临界区之前，当前线程将一直处于阻塞状态：

```C++
pthread_mutex_lock(&mutex);
//临界区
pthread_mutex_unlock(&mutex);
```

互斥量相当于一把锁，阻止多个线程同时访问。 还有一点需要注意，线程退出临界区时，如果忘了调用pthread_mutex_unlock函数，那么其他为了进入临界区而调用 pthread_mutex_lock函数的线程就无法摆脱阻塞状态 。 这种情况称为"死锁“ ( Dead-lock)

mutex.c

```C++
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>
#define NUM_THREAD 100
void * thread_inc(void * arg);
void * thread_des(void * arg);
long long num=0;//64位整型
pthread_mutex_t mutex;//声明互斥量

int main(int argc,char *argv[]) {
    pthread_t thread_id[NUM_THREAD];
    int i;

    pthread_mutex_init(&mutex,NULL);//初始化互斥量
    for (i = 0; i < NUM_THREAD; ++i) {
        if (i % 2)
            pthread_create(&thread_id[i], NULL, thread_inc, NULL);
        else
            pthread_create(&thread_id[i], NULL, thread_des, NULL);
    }
    for (i = 0; i < NUM_THREAD; ++i) {
        pthread_join(thread_id[i], NULL);
    }

    printf("result : %lld \n", num);
  	pthread_mutex_destroy(&mutex);//销毁互斥量
    return 0;
}

void * thread_inc(void * arg){
    int i;
    pthread_mutex_lock(&mutex);//临界区带上了for循环
    for (i = 0; i < 5000000; ++i) {
        num+=1;
    }
    pthread_mutex_unlock(&mutex);
    return NULL;
}

void * thread_des(void * arg) {
    int i;
    for (i = 0; i < 5000000; ++i) {
        pthread_mutex_lock(&mutex);//临界区只有自减
        num-=1;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}
```

确认运行结果需要等待较长时间 。 因为互斥量lock、 unlock函数的调用过程要比想象中花费更长时间。

threades函数比 threadinc函数多调用 49999999次互斥量lock、unlock函数，表现出人可以感知的速度差异。如果不太关注线程的等待时间，可以适当扩展临界区 。但num的值增加到 50000000前不允许其他线程访问，这反而成了缺点。其实这里没有正确答案。

### 18.4.3信号量

信号量与互斥量极为相似，信号量创建及销毁方法：

```C++
int sem_init(sem_t * sem, int pshared, unsigned int value);
int sem_destroy(sem_t * sem);
//成功时返回0，失败时返回其他值
```

sem：信号量的变量地址值

pshared：传递其他值时，创建可由多个进程共享的信号量 ; 传递0时，创建只允许 1个进程 内部使用的信号量。我们需要完成同一进程内的线程同步，故传递 0

value：信号量初始值

信号量中相当于互斥量lock、 unlock的函数：

```C++
int sem_post(sem_t * sem);
int sem_wait(sem_t * sem);
//成功时返回0，失败时返回其他值
```

调用sem_init函数时，操作系统将创建信号量对象， 此对象中记录着"信号量值"(Semaphore Value) 整数。该值在调用sem_post函数时增1，调用sem_wait函数时减1。但信号量的值不能小于 0，因此， 在信号量为0的情况下调用sem_wait函数时， 调用函数的线程将进入阻塞状态(因为函数未返回)。此时如果有其他线程调用sem_post函数， 信号量的值将变为1，而原本阻塞的线程可以将该信号量重新减为 0并跳出阻塞状态。实际上就是通过这种特性完成临界区的同步操作：

```C++
sem_wait(&sem);//信号量变为0
//临界区
sem_post(&sem);//信号量变为1
```

信号量的值在0和1之间跳转，因此，具有这种特性的机制称为"二进制信号量"。关于控制访问顺序的同步。 该示例的场景如下：

"线程A从用户输入得到值后存入全局变量num，此时线程B将取走该值并累加。该过程共进行5次，完成后输出总和并退出程序。“

Semaphore.c

```C++
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

void *read(void *arg);
void *accu(void *arg);
static sem_t sem_one;
static sem_t sem_two;
static int num;

int main(int argc,char *argv[]) {
    pthread_t id_t1, id_t2;//两个信号量一个是0，一个是1
    sem_init(&sem_one, 0, 0);
    sem_init(&sem_two, 0, 1);

    pthread_create(&id_t1, NULL, read, NULL);
    pthread_create(&id_t2, NULL, accu, NULL);

    pthread_join(id_t1, NULL);
    pthread_join(id_t2, NULL);

    sem_destroy(&sem_one);
    sem_destroy(&sem_two);
    return 0;
}

void *read(void *arg){
    int i;
    for (i = 0; i < 5; ++i) {
        fputs("Input num:",stdout);
        sem_wait(&sem_two);//为了防止在调用accu函数的线程还未取走数据的情况下，调用read函数的线程覆盖原值
        scanf("%d",&num);
        sem_post(&sem_one);//为了防止调用read函数的线程写入新值前，accu函数取走(再取走旧值)数据。
    }
    return NULL;
}
void *accu(void *arg){
    int sum=0,i;
    for (i = 0; i < 5; ++i) {
        sem_wait(&sem_one);//为了防止调用read函数的线程写入新值前，accu函数取走(再取走旧值)数据。
        sum+=num;
        sem_post(&sem_two);//为了防止在调用accu函数的线程还未取走数据的情况下，调用read函数的线程覆盖原值
    }
    printf("Result:%d\n",sum);
    return NULL;
}
```

sem_one=0,sem_two=1

Read:two=1-1,one=0+1

Accu:one=1-1,two=0+1

## 18.5线程的销毁和多线程并发服务器端的实现

### 18.5.1销毁线程的3种方法

Linux线程并不是在首次调用的线程main函数返回时自动销毁，所以用如下2种方法之一加以明确。 否则由线程创建的内存空间将一直存在：

- 调用pthread_join函数；
- 调用pthread_detach函数；

pthread_join：调用该函数时，不仅会等待线程终止，还会引导线程销毁。 但该函数的问题是，线程终止前，调用该函数的线程将进入阻塞状态。 因此，通常通过如下函数调用引导线程销毁：

```C++
int pthread_detach(pthread_t t hread);
//成功时返回0，失败时返回其他值
```

述函数不会引起线程终止或进入阻塞状态，可以通过该函数引导销毁线程创建的内存空间。调用该函数后不能再针对相应线程调用pthread_join函数。

### 18.5.2多线程并发服务器端的实现

多个客户端之间可以交换信息的简单的聊天程序。

chat_server.c

```C++
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>
#include <iostream>
#define BUF_SIZE 100
#define MAX_CLNT 256

void * handle_clnt(void * arg);
void send_msg(char * msg, int len);
void error_handling(char * msg);

int clnt_cnt = 0;//用于管理接入的客户端套接字的变量和数组。访问这2个变量的代码将构成临界区。
int clnt_socks[MAX_CLNT];
pthread_mutex_t mutx;//互斥量

int main(int argc,char *argv[]){
    int serv_sock,clnt_sock;
    struct sockaddr_in serv_adr,clnt_adr;
    int clnt_adr_sz;
    pthread_t t_id;
    if(argc!=2){
        printf("Usage : %s <port>\n",argv[0]);
        exit(1);
    }
    pthread_mutex_init(&mutx,NULL);
    serv_sock=socket(PF_INET,SOCK_STREAM,0);//获取服务器套接字

    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    serv_adr.sin_port=htons(atoi(argv[1]));

    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1){//服务器套接字绑定端口，协议，ip
        error_handling("bind() error");
    }
    if(listen(serv_sock,5)==-1){//服务器接口开始监听
        error_handling("listen() error");
    }
		//循环监听
    while(1){
        clnt_adr_sz=sizeof(clnt_adr);
        clnt_sock=accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);

        pthread_mutex_lock(&mutx);
        clnt_socks[clnt_cnt++]=clnt_sock;//每当有新连接时，将相关信息写入变量clnLcnt和clnLsocks
        pthread_mutex_unlock(&mutx);

        pthread_create(&t_id,NULL,handle_clnt,(void*)&clnt_sock);//创建线程向新接入的客户端提供服务。
        pthread_detach(t_id);//调用 pthread_detach函数从内存中完全销毁己终止的线程。
        printf("Connected client IP: %s \n",inet_ntoa(clnt_adr.sin_addr));
    }
    close(serv_sock);
    return 0;
}
void * handle_clnt(void * arg){
    int clnt_sock=*((int*)arg);
    int str_len=0,i;
    char msg[BUF_SIZE];
		//从clnt套接字中读取消息
    while((str_len=read(clnt_sock,msg,sizeof(msg)))!=0){
        send_msg(msg,str_len);
    }
		
    pthread_mutex_lock(&mutx);
    for(i=0;i<clnt_cnt;i++){//遍历所有clnt_sock
        if(clnt_sock==clnt_socks[i]){//找到当前的clnt_sock了
            while(i++<clnt_cnt-1){//如果当前sock的不是数组最后一个元素
                clnt_socks[i]=clnt_socks[i+1];//用当前sock的下一个sock覆盖当前sock
            }
            break;
        }
    }
    clnt_cnt--;
    pthread_mutex_unlock(&mutx);
    close(clnt_sock);//关闭当前clntsock
    return NULL;
}
void send_msg(char * msg,int len){//该函数负责向所有连接的客户端发送消息。
    int i;
    pthread_mutex_lock(&mutx);
    for(i=0;i<clnt_cnt;i++){
        write(clnt_socks[i],msg,len);
    }
    pthread_mutex_unlock(&mutx);
}
void error_handling(char * msg){
    fputs(msg,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

 添加或删除客户端时 ， 变量clnt_cnt和数组clnt_socks同时发生变化 。 因此，在如下情形中均会导致数据不一致，从而引发严重错误：

- 线程A从数组clnt_socks中删除套接字信息，同时线程B读取clnt_cnt变量；
- 线程A读取变量clnt_cnt，同时线程B将套接字信息添加至clnt_socks数组;

Chat_clnt.c

```C++
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>
#include <iostream>
#define BUF_SIZE 100
#define NAME_SIZE 20

void * send_msg(void * arg);
void * recv_msg(void * arg);
void error_handling(char * msg);

char name[NAME_SIZE]="[DEFAULT]";
char msg[BUF_SIZE];

int main(int argc,char *argv[]){
    int sock;
    struct sockaddr_in serv_addr;
    pthread_t snd_thread,rcv_thread;
    void * thread_return;
    if(argc!=4){
        printf("Usage : %s <IP> <port> <name>\n",argv[0]);
        exit(1);
    }

    sprintf(name,"[%s]",argv[3]);//客户端名字
    sock=socket(PF_INET,SOCK_STREAM,0);//获取客户端sock

    memset(&serv_addr,0,sizeof(serv_addr));
    serv_addr.sin_family=AF_INET;
    serv_addr.sin_addr.s_addr=inet_addr(argv[1]);//IP地址
    serv_addr.sin_port=htons(atoi(argv[2]));//端口号

    if(connect(sock,(struct sockaddr*)&serv_addr,sizeof(serv_addr))==-1){
        error_handling("connect() error");
    }

    pthread_create(&snd_thread,NULL,send_msg,(void*)&sock);
    pthread_create(&rcv_thread,NULL,recv_msg,(void*)&sock);
    pthread_join(snd_thread,&thread_return);
    pthread_join(rcv_thread,&thread_return);
    close(sock);
    return 0;
}
void * send_msg(void * arg){//发送线程
    int sock=*((int*)arg);
    char name_msg[NAME_SIZE+BUF_SIZE];
    while(1){
        fgets(msg,BUF_SIZE,stdin);
        if(!strcmp(msg,"q\n")||!strcmp(msg,"Q\n")){
            close(sock);
            exit(0);
        }
        sprintf(name_msg,"%s %s",name,msg);
        write(sock,name_msg,strlen(name_msg));
    }
    return NULL;
}
void * recv_msg(void * arg){//接收线程
    int sock=*((int*)arg);
    char name_msg[NAME_SIZE+BUF_SIZE];
    int str_len;
    while(1){
        str_len=read(sock,name_msg,NAME_SIZE+BUF_SIZE-1);
        if(str_len==-1){
            return (void*)-1;
        }
        name_msg[str_len]=0;
        fputs(name_msg,stdout);
    }
    return NULL;
}
void error_handling(char * msg){
    fputs(msg,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

## 18.6习题

- 单CPU系统中如何同时执行多个进程?请解释该过程中发生的上下文切换。
- 为何线程的上下文切换速度相对更?线程间数据交换为何不需要类似IPC的特别技术?
- 请从执行流角度说明进程和线程的区别。
- 下列关于临界区的说法错误的是?
- - a. 临界区是多个线程同时访问时发生问题的区域。
  - b. 线程安全的函数中不存在临界区，即便多个线程同时调用也不会发生问题。
  - C. 1个临界区只能由1个代码块，而非多个代码块构成。换言之，线程A执行的代码块A和线程B执行的代码块B之间绝对不会构成临界区。
  - d.临界区由访问全局变量的代码构成。其他变量中不会发生问题。
- 下列关于线程同步的描述错误的是?
- - a. 线程同步就是限制访问临界区。
  - b. 线程同步也具有控制线程执行顺序的含义。
  - C. 互斥量和信号量是典型的同步技术。
  - d. 线程同步是代替进程IPC的技术。
- 请说明完全销毁Linux线程的2种方法。
- 请利用多线程技术实现回声服务器端，但要让所有线程共享保存客户端消息的内存空间( char数组 )。这么做只是为了应用本章的同步技术其实不符合常理。
- 上一题要求所有线程共享保存回声消息的内存空间，如果采用这种方式，无论是否同步都会产生问题。请说明每种情况各产生哪些问题。

# 第三部分 基于windows的编程

# 第十九章 Windows平台下线程的使用

## 19.1内核对象

### 19.1.1内核对象的定义

操作系统创建的资源 (Resource) 有很多种，如进程、线程、文件及即将介绍的信号量、互斥量等。 其中大部分都是通过程序员的请求创建的，而且请求方式(请求使用的函数)各不相同。但它们之间也有如下共同点：

"都是由Windows操作系统创建并管理的资源。"

不同资源类型在"管理"方式上也有差异。 例如，文件管理中应注册并更新文件相关的数据I/O位置、文件的打开模式( read or write )等。如果是线程，则应注册并维护线程ID、线程所属进程等信息。

操作系统为了**以记录相关信息的方式**管理各种资源，在其内部生成数据块 (亦可视为结构体变量)。当然，每种资源需要维护的信息不同，所以每种资源拥有的数据块格式也有差异。这类数据块称为"内核对象"。

假设在 Windows下创建了mydata.txt文件，此时 Windows操作系统将生成 1个数据块以便管理，该数据块就是内核对象。同理， Windows在创建进程、 线程、 线程同步信号量时也会生成相应的内核对象，用于管理操作系统资源。 

### 19.1.2内核对象归操作系统所有

线程 、 文件等资源的创建请求均在进程内部完成，因此，很容易产生"此时创建的内核对象所有者就是进程"的错觉。其实，内核对象所有者是内核(操作系统 )。"所有者是内核"具有如下含义 :

"内核对象的创建、管理 、 销毁时机的决定等工作均由操作系统完成!"

内核对象就是为了管理线程 、 文件等资源而由操作系统创建的数据块，其创建者和所有者均 为操作系统 。

## 19.2基于Windows的线程创建

### 19.2.1进程和线程的关系

"程序开始运行后调用main函数的主体是进程还是线程?"

调用main函数的主体是线程！实际上，过去的正确答案可能是进程(特别是在UNIX系列的操作系统中)。 因为早期的操作系统并不支持线程?为了创建线程，经常需要特殊的库函数支持。

非显式创建线程的程序 (如基于select的服务器端)可描述如下:

"单一线程模型的应用程序"

反之，显式创建单独线程的程序可描述如下 :

"多线程模型的应用程序"

这就意味着main函数的运行同样基于线程完成，此时进程可以比喻为装有线程的篮子。实际的运行主体是线程。

### 19.2.2Windows 中线程的创建方法

该函数将创建线程，操作系统为了管理这些资源也将同时创建内核对象。最后返回用于区分内核对象的整数型"句柄"(Handle)：

```C++
HANDLE CreateThread(
  LPSECURITY_ATTRIBUTES lpThreadAttributes,
  SIZE_T dwStackSize,
  LPTHREAD_START_ROUTINE plStartAddress,
  LPVOID lpParameter,
  DWORD dwCreationFlags,
  LPDWORD lpThreadId
)
  //成功时返回线程句柄，失败时返回NULL
```

lpThreadAttributes：线程相关安全信息，默认设置使用NULL

dwStackSize：要分配给线程的栈大小，传递0时生成默认大小的栈

plStartAddress：传递线程的main函数信息

lpParameter：调用main函数时传递的参数信息

dwCreationFlags：用于指定线程创建后的行为，传递0时，线程创建后立即进入可执行状态

lpThreadId：用于保存线程ID的变量地址值

只需要考虑 lpStartAddress和lpParameter这2个参数，剩下的只需传递0或NULL即可。

**Windows线程的销毁时间点**

Windows线程在首次调用的main函数返回时销毁（销毁时间点和销毁方法与Linux不同）

### 19.2.3编写多线程程序的环境设置

VC++环境下需要设置"C/C++RuntimeLibrary"(以下简称CRT)，这是调用C/C++标准函数时必需的库。过去的VC++6.0版默认只包含支持单线程的(只能在单线程模型下正常工作的)库，需要自行配置。 但各位使用的VC++Express Edîtion 2005或更高版本中只有支持多线程的库，不用另行设置环境。 即使如此，还是通过图给出CRT的指定位置。在菜单中选择"项目"→"属性"，或使用快捷键A1+F7打开库的配置页，即可看到相关界面。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230825151243883.png" alt="image-20230825151243883" style="zoom:50%;" />

### 19.2.4创建"使用线程安全标准 C 函数 " 的线程

如果线程要调用C/C++标准函数，需要通过如下方法创建线程。因为通过CreateThread函数调用创建出的线程在使用C/C++标准函数时并不稳定：

```C++
uintptr_t _beginthreadex(
	void *security,
  unsigned stack_size,
  unsigned (*start_address)(void *),
  void *arglist,
  unsigned initflag,
  unsigned *thrdaddr
);
//成功时返回线程句柄，失败时返回0
```

参数个数及各参数的含义和顺序均相同，只是变量名和参数类型有所不同。因此，用上述函数替换CreateThread函数时，只需适当更改数据类型。下面通过上述函数编写示例。

**定义于_beginthreadex函数之前的beginthread函数**

该函数的问题在于，它会让创建线程时返回的句柄失效，以防止访问内核对象。_beginthreadex就是为了解决这一问题而定义的函数。

下述示例将通过声明CreateThread 函数的返回值类型HANDLE (这同样是整数型)保存返回的线程句柄：

Thread1_win.c

```C++
#include <windows.h>
#include <stdio.h>
#include <process.h>
unsigned WINAPI ThreadFunc(void *arg);

int main(int argc,char * argv[]) {
    HANDLE hThread;
    unsigned threadID;
    int param = 5;

    //将ThreadFunc作为线程的main函数，并向其传递变量param的地址，同时请求创建线程
    hThread = (HANDLE) _beginthreadex(NULL, 0, ThreadFunc, (void *) &param, 0, &threadID);
    if (hThread == 0) {
        puts("_beginthreadex error");
        return -1;
    }
    Sleep(3000);//Sleep函数以1/1000秒为单位进入阻塞状态，即3秒钟
    puts("end of main");
    return 0;
}

//WINAPI是Windows固有的关键字，它用于指定参数传递方向、分配的栈返回方式，
//等函数调用相关规定。插入它是为了遵守_beginthreadex函数要求的调用规定。
unsigned WINAPI ThreadFunc(void *arg) {
    int i;
    int cnt=*((int *)arg);
    for (i = 0; i < ; ++i) {
        Sleep(1000);
        puts("running thread");
    }
    return 0;
}
```

与Linux相同， Windows同样在main函数返回后终止进程，也同时终止其中包含的所有线程。 因此，需要特殊方法解决该问题。

**句柄、内核对象和ID之间的关系**

线程也属于操作系统管理的资源，会创建内核对象，并为了引用内核对象创建句柄，可以利用句柄发出请求：“我会一直等到该句柄指向的线程终止”。句柄区分内核对象，内核对象区分线程，所以句柄是区分线程的主要依据，句柄和ID有如下显著特点：“句柄的整数值在不同的进程中可能重复出现，但线程ID在夸进程范围内不会出现重复”。线程ID用于区分操作系统创建的所有进程，但通常没有这种需求。

## 19.3内核对象的2种状态

资源类型不同，内核对象也含有不同信息，其中实现过程中需要特别关心信息被赋予某种状态，例如，线程内核对象中需要重点关注线程是否终止，终止状态：signaled状态，未终止状态：unsignaled状态

### 19.3.1内核对象的状态及状态查看

已发信号状态，未发信号状态

"该进程何时终止?"或"该线程何时终止?"操作系统将这些重要信息保存到内核对象，同时给出如下约定 :

**"进程或线程终止时，我会把相应的内核对象改为signaled状态!"**

进程和线程的内核对象初始状态是non_signaled状态。

内核对象带有1个boolean变量，其初始值为FALSE，此时的状态就是non-signaled状态。如果发生约定的情况("发生了事件"，以下会酌情使用这种表达)，把该变量改为TRUE，此时的状态就是signaled状态。

### 19.3.2WaitForSingleObject&WaitForMultipleObjects

该函数针对单个内核对象验证signaled状态：

```C++
DWORD WaitForSingleObject(HANDLE hHandle,DOWRD dwMilliseconds);
//成功时返回事件信息，失败时返回WAIT_FAILED
```

hHandle：查看状态的内核对象句柄，**可以是互斥量、事件、信号量、线程、进程等**

dwMilliseconds：以1/1000秒为单位指定超时，传递INFINITE时不会返回，直到内核对象变成signaled状态

返回值：进入signaled状态返回WAIT_OBJECT_0，超时返回WAIT_TIMEOUT

该函数由于发生事件（变成signaled状态）返回时，有时会把相应内核对象再次改成non-signaled状态，这种可以再次进入non-signaled状态的内核对象称为“auto-reset模式”的内核对象，而不会自动跳转到non-signaled状态的内核对象称为“manual-reset模式”的内核对象。

该函数可以验证多个内核对象状态：

```C++
DWORD WaitForMultipleObjects(DWORD nCount,const HANDLE * lpHandles,BOOL bWaitAll,DWORD dwMilliseconds);
//成功时返回事件信息，失败时返回WAIT_FILED
```

nCount：需要验证的内核数

lpHandles：存有内核对象句柄的数组地址值

bWaitAll：如果为TRUE，则所有内核对象全部变成signaled时返回，如果为False，只要有一个signaled时就返回

dwMilliseconds：超时

利用WaitForSingleObject函数尝试解决threadl-win-c的问题：

```C++
#include <stdio.h>
#include <windows.h>
#include <process.h>
unsigned WINAPI ThreadFunc(void *arg);

int main(int argc, char *argv[])
{
    HANDLE hThread;
    DWORD wr;
    unsigned threadID;
    int param = 5;

    hThread = (HANDLE)_beginthreadex(NULL, 0, ThreadFunc, (void*)&param, 0, &threadID);
    if (hThread == 0)
    {
        puts("_beginthreadex() error");
        return -1;
    }
		//等待线程终止，
    if(wr=WaitForSingleObject(hThread, INFINITE)==WAIT_FAILED)
    {
        puts("thread wait error");
        return -1;
    }
		//查看线程返回原因
    printf("wait result: %s\n", (wr==WAIT_OBJECT_0)?"signaled":"time-out");
    puts("end of main");
    return 0;
}

unsigned WINAPI ThreadFunc(void *arg)
{
    int i;
    int cnt = *((int*)arg);
    for (i = 0; i < cnt; i++)
    {
        Sleep(1000);
        puts("running thread");
    }
    return 0;
}
```

下列示例将涉及WaitForMultipleObjects 函数的应用：

```C++
#include <stdio.h>
#include <windows.h>
#include <process.h>
#define NUM_THREAD 50
unsigned WINAPI threadInc(void *arg);
unsigned WINAPI threadDes(void *arg);
long long num = 0;

int main(int argc, char *argv[])
{
    HANDLE tHandles[NUM_THREAD];
    int i;

    printf("sizeof long long: %d \n", sizeof(long long));
    for (i = 0; i < NUM_THREAD; i++)
    {
        if (i % 2)
            tHandles[i] = (HANDLE)_beginthreadex(NULL, 0, threadInc, NULL, 0, NULL);
        else
            tHandles[i] = (HANDLE)_beginthreadex(NULL, 0, threadDes, NULL, 0, NULL);
    }

    waitForMultipleObjects(tHandles, NUM_THREAD);
    printf("result: %lld \n", num);
    return 0;
}

unsigned WINAPI threadInc(void *arg)
{
    int i;
    for (i = 0; i < 50000000; i++)
        num += 1;
    return 0;
}

unsigned WINAPI threadDes(void *arg)
{
    int i;
    for (i = 0; i < 50000000; i++)
        num -= 1;
    return 0;
}
```

即使多运行几次也无法得到正确结果，而且每次结果都不同。可以利用第20章的同步技术得到预想的结果。

## 19.4习题

- 下列关于内核对象的说法错误的是?
  - a. 内核对象是操作系统保存各种资源信息的数据块
  - b. 内核对象的所有者是创建该内核对象的进程
  - c. 由用户进程创建并管理内核对象
  - d. 无论操作系统创建和管理的资源类型是什么，内核对象的数据块结构都完全相同
- 现代操作系统大部分都在操作系统级别支持线程。根据该情况判断下列描述中错误的是?
  - a. 调用main函数的也是线程
  - b. 如果进程不创建线程，则进程内不存在任何线程
  - c. 多线程模型是进程内可以创建额外线程的程序类型
  - d. 单一线程模型是进程内只额外创建1个线程的程序模型
- 请比较从内存中完全销毁 Windows线程和 Linux线程的方法。
- 通过线程创建过程解释内核对象、线程 、 句柄之间的关系。
- 判断下列关于内核对象描述的正误。
  - 内核对象只有 signaled和non也gnaled这2种状态 。 ( )
  - 内核对象需要转为 signaled状态时，需要程序员亲自将内核对象的状态改为 signaled状态。()
  - 线程的内核对象在线程运行时处于 signaled状态事线程终止则进入 non-signaled状态。()
- 请解释"auto-reset模式"和 "manual-reset模式"的内核对象。 区分二者的内核对象特征是什么?

# 第二十章 windows中的线程同步

## 20.1同步方法的分类及CRITICAL_SECTION同步

### 20.1.1用户模式(Usermode) 和内核模式 (Kernalmode)

Windows操作系统的运行方式(程序运行方式)是"双模式操作" (Dual-mode Operation ), 这意味着 Windows在运行过程中存在如下2种模式：

- 用户模式:运行应用程序的基本模式，禁止访问物理设备，而且会限制访问的内存区域。
- 内核模式:操作系统运行时的模式，不仅不会限制访问的内存区域，而且访问的硬件设备也不会受限 。

内核是操作系统的核心模块，可以简单定义为如下形式：

- 用户模式：应用程序的运行模式 
- 内核模式 : 操作系统的运行模式

实际上，在应用程序运行过程中， Windows操作系统不会一直停留在用户模式，而是在用户模式和内核模式之间切换。例如，各位可以在Windows中创建线程。 虽然创建线程的请求是由应用程序的函数调用完成，但**实际创建线程的是操作系统**。因此，创建线程的过程中无法避免向内核模式的转换。

定义这2种模式主要是为了提高安全性。应用程序的运行时错误会破坏操作系统及各种资源。特别是C/C++可以进行指针运算，很容易发生这类问题：例如，因为错误的指针运算覆盖了操作系统中存有重要数据的内存区域，这很可能引起操作系统崩溃 。 但实际上各位从未经历过这类事件，因为用户模式会保护与操作系统有关的内存区域。 因此、即使遇到错误的指针运算也仅停止应用程序的运行，而不会影响操作系统。总之，像线程这种伴随着内核对象创建的资源创建过程中，都要默认经历如下模式转换过程 :

用户模式→内核模式→用户模式

从用户模式切换到内核模式是为了创建资源，从内核模式再次切换到用户模式是为了执行应用程序的剩余部分。不仅是资源的创建，与内核对象有关的所有事务都在内核模式下进行。模式切换对系统而言其实也是一种负担，频繁的模式切换会影响性能。

### 20.1.2用户模式同步

用户模式同步是用户模式下进行的同步，即无需操作系统的帮助而在应用程序级别进行的同步。用户模式同步的最大优点是连度快。无需切换到内核模式，仅考虑这一点也比经历内核模式切换的其他方法要快。而且使用方法相对简单，因此，适当运用用户模式同步并无坏处。 但因为这种同步方法不会借助操作系统的力量，其功能上存在一定局限性。

### 20.1.3内核模式同步

内核模式同步的优点：

- 比用户模式同步提供的功能更多
- 可以指定超时防止产生死锁 

特别是在内核模式同步中，可以跨越进程进行线程同步。与此同时，由于无法避免用户模式和内核模式之间的切换，所以性能上会受到一定影响。

**知识补给站死锁**

请考虑如下情况 。 有人进入洗手间后把门锁上，一会儿又来人开始在门外等待。 本来里面的人应该先解锁再离开，但现在不知何种原因，里面的人通过小窗户爬出了洗手间。当然谁都不得而知，那在外面等的人该怎么办呢?本来应该询问里面的情况并决定是否去其他洗手间，但此人一直在等待，直到洗手间被强行关闭(程序强行退出)。

虽然多少有些荒诞，但这种情况描述的就是死锁。发生死锁时，等待进入临界区的阻塞状态的线程无法退出。死锁发生的原因有很多，以第18章的Mutex为例，调用pthread_mutex_lock函数进入临界区的线程如采不调用 pthread_mutex_unlock函数，将发生死锁。 

### 20.1.4基于CRITICAL_SECTION（临界区域）的同步，用户模式

基于CRITICAL_SECTION的同步中将创建并运用 "CRITICAL_SECTION对象"，但这并非内核对象。与其他同步对象相同，它是进入临界区的一把"钥匙" (Key )，为了进入临界区，需要得到CRITICAL_SECTION对象这把钥匙，离开时应上交CRITICAL_SECTION对象 (以下简称CS)：

```C++
void InìtializeCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
void DeleteCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
```

lpCriticalSection：CRITICAL_SECTION对象的地址值

LPCRITICAL_SECTION是CRITICAL_SECTION的指针类型，DeleteCriticalSection并不是销毁CRITICAL_SECTION对象的函数，该函数的作用是销毁CRITICAL_SECTION对象使用过的资源。

获取(拥有)及释放cs对象的函数，可以简单理解为获取和释放"钥匙"的函数：

```C++
void EnterCriticalSection(LPCRITICAL_SECTION lpCriticalSection};
void LeaveCriticalSection(LPCRITICAL_SECTION lpCriticalSection);
```

lpCriticalSection：获取(拥有)和释放的CRITICAL_SECTION对象的地址值。

将第19章的示例threac_win.c改为同步程序：

SyncCS_win.c

```C++
#include <stdio.h>
#include <windows.h>
#include <process.h>

#define MAX_THREAD 50
unsigned WINAPI threadInc(void *arg);
unsigned WINAPI threadDes(void *arg);

long long num = 0;
CRITICAL_SECTION cs;

int main(int argc, char *argv[])
{
    HANDLE tHandles[MAX_THREAD];
    int i;

    InitializeCriticalSection(&cs);
    for (i = 0; i < MAX_THREAD; i++)
    {
        if (i % 2)
            tHandles[i] = (HANDLE)_beginthreadex(NULL, 0, threadInc, NULL, 0, NULL);
        else
            tHandles[i] = (HANDLE)_beginthreadex(NULL, 0, threadDes, NULL, 0, NULL);
    }

    WaitForMultipleObjects(MAX_THREAD, tHandles, TRUE, INFINITE);
    DeleteCriticalSection(&cs);
    printf("result: %lld \n", num);
    return 0;
}
unsigned WINAPI threadInc(void *arg)
{
    int i;
    EnterCriticalSection(&cs);
    for (i = 0; i < 50000000; i++)
        num += 1;
    LeaveCriticalSection(&cs);
    return 0;
}
unsigned WINAPI threadDes(void *arg)
{
    int i;
    EnterCriticalSection(&cs);
    for (i = 0; i < 50000000; i++)
        num -= 1;
    LeaveCriticalSection(&cs);
    return 0;
}
```

程序中将整个循环纳入临界区，主要是为了减少运行时间。如果只将访问num的语句纳入临界区，那将不知何时才能得到运行结果。

## 20.2内核模式的同步方法

典型的内核模式同步方法有基于事件 (Event)、信号量、互斥量等内核对象的同步：

### 20.2.1基于互斥量 (MutualExclusion) 对象的同步

与基于cs对象的同步方法类似，因此，互斥量对象同样可以理解为"钥匙" ：

```C++
HANDLE CreateMutex(LPSECURITV_ATIRIBUTES lpMutexAttributes, BOOL bInitialOWner, LPCTSTR lpName);
//成功时返回创建的互斥量对象句柄，失败时返回NULL
```

lpMutexAttributes：安全相关的配置信息，使用默认安全设置时可以传递 NULL。

bInitialOWner：为TRUE，则创建出的互斥量对象属于调用该函数的线程，同时进入non-signaled状态; 如果为 FALSE，则创建出的互斥量对象不属于任何线程，此时状态为signaled。

lpName：用于命名互斥量对象。 传入NULL时创建无名的互斥量对象。

如果互斥量对象不属于任何拥有者，则将进入signaled状态。利用该特点进行同步。另外，互斥量属于内核对象，所以通过如下函数销毁：

```C++
BOOL CloseHandle(HANDLE hObject);
//成功时返回true，失败时返回false
```

hObject：要销毁的内核对象的句柄

同样可以销毁即将介绍的信号量及事件

获取和释放互斥量的函数，但我认为只需介绍释放的函数，因为获取是通过各位熟悉的WaitForSingleObject函数完成的：

获取互斥量和释放互斥量就是上锁与解锁

```C++
BOOL ReleaseMutex(HANDLE hMutex);
//成功时返回true，失败时返回false
```

hMutex：需要释放(解除拥有 ) 的互斥量对象句柄 

互斥量被某一线程获取时(拥有时)为non-signaled状态，释放时(未拥有时)进入signaled状态 。 因此，可以使用WaitForSingleObject函数验证互斥量是否已分配。 该函数的调用结果有如下2种：

- 调用后进入阻塞状态:互斥量对象已被其他线程获取，现处于 non-signaled状态
- 调用后直接返回:其他线程未占用互斥量对象，现处于 signaled状态

互斥量在WaitForSingleObject函数返回时自动进入non-signaled状态，因为它是第 19章介绍过的 "auto-reset" 模式的内核对象。 结果， WaitForSingleObject函数成为申请互斥量时调用的函数。 因此，基于互斥量的临界区保护代码如下：

```C++
WaitForSingleObject(hMutex, INFINITE);
//临界区
ReleaseMutex(hMutex);
```

WaitForSingleObject函数使互斥量进入non-signaled状态，限制访问临界区，所以相当于临界 区的门禁系统。相反， ReleaseMutex函数使互斥量重新进入signal状态，所以相当于临界区的出口 。下面将之前介绍过的SyncCS_win.c示例改为互斥量对象的实现方式：



### 20.2.2基于信号量对象的同步

### 20.2.3基于事件对象的同步

## 20.3Windows 平台下实现多线程服务器端

## 20.4习题

# 第二十一章 异步通知I/O模型

## 21.1理解异步通知I/O模型

### 21.1.1理解同步和异步

Windows示例中主要通过send& recv函数进行同步I/O，

调用 send函数的瞬间开始传输数据，send函数执行完(返回)的时刻完成数据传输；

调用recv函数的瞬间开始接收数据，recv函数执行完 ( 返回)的时刻完成数据接收；

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230810154155260.png" alt="image-20230810154155260" style="zoom:50%;" />

异步I/O：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230810154220511.png" alt="image-20230810154220511" style="zoom:50%;" />

1. **同步 I/O**：就像你去餐馆吃饭，点完菜后。你得等着，直到每道菜都准备好，然后才能继续吃下一道菜。在计算机中，这就像程序发送一个 I/O 请求，然后程序会等待，直到数据准备好或操作完成，然后再继续执行后续操作。这就像你等待食物上菜一样，会浪费一些时间，但你能确保每道菜都在你的面前。
2. **异步 I/O**：相反，就像你点了菜后，可以离开餐馆，去做其他事情。然后，餐厅会通知你，一道一道的菜准备好了，你可以回去享用。在计算机中，这就像程序发送一个 I/O 请求，然后继续执行其他任务。当数据准备好或操作完成时，程序会得到通知，然后再处理数据。这样可以节省时间，因为程序不必一直等待，可以继续执行其他任务。

### 21.1.2同步I/O的缺点及异步方式的解决方案

从图 21中很容易找到同步I/O的缺点:"进行I/O的过程中函数无法返回，所以不能执行其他任务!"而图21-2 中 ， 无论数据是否完成交换都返回函数，这就意味着可以执行其他任务 。 所以说"异步方式能够比同步方式更有效地使用 cpu" 。

### 21.1.3理解异步通知I/O模型

通知I/O的含义：通知输入缓冲收到数据并需要读取，以及输出缓冲为空故可以发送数据。

"通知I/O"是指发生了I/O相关的特定情况，典型的通知I/O模型是select方式，select监视的3种情况其中具代表性的就是"收到数据的情况"，select函数就是从返回调用的函数时通知需要I/O处理的，或可以进行I/O处理的情况，这种通知是以同步方式进行的，原因在于，**需要I/O或可以进行I/O的时间点(简言之就是I/O相关事件发生的时间点)与select函数的返回时间点一致。**

**也就是函数需要等在这里直到select函数返回，则说明需要进行IO了**

通知I/O模型的含义，与 "select函数只在需要或可以进行I/O的情况下返回" 不同，异步通知I/O模型中函数的返回与I/O状态无关，WSAEventSelect函数就是select函数的异步版本。

"既然函数的返回与I/O状态无关，那是否需要监视I/O状态变化?"

需要!异步通知I/O中，**指定I/O监视对象的函数和实际验证状态变化的函数是相互分离的**。 因此，指定监视对象后可以离开执行其他任务，最后再回来验证状态变化。 

**异步I/O不需要一直监视着对象，需要验证的时候验证一下是否需要IO**

**知识补给站**

select函数也可以设置超时时间，设置超时时间可以在未发生I/O状态变化的情况下防止函数阻塞，所以可编 写类似异步方式的代码。但之后为了验证I/O的状态变化需要再次集中句柄(文件描述符)，以便再次调用 select 函数。

## 21.2理解和实现异步通知I/O模型

实现方法有2种: 一种是使用本书介绍的WSAEventSelect函数，另一种是使用WSAAsyncSelect函数。 WSA：Windows Sockets API

### 21.2.1WSAEventSelect函数和通知

告知IO状态变化的操作就是"通知"，IO的状态变化可以分为不同情况：

- 套接字的状态变化:套接字的IO状态变化
- 发生套接字相关事件:发生套接字IO相关事件

WSAEventSelect用于指定某一套接字为事件监视对象：

```C++
int WSAEventSelect(SOCKET s, WSAEVENT hEventObject, long lNetworkEvents);
//成功时返回0，失败返回SOCKET_ERROR
```

s：监视的套接字句柄

hEventObject：传递事件对象句柄以验证事件发生与否，将事件对象和套接字绑定，通过查看事件对象来确定该套接字是否发生了某事件

lNetworkEvents：希望监视的事件类型信息

s的套接宇内只要发生lNetworkEvents中指定的事件之一，WSAEventSelect函数就将hEventObject句柄所指内核对象改为 signaled状态，该函数又称**"连接事件对象和套接字的函数”**。

无论事件发生与否，WSAEventSelect函数调用后都会直接返回，所以可以执行其他任务。也就是说，该函数以异步通知方式工作。

该函数第三个参数的事件类型信息，可以通过位或运算同时指定多个信息：

- FD _READ：是否存在需要接收的数据；
- FD _WRITE：能否以非阻塞方式传输数据；
- FD _OOB：能否收到带外数据；
- FD_ACCEPT: 是否有新的连接请求；
- FD_CLOSE是否有断开连接的请求；

“select函数可以针对多个套接字对象调用，但WSAEventSelect函数只能针对 1 个套接字对象调用？”

select函数返回时，为了验证事件是否再次发生，需要再次针对所有句柄调用函数，但通过WSAEventSelect函数传递的套接字信息已经注册到操作系统，则无需再次操作。

需要补充如下内容：

- WSAEventSelect函数的第二个参数中用到的事件对象的创建方法；
- 调用WSAEventSelect函数后发生事件的验证方法；
- 验证事件发生后事件类型的查看方法；

### 21.2.2manual-reset模式事件对象的其他创建方法

CreateEvent函数在创建事件对象时，可以在auto-reset模式和 manual-reset模式中任选其一。但我们只需要 manual-reset模式non-signaled状态的事件对象，所以利用如下函数创建较为方便：

```C++
WSAEVENT WSACreateEvent(void);
//成功时返回事件对象句柄，失败时返回 WSA_INVALID_EVENT
```

返回类型 WSAEVENT的定义如下 :

```C++
#define WSAEVENT HANDLE
```

实际上就是内核对象句柄，为了销毁通过上述函数创建的事件对象，系统提供了如下函数：

```C++
BOOL WSACloseEvent(WSAEVENT hEvent);
//成功时返回 TRUE，失败时返回 FALSE
```

### 21.2.3验证是否发生事件

查看事件对象。 完成该任务的函数如下，除了多 1个参数外，其余部分与 WaitForMultipleObjects函数完全相同：

```C++
DWORD WSAWaitForMultipleEvents(DWORD cEvents, const WSAEVENT * lphEvents, BOOL fWaitAll, DWORD dwTimeout,BOOL
fAlertable);
//成功时返回发生事件的对象信息；失败时返回 WSA_INVALID_EVENT
```

cEvents：需要验证是否转为 signaled状态的事件对象的个数 

lphEvents：存有事件对象句柄的数组地址值

fWaitAll：传递TRUE时 ，所有事件对象在 signaled状态时返回 ; 传递 FALSE时，只要其中 1个变为 signaled状态就返回 。

dwTimeout：以 1/1000秒为单位指定超时，传递WSA_INFINITE时，直到变为 signaled状态时才会返回 。

fAlertable：传递TRUE时进入alertable wait (可警告等待)状态(第22章)

返回值：返回值减去常量WSA_WAlLEVENT_0时，可以得到转变为signaled状态的事件对象句柄对应的索引，可以通过该索引在第二个参数指定的数组中查找句柄。如果有多个事件对象变为 signaled状态，则会得到其中较小的值。发生超时将返回WAIT_TIMEOUT。

“WSAWaitForMultipleEvents函数如何得到转为signaled状态的所有事件对象句柄的信息?"

只通过1次函数调用无法得到转为signaled状态的所有事件对象句柄的信息。通过该函数可以得到转为signaled状态的事件对象中的第一个(按数组中的保存顺序)索引值。但可以利用"事件对象为manual-reset模式"的特点，通过如下方式获得所有 signaled状态的事件对象：

```C++
int posInfo,startIdx,i;
...
posInfo=WSAWaitForMultipleEvents(numOfSock,hEventArray,FALSE,WSA_INFINITE,FALSE);
startIdx=posInfo-WSA_WAIT_EVENT_0;
...
for(i=startIdx;i<numOfSock;i++){
  int sigEventIdx=WSAWaitForMultipleEvents(l, &hEventArray[i], TRUE, 0, FALSE);
}
```

循环中从第一个事件对象到最后一个事件对象逐一依序验证是否转为signaled状态 (超时信息为0，所以调用函数后立即返回 )。之所以能做到这一点，完全是因为事件对象为manual-reset模式。

### 21.2.4区分事件类型

确定相应对象进入signaled状态的原因。调用此函数时，不仅需要 signaled状态的事件对象句柄，还需要与之连接的(由 WSAEventSelect函数调用引发的) 发生事件的套接字句柄。

```C++
int WSAEnumNetworkEvents(SOCKET s, WSAEVENT hEventObject, LPWSANETWORKEVENTS lpNetworkEvents);
//成功时返回0，失败时返回SOCKET-ERROR
```

s：发生事件的套接字句柄

hEventObject：和套接字绑定的事件对象句柄

lpNetworkEvents：保存发生的事件类型信息和错误信息的WSANETWORKEVENTS结构体变量地址值 

函数将manual-reset模式的事件对象改为non-signaled状态 ，所以得到发生的事件类型后，不必单独调用ResetEvent函数。下面介绍与上述函数有关的WSANETWORKEVENTS结构体：

```C++
typedef struct _WSANETWORKEVENTS
{
  long lNetworkEvent;//保存发生的事件信息，与WSAEventSelect函数的第三个参数相同
  //需要接收数据时，该成员为FD_READ; 有连接请求时，该成员为FD_ACCEPT。
  int iErrorCode[FD_MAX_EVENTS];
  //错误信息将保存到声明为成员的iErrorCode数组
}WSANETWORKEVENTS,* LPWSANETWORKEVENTS;
```

可通过如下方式查看发生的事件类型：

```C++
WSANETWORKEVENTS netEvents;
...
WSAEnumNetworkEvents(hSock, hEvent, &netEvents);
if(netEvents.lNetworkEvents & FD_ACCEPT){
  //FD_ACCEPT事件的处理
}
if(netEvents..lNetworkEvents & FD_READ){
  //FD_READ事件的处理
}
if(netEvents.lNetwor、kEvents & FD_CLOSE){
  //FD_CLOSE事件的处理
}
```

错误验证方法如下：

"如果发生FD XXX相关错误，则在iErrorCode[FD_XXX_BIT]中保存除0以外的其他值"

```C++
WSANETWORKEVENTS netEvents;
...
WSAEnumNetworkEvents(hSock, hEvent, &netEvents);
...
if(netEvents.iError、Code[FD_READ_BIT] != 0){
  //发生 FD-READ 事件相关错误
}
```

### 21.2.5利用异步通知I/O模型实现回声服务器端

AsynNotiEchoServ_win.c

```C++
#include <stdio.h>
#include <string.h>
#include <winsock2.h>

#define BUF_SIZE 100

void CompressSockets(SOCKET hSockArr[],int idx,int total);
void CompressEvents(WSAEVENT hEventArr[],int idx,int total);
void ErrorHandliing(char * msg);

int main(int argc,char * argv[]){
    WSADATA wsaData;
    SOCKET hServSock,hClntSock;
    SOCKADDR_IN servAdr,clntAdr;

    SOCKET hSockArr[WSA_MXIMUM_WAIT_EVENTS];
    WSAEVENT hEventArr[WSA_MXIMUM_WAIT_EVENTS];
    WSAEVENT newEvent;
    WSANETWORKEVENTS netEvents;

    int numOfClntSock=0;
    int strLen,i;
    int posInfo,startIndx;
    int clntAdrLen;
    char msg[BUF_SIZE];

    if (argc!=2){
        printf("Usage:%s<port>\n",argv[0]);
        exit(1);
    }
    if (WSAStartup(MAKEWORD(2,2),&wsaData)!=0){
        ErrorHandliing("WSAStartup error");
    }

    hServSock=socket(PF_INET,SOCK_STREAM,0);
    memset(&servAdr,0,sizeof(hSockArr));
    servAdr.sin_family=AF_INET;
    servAdr.sin_addr.s_addr=htonl(INADDR_ANY);
    servAdr.sin_port=htons(atoi(argv[1]));

    if(bind(hServSock,(SOCKADDR *)&servAdr, sizeof(servAdr))==SOCKET_ERROR){
        ErrorHandliing("bind error");
    }

    if(listen(hServSock,5)==SOCKET_ERROR){
        ErrorHandliing("listen error");
    }

    newEvent=WSACreateEvent();//针对FD_ACCEPT事件调用了WSAEventSelect函数
    if(WSAEventSelect(hServSock,newEvent,FD_ACCEPT)==SOCKET_ERROR){
        ErrorHandliing("WSAEventSelect error");
    }
    //把通过WSAEventSelect函数连接的套接字和事件对象的句柄分别存入hSockArr和hEventArr数组，
    //但应该维持它们之间的关系。也就是说，应该可以通过hSockArr[idx]找到连接到套接字的事件对象，
    //反之，也可以通过hEventArr[idx]找到连接到事件对象的套接字。 因此，该示例将套接字和事件对象句柄保存到数组时统一了保存位置。
    //也就有了下列公式 (程序中应该遵守该公式):
    //与hSockArr[n]中的套接字相连的事件对象应保存到hEventArr[n]。
    //与hEventArr[n]中的事件对象相连的套接字应保存到hSockArr[n]。
    hSockArr[numOfClntSock]=hServSock;
    hEventArr[numOfClntSock]=newEvent;
    numOfClntSock++;

    while(1){
        posInfo=WSAWaitForMultiplEvents(numOfClntSock,hEventArr,FALSE,WSA_INFINITE,FALSE);
        startIndx=posInfo-WSA_WAIT_EVENT_0;

        for (i = startIndx; i < numOfClntSock; ++i) {
            int sigEventIdx=WSAWaitForMultiplEvents(1,&hEventArr[i],TRUE,0,FALSE);
            if(sigEventIdx==WSA_WAIT_FAILED||sigEventIdx==WSA_WAIT_TIMEOUT){
                continue;
            }
            else {
                sigEventIdx=i;
                WSAEnumNetworkEvents(hSockArr[sigEventIdx],hEventArr[sigEventIdx],&netEvents);
                if(netEvents.lpNetworkEvents&FD_ACCEPT){//请求超时
                    if(netEvents.iErrorCode[FD_ACCEPT_BIT]!=0){
                        puts("Accept error");
                        break;
                    }
                    clntAdrLen= sizeof(clntAdr);
                    hClntSock=accept(hSockArr[sigEventIdx],(SOCKADDR *)&clntAdr,&clntAdrLen);
                    newEvent=WSACreateEvent();
                    WSAEventSelect(hClntSock,newEvent,FD_READ|FD_CLOSE);

                    hEventArr[numOfClntSock]=newEvent;
                    hSockArr[numOfClntSock]=hClntSock;
                    numOfClntSock++;
                }

                if (netEvents.lNetworkEvents&FD_READ){
                    if(netEvents.iErrorCode[FD_READ_BIT]!=0){
                        puts("Read error");
                        break;
                    }
                    strLen=recv(hSockArr[sigEventIdx],msg, sizeof(msg),0);
                    send(hSockArr[sigEventIdx],msg,strLen,0);
                }

                if(netEvents.lNetworkEvents&FD_CLOSE){
                    if(netEvents.iErrorCode[FD_READ_BIT]!=0){
                        puts("Close error");
                        break;
                    }
                    WSACloseEvent(hEventArr[sigEventIdx]);
                    closesocket(hSockArr[sigEventIdx]);

                    numOfClntSock--;
                    CompressSockets(hSockArr,sigEventIdx,numOfClntSock);
                    CompressEvents(hEventArr,sigEventIdx,numOfClntSock);
                }
            }
        }
    }
    WSACleanup();
    return 0;
}
void CompressSockets(SOCKET hSockArr[],int idx,int total){
    int i;
    for (i = idx; i < total; i++) {
        hSockArr[i]=hSockArr[i+1]
    }
}
void CompressEvents(WSAEVENT hEventArr[],int idx,int total){
    int i;
    for (i = idx; i < total; i++) {
        hEventArr[i]=hEventArr[i+1]
    }
}
void ErrorHandliing(char * msg){
    fputs(msg,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

## 21.3习题

- 结合send& recv函数解释同步和异步方式的I/O。 并请说明同步I/O的缺点，以及怎样通过异步I/O进行解决。
- 异步I/O并不是所有情况下的最佳选择。它具有哪些缺点?何种情况下同步I/O更优?可以参考异步I/O相关源代码，亦可结合线程进行说明 。
- 判断下列关于 select模型描述的正误：
- - select模型通过函数的返回值通知I/O相关事件，故可视为通知IO模型 。 ( )
  - select模型中IO相关事件的发生时间点和函数返回的时间点一致，故不属于异步模型 。 ()
  - WSAEventSelect函数可视为select方式的异步模型，因为该函数的IJO相关事件的通知方式为异步方式。 ( )
- 请从源代码的角度说明 select函数和 WSAEventSelect函数在使用上的差异。
- 第17章的epoll可以在条件触发和边缘触发这2种方式下工作。 哪种方式更适合异步IO模型?为什么?请概括说明。
- Linux中的epoll同样属于异步IO模型。请说明原因。
- 如何获取WSAWaitForMultipleEvents函数可以监视的最大句柄数?请编写代码读取该值。
- 为何异步通知IO模型中的事件对象必须是manual-reset模式?
- 请在本章的通知IO模型的基础上编写聊天服务器端。要求该服务器端能够结合第20章的聊天客户端chat clnt win.c运行。

# 第二十二章 重叠I/O模型

## 22.1理解重叠I/O模型

第21章异步处理的并非IO，而是"通知"，本章讲解的才是以异步方式处理IO的方法。**重叠IO就是多个异步IO同时进行**

### 22.1.1重叠IO

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230908144614496.png" alt="image-20230908144614496" style="zoom:50%;" />

通过上图说明过异步IO模型， 实际上，这种异步IO就相当于重叠IO。即调用发送后立即返回，发送自己在后台发送，调用接受后立即返回，接收在后台接受。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230908145020721.png" alt="image-20230908145020721" style="zoom:50%;" />

同一线程内部向多个目标传输(或从多个目标接收)数据引起的IO重叠现象称为"重叠IO“为了完成这项任务，调用的IO函数应立即返回，只有这样才能发送后续数据。 从结果上看，利用上述模型收发数据时，**最重要的前提条件就是异步IO。而且，为了完成异步IO，调用的IO函数应以非阻塞模式工作。**

### 22.1.2本章讨论的重叠IO的重点不在于IO

Windows中重叠IO的重点并非IO本身， 而是如何确认IO完成时的状态。不管是输入还是输出，只要是非阻塞模式的，就要另外确认执行结果。 即确认是否传输完成和接受完成，确认执行结果前需要经过特殊的处理过程。

### 22.1.3创建重叠IO套接字

首先要创建适用于重叠IO的套接字：

双字节DWORD是一个32位的无符号整数类型，可以存储0到4294967295之间的值。在Windows操作系统中，DWORD常用于存储各种设置和配置信息，例如注册表键值、API函数参数等。

```C++
SOCKET WSASocket(int af,int type, int protocol, LPWSAPROTOCOL_INFO lpProtocolInfo, GROUP g,DWORD dwFlags);
//成功时返回套接字句柄，失败时返回INVALID_SOCKET
```

af：协议族信息

type：套接字的传输类型

protocol：2个套接字之间使用的协议类型

lpProtocolInfo：创建的套接字信息的WSAPROTOCOL_INFO结构体变量地址值，不需要时传递NULL

g：为扩展函数而预约的参数，可以使用 0

dwFlags：套接字属性信息

可以向最后一个参数传递**WSA_FLAG_OVERLAPPED**，赋予创建出的套接字重叠IO特性：创建出可以进行重叠IO的阻塞模式的套接字：为了和之前的套接字编程兼容，所以其实还是要手动设置成非阻塞模式

```C++
WSASocket(PF_INET , SOCK_STREAM , 0 , NULL , 0 , WSA_FLAG_OVERLAPPED);
```

### 22.1.4执行重叠IO的WSASend函数

重叠IO中使用的数据输出函数：

```C++
int WSASend(SOCKET s,LPWSABUF lpBuffers,DWORD dwBufferCount, LPDWORD lpNumberOfBytesSent, DWROD dwFlags, LPWSAOVERLAPPED lpOverlapped, LPWSAOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine);
//成功时返回0，失败时返回SOCKET_ERROR
```

s：套接字句柄

lpBuffers：WSABUF结构体变量数组的地址值， WSABUF中存有待传输数据，**数组指针**

dwBufferCount：第二个参数中数组的长度，**数组长度**

lpNumberOfBytesSent：用于保存实际发送字节数的变量地址值(稍后进行说明)

dwFlags：用于更改数据传输特性，如传递MSG_OOB时才发送OOB模式的数据，辅助信道通信

**lpOverlapped**：WSAOVERLAPPED结构体变量的地址值，使用事件对象，用于确认完成数据传输

lpCompletionRoutine：传入Completion Routine函数的入口地址值，可以通过**该函数确认是否完成数据传输**

```C++
typedef struct __WASBUF{//该结构体中存有待传输数据的地址和大小等信息
  u_long len;			//待传输数据大小
  chat FAR * buf;	//缓冲地址值
}WSABUF,* LPWSABUF;
```

上述函数的调用示例：

```C++
WSAEVENT event;
WSAOVERLAPPED overlapped;//当异步I/O操作完成时，系统会调用WSAOVERLAPPED中的回调函数，在回调函数中获取操作结果并进行后续处理。
WSABUF dataBuf;
char buf[BUF_SIZE] = {"待传输的数据"};
int recvBytes = 0;
......
event =WSACreateEvent();
//WSAOVERLAPPED的设置，暂时只需要设置事件
memset(&overlapped, 0, sizeof(overlapped)); // 所有位初始化为0!
overlapped.hEvent =event;
dataBuf.len = sizeof(buf);
dataBuf.buf= buf;
WSASend(hSocket, &dataBuf, 1, &recvBytes, 0, &overlapped, NULL); 
......
```

第三个参数设置为 1，因为第二个参数中待传输数据的缓冲个数为1，**即WSABUF dataBuf;而不是WSABUF dataBuf[2];**

第六个参数中的WSAOVERLAPPED结构体定义如下：

```C++
typedef struct _WSAOVERLAPPED{//重叠IO的监视者
  DWORD Internal;//操作系统内部成员
  DWORD InternalHigh;//操作系统内部成员
  DWORD Offset;//特殊用途
  DWORD OffsetHigh;//特殊用途
  WSAEVENT hEvent;//事件对象句柄，当当前重叠套接字完成异步IO之后，会给该句柄对应事件发送信号，由此可以确认异步IO是否完成
}WSAOVERLAPPED，*LPWSAOVERLAPPED
```

"为了进行重叠IO，WSASend函数的lpOverlapped参数中应该传递有效的结构体变量地址，而不是NULL"

如果向lpOverlapped传递NULL，WSASend函数的第一个参数中的句柄所指的套接字将以阻塞模式工作。

“利用WSASend函数同时向多个目标传输数据时， 需要分别构建传入第六个参数的 WSAOVERLAPPED结构体变量 。”

进行重叠IO的过程中，操作系统将使用WSAOVERLAPPED结构体变量。

### 22.1.5关于WSASend再补充一点

通过WSASend函数的lpNumberOfBytesSent参数可以获得实际传输的数据大小。关于这一点不感到困惑吗?

"既然WSASend函数在调用后立即返回，那如何得到传输的数据大小呢?"（只有传输数据足够小的时候可以）

实际上， WSASend函数调用过程中，函数返回时间点和数据传输完成时间点并非总不一致。如果输出缓冲是空的，且传输的数据并不大，那么函数调用后可以立即完成数据传输。此时，WSASend函数将返回0，而lpNumberOfBytesSent中将保存实际传输的数据大小的信息。反之，WSASend函数返回后仍需要传输数据时，将返回SOCKET_ERROR，并将WSA_IO_PENDING注册为错误代码，该代码可以通过WSAGetLastError函数(稍后再介绍)得到，这时应该通过如下函数获取实际传输的数据大小 ：

```C++
BOOL WSAGetOverlappedResult(SOCKET s, LPWSAOVERLAPPED lpOverlapped, LPDWORD lpcbTransfer, BOOL fWait, LPDWORD lpdwFlags);
//成功时返回TRUE，失败时返回FALSE。
```

s：进行重叠IO的套接字句柄 

lpOverlapped：进行重叠IO时传递的WSAOVERLAPPED结构体变量的地址值 

lpcbTransfer：用于保存实际传输的字节数的变量地址

fWait：如果调用该函数时仍在进行IO，fWait为TRUE时等待IO完成，fWait为FALSE时将返回FALSE并跳出函数

lpdwFlags：调用WSARecv函数时，用于获取附加信息 ( 例如OOB消息)。如果不需要，可以传递NULL

通过此函数**不仅可以获取数据传输结果，还可以验证接收数据的状态。**

### 22.1.6进行重叠IO的WSARecv 函数

```C++
int WSARecv(SOCKET s,LPWSABUF lpBuffers,DWORD dwBufferCount, LPDWORD lpNumberOfBytesRecvd, LPDWROD lpFlags, LPWSAOVERLAPPED lpOverlapped, LPWSAOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine);
//成功时返回0，失败时返回SOCKET_ERROR
```

s：套接字句柄

lpBuffers：WSABUF结构体变量数组的地址值， WSABUF中存有待传输数据

dwBufferCount：第二个参数中数组的长度

lpNumberOfBytesRecvd：保存接收的数据大小信息的变量地址值

lpFlags：用于设置或读取传输特性信息

lpOverlapped：WSAOVER LAPPED结构体变量的地址值，使用事件对象，用于确认完成数据读取

lpCompletionRoutine：传入Completion Routine函数的入口地址值，可以通过该函数确认是否完成数据读取

## 22.2重叠I/O的I/O完成确认

重叠IO中有2种方法确认IO的完成并获取结果：

利用 WSASend、 WSARecv函数的第六个参数，基于**事件对象**；

利用WSASend、 WSARecv函数的第七个参数，基于**CompletionRoutine**；

### 22.2.1使用事件对象

通过该示例验证如下2点：

- 完成IO时，WSAOVERLAPPED结构体变量引用的事件对象将变为signaled状态；
- 为了验证IO的完成和完成结果，需要调用WSAGetOverlappedResult函数（获取实际传输的数据大小）；

OverlappedSend_win.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
void ErrorHandling(char *message);

int main(int argc, char *argv[]) {
    WSADATA wsaData;//Windows Sockets初始化信息
    SOCKET hSocket;//用于保存创建的套接字句柄
    SOCKADDR_IN servAddr;//用于保存服务器端地址信息

    WSABUF dataBuf;//用于保存待发送的数据
    char msg[] = "Network is Computer!";
    int sendBytes = 0;//保存实际发送的数据长度

    WSAEVENT evObj;//用于保存事件对象句柄
    WSAOVERLAPPED overlapped;//用于保存重叠I/O操作的信息

    if(argc != 3) {
        printf("Usage : %s <IP> <port>\n", argv[0]);
        exit(1);
    }
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)//初始化Windows Sockets API
        ErrorHandling("WSAStartup() error!");

    hSocket = WSASocket(PF_INET, SOCK_STREAM, 0, NULL, 0, WSA_FLAG_OVERLAPPED);//创建套接字
    memset(&servAddr, 0, sizeof(servAddr));
    servAddr.sin_family = AF_INET;
    servAddr.sin_addr.s_addr=inet_addr(argv[1]);
    servAddr.sin_port = htons(atoi(argv[2]));

    if(connect(hSocket, (SOCKADDR*)&servAddr, sizeof(servAddr)) == SOCKET_ERROR)
        ErrorHandling("connect() error!");

    evObj = WSACreateEvent();//创建事件对象
    memset(&overlapped, 0, sizeof(overlapped));//初始化重叠I/O操作信息，并设置它的事件对象
    overlapped.hEvent = evObj;
    dataBuf.len = strlen(msg) + 1;//设置待发送的数据长度
    dataBuf.buf = msg;//设置待发送的数据

    //调用WSASend函数进行异步发送，DWORD：Windows Api中的32位无符号int
    if(WSASend(hSocket, &dataBuf, 1, (LPDWORD)&sendBytes, 0, &overlapped, NULL) == SOCKET_ERROR) {
        if(WSAGetLastError() == WSA_IO_PENDING) {
            puts("Background data send");
            WSAWaitForMultipleEvents(1, &evObj, TRUE, WSA_INFINITE, TRUE);//等待事件对象发出信号
            WSAGetOverlappedResult(hSocket, &overlapped, (LPDWORD)&sendBytes, FALSE, NULL);//获取重叠I/O操作的结果
        }
        else {
            ErrorHandling("WSASend() error!");
        }
    }

    printf("Send data size: %d \n", sendBytes);
    WSACloseEvent(evObj);//关闭事件对象
    closesocket(hSocket);//关闭套接字
    WSACleanup();//终止Windows Sockets API
    return 0;
}

void ErrorHandling(char *message) {
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```

调用套接字相关函数后，可以通过该函数获取错误信息：

```C++
int WSAGetLastError(void);
//返回错误代码
```

上述示例中该函数的返回值为WSA_IO_PENDING， 由此可以判断WSASend函数的调用结果并非发生了错误，而是尚未完成 (Pending) 的状态。 下面介绍与上述示例配套使用的Receiver， 该示例的结构与之前的Sender类似：

OverlappedRecv_win.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
void ErrorHandling(char *message);

int main(int argc, char *argv[]) {
    WSADATA wsaData;//Windows Sockets初始化信息
    SOCKET hLisnSock, hRecvSock;//分别用于保存监听套接字和数据传输套接字
    SOCKADDR_IN lisnAdr, recvAdr;//分别用于保存监听套接字和数据传输套接字的地址信息
    int recvAdrSz;//用于保存recvAdr结构体的大小

    WSABUF dataBuf;//用于保存接收数据的缓冲区
    WSAEVENT evObj;//用于保存事件对象
    WSAOVERLAPPED overlapped;//用于保存重叠结构体

    char buf[BUF_SIZE];
    int recvBytes = 0, flags = 0;
    if (argc != 2) {
        printf("Usage : %s <port>\n", argv[0]);
        exit(1);
    }
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)//初始化Windows Sockets环境
        ErrorHandling("WSAStartup() error!");

    hLisnSock = WSASocket(PF_INET, SOCK_STREAM, 0, NULL, 0, WSA_FLAG_OVERLAPPED);//创建监听套接字
    memset(&lisnAdr, 0, sizeof(lisnAdr));//初始化lisnAdr结构体
    lisnAdr.sin_family = AF_INET;
    lisnAdr.sin_addr.s_addr = htonl(INADDR_ANY);
    lisnAdr.sin_port = htons(atoi(argv[1]));

    if (bind(hLisnSock, (SOCKADDR *) &lisnAdr, sizeof(lisnAdr)) == SOCKET_ERROR)//绑定监听套接字
        ErrorHandling("bind() error");
    if (listen(hLisnSock, 5) == SOCKET_ERROR)//将监听套接字转换为可接收连接状态
        ErrorHandling("listen() error");

    recvAdrSz = sizeof(recvAdr);
    hRecvSock = accept(hLisnSock, (SOCKADDR *) &recvAdr, &recvAdrSz);//接收连接请求

    evObj = WSACreateEvent();//创建事件对象
    memset(&overlapped, 0, sizeof(overlapped));//初始化重叠结构体，并绑定事件
    overlapped.hEvent = evObj;
    dataBuf.len = BUF_SIZE;//设置WSABUF
    dataBuf.buf = buf;

    if (WSARecv(hRecvSock, &dataBuf, 1, (LPDWORD) &recvBytes, (LPDWORD) &flags, &overlapped, NULL) == SOCKET_ERROR) {
        if (WSAGetLastError() == WSA_IO_PENDING)//如果发生WSA_IO_PENDING错误，说明数据还未接收完毕
            puts("Background data receive");
            WSAWaitForMultipleEvents(1, &evObj, TRUE, WSA_INFINITE, TRUE);//等待事件对象发生
            WSAGetOverlappedResult(hRecvSock, &overlapped, (LPDWORD) &recvBytes, FALSE, NULL);//获取重叠结构体中的数据
    } else
        ErrorHandling("WSARecv() error");

    printf("Received message: %s \n", buf);
    WSACloseEvent(evObj);//关闭事件对象
    closesocket(hRecvSock);//关闭数据传输套接字
    closesocket(hLisnSock);//关闭监听套接字
    WSACleanup();//终止Windows Sockets环境
    return 0;
}

void ErrorHandling(char *message) {
    fputs(message, stderr);
    fputc('\n', stderr);
    exit(1);
}
```

### 22.2.2使用Completion Routine函数

笔记：重叠IO相当于我去烧水，烧水的时候我可以先去做其他事，但是不是用多线程实现的，当我做完自己的事之后我将自己设置为闲置并去检查是否烧开，如果已经烧开则执行回调函数，如果没有烧开，我就等到烧开然后执行回调函数

前面的示例通过事件对象验证了IO完成与与否，CompletionRoutine ( 以下简称CR) 函数验证IO完成情况。"注册CR"具有如下含义 :

"Pending的IO完成时调用此函数!"就是完成回调函数，IO完成时调用注册过的函数进行事后处理。

如果执行重要任务时突然调用CompletionRoutine，则有可能破坏程序的正常执行流。因此，操作系统通常会预先定义规则:

"只有请求IO的线程处于alertable wait状态时才能调用Completion Routine函数!"

”alertable wait状态“是等待接收操作系统消息的线程状态。调用下列函数时进入alertable wait状态：

WaitForSingleObjectEx

WaitForMultipleObjectsEx

WSAWaitForMultipleEvents

SleepEx

第一 、第二、 第四个函数提供的功能与WaitForSingleObject、 WaitForMultipleObjects、 Sleep 函数相同。只增加了1个参数，如果该参数为TRUE，则相应线程将进入alertablewait（可响应等待）状 态。另外，第21章介绍过以WSA为前缀的函数，该函数的最后一个参数设置为TRUE时，线程同样进入可响应等待状态。启动IO任务之后，可调用上述函数验证IO是否完成，此时操作系统知道线程进入可响应等待状态，如果有已经完成的IO，则调用相应的完成回调函数，调用后上述函数全部返回WAIT_IO_COMPLETION，并开始执行接下来的程序，将OverlappedRecv_win.c改成Completion Routine方式：

CmplRoutinesRecv_win.c

```c++
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>

#define BUF_SIZE 1024
void CALLBACK CompRoutine(DWORD,DWORD,LPWSAOVERLAPPED,DWORD);
void ErrorHandling(char *message);

WSABUF dataBuf;
char buf[BUF_SIZE];
int recvBytes=0;

int main(int argc ,char * argv[]) {
    WSADATA wsaData;
    SOCKET hLisnSock, hRecvSock;
    SOCKADDR_IN lisnAdr, recvAdr;
    WSAOVERLAPPED overLapped;
    WSAEVENT evObj;

    int idx, recvAdrSz, flag = 0;
    if (argc != 2) {
        printf("Usage:%s<port>\n", argv[0]);
        exit(1);
    }
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
        ErrorHandling("WSAStartup Error");

    hLisnSock = WSASocket(PF_INET, SOCK_STREAM, 0, NULL, 0, WSA_FLAG_OVERLAPPED);

    memset(&lisnAdr, 0, sizeof(lisnAdr));
    lisnAdr.sin_family = AF_INET;
    lisnAdr.sin_addr.s_addr = htonl(INADDR_ANY);
    lisnAdr.sin_port = htons(atoi(argv[1]));

    if (bind(hLisnSock, (SOCKADDR * ) & lisnAdr, sizeof(lisnAdr)) == SOCKET_ERROR)
        ErrorHandling("bind error");
    if (listen(hLisnSock, 5) == SOCKET_ERROR)
        ErrorHandling("listen error");

    recvAdrSz = sizeof(recvAdr);
    hRecvSock = accept(hLisnSock, (SOCKADDR * ) & recvAdr, &recvAdrSz);
    if (hRecvSock == INVALID_SOCKET)
        ErrorHandling("accept error");

    memset(&overLapped, 0, sizeof(overLapped));
    dataBuf.len = BUF_SIZE;
    dataBuf.buf = buf;
    evObj = WSACreateEvent;
		//调用WSARecv函数同时传递回调函数的地址，注意：采用回调函数方式，也要指定WSAOVERPLAPPED对象地址，但是可以没有hEvent
  	//发起一个异步数据接受操作，尝试从hRecvSock中接受数据，但不会等待数据立即到达而是立即返回。然后它会继续执行后续代码，不会阻塞等待数据。如果数据尚未到达，该函数会返回 WSA_IO_PENDING，表示数据接收操作尚未完成。
    if (WSARecv(hRecvSock, &(hbInfo->wsaBuf), 1, &recvBytes, &flagInfo, lpOvLp, CompRoutine) == SOCKET_ERROR) {
      //验证接受数据的操作是否在pending状态：尚未完成状态，说明后台还在接受数据
      if (WSAGetLastError() == WSA_IO_PENDING)
            puts("Background data receive");
    }
		//为了使main线程进入可响应等待状态，为了调用次函数，创建了多余的evObj，使用SleepEx可以避免它的产生
  	//使mian线程进入可响应等待状态，并等待后台IO结束后该函数就会返回，返回WAIT_IO_COMPLETION意味着IO正常结束，否则说明发生错误，
  	//并且该函数返回后意味着后台执行的异步接收完成，系统将会在后台调用执行CompRoutine函数
  	//如果在调用 WSAWaitForMultipleEvents 之前异步操作已经完成了，Completion Routine 函数的调用将在调用 WSAWaitForMultipleEvents 之后，与 WSAWaitForMultipleEvents 的返回结果之后立即执行。也就是说，Completion Routine 函数的调用是在与 WSAWaitForMultipleEvents 调用同时或在其之后执行的，不会在 WSAWaitForMultipleEvents 调用之前执行。
    idx = WSAWaitForMultipleEvents(1, &evObj, FALSE, WSA_INFINITE, TRUE);
  	//WAIT_IO_COMPLETION意味着IO正常结束
    if (idx == WAIT_IO_COMPLETION)
        puts("Overlapped IO Completed");
    else
        ErrorHandling("WSARecv error");
    WSACloseEvent(evObj);
    closesocket(hRecvSock);
    closesocket(hLisnSock);
    WSACleanup();
    return 0;
}

    void CompRoutine(DWORD dwError,DWORD szRecvBytes,LPWSAOVERLAPPED lpOverlapped,DWORD flags){
        if(dwError!=0)
            ErrorHandling("CompRoutine error");
        else{
            recvBytes = szRecvBytes;
            printf("Received message:%s\n",buf);
            }
    }
    void ErrorHandling(char *message){
        fputs(message,stderr);
        fputc('\n',stderr);
        exit(1);
    }
```

下面给出传入WSARecv函数的最后一个参数的Completion Routine函数原型：

```C++
void CompRoutine(DWORD dwError,DWORD szRecvBytes,LPWSAOVERLAPPED lpOverlapped,DWORD flags)
```

其中第一个参数中写入错误信息(WSARecv正常结束时写入0)，第二个参数中写入WSARecv实际收发的字节数。第三个参数中写入WSASend、WSARecv函数的参数lpOverlapped，dwFlags中写入调用IO函数时传入的特性信息或 0。返回值类型void后插入的CALLBACK关键字与main函数中声明的关键字WINAPI相同，都是用于声明函数的调用规范，所以定义CompletionRoutine函数时必须添加。

## 22.3习题

- 异步通知IO模型与重叠IO模型在异步处理方面有哪些区别？

  异步通知：

​	在异步通知IO模型中，应用程序通常发起IO请求，然后继续执行其他任务，不需要等待IO操作完成。

​	当IO操作完成时，操作系统会向应用程序发送通知，通常是通过信号、回调函数、消息队列等方式。

​	应用程序需要提前注册一个回调函数或处理程序来处理IO完成事件，以便在事件发生时执行相应的处理逻辑。

​	异步通知IO模型对于处理多个IO操作时非常有效，因为应用程序可以同时发起多个IO请求，然后等待通知。

​	重叠IO：

​	在重叠IO模型中，应用程序通常使用操作系统提供的异步IO函数（如Windows的`WSARecv`）来发起IO请求，然后继续执行其他任务。

​	重叠IO函数会将IO请求提交给操作系统内核，但不会阻塞应用程序的执行，应用程序可以继续做其他事情。

​	当IO操作完成时，操作系统会在后台自动将数据复制到应用程序的缓冲区中，并通知应用程序IO完成。

​	应用程序需要轮询或使用事件通知等机制来检查IO是否完成，然后处理已完成的IO请求。

​	异步通知IO模型是你发起IO操作后，可以继续做其他事情，等待通知；而重叠IO模型是你可以同时发起多个IO操作，它们可以并行执行，不需要等待一个操作完成后再进行下一个。异步IO是重叠IO的基础。重叠IO一定是异步IO。

- 请分析非阻塞IO、异步IO、 重叠IO之间的关系。
  - 非阻塞IO是一种简单的方式，允许你轮询IO操作的状态。
  - 异步IO更高级，让你在IO操作完成后接收通知，而不需要主动轮询。
  - 重叠IO是异步IO的一种特殊形式，它允许同时处理多个IO操作，提高效率。
- 阅读如下代码，请指出问题并给出解决方案 ：

```C++
while(1){
  hServSock=accept(hLisnSock,(SOCKADDR*)&recvAdr,&recvAdrSz);
  evObj=WSACreateEvent();
  memset(&overlapped,0,sizeof(overlapped));
  overlapped.hEvent=evObj;
  dataBuf.len=BUF_SIZE;
  dataBuf.buf=buf;
  WSARecv(hRecvSock,&dataBuf,1,&recvBytes,&flags,&overlapped,NULL);
}
```

​	若accept函数返回失败，WSARecv传入的overlapped将会失效，因为overlapped事先没有任何传入，若accept返回成功，因为所有的overlapped结构都相同，可以使用一个overlapped给多个WSARecv使用

- 请从源代码角度说明调用WSASend函数后如何如何验证IO是否进入Pending状态 。

  ```c++
  int error = WSAGetLastError();
      
      if (error == WSA_IO_PENDING) {
          // WSASend 进入了 Pending 状态
          // 在这里可以执行一些处理或等待操作完成
      } else {
          // 发生了其他错误，需要处理错误情况
          // 错误码可通过 error 变量获取
      }
  ```

- 线程的alertable wait状态的含义是什么?说出能使线程进入这种状态的 2个函数。

线程的 alertable wait 状态是一种等待状态，它允许线程在等待期间执行特定的回调函数。在 alertable wait 状态下，线程会等待某个事件发生，但同时也会响应异步回调函数（例如 I/O 完成时的回调函数）。

SleepEx是直接将main线程变成响应等待状态，这样当IO完成时就会继续执行

WSAWaitForMultipleEvents也会将mian线程编程响应等待状态，并且参数中的事件会因为IO完成而发出信号，

# 第二十三章 IOCP

## 23.1通过重叠I/O理解IOCP

### 23.1.1epoll和IOCP的性能比较

每种操作系统(内核级别)都会提供特有的IO模型以提高性能。其中最具代表性的有Linux的epoll、BSD的kqueue及本章的Windows的IOCP。

### 23.1.2实现非阻塞模式的套接字

在Windows中通过如下函数调用将套接字属性改为非阻塞模式：WSASocket就是非阻塞的套接字

```C++
SOCKET hLisnSock,
int mode = 1;
......
hListSock = WSASocket(PF_INET) SOCK_STREAM, 0, NULL, 0, WSA_FLAG_OVERLAPPED);
ioctlsocket(hLisnSock) FIONBIO, &mode); //变成非阻塞套接字
......
```

ioctlsocket函数负责控制套接字IO方式，其调用具有如下含义：

"将hLisnSock句柄引用的套接字IO模式 (FIONBIO)改为变量mode中指定的形式。"

FIONBIO是用于更改套接字IO模式的选项，该函数的第三个参数中传入的变量中若存有0，则说明套接字是阻塞模式的;如果存有非0值，则说明已将套接字模式改为非阻塞模式。 改为非阻塞模式后，除了以非阻塞模式进行IO外，还具有如下特点。

- 如果在没有客户端连接请求的状态下调用 accept函数，将直接返回INVALID_SOCKET。 调用 WSAGetLastError函数时返回 WSAEWOULDBLOCK。
- 调用accept函数时创建的套接字同样具有非阻塞属性

因此，针对非阻塞套接字调用accept函数并返回INVALID_SOCKET时，应该通过WSAGetLastError函数确认返回INVALID_SOCKET的理由，再进行适当处理。

### 23.1.3以纯重叠I/O方式实现回声服务器端

为了正确理解IOCP，应当尝试用纯重叠IO方式实现服务器端。

CmplRoulEchoServ_win.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>

#define BUF_SIZE 1024
void CALLBACK ReadCompRoutine(DWORD,DWORD,LPWSAOVERLAPPED,DWORD);
void CALLBACK WriteCompRoutine(DWORD,DWORD,LPWSAOVERLAPPED,DWORD);
void ErrorHandling(char *message);

//该结构体中的信息用于数据交换
typedef struct{
    SOCKET hClntSock;//套接字句柄
    char buf[BUF_SIZE];//缓冲区
    WSABUF wsaBuf;//保存缓冲区的信息
}PER_TO_DATA,*LPPER_TO_DATA;

int main(int argc ,char * argv[]){
    WSADATA wsaData;
    SOCKET hLisnSock,hRecvSock;
    SOCKADDR_IN lisnAdr,recvAdr;
    LPWSAOVERLAPPED lpOvLp;
    DWORD recvBytes;
    LPPER_TO_DATA hbInfo;
    int mode=1,recvAdrSz,flagInfo=0;

    if (argc!=2){
        printf("Usage:%s<port>\n",argv[0]);
        exit(1);
    }

    if (WSAStartup(MAKEWORD(2,2),&wsaData)!=0)
        ErrorHandling("WSAStartup Error");

    hLisnSock=WSASocket(PF_INET,SOCK_STREAM,0,NULL,0,WSA_FLAG_OVERLAPPED);
    ioctlsocket(hLisnSock,FIONBIO,&mode);//将套接字改为非阻塞模式

    memset(&lisnAdr,0,sizeof(lisnAdr));
    lisnAdr.sin_family=AF_INET;
    lisnAdr.sin_addr.s_addr= htonl(INADDR_ANY);
    lisnAdr.sin_port=htons(atoi(argv[1]));

    if (bind(hLisnSock,(SOCKADDR*)&lisnAdr,sizeof(lisnAdr))==SOCKET_ERROR)
        ErrorHandling("bind error");
    if (listen(hLisnSock,5)==SOCKET_ERROR)
        ErrorHandling("listen error");

    recvAdrSz=sizeof (recvAdr);
    while(1){
      	//直接进入响应等待状态，IO完成后会被唤醒，第一次的等待其实没有意义
      	//True表示在休眠期间允许IO完成或者APC（线程异步调用）被调度时唤醒线程，而不需要等待完整的时间
        SleepEx(100,TRUE);//用于可响应等待状态
        //循环接受客户端的连接请求
        hRecvSock=accept(hLisnSock,(SOCKADDR*)&recvAdr,&recvAdrSz);
        if(hRecvSock==INVALID_SOCKET){
            if (WSAGetLastError()==WSAEWOULDBLOCK)
                continue;
            else
                ErrorHandling("accept error");
        }
        puts("Client connected...");
				
      	//动态分配LPWSAOVERLAPPED，每个客户端都需要独立的WSAOVERLAPPED结构体变量，这也是上一章习题中的问题，不可以共用WSAOVERLAPPED
        lpOvLp=(LPWSAOVERLAPPED)malloc(sizeof(WSAOVERLAPPED));
        memset(lpOvLp,0,sizeof(WSAOVERLAPPED));//因为不使用WSAOVERLAPPED自带的事件来判断IO是否完成，所以全设置为0就可以了

      	//动态分配LPPER_TO_DATA
        hbInfo=(LPPER_TO_DATA) malloc(sizeof(PER_TO_DATA));
        hbInfo->hClntSock=(DWORD)hRecvSock;
        (hbInfo->wsaBuf).buf=hbInfo->buf;
        (hbInfo->wsaBuf).len=BUF_SIZE;
				//基于CompletionRoutine函数的重叠IO中不需要事件对象，因此hEvent可以写入其他信息，之所以写入hbInfo，是因为回调函数可以因此使用到LPPER_TO_DATA结构体的内容
        lpOvLp->hEvent=(HANDLE)hbInfo;
      	//设置ReadCompRoutine为完成回调函数，并开始重叠IO接收，接收hRecvSock里的内容到hbInfo->wsaBuf缓冲区，字节数为recvBytes，回调函数为ReadCompRoutine。
        WSARecv(hRecvSock,&(hbInfo->wsaBuf),1,&recvBytes,&flagInfo,lpOvLp,ReadCompRoutine);
    }
    closesocket(hRecvSock);
    closesocket(hLisnSock);
    WSACleanup();
    return 0;
}

void ReadCompRoutine(DWORD dwError,DWORD szRecvBytes,LPWSAOVERLAPPED lpOverlapped,DWORD flags){
    LPPER_TO_DATA hbInfo=(LPPER_TO_DATA)(lpOverlapped->hEvent);//获取到LPPER_TO_DATA结构体
    SOCKET hSock=hbInfo->hClntSock;//获取到客户端套接字
    LPWSABUF bufInfo=&(hbInfo->wsaBuf);//获取到缓冲区
    DWORD sentBytes;

    if(szRecvBytes==0){//接收字节为0，则关闭客户端套接字，并释放掉重叠IO信息结构体
        closesocket(hSock);
        free(lpOverlapped->hEvent);free(lpOverlapped);
        puts("Client disconnect...");
    }
    else{//否则将数据用重叠IO发送回去，发送后调用写回调函数
        bufInfo->len=szRecvBytes;
        WSASend(hSock,bufInfo,1,&sentBytes,0,lpOverlapped,WriteCompRoutine);
    }
}
void WriteCompRoutine(DWORD dwError,DWORD szRecvBytes,LPWSAOVERLAPPED lpOverlapped,DWORD flags){
    LPPER_TO_DATA hbInfo=(LPPER_TO_DATA)(lpOverlapped->hEvent);//获取到LPPER_TO_DATA结构体
    SOCKET hSock=hbInfo->hClntSock;//获取到客户端套接字
    LPWSABUF bufInfo=&(hbInfo->wsaBuf);//获取到缓冲区
    DWORD recvBytes;
    int flagInfo=0;
    WSARecv(hSock,bufInfo,1,&recvBytes,&flagInfo,lpOverlapped,ReadCompRoutine);//重叠IO接收，接受后调用读回调函数
}
void ErrorHandling(char *message){
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

后台接收->读回调.发送回去->写回调.后台接收，如此就形成了自动的回声服务器

### 23.1.4重新实现客户端

需要按照第5章的提示解决客户端存在的问题，并结合改进后的客户端运行本章服务器端。 之前已介绍过解决方法， 故只给出代码。

StableEchoClnt_win.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
#include <cstring>

#define BUF_SIZE 1024
void ErrorHandling(char *message);

WSABUF dataBuf;
char buf[BUF_SIZE];
int recvBytes=0;

int main(int argc ,char * argv[]) {
    WSADATA wsaData;
    SOCKET hSocket;
    SOCKADDR_IN servAdr;
    char message[BUF_SIZE];
    int strLen, readLen;

    if (argc != 3) {
        printf("Usage:%s<port>\n", argv[0]);
        exit(1);
    }
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
        ErrorHandling("WSAStartup Error");

    hSocket = socket(PF_INET, SOCK_STREAM, 0);
    if (hSocket == INVALID_SOCKET)
        ErrorHandling("socket error");

    memset(&servAdr, 0, sizeof(servAdr));
    lisnAdr.sin_family = AF_INET;
    lisnAdr.sin_addr.s_addr = inet_addr(argv[1]);
    lisnAdr.sin_port = htons(atoi(argv[2]));

    if (connect(hSocket, (SOCKADDR * ) & servAdr, sizeof(servAdr)) == SOCKET_ERROR)
        ErrorHandling("connect error");
    else
        puts("Connected......");

    while (1) {
        fputs("input message (q to quit)", stdout);
        fgets(message, BUF_SIZE, stdin);
        if (!strcmp(message, 'q\n') || !strcmp(message, 'Q\n'))
            break;

        strLen = strlen(message);
        send(hSocket, message, strLen, 0);
        readLen = 0;
        while (1) {
            readLen += recv(hSocket, &message[readLen], BUF_SIZE - 1, 0);
            if (readLen >= strLen)
                break;
        }
        message[strLen] = 0;
        printf("message from serv:%s", message);
    }

    closesocket(hSocket);
    WSACleanup();
    return 0;
}

void ErrorHandling(char *message){
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

### 23.1.5从重叠I/O模型到IOCP模型

重叠IO模型回声服务器端的缺点：

”重复调用非阻塞模式的accept函数和以进入alertable wait状态为目的的SleepEx函数将影响性能!“

既不能为了处理连接请求而只调用accept 函数，也不能为了CompletionRoutine而只调SleepEx函数，因此轮流调用了非阻塞模式的 accept函数和SleepEx函数(设置较短的超时时间)。 这恰恰是影响性能的代码结构。

如果调用完accept函数后能够开一个线程去处理SleepEx函数及IO处理能够提高性能。就是IOCP创建专门的IO线程负责和所有客户端进行IO。

**知识补给站**：负责全部IO

有些读者会误认为IOCP就是创建专职IO线程的一种模型。IOCP为了负责全部IO工作，的确需要创建至少 1个线程 。 但"在服务器端负责全部IO"实质上相当于负责服器端的全部工作（包括IO操作的正确执行，并在需要时通知IO的完成），因此，它并不是仅负责IO工作，而是至少创建 1个线程并使其负责全部IO的前后处理。 尝试理解IOCP时不要把焦点集中于线程，而是注意观察如下2点。 

IO是否以非阻塞模式工作 ?
如何确定非阻塞模式的IO是否完成?

之前讲过的所有IO模型及本章的IOCP模型都可以按上述标准进行区分。 各位会发现，我讲解IOCP时也不会刻意强调创建线程的事实 。

## 23.2分阶段实现IOCP

IOCP流程：先创建完成端口作为管理IO操作的中心来多线程跟踪已完成的IO操作；然后创建多个专门的IO线程来负责具体的处理IO操作，这些线程会等待IOCP对象通知他们有IO操作完成时进行处理；当需要执行IO操作的时候交给IOCP对象进行操作；IOCP对象将要做的IO操作放入等待队列中，IO线程们一起重复检查等待队列，如果有新的IO操作就交给检查到的IO线程来执行相应操作；IO操作完成时，相关信息放入已完成IO队列，IOCP通知等待的已完成IO处理线程有IO操作已经完成需要进行处理；已完成IO处理线程收到通知后获取已完成的IO操作的信息并执行相应操作

也就是IO操作是多线程的，已完成IO操作的处理也是多线程的

### 23.2.1创建完成端口

已完成的IO消息就是那些已经完成传输的IO对象消息

IOCP中已完成的IO信息将会被注册到完成端口对象（Completion Port），首先经过如下请求过程：

“该套接字的IO完成时，请把状态信息注册到指定CP对象”

这个过程叫做“套接字和完成端口对象之间的连接请求”，所以需要先做两个工作：

- 创建完成端口对象

- 建立完成端口对象和套接字之间的联系


此时的套接字必须是重叠IO套接字，上述两个工作可以通过一个函数完成，先介绍创建完成端口对象：

```c++
HANDLE CreateIoCompletionPort(HANDLE FileHandle,HANDLE ExistingCompletionPort,ULONG_PTR CompletionKey,DWORD NumberofConcurrentThreads);
//成功时返回CP对象句柄，失败时返回NULL
```

FileHandle：创建CP时传递INVALID_HANDLE_VALUE

ExistingCompletionPort：创建CP时传递NULL

CompletionKey：创建CP时传递0；

NumberofConcurrentThreads：分配给CP对象的用于已完成IO处理的线程数，例如2时，说明分配给CP对象的同时运行的已完成IO处理线程数最多为2个，如果是0，系统中cpu个数就是最大线程数

创建CP对象时只有最后一个参数有意义，一般用如下代码将分配给CP对象的已完成IO处理线程数为2

```C++
HANDLE hCpObject;
......
hCpObject=CreateIoCompletionPort(INVALID_HANDLE_VALUE,NULL,0,2);
```

### 23.2.2连接完成端口对象和套接字

将FileHandle句柄指向的套接字和ExistingCompletionPort指向的CP对象相连接的函数调用如下：

```C++
HANDLE hCpObject;
SOCKET hSock;
......
CreateIoCompletionPort((HANDLE)hSock,hCpObject,(DWORD)ioInfo,0);
```

此时hSock相关信息就被注册到hCpObject指向的CP对象中了。

### 23.2.34确定完成端口已完成的IO和线程的IO处理

笔记：IOCP中有一个或多个已完成IO处理线程，负责处理所有IO操作，已完成的IO是已经完成的输入输出操作，当一个IO操作完成时，系统会将相关信息放入一个队列中，表示该操作已经完成。已完成IO处理线程负责从已完成的IO队列中获取信息并处理，这些已完成IO处理线程不断检查已经完成的IO队列看是否有新完成的IO操作，如果有，IO线程会获取相关信息然后执行处理，处理完一个后已完成IO处理线程再次检查已完成的IO队列，IOCP中的完成端口允许已完成IO处理线程并发处理已完成的IO操作，而不需要每一个已完成IO处理线程都创建一个新线程。

如何确认CP对象中注册的已完成的IO？函数如下：

```c++
BOOL GetQueueCompletionStatus(HANDLE CompletionPort, LPDWORD lpNumberOfBytes, PULONG_PTR lpCompletionKey, LPOVERLAPPED *lpOverlapped, DWORD dwMilliseconds);
//成功时返回TRUE，失败时返回FALSE
```

CompletionPort：CP对象句柄

lpNumberOfBytes：用于保存IO过程中传输的数据大小的变量指针

lpCompletionKey：用于保存CreateIoCompletionPort第三个参数CompletionKey的变量指针

lpOverlapped：用于保存WSASend、WSARecv函数传递的OVERLAPPED结构体地址的指针

dwMilliseconds：超时信息，超时之后将返回FALSE并返回函数，传递INFINITE时，程序将阻塞，直到已完成IO信息写入CP对象

处理IOCP中已完成IO的线程来调用上述函数：已完成IO处理线程

“那IO如何分配给线程呢？”

IOCP将创建全职IO线程，该线程对所有客户端进行IO，CreateIoCompletionPort函数中也有参数指定分配给CP对象的最大线程数，各位可能有疑问：

“是否自动创建线程并处理IO？”

并不是，应该由程序员自行创建调用WSASend、WSARecv的IO函数的线程，只是该线程为了确认IO的完成会调用GetQueueCompletionStatus函数，虽然任何线程都能调用GetQueueCompletionStatus函数，但实际得到IO信息的线程数不会超过调用CreateIoCompletionPort函数时指定的最大线程数。

笔记：IOCP中有一个或多个已完成IO处理线程处理已完成的IO操作，这些线程调用GetQueueCompletionStatus来检查是否有IO操作已经完成，这个线程需要程序员自己创建并编写处理IO完成的处理，IOCP可以限制处理IO完成的线程数量，通过调用CreateIoCompletionPort时指定的最大线程数来控制，所有IO线程都可以调用GetQueueCompletionStatus来获取IO完成的信息，但是处理这些信息的线程数量是有限制的，IOCP会动态的将IO完成处理分配给空闲的线程，但不会超过指定的最大线程数。

**知识补给站**：**分配给完成端口的合理的线程个数**

分配给CP对象的合理的线程数应该和cpu个数相同

### 23.2.5实现基于IOCP的回声服务端

·IOCPEchoServ_win.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <process.h>
#include <winsock2.h>
#include <windows.h>

#define BUF_SIZE 100
#define READ 3
#define WRITE 5

typedef struct {//socket信息，保存和客户端相连的套接字，请观察该结构体何时被分配空间、如何被传递、如何被使用
    SOCKET hClntSock;
    SOCKADDR_IN clntAdr;
}PER_HANDLE_DATA,* LPPER_HANDLE_DATA;

typedef struct {//buffer信息，将IO中使用的缓冲和重叠IO信息OVERLAPPED结构体封装到同一结构体
    OVERLAPPED overlapped;//注意：结构体的变量地址和结构体第一个成员的地址值相同
    WSABUF wsaBuf;
    char buffer[BUF_SIZE];
    int rwMode;
}PER_IO_DATA,* LPPER_IO_DATA;

DWORD WINAPI EchoThreadMain(LPVOID CompletionPortIO);
void ErrorHandling(char *message);

int main(int argc, char *argv[]){
    WSADATA wsaData;
    HANDLE hComPort;
    SYSTEM_INFO sysInfo;
    LPPER_IO_DATA ioInfo;
    LPPER_HANDLE_DATA handleInfo;

    SOCKET hServSock;
    SOCKADDR_IN servAdr;
    int recvBytes,i,flags=0;
    if(WSAStartup(MAKEWORD(2,2),&wsaData)!=0)
        ErrorHandling("WSAStratup error");
    //创建完成端口对象，并且最大线程数为CPU数目
    hComPort= CreateIoCompletionPort(INVALID_HANDLE_VALUE,NULL,0,0);
    //获取当前系统信息
    GetSystemInfo(&sysInfo);
    //创建了与cpu个数相当的线程，并且创建线程时传递创建的完成端口对象句柄，线程会通过这个句柄访问完成端口对象，也就是我们说的完成对象将通过该句柄分配到线程
    for (i = 0; i < sysInfo.dwNumberOfProcessors; i++) {
        _beginthreadex(NULL,0,EchoThreadMain,(LPVOID)hComPort,0,NULL);
    }
	//创建服务器套接字，并且是支持重叠IO的服务器套接字，并设置好协议、地址和端口
    hServSock= WSASocket(AF_INET,SOCK_STREAM,0,NULL,0,WSA_FLAG_OVERLAPPED);
    memset(&servAdr,0, sizeof(servAdr));
    servAdr.sin_family=AF_INET;
    servAdr.sin_addr.s_addr= htonl(INADDR_ANY);
    servAdr.sin_port= htons(atoi(argv[1]));
	//绑定套接字和地址信息，并开始监听
    bind(hServSock,(SOCKADDR*)&servAdr, sizeof(servAdr));
    listen(hServSock,5);

    while(1){
        SOCKET hClntSock;
        SOCKADDR_IN clntAdr;
        int addrLen=sizeof(clntAdr);
		//接受客户端套接字的连接请求并获得客户端地址
        hClntSock= accept(hServSock,(SOCKADDR*)&clntAdr,&addrLen);
        //动态分配LPPER_HANDLE_DATA对象，并写入客户端套接字和客户端地址信息
        handleInfo=(LPPER_HANDLE_DATA)malloc(sizeof(PER_HANDLE_DATA));
        handleInfo->hClntSock=hClntSock;
        memcpy(&(handleInfo->clntAdr),&clntAdr,addrLen);
		//将客户端套接字信息注册到完成端口
        CreateIoCompletionPort((HANDLE)hClntSock,hComPort,(DWORD)handleInfo,0);
        //动态分配LPPER_IO_DATA对象
        ioInfo=(LPPER_IO_DATA) malloc(sizeof(PER_IO_DATA));
        memset(&(ioInfo->overlapped),0, sizeof(OVERLAPPED));
        //准备了OVERLAPPED结构体变量WSABUF结构体变量及缓冲
        ioInfo->wsaBuf.len=BUF_SIZE;
        ioInfo->wsaBuf.buf=ioInfo->buffer;
        //IOCP本身不会区分输入完成还是输出完成的状态，无论是输入还是输出，只通知完成IO的状态，所以需要额外的变量区分是输入IO还是输出IO
        ioInfo->rwMode=READ;
        //这里的第七个参数overlapped可以在GetQueueCompletionStatus函数返回时得到。但是该结构体变量地址和第一个成员的地址相同，也就相当于传入了PER_IO_DATA的指针
        WSARecv(handleInfo->hClntSock,&(ioInfo->wsaBuf),1,&recvBytes,&flags,&(ioInfo->overlapped),NULL);
    }
    return 0;
}
//由线程来运行EchoThreadMain函数，该线程调用GetQueuedCompletionStatus
DWORD WINAPI EchoThreadMain(LPVOID pComPort){
    HANDLE hComPort=(HANDLE)pComPort;
    SOCKET sock;
    DWORD bytesTrans;
    LPPER_HANDLE_DATA handleInfo;
    LPPER_IO_DATA ioInfo;
    DWORD flags =0;
    
    while(1){
        //获取完成端口对象中已完成IO信息，返回时可以通过handleInfo、ioInfo获得之前提到的两个信息
        //但是一次只能从队列中获取一个已完成IO的信息，也就是客户端套接字、地址和重叠IObuffer信息
        GetQueuedCompletionStatus(hComPort,&bytesTrans,(LPDWORD)&handleInfo,(LPOVERLAPPED*)&ioInfo,INFINITE);
        sock=handleInfo->hClntSock;//已完成IO的套接字
        //判断是输入完成IO还是输出完成IO
        if (ioInfo->rwMode==READ){
            //如果是输入IO
            puts("message received");
            if (bytesTrans==0){
                closesocket(sock);
                free(handleInfo);free(ioInfo);
                continue;
            }
            //将服务器收到的消息回声给客户端
            memset(&(ioInfo->overlapped),0, sizeof(OVERLAPPED));
            ioInfo->wsaBuf.len=bytesTrans;
            ioInfo->rwMode=WRITE;
            WSASend(sock,&(ioInfo->wsaBuf),1,NULL,0,&(ioInfo->overlapped),NULL);
            //再次发送消息后接收客户端消息
            ioInfo=(LPPER_IO_DATA) malloc(sizeof(PER_IO_DATA));
            ioInfo->wsaBuf.len=BUF_SIZE;
            ioInfo->wsaBuf.buf=ioInfo->buffer;
            ioInfo->rwMode=READ;
            WSARecv(sock,&(ioInfo->wsaBuf),1,NULL,&flags,&(ioInfo->overlapped),NULL);
        }
        else{//完成的IO为输出时执行以下
            puts("message sent");
            free(ioInfo);
        }
    }
    return 0;
}

void ErrorHandling(char *message){
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

这里的IO第一次接收操作并不是用多线程进行的，而是用重叠IO放到后台进行的

程序流程：

假设有两台客户端：创建完成端口->创建多个回声线程并把完成端口的句柄作为参数传递给线程，此时这些线程就在后端的GetQueuedCompletionStatus函数这里阻塞等待了->设置好套接字->当第一个客户端进行连接时，将重叠套接字注册到完成端口，并用重叠IO进行后台接收

1：->重叠IO接收完成后，IO线程GetQueuedCompletionStatus函数返回并进行后续处理，也就是回声->回声后继续调用重叠IO进行后台接受，此时IO将会被设置成未完成状态，该线程进入下一次循环并在GetQueuedCompletionStatus函数阻塞

2：->主线程进入下一次循环来接受第二个客户端的连接与接受

### 23.2.6IOCP性能更优的原因

之前已介绍了Linux和Windows下多种服务器端模型，各位应该可以分析出它们在性能上的优势。将其在代码级别与select进行对比，可以发现如下特点：

因为是非阻塞模式的IO，所以不会由IO引发延迟。

查找已完成IO时无需添加循环。

无需将作为IO对象的套接字句柄保存到数组进行管理。

可以调整处理IO的线程数，所以可在实验数据的基础上选用合适的线程数。

## 23.3习题

- 完成端口对象将分配多个线程用于处理IO。如何创建这些线程?如何分配?请从源代码级别进行说明。

  先创建完成端口对象

  ```C++
  HANDLE hCompletionPort = CreateIoCompletionPort(INVALID_HANDLE_VALUE, NULL, 0, numThreads);
  ```

  再创建工作线程

  ```C++
  for (int i = 0; i < numThreads; ++i) {
      // 创建线程并传递完成端口对象给线程
      HANDLE hThread = CreateThread(NULL, 0, WorkerThread, hCompletionPort, 0, NULL);
  }
  ```

  在工作线程中等待IO完成队列

  ```C++
  DWORD bytesTransferred;
  ULONG_PTR completionKey;
  LPOVERLAPPED overlapped;
  
  while (true) {
      // 等待IO完成信息
      BOOL result = GetQueuedCompletionStatus(hCompletionPort, &bytesTransferred, &completionKey, &overlapped, INFINITE);
  
      if (!result) {
          // 处理出错情况
          DWORD error = GetLastError();
          // 处理错误...
      }
  
      // 处理已完成的IO操作
      // 在这里你可以根据 completionKey 等信息，知道是哪个IO操作完成了，然后进行相应的处理。
  }
  ```

  每个线程中这个循环会一直运行来处理完成IO队列中的IO

- CreateloCompletionPort函数与其他函数不同，提供2种功能。请问是哪2种?

  创建完成端口，把IO信息注册到完成端口中

- 完成端口对象和套接字之间的连接意味着什么?如何连接?

  连接之后意味着你可以在套接字上执行异步IO操作，例如 `WSARecv` 或 `WSASend`。这些操作将会通过完成端口对象通知你IO操作的完成情况。在工作线程中使用 `GetQueuedCompletionStatus` 函数等待IO完成信息，一旦IO操作完成，你将收到有关完成操作的信息，包括套接字句柄，IO字节数，以及自定义的信息。

- 下列关于IOCP的说法错误的是 ?

  - a. 以最少的线程处理多数IO的结构，因此可以减少上下文切换引起的性能低下。

  - b.执行IO的过程中，服务器端无需等待IO完成，可以执行其他任务，故能提高CPU效率。

  - c.I/O完成时会自动调用相关completionRoutine函数，因此没必要调用特定函数以等待IO完成。

  - d. 除Windows外， 其他操作系统同样支持IOCP，所以这种模型具有良好的移植性 。

    d：IOCP（Input/Output Completion Port）是Windows特有的异步IO模型，不支持跨平台移植。在其他操作系统上，需要使用不同的异步IO模型，因为它们不提供IOCP的等效功能。因此，IOCP不具备良好的移植性，它是特定于Windows的。

- 判断下列关于IOCP中选择合理线程数的方法是否合适。

  - 通常会选择与CPU数同样数量的线程。 (O )
  - 最好在条件允许的范围内通过实验决定线程数 。 (O )
  - 分配的线程数越多越好。 例如， 1个线程就足够的情况下应该多分配几个，比如创建3个线程分配给IOCP (X )

- 利用本章的IOCP模型实现聊天服务器端，该聊天服务器端应当结合第20章的聊天客户端chat_clnt_win.C正常运行。编写程序时不必刻意套用本章IOCP示例中的框架，那样反而会加大实现难度。

# 第二十四章 制作HTTP服务器端

## 24.1HTTP概要

对Web服务器端的定义："基于HTTP协议，将网页对应文件传输给客户端的服务器端 。"

HTTP是以超文本传输为目的而设计的应用层协议，这种协议同样属于基于TCP/IP实现的协议，因此我们也可以直接实现 HTTP。从结果上 看，实现该协议相当于实现Web服务器端。另外，浏览器也属于基于套接字的客户端，因为连接到任意Web服务器端时，浏览器内部也会创建套接字。只不过浏览器多了一项功能，它将服务器端传输的HTML格式的超文本解析为可读性较强的视图。总之，Web服务器端是以 HTTP协议为基础传输超文本的服务器端。

### 24.1.1HTTP

#### 24.1.1.1无状态的 Stateless协议

HTTP协议的请求及响应方式设计如图：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230925134400755.png" alt="image-20230925134400755" style="zoom:50%;" />

服务器端响应客户端请求后立即断开连接。 换言之，服务器端不会维持客户端状态。 即使同一客户端再次发送请求，服务器端也无法辨认出是原先那个，而会以相同 方式处也新请求 。 因此， HTTP又称”无状态的 Stateless协议“

为了弥补HTTP无法保持连接的缺点，Web 编程中通常会使用Cookie和Session技术。相信各位都接触过购物网站的购物车功能，即使关闭浏览器也不会丢失购物车内的信息(甚至不用登录)。这种保持状态的功能都是通过Cookie和Session技术实现的。

#### 24.1.1.2请求消息（Request Message）的结构

Web服务器端需要解析并响应客户端请求，客户端和服务器端之间的数据请求方式标准如图：

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230925134903199.png" alt="image-20230925134903199" style="zoom:50%;" />

请求消息可以分为请求行、消息头、消息体等3个部分。其中，请求行含有请求方式(请求目的)信息。典型的请求方式有GET和POST， GET主要用于请求数据，POST主要用于传输数据。为了降低复杂度，我们实现只能响应GET请求的Web服务器端。 

下面解释图请求行信息。 其中“GET/index.html HTTP/1.1" 具有如下含义：

"请求( GET ) index.html文件，希望以 1.1版本的HTTP协议进行通信。"

请求行只能通过1行( line )发送，因此，服务器端很容易从HTTP请求中提取第一行，并分析请求行中的信息。

请求行下面的消息头中包含发送请求的(将要接收响应信息的)浏览器信息、用户认证信息等关于HTTP消息的附加信息。最后的消息体中装有客户端向服务器端传输的数据，为了装入数据，需要以POST方式发送请求。但我们的目标是实现GET方式的服务器端，所以可以忽略这部分内容。另外，消息体和消息头之间以空行分开因此不会发生边界问题。

#### 24.1.1.3响应消息（Response Message）的结构

Web服务器向客户端传递的响应消息的结构如图：由状态行、头信息、消息体等3个部分构成。状态行中含有关于请求的状态信息

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230925141418188.png" alt="image-20230925141418188" style="zoom:50%;" />

第一个字符串状态行中含有关于客户端请求的处理结果。例如客户端请求index.html文件时，表示 index.html文件是否存在、服务器端是否发生问题而无法响应等不同情况的信息将写入状态行。图中的 "HTTP/1.1 2000K" 具有如下含义:

"我想用HTTP1.1版本进行响应，你的请求已正确处理 (200 OK)。"

表示"客户端请求的执行结果"的数字称为状态码，典型的有以下几种：

200 OK: 成功处理了请求!

404 Not Found: 请求的文件不存在!

400 Bad Request: 请求方式错误，请检查 !

消息头中含有传输的数据类型和长度等信息。图中的消息头含有如下信息:

“服务器端名为SimpleWebServert，传输的数据类型为text/html (html格式的文本数据)。数据长度不超过2048字节。"

最后插入1个空行后，通过消息体发送客户端请求的文件数据。以上就是实现Web服务器端过程中必要的HTTP协议。 

## 24.2实现简单的web服务器端

### 24.2.1实现基于 Windows的多线程Web服务器端

Web服务器端采用HTTP协议，即使用 IOCP或epoll模型也不会大幅提升性能。在服务器端和客户端保持较长连接的前提下频繁发送大小不一的消息时(最典型的就是网游服务器端)，才能真正发挥出这2种模型的优势。

通过多线程模型实现Web服务器端。也就是说，客户端每次请求时，都创建1个新线程响应客户端请求。

Webserv_win.c

```C++
#include <stdio.h>
#include <stdlib.h>
#include <string>
#include <winsock2.h>
#include <process.h>

#define BUF_SIZE 2048
#define BUF_SMALL 100

unsigned WINAPI RequestHandler(void *arg);
char * ContentType(char * file);
void SendData(SOCKET sock,char * ct,char * fileName);
void SendErrorMSG(SOCKET sock);
void ErrorHandling(char *message);

int main(int argc ,char * argv[]) {
    WSADATA wsaData;
    SOCKET hServSock, hClntSock;
    SOCKADDR_IN servAdr, clntAdr;

    HANDLE hThread;
    DWORD dwThreadId;
    int clntAdrSize;

    if (argc != 2) {
        printf("Usage:%s<port>\n", argv[0]);
        exit(1);
    }

    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
        ErrorHandling("WSAStartup Error");
		//获取并设置服务器套接字信息
    hServSock = socket(PF_INET, SOCK_STREAM, 0);
    memset(&servAdr, 0, sizeof(servAdr));
    lisnAdr.sin_family = AF_INET;
    lisnAdr.sin_addr.s_addr = htonl(INADDR_ANY);
    lisnAdr.sin_port = htons(atoi(argv[1]));
		//绑定服务器套接字并开始监听
    if (bind(hServSock, (SOCKADDR * ) & servAdr, sizeof(servAdr)) == SOCKET_ERROR)
        ErrorHandling("bind error");
    if (listen(hServSock, 5) == SOCKET_ERROR)
        ErrorHandling("listen error");

    /* 请求及响应 */
  	//循环接受连接请求
    while (1) {
        clntAdrSize = sizeof(clntAdr);
        hClntSock = accept(hServSock, (SOCKADDR * ) & clntAdr, &clntAdrSize);
        printf("Connection Request:%s:%d\n", inet_ntoa(clntAdr.sin_addr), ntohs(clntAdr.sin_port));
      	//开启RequestHandler线程来处理请求
        hThread = (HANDLE) _beginthreadex(NULL, 0, RequestHandler, (void *) hClntSock, 0, (unsigned *) &dwThreadID);
    }
    closesocket(hServSock);
    WSACleanup();
    return 0;
}

unsigned WINAPI RequestHandler(void *arg) {
    SOCKET hClntSock=(SOCKET)arg;//客户端套接字
    char buf[BUF_SIZE];
    char method[BUF_SMALL];
    char ct[BUF_SMALL];
    char fileName[BUF_SMALL];

    recv(hClntSock,buf,BUF_SIZE,0);//接收数据
    if(strstr(buf,"HTTP/")==NULL) {//查看是否是HTTP提出的请求
        SendErrorMSG(hClntSock);
        closesocket(hClntSock);
        return 1;
    }

    strcpy(method,strtok(buf," /"));
    if(strcmp(method,"GET"))//查看是否是GET方式的请求
        SendErrorMSG(hClntSock);

    strcpy(fileName,strtok(NULL," /"));//查看请求文件名
    strcpy(ct,ContentType(fileName));//查看content-type
    SendData(hClntSock,ct,fileName);//响应：传输该文件
    return 0;
};

void SendData(SOCKET sock,char * ct,char * fileName) {
    char protocol[]="HTTP/1.0 200 OK\r\n";
    char servName[]="Server:simple web server\r\n";
    char cntLen[]="Content-length:2048\r\n";
    char cntType[BUF_SMALL];
    char buf[BUF_SIZE];
    FILE * sendFile;

    sprintf(cntType,"Content-type:%s\r\n\r\n",ct);
    if ((sendFile= fopen(fileName,"r"))==NULL) {
        SendErrorMSG(sock);
        return;
    }

    /* 传输头信息 */
    send(sock,protocol, strlen(protocol),0);
    send(sock,servName, strlen(servName),0);
    send(sock,cntLen, strlen(cntLen),0);
    send(sock,cntType, strlen(cntType),0);

    /* 传输请求数据 */
    while(fgets(buf,BUF_SIZE,sendFile)!=NULL)
        send(sock,buf, strlen(buf),0);

    closesocket(sock);//由HTTP协议响应后断开
}

void SendErrorMSG(SOCKET sock){//发生错误时传递信息
    char protocol[]="HTTP/1.0 400 Bad Request\r\n";
    char servName[]="Server:simple web server\r\n";
    char cntLen[]="Content-length:2048\r\n";
    char cntType[]="Content-type:text/html\r\n\r\n";
    char content[]="<html><head><title>NETWORK</title></head>"
                   "<body><font size=+5><br> 发生错误！查看请求文件名和请求方式！"
                   "</font></body></html>";
    send(sock,protocol, strlen(protocol),0);
    send(sock,servName, strlen(servName),0);
    send(sock,cntLen, strlen(cntLen),0);
    send(sock,cntType, strlen(cntType),0);
    send(sock,content, strlen(content),0);
    closesocket(sock);
}

char * ContentType(char *file){//区分content type
    char extension[BUF_SMALL];
    char fileName[BUF_SMALL];
    strcpy(fileName,file);
    strtok(fileName,".");
    strcpy(extension, strtok(NULL,"."));
    if(!strcmp(extension,"html")||!strcmp(extension,"htm"))
        return "text/html";
    else
        return "text/plain";
}

void ErrorHandling(char *message){
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230925161705739.png" alt="image-20230925161705739" style="zoom:50%;" />

地址栏中输入了如下地址：http://localhost:9190/index.html

该请求相当于连接到IP为127.0.0.1、 端口为9190的套接字，并请求获取index.htm1文件。 可以 用回送地址127.0.0.1代替 "localhost"。 

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20230925161822582.png" alt="image-20230925161822582" style="zoom:50%;" />

### 

### 24.2.2实现基于 Linux的多线程 Web服务器端

Linux下的Web服务器端与上述示例不同，将使用标准IO函数。 

```C++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <pthread.h>

#define BUF_SIZE 2048
#define SMALL_BUF 100

void * request_handler(void *arg);
char * content_type(char * file);
void send_data(FILE *fp,char * ct,char * file_name);
void send_error(FILE *fp );
void error_handling(char *message);

int main(int argc ,char * argv[]) {
    int serv_sock, clnt_sock;
    struct sockaddr_in serv_adr, clnt_adr;
    int clnt_adr_size;
    char buf[BUF_SIZE];
    pthread_t t_id;
    if (argc != 2) {
        printf("Usage:%s<port>\n", argv[0]);
        exit(1);
    }

    serv_sock = socket(PF_INET, SOCK_STREAM, 0);
    memset(&serv_adr, 0, sizeof(servAdr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_adr.sin_port = htons(atoi(argv[1]));

    if (bind(serv_sock, (struct sockaddr* ) & serv_adr, sizeof(serv_adr)) == -1)
        error_handling("bind error");
    if (listen(serv_sock, 20) == -1)
        error_handling("listen error");

    /* 请求及响应 */
    while (1) {
        clnt_adr_size = sizeof(clnt_adr);
        clnt_sock = accept(serv_sock, (struct sockaddr* ) & clnt_adr, &clnt_adr_size);
        printf("Connection Request:%s:%d\n", inet_ntoa(clnt_adr.sin_addr), ntohs(clnt_adr.sin_port));
        pthread_create(&t_id,NULL,request_handler,&clnt_sock);
        pthread_detach(t_id);
    }
    close(serv_sock);
    return 0;
}

void* request_handler(void *arg) {
    int clnt_sock = *((int *) arg);
    char req_line[SMALL_BUF];
    FILE *clnt_read;
    FILE *clnt_write;

    char method[10];
    char ct[15];
    char file_name[30];

    clnt_read = fdopen(clnt_sock, "r");
    clnt_write = fdopen(dup(clnt_sock), "w");
    fgets(req_line, SMALL_BUF, clnt_read);
    if (strstr(req_line, "HTTP/") == NULL) {
        send_error(clnt_write);
        fclose(clnt_read);
        fclose(clnt_write);
        return;
    }

    strcpy(method, strtok(req_line, " /"));
    strcpy(file_name, strtok(NULL, " /"));
    strcpy(ct, content_type(file_name));
    if (strcmp(method, "GET") != 0) {
        send_error(clnt_write);
        fclose(clnt_read);
        fclose(clnt_write);
        return;
    }

    fclose(clnt_read);
    send_data(clnt_write, ct, file_name);
}

void send_data(FILE* fp,char * ct,char * file_name) {
    char protocol[]="HTTP/1.0 200 OK\r\n";
    char serv[]="Server:simple web server\r\n";
    char cnt_len[]="Content-length:2048\r\n";
    char cnt_type[SMALL_BUF];
    char buf[BUF_SIZE];
    FILE * send_file;

    sprintf(cnt_type,"Content-type:%s\r\n\r\n",ct);
    send_file=fopen(file_name,"r");
    if (send_file==NULL){
        send_error(fp);
        return;
    }

    /* 传输头信息 */
    fputs(protocol,fp);
    fputs(serv,fp);
    fputs(cnt_len,fp);
    fputs(cnt_type,fp);

    /* 传输请求数据 */
    while(fgets(buf,BUF_SIZE, send_file)!=NULL){
        fputs(buf,fp);
        fflush(fp);
    }
    fflush(fp);
    fclose(fp);
}

char * content_type(char *file){//区分content type
    char extension[SMALL_BUF];
    char file_name[SMALL_BUF];
    strcpy(file_name,file);
    strtok(file_name,".");
    strcpy(extension, strtok(NULL,"."));

    if(!strcmp(extension,"html")||!strcmp(extension,"htm"))
        return "text/html";
    else
        return "text/plain";
}

void send_error(FILE* fp){//发生错误时传递信息
    char protocol[]="HTTP/1.0 400 Bad Request\r\n";
    char serv_name[]="Server:simple web server\r\n";
    char cnt_len[]="Content-length:2048\r\n";
    char cnt_type[]="Content-type:text/html\r\n\r\n";
    char content[]="<html><head><title>NETWORK</title></head>"
                   "<body><font size=+5><br> 发生错误！查看请求文件名和请求方式！"
                   "</font></body></html>";
    fputs(protocol,fp);
    fputs(serv_name,fp);
    fputs(cnt_len,fp);
    fputs(cnt_type,fp);
    fputs(content,fp);
    fflush(fp);
}

void error_handling(char *message){
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

## 24.3习题

- 下列关于Web服务器端和Web浏览器的说法错误的是?
  - a. Web浏览器并不是通过自身创建的套接字连接服务器端的客户端
  - b. Web服务器通过TCP套接字提供服务，因为它将保持较长的客户端连接并交换数据
  - c. 超文本与普通文本的最大区别是其具有可跳转的特性
  - d. Web服务器端可视为向浏览器提供请求文件的文件传输服务器端
  - e. 除Web浏览器外，其他客户端都无法访问 Web服务器端
- 下列关于HTTP协议的描述错误的是?
  - a. HTTP协议是无状态的Stateless协议，不仅可以通过TCP实现，还可通过UDP实现
  - b. HTTP协议是无状态的Stateless协议，因为其在1次请求和响应过程完成后立即断开连接。 因此，如果同一服务器端和客户端需要3次请求及响应，则意味着要经过3次套接字创建过程 
  - c. 服务器端向客户端传递的状态码中含有请求处理结果信息 
  - d. HTTP协议是基于因特网的协议，因此，为了同时向大量客户端提供服务， HTTP被设计为 Stateless协议
- IOCP和epol1是可以保证高性能的典型服务器端模型，但如果在基于HTTP协议的Web服务器端使用这些模型，则无法保证一定能得到高性能 。 请说明原因 。

# 第二十五章 进阶内容

## 25.1网络编程学习的其他内容

## 25.2网络编程相关书籍介绍

### 25.2.1系统编程相关书籍

《Unix环境高级编程（第二版）》

《Windows核心编程》

### 25.2.2协议相关书籍

《TCP/IP详解》 (卷 1-卷 3)

《TCP/IP协议族》

总笔记：回调函数：可以定义一个函数，将这个函数的引用传递给其他任务或者函数，当某个条件满足的时候，这个回调函数会被调用，比如说用户点击按钮时，可以触发一个回调函数来响应按钮点击事件。这个回调函数 会在用户点击按钮时被执行。

异步操作：函数回调用于异步操作，例如在网络编程中，可以发起一个IO操作，并在操作完成时调用回调函数来处理响应数据，在等待响应的过程中，程序不会被阻塞，可以干其他事情，当 响应达到时，程序将自己变成可响应状态并在后台处理回调函数内容。